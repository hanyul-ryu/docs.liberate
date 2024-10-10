# Homomorphic Statistics

The purpose of this tutorial is to calculate the mean and variance for all the data using approximately 10 million encrypted records. This will enable more accurate and reliable data analysis. Additionally, by using Liberate.FHE as in this tutorial, you can confirm that it is possible to implement Homomorphic operations with very simple code.

In this tutorial, you can find the complete codes for mean and variance as follows:

{% code title="var.py" %}
```python
import numpy as np
from liberate import fhe
from liberate.fhe import presets

# generate engine
params = presets.params["gold"]
engine = fhe.ckks_engine(verbose=True, **params)

# generate Keys
sk = engine.create_secret_key()
pk = engine.create_public_key(sk)
evk = engine.create_evk(sk)
gk = engine.create_galois_key(sk)

# set num of data
target_N = 10 ** 7  # 10,000,000
required_num_ct = round(target_N / engine.num_slots)

# generate Data
cts = []
ms = []
age_range = [0, 100]
for _ in range(required_num_ct):
    m = np.random.randint(age_range[0], age_range[1], engine.num_slots)
    ct = engine.encorypt(m, pk, level=engine.num_levels - 8)
    # ct = engine.encorypt(m, pk, level=0)
    ms.append(m)
    cts.append(ct)

#################################
########    calc mean    ########
#################################
result = engine.add(cts[0], cts[1])

for ct in cts[2:]:
    result = engine.add(result, ct)
ct_mean = engine.mean(result, gk, alpha=required_num_ct)

calculated_mean = engine.decrode(ct_mean, sk)
target_mean = np.array(ms).mean()

print("=====" * 15)
print(f">>\tcalc mean\t:\t{calculated_mean[0].real:20.13f}")
print(f">>\ttarget mean\t:\t{target_mean:20.13f}")
print(f">>\tabs max error\t:\t{np.abs(calculated_mean.real - target_mean).max():20.13e}")

################################
########    calc var    ########
################################
devs = []
for ct in cts:
    dev = engine.sub(ct, ct_mean)

    dev = engine.square(dev, evk)
    devs.append(dev)

result = engine.cc_add(devs[0], devs[1])

for dev in devs[2:]:
    result = engine.cc_add(dev, result)


ct_var = engine.mean(result, gk, alpha=required_num_ct)

################################
####    decrypt & decode    ####
################################
calculated_var = engine.decrode(ct_var, sk).mean()
target_var = np.array(ms).real.var() + np.array(ms).imag.var() * 1j

print("=====" * 15)
print(f">>\tcalc var\t: {calculated_var.real:20.13e}")
print(f">>\ttarget var\t: {target_var.real:20.13e}")
print(f">>\terror\t\t: {(calculated_var - target_var).real:20.13e}")
```
{% endcode %}

## Setup

### Import necessary packages

```python
import numpy as np
from liberate import fhe
from liberate.fhe import presets
```

Import the package you want to use. `fhe` is an essential package from the Liberate Library for homomorphic encryption. `fhe.presets` is a package that provides convenience for importing pre-configured homomorphic parameters.

### Generate CKKS engine

```python
# generate engine
params = presets.params["gold"]
engine = fhe.ckks_engine(verbose=True, **params)
```

To create the CKKS engine, we utilize pre-configured parameters. For this tutorial, the engine has been created using the [`gold` parameter](../api-references/docs.md#preset-parameters). In the case of `gold`, the value of `logN` is 16, there are 4 `special primes`, and a total of 34 levels are available.3.&#x20;

### Generate encryption keys

```python
# generate Keys
sk = engine.create_secret_key()
pk = engine.create_public_key(sk)
evk = engine.create_evk(sk)
gk = engine.create_galois_key(sk)
```

We will generate the keys to be used in the computation. The keys used in this tutorial are as follows.

* `sk`: The secret key is a confidential piece of information that is kept secret by the data owner or a trusted entity. The secret key is typically used for decrypting data. In homomorphic encryption, it is used to decrypt the results of computations performed on encrypted data. The secrecy of the secret key is crucial for the security of the encrypted data. If compromised, an attacker could decrypt the results and potentially gain access to sensitive information.
* `pk` : The public key is a component of the homomorphic encryption system that is openly shared. The public key is used for encrypting data. While it can be used to encrypt data, it cannot be used to decrypt the results of computations performed on that data. Unlike the secret key, the public key is shared openly and does not need to be protected in the same way. Its security is not compromised even if it is known to potential adversaries.
* `evk` : The evaluation key is a cryptographic component used in homomorphic encryption schemes that support relinearization. It is used during the relinearization process to transform ciphertexts, typically after multiplication operations. While not as sensitive as the secret key, the relinearization key may still require protection, and its use is generally limited to trusted entities.
* `gk` : The Galois key is a cryptographic component used in certain homomorphic encryption schemes to support specific operations, particularly rotations and linear transformations in the Galois field. The Galois key enables the efficient execution of mathematical operations on encrypted data while preserving the confidentiality of the underlying information. While the Galois key is an important component, it is not as sensitive as the secret key in homomorphic encryption.

### Generate data and encrypt data

```python
# set num of data
target_N = 10 ** 7  # 10,000,000
required_num_ct = round(target_N / engine.num_slots)

# generate Data
cts = []
ms = []
age_range = [0, 99]
for _ in range(required_num_ct):
    m = np.random.randint(age_range[0], age_range[1], engine.num_slots)
    ct = engine.encorypt(m, pk, level=0)
    # ct = engine.encorypt(m, pk, level=engine.num_levels - 8)
    ms.append(m)
    cts.append(ct)
```

In this tutorial, we will calculate the average and variance of approximately 10 million data points. These 10 million data points will be generated assuming they represent the ages of the population in a specific city. Therefore, we assume the age ranges from _0 years old_ to _99 years old_ and generate them using a random function. Additionally, we will trim the data slightly to match the number of security parameters we have set, rather than exactly 10 million. Since we have set `logN` to 16, the number of available slots is 32,768. Hence, the number of ciphertexts we will use is approximately round(10,000,000/32,768) = 305. We will then proceed with encoding and encrypting using the encrypt function. With the data prepared, we can now begin by calculating the mean (average).

## Mean

The code for calculating mean in Liberate.FHE is as follows.

1. Calculate the total of all the values in the dataset.&#x20;

$$
\sum_{i=0}^{n-1}x_{i} = x_{0}+x_{1}+\cdots+x_{n-1}
$$

2. To calculate the average, divide the sum by the number of values $$n$$.

$$
\text{E}\left(X\right) = \frac{\sum_{i=0}^{n-1}x_{i}}{n}
$$

Therefore, we use the `add` function to calculate the sum of all ciphertexts.

```python
result = engine.add(cts[0], cts[1])
for ct in cts[2:]:
    result = engine.add(result, ct)
```

To calculate the average of all encrypted texts, use the mean provided by Liberate.FHE.

```python
ct_mean = engine.mean(result, gk, alpha=required_num_ct)
```

## Variance

Now, we can use the average value obtained in this way to calculate the variance.

The formula to calculate dispersion is as follows.

1. Calculate the average of the data : Utilize the previously computed average.
2. Subtract the average from each data point.

$$
\text{deviation} = \left(x_{i}-\text{E}\left(X\right)\right)
$$

3. Calculate the sum of all squared deviations. \

4. Divide the sum of squared deviation by the number of values.

$$
\text{Var}\left(X\right) = \frac{\sum_{i=0}^{n-1}\left(x_{i}-\text{E}\left(X\right)\right)^{2}}{n}
$$

```python
devs = []
for ct in cts:
    dev = engine.sub(ct, ct_mean)
    dev = engine.square(dev, evk)
    devs.append(dev)

result = engine.cc_add(devs[0], devs[1])
for dev in devs[2:]:
    result = engine.cc_add(dev, result)

ct_var = engine.mean(result, gk, alpha=required_num_ct)
```

To calculate the difference between each data point and the previously calculated average, we can use the `sub` function from Liberate.FHE. Then, we can calculate the square of the differences using the `square` function. By summing up the squared differences and using the `mean` function, we can obtain the variance.



## Result verify

```python
print("=====" * 15)
print(f">>\tcalc mean\t:\t{calculated_mean[0].real:20.13f}")
print(f">>\ttarget mean\t:\t{target_mean:20.13f}")
print(f">>\tabs max error\t:\t{np.abs(calculated_mean.real - target_mean).max():20.13e}")
```

```bash
>>	calc mean	:	    49.4955522744958
>>	target mean	:	    49.4953527231685
>>	abs max error	:	 1.9955132722060e-04
```

```python
print("=====" * 15)
print(f">>\tcalc var\t: {calculated_var.real:20.13e}")
print(f">>\ttarget var\t: {target_var.real:20.13e}")
print(f">>\terror\t\t: {(calculated_var - target_var).real:20.13e}")
```

```bash
>>	calc var	:  8.3310567454940e+02
>>	target var	:  8.3310766262893e+02
>>	error		: -1.9880795293830e-03
```

