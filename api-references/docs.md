# Docs

## Liberate.FHE API Documentation

This comprehensive document provides in-depth information on the usage and functions of the groundbreaking Liberate.FHE library for Homomorphic Encryption. It covers various essential aspects, starting with a detailed explanation of the context and engine generation, which forms the foundation for the library's powerful capabilities. It then delves into the key generation process, ensuring the secure creation of encryption keys that are vital for protecting sensitive data.

Moreover, this document elaborates on the encoding and decoding mechanisms employed by the Liberate.FHE library, enabling seamless transformation of data into encrypted form and vice versa. It also explores the encryption and decryption processes, shedding light on the secure procedures employed to safeguard information while allowing computations to be performed on encrypted data.

In addition to these core functionalities, the document provides insights into the diverse arithmetic functions available within the Liberate.FHE library. These functions empower users to perform mathematical operations on encrypted data, opening up a world of possibilities for secure computations. Furthermore, it highlights support functions that enhance the overall usability and efficiency of the library, ensuring smooth integration and seamless operation.

Last but not least, this document showcases the utility functions offered by the Liberate.FHE library, including the ability to save, load, and print data structures. These functions provide convenient ways to manage and manipulate encrypted data, enabling users to easily store, retrieve, and analyze information as needed.

Overall, this comprehensive document serves as an indispensable guide for users seeking to harness the full potential of the Liberate.FHE library for Homomorphic Encryption. It covers a wide range of topics, ensuring a thorough understanding of the library's capabilities and empowering users to leverage its advanced features for secure and efficient computations.

## Before Moving on...

This documentation only covers the _high-level_ APIs of the library. That is, only the publicly exposed functions are explained. There are numerous internal functions that compose the library; however, those are not documented herein.

## Composition of This Document

The function API is broken down into 7 parts:

1. Context generation
2. Key generation
3. The basic blocks of encoding/decoding and encrypting/decrypting
4. Arithmetic functions
5. Support functions
6. Runtime reflection
7. Utility functions such as save, load, export, import, upload (to GPUs), and download (from GPUs)

## General Concept and Particular Features of the Library

Liberate.FHE differentiates itself from other packages in that

### Arithmetic-Wise Differences

1. All intermediate calculations are explicitly executed as integers. No _float_ or _double_ conversions occur during computation.
2. All intermediate calculations and modular reductions are _exact_, as they are done using the Montgomery Reduction technique.
3. Texts, including Cipher, Key, and Plain, are stored in GPUs in split form. In particular, the split is alongside the RNS channels, with the exception of the special primes. Special primes are repeated (copied) onto all the GPUs for faster execution of key switching. For details of the partitioning scheme, refer to `rns_partition.py` under the `ntt` folder.

### Algorithm-Wise Differences

1. Rescaling is done before the multiplication, not _after_ the multiplication.
2. The input message is pre-permuted before encoding. This is done to achieve a cyclic-rotation effect by permuting the coefficients. Similarly, the decoded message is post-permuted to restore the intended form..
3. As a result of _Rescale Before Multiplication_, intermediate cipher texts are multiplied with the square of the scale factor. That is $$\Delta^{2}$$. To compensate for the squared $$\Delta$$ , cipher texts are rescaled right before decryption as well.
4. There are 2 proprietary formulas used for rescaling and key switching. The effect of using these methods results in algorithmic exactness.
5. Primes are selected according to a proprietary condition that prevents _drift_ of the rescale error, that is $$\Delta^{2}/q^{2}$$. The square term comes from multiplication because rescale error is always the consequence of binary multiplication.
6. The deviation error, relative to the size of the text, that arises from rescaling is explicitly corrected during the decryption stage. This correction significantly enhances the accuracy of homomorphic operations.

## Context Generation

There are 2 stages of the context generation.

The first is the CKKS context generation. The generated CKKS context includes basic information about the polynomial length, primes, and _etc_. The generated context is also saved in the cache folder for faster access at later times if a context with the same configuration is requested.

By default, scale primes with the bit lengths of from 20 to 59 are generated. However, usage of primes under 30 bits are not recommended for accuracy reasons as density of primes in the bit range are sparse, hampering downed the quality of the primes distribution. Also, note that the context support 2 precision types, that are 32 bits and 64 bits integers. Although, it is highly unlikely the 32 bits precision will be used in real situations, the case is included for completeness.

The grammar of calling the context generate functions is as follows:

{% code title="generate_ckks_context" %}
```python
from liberate.fhe.context import ckks_context

ctx = ckks_context(buffer_bit_length=62,
                   scale_bits=40,
                   logN=15,
                   num_scales=None,
                   num_special_primes=2,
                   sigma=3.2,
                    uniform_ternary_secret=True,
                   cache_folder='cache/',
                   security_bits=128,
                   quantum='post_quantum',
                   distribution='uniform',
                   read_cache=True,
                   save_cache=True,
                   verbose=False
                  )
```
{% endcode %}

where,

* `buffer_bit_length` : Specifies the bit length of coefficients. For 64-bit integers, use 62 (default), and for 32-bit integers, use 30.
* `scale_bits` : Specifies the scale. The scale will be set to . Note that the bits allocated for the integral part of the message will be `buffer_bit_length - scale_bits - 2`.
* `logN` : Specifies the length of the message polynomial. The length of the polynomial is $$2^{\text{logN}}$$ , and the actual length of the message that can be encoded in the polynomial is . The means you $$2^{\text{logN}}/2 = 2^{\text{logN} - 1}$$ can fill the polynomial with $$2^{\text{logN}-1}$$ complex numbers.
* `num_scales` : Specifies the number of utilizable homomorphic levels. If not specified explicitly, the context generator will generate the _maximum_ number of levels available given the `logN` and the security requirements.
* `num_special_primes` : This number corresponds to the factor in hybrid key switching. The specified number of primes will be subtracted from the available levels and then used for key switching.
* `sigma`: The standard deviation of the discrete Gaussian sampling, when generating small errors.
* `uniform_ternary_secret`: Selects the random algorithm when sampling the secret key. Currently this parameter has _no effect_, and the engine sticks to the uniform ternary sampling method.
* `cache_folder`: The path to the cache folder.
* `security_bits`: Specifies the strength of the security in terms of bits.
* `quantum`: Specifies the quantum security measures. The value can be either `post_quantum` or `pre_quantum`.
* `distribution`: Specifies the security model. The security level is measured by sampling and measuring the hardness. This parameter selects the sampling method in the process. Note that this is _not_ the sampling distribution for error generation.
* `read_cache`: If set to true, and if the cache for the input parameters exists, the context generator will read in the cache instead of generating a fresh one.
* `save_cache`: If set to true, a newly generated context will be saved to a cache file.
* `verbose`: If set to true, invoking the generator will print out diagnostic messages.

## CKKS Engine Generation

The second is the engine generation. An engine contains numerous pre-calculated caches for calculating the NTT transformation and modular operations. The pre-calculated caches are moved to available GPUs at the generation time so that following calculation won't need to move around the caches. Note that these engine parameters are volatile, that means they are _not_ saved to the disk.

Note that you _do not_ need to generate the ckks\_context and hand it over to the engine generator, as it is done internally, and automatically.

At the time of engine generation, you can specify which GPUs to use. This will give you the opportunity to experiment with deployment configurations.

The calling API is identical to that of context generation, such that

```python
from liberate import fhe

engine = fhe.ckks_engine(
            buffer_bit_length=62,
            scale_bits=40,
            logN=15,
            num_scales=None,
            num_special_primes=2,
            sigma=3.2,
            uniform_ternary_secret=True,
            cache_folder='cache/',
            security_bits=128,
            quantum='post_quantum',
            distribution='uniform',
            read_cache=True,
            save_cache=True,
            verbose=False
)
```

Semantics of the parameters are the same as the context generation.

In most usage scenarios, you would only need to specify the `logN` and all the rest will be automatically generated.

For example,

```python
from liberate import fhe

engine = fhe.ckks_engine(logN=15)
```

will generate the most typical CKKS engine for you.

## Preset Parameters

We have preset parameters that can deliver the best performance for user convenience. These settings are composed of values that can yield optimal results in `logN`, `num_special_primes`, and number of GPUs (`devices`). We have named these settings `bronze`, `silver`, `gold`, and `platinum`_(you may have heard these names somewhere before. Yes, that's right. Just as you might think)_. `Bronze` corresponds to the setting when `logN` is _14_, and the subsequent settings are composed of values _15_, _16_, and _17_ respectively. These settings are provided in the form of Python dictionaries, and users can easily modify them as they wish. By providing these preset values, users can avoid the hassle of manually adjusting parameters to achieve the optimal results.

```python
from liberate import fhe
from liberate.fhe import presets

grade = "silver"
params = presets.params[grade]
params["num_scales"] = 5 # you can modify easily

engine = fhe.ckks_engine(**params)

```

The arguments provided in `presets.params` are as follows:

<table data-full-width="false"><thead><tr><th width="118">grade</th><th width="76">logN</th><th>special primes</th><th width="97">devices</th><th>scale_bits</th><th>num slots</th><th>num levels</th></tr></thead><tbody><tr><td>bronze</td><td>14</td><td>1</td><td>1</td><td>40</td><td>8192</td><td>7</td></tr><tr><td>silver</td><td>15</td><td>2</td><td>1</td><td>40</td><td>16384</td><td>16</td></tr><tr><td>gold</td><td>16</td><td>4</td><td>Full</td><td>40</td><td>32768</td><td>34</td></tr><tr><td>platinium</td><td>17</td><td>6</td><td>Full</td><td>40</td><td>65536</td><td>72</td></tr></tbody></table>

And you can use _level 7_ in `bronze`, _level 16_ in `silver`, _level 34_ in `gold`, and _level 72_ in `platinum`.

You can find descriptions for each parameter in the [Liberate.FHE document](docs.md#context-generation).

## Key Generations

### The Secret Key Generation

```python
sk = engine.create_secret_key()
```

The secret keysk is the randomly generated secret key. The Homomorphic Encryption Standard recommends refresh of the random seed occasionally. In case of refresh, issue

```python
engine.rng.refresh()
```

and then, generate a secret key.

### The Public key Generation

Generating a public keypk requires a sk. Equipped with a sk, you can issue

```python
pk = engine.create_pulic_key(sk=sk)
```

to obtain a public key.

Note that, public keys generated with the same `sk` have all different values, as it is the security requirement. Although different in values, all the cipher messages that are generated with the same `sk` will work equally well under homomorphic operations; cipher texts encrypted under the same `sk` but different `pk` can be mixed in homomorphic operation. However, in any operation, their currency level must match.

### The Evaluation Key Generation

The Evaluation Keyevk is used for multiplication. It can be generated with

```python
evk = engine.create_evk(sk=sk)
```

### The Rotation key Generations

The galois key galk is a collection of key switch keys that enable the $$n$$ cyclic rotations. For a polynomial of length $$N$$, all possible rotations can be represented by $$\log_{2}\left(N\right)$$ rotations. The galk contains such $$\log_{2}\left(N\right)$$ keys. The key generation API is

```python
galk = engine.create_galois_key(sk=sk)
```

.

In some occasions, you may want to generate a rotation key for a particular rotation. The most typically expected case is where the rotation is repeated frequently is a computation circuit and you want to accelerate the rotation by directly applying a rotation instead of a successive composition of rotations. In such case, use the following API

```python
rotk = engine.create_rotation_key(sk=sk, delta=my_shift)
```

. Issuing the above function will generate a rotation key valid for the particular rotation.

### The Conjugate Key Generation

Conjugation is a transformation of a complex number $$\alpha + b\mathbf{i}$$ to $$\alpha -b\mathbf{i}$$. The key for such operation can be generated with

```python
conjk = engine.create_conjugation_key(sk=sk)
```

### The Key Switch Key Generation

You can change the secret key of a cipher text to a different secret key. The key switch key is used for such operation. You can issue the following command to generate the key switch key

```python
ksk = engine.create_key_switch_key(sk_from, sk_to)
```

, where ksk is the key switch key, sk\_from is the secret key used to encrypt the cipher text, and sk\_to is the new secret key with which you want to encrypt the cipher text.

## The Basic Usage Cycle of Encode/Decode and Encrypt/Decrypt

A message is an array of complex numbers of which the length $$2^{\text{logN}-1}$$.

In most cases, the message goes through a usage cycle as

**encode → encrypt → Homomorphic Operations → decrypt → decode**

The following subsections explain the API for each stage in the usage cycle, except for the homomorphic operations phase. APIs for the operation phase are enumerated in a separate (following) section.

### Encode

```python
pt = engine.encode(m, level=0)
```

, where pt denotes a plain text and m a message and level denotes the homomorphic level you wnat to use. Note that the pt you get is a pre-permuted version of the encoded plain text.

### Enecrypt

```python
ct = engine.encrypt(pt, pk, level)
```

, where ct denotes the cipher text, pk is a public key and level is homomorphic level you want to use.

### Encorypt

```python
ct = engine.encorypt(m, pk, level)
```

, where ct denotes the cipher text, pk is a public key and level is homomorphic level you want to use.

### Decrypt

```python
pt = engine.decrypt(ct, sk)
```

, where `sk` is a secret key.

### Decode

```python
m = engine.decode(pt, level)
```

, where `sk` is a secret key and level is homomorphic level you want to use.

Note that you do _not_ need to post-permute the message `m`. It is automatically handled for you.

### Decrode

```python
m = engine.decrode(ct, sk)
```

, where `sk` is a secret key.

## Arithmetic Functions

There are numerous homomorphic arithmetic functions supported. The following subsections explain each of them.

### Add / Subtract

```python
ct_add = engine.add(ct_a, ct_b)
```

Note that `a` and `b` can both be cipher texts, or can be plain texts (or a messages). If one of the operands is message, the message will be automatically encoded to match the required format of the plain text.

Also that `a` and `b` can both be cipher text or can be plain texts(or messages). If one of the operands is message, the message will be automatically encoded to match the required format of the plain text.

Also note that, the sace where both `a` and `b` are plain texts(messages) is mermitted since triplet of additions $$\left(a+b+c\right)$$ may occur and two of the operands are plain texts.

> If `a` and `b` are at different levels, the engine will do a proper leveling on one of the inputs and calculate the results. The resultant will have the highest level of the two.

Likewise, for subtraction use

```python
ct_sub = engine.sub(a, b)
```

.

A word of warning here.

{% hint style="info" %}
**Do NOT attempt to add or subtract cipher texts directly. Use the API.**

Since the cipher texts are, by implementation, **PyTorch tensors**, you may be attempted to add or subtract them directly by using `a + b` of `a - b`. Don't. Although, the numbers contained in the tensors are

1. Coefficients of the Number Theoretic Transformation (NTT) coefficients.
2. The Montgomery form numbers.

The context engine will do proper computations for you, and hence do not attempt to do it yourself unless you are undeniably certain about what you're doing.
{% endhint %}

### Multiplication

```python
ct_mult = engine.mult(ct_a, ct_b, evk)
```

. Composition and permittance of the input parameters are the same as the add/substration. However, one caveat still persists: Division is _not_ provided as an API function.

Since division is inherently iterative, meaning it consumes multiple levels, its API is _not_ provided in the engine API. You will have to devise your own function using the Newton-Rhapson or the Goldschmidt algorithm.

> If `a` and `b` are at different levels, the engine will do a proper leveling on one of the inputs and calculate the results. The resultant will have the highest level of the two.

### Square

```python
ct_squared = engine.square(ct, evk)
```

. Most of the aspects are similar to multiplication, but it is a bit faster than using the [`mult`](docs.md#multiplication) function because it has been optimized.

### Rotate

Cyclically rotate a cipher text using

```python
ct_rotated = engine.rotated_galois(ct, galk, shift, return_cirtuit=False)
```

, where `ct` is the cipher text, `ksk` is the `galk`, `shift` is the shift distance of the rotation. And if you use the `return_circuit` option, you can check wich index you have moved to. Signedness of the `shift` determines the direction of the shift. A positive `shift` denotes shifting _right_, and a negative the opposite direction of _left_.

In case you generated a key switching key that can accommodate _one_ rotation, you can also issue

```python
ct_rotated = engine.rotated_single(ct, rotk, shift)
```

.

### Conjugate

You can conjugate a cipher text by simply calling

```python
ct_conj = engine.conjugate(ct, conjk)
```

.

### Key Switch

Note that key switching occurs internally when performing multiplication, rotation, and conjugation. For some reason if you want to change the secret key used to encrypt a cipher text, you can do

```python
ct_new = engine.switch_key(ct, ksk)
```

, where the original secret key and the new secret key are embedded in the key switching key `ksk`. Note that the `ct` must already have been encrypted with the secret key that matches the original secret key embedded in the `ksk`. Otherwise, you will be confronted with a broken cipher text.

## Support Functions

If necessary, you can use the following functions to have more control over homomorphic operations. The functions are issued in the calls of homomorphic multiplication, however you can have finer control over how the multiplications are done by calling them individually.

### Atomic Operations of Multiplications

Homomorphic multiplication, is in fact a sequential combination of `rescale`, `cc_mult`, and `relinearize`. In building a computation circuit you can apply some optimization techniques at atomic levels to achieve greater performance. The following are such atomic functions that consist a multiplication.

```python
d0, d1, d2 = engine.cc_mult(ct_a, ct_b, relin=False).data
```

, where `a` and `b` are input parameters that can be a cipher text, an encoded plain text, or a message respectively, and `d0`, `d1`, and `d2` are components of a raw product.

```python
ct_relin = engine.relinearlization(ct_mult, evk)
```

, where `ct_mult` has d0, d1, and d2 are the results of applying the something manual multiply function, and `new_ct` is the relinearized cipher text.

```python
ct_rescaed = engine.rescale(ct, exact_rounding=True)
```

, `ct_rescaled` have plus one levels compared to the `ct`.

The rationale behind the design decision is that the rescaling is always done _at least_ in pairs. Obviously, in multiplications two cipher texts get involved and since we are rescaling _before_ multiplication the operation necessitates rescaling of the two involving cipher texts.

### Level up

A circumstance will occur frequently where you want to operate on two cipher text but their levels are different. The level up function is the rescue for such circumstances. Matching levels of cipher texts is not an easy business as it might look at first glance; Deviation from rescaling kicks in and the deviation must match at both operands together with the RNS channel lengths. The following function will do the job for you and ease up the headache.

```python
ct_new = engine.level_up(ct, level_to)
```

The engine will do multiple (if necessasary) levelings to make the level of the `new_ct` to reach the `level_to`. If the level of the `ct` is already higher or equal to the `level_to` the function will return raising an exception.

## Runtime Reflection

You can investigate the status of the cipher text or the engine at runtime, using

```python
num_levels = engine.num_levels
```

The `num_level` gives you the number of multiplications you can do with a cipher text.

You can investigate the level at which the cipher text is by issuing

```python
level = ct.level
```

. Note that the `level` goes _up_ from zero until it reaches depletion at the `num_level`. That means a freshly encrypted cipher texts will always have the _level 0_.

You can figure out the kind of a text by calling

```python
data_struct_str = data_struct.kind
```

. The text can be a `plain text`, a `key`, or an `array of keys` (a key switching key). The `data_struct_str` will give you a self-explanatory description of what kind of the `data_struct` is.

## Utility functions such as save, load, export, import, upload (to GPUs), and download (from GPUs).

### Save

```python
engine.save(data_struct, filename=None)
```

You can save all variable with `data_struct` like `cipher text`, `secret key`, `public key`, `key swithching key`, `rotation key` and `galois key`.

```python
sk = engine.create_secret_key()
engine.save(sk, filename="./sk.pkl")
```

### Load

```python
data_struct = engine.load(filename, move_to_gpu=True)
```

You can load all variable with `data_struct` like `cipher text`, `secret key`, `public key`, `key swithching key`, `rotation key` and `galois key` and you can use `move_to_gpu` to select whether to move to GPU(True) or CPU(False).

```python
sk = engine.load(filename="./sk.pkl", move_to_gpu=False)
```

### Clone

```
data_struct_ = engine.clone(data_struct)
```

You can copy a variable with data\_struct using Clone function.

### CPU

```python
data_struct_cpu = engine.cpu(data_struct)
```

### CUDA

```python
data_struct_gpu = engine.cuda(data_struct)
```

### Print Data Struct

```python
engine.print_data_structure(text, level)
──┬── cipher text
  ├── tensor at device 1 with shape torch.Size([3, 32768]).
  ├── tensor at device 0 with shape torch.Size([1, 32768]).
  ├── tensor at device 1 with shape torch.Size([3, 32768]).
  └── tensor at device 0 with shape torch.Size([1, 32768]).
```

_**TBD**_
