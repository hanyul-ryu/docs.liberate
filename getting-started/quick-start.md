# Quick Start

## Quick Start

The following example shows the high level APIs of Liberate.FHE which can be used quickly and easily.

1. [Import packages for liberate.FHE](quick-start.md#1.-import-packages-for-liberate.fhe)
2. [Configure security parameters & generate CKKS engine](quick-start.md#2.-configure-homomorphic-paramters-and-generate-ckks-engine)
3. [Create keys](quick-start.md#3.-create-homomorphic-keys)
4. [Encode and encrypt](quick-start.md#4.-encode-and-encrypt-data)
5. [Perform homomorphic operations with encrypted data](quick-start.md#5.-perform-operation-with-encrypted-data)
6. [Decrypt and decode](quick-start.md#6.-decrypt-and-decode)

{% code title="example.py" overflow="wrap" fullWidth="false" %}
```python
from liberate import fhe
from liberate.fhe import presets

# Generate CKKS engine with preset parameters
grade = "silver"  # logN=15
params = presets.params[grade]

engine = fhe.ckks_engine(**params, verbose=True)

# Generate Keys
sk = engine.create_secret_key()
pk = engine.create_public_key(sk)
evk = engine.create_evk(sk)

# Generate test data
m0 = engine.example(-1, 1)
m1 = engine.example(-10, 10)

# encode & encrypt data
ct0 = engine.encorypt(m0, pk)
ct1 = engine.encorypt(m1, pk, level=5)

# (a + b) * b - a
result = (m0 + m1) * m1 - m0
ct_add = engine.add(ct0, ct1)  # auto leveling
ct_mult = engine.mult(ct1, ct_add, evk)
ct_result = engine.sub(ct_mult, ct0)

# decrypt & decode data
result_decrypted = engine.decrode(ct_result, sk)

```
{% endcode %}

## 1. Import packages for liberate.fhe

To start using homomorphic encryption such as the CKKS scheme with the `liberate.fhe` package, you need to import the necessary packages. The `liberate.fhe` package is the essential for using the CKKS scheme. Additionally, pre-configured information can be found in the `liberate.fhe` package's [presets](../api-references/docs.md#preset-parameters).

```python
from liberate import fhe
from liberate.fhe import presets
```

## 2. Configure security parameters & generate CKKS engine

To use homomorphic encryption, you need to configure security parameters according to your requirements. These parameters should be decided based on several factors such as the number of data elements, the number of multiplications, and the desired precision. The presets provide parameter options of different levels for convenience. You can also modify the parameters if needed.

```python
# use presets
grade = "silver"
params = presets.params[grade]

# simple modify
params["scale_bits"] = 39
```

After setting the parameters, you can generate the engine by providing the parameters and any additional options you need.

```python
# Generate the engine
engine = fhe.ckks_engine(**presets,
                         security_bits=128, # Additional options
)

```

For more detailed information on the parameters used for engine generation, you can refer to the [Liberate.FHE documentation](../api-references/docs.md#context-generation).

## 3. Create keys

Homomorphic encryption uses various types of keys for encryption, decryption and homomorphic operations (a.k.a. homomorphic evaluations). The most basic keys are a Secret Key `sk` and a Public Key `pk`. The `sk` is a private key used for decryption and should not be exposed to external entities. The `pk` is a key used for encryption and is shared with external users. In addition, an Evaluation Key `evk`, also known as a Re-linearization Key, is generated for multiplication operations.

```python
# Create the secret key
sk = engine.create_secret_key()

# Create the public key using the secret key
pk = engine.create_public_key(sk=sk)

# Create the evaluation key using the secret key
evk = engine.create_evk(sk=sk)

```

Please refer to the [Liberate.FHE documentation](../api-references/docs.md#key-generations) for information on generating different keys.

## 4. Encode and encrypt data

In CKKS, data is encrypted through the _Encode -> Encrypt_ process.

```python
# Generate test data
message_a = engine.example(amin=-1., amax=1.)
message_b = engine.example(amin=-1., amax=1.)

# Encode the data
pt_a = engine.encode(m=message_a, level=0)
pt_b = engine.encode(m=message_b, level=0)

# Encrypt the encoded data
ct_a = engine.encrypt(pt=pt_a, pk=pk, level=0)
ct_b = engine.encrypt(pt=pt_b, pk=pk, level=0)

```

The `liberate.fhe` library provides a shortcut function called `encorypt` for convenience, which combines the encode and encrypt steps.

```python
# Generate test data
message_a = engine.example(amin=-1., amax=1.)
message_b = engine.example(amin=-1., amax=1.)

# 
ct_a = engine.encorypt(m=message_a, pk=pk, level=0)
ct_b = engine.encorypt(m=message_b, pk=pk, level=0)
```

## 5. Perform homomorphic operations with encrypted data

To perform operations using encrypted data, you can utilize the homomorphic operation functions provided by the `liberate.FHE` library.&#x20;

```python
# Perform (a * b) + a
ct_mult = engine.mult(a=ct_a, b=ct_b, evk=evk)
ct_result = engine.add(a=ct_mult, b=ct_a)
```

For more detailed information on the homomorphic operations used, you can refer to the [Liberate.FHE documentation](../api-references/docs.md#arithmetic-functions).

## 6. Decrypt and decode

After performing the homomorphic operations, you can decrypt the ciphertext `ct` and decode it to obtain the original data.

```python
# Decrypt the ciphertext
pt_result = engine.decrypt(ct=ct_mult, sk=sk)

# Decode the decrypted result to obtain the original data
message_result = engine.decode(m=pt_result, level=ct_mult.level)
```

The `liberate.FHE` library provides shortcut functions called `decrode` for convenience, which combines the decrypt and decode steps.

```python
message_result = engine.decrode(ct=ct_result, sk=sk)
```

***

This section is a brief guide on how to start programming homomorphic encryption applications using the `liberate.fhe` package with the CKKS scheme. It includes steps such as package import, engine configuration and creation, homomorphic key generation, data encoding and encryption, performing operations on encrypted data, decryption and decoding. For more detailed information, please refer to the [Liberate.FHE documentation](../api-references/docs.md).
