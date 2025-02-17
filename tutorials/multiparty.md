# Multiparty

## Multiparty

The Multiparty FHE (MFHE) approach combines the features of a distributed trust model and the privacy-preserving capabilities of homomorphic encryption. In the MFHE, no single participant stores the secret key for decryption. This decryption system ensures that no one can access encrypted data alone, and decryption requires consensus from all participants.

Liberate.FHE provides a $$n$$ of $$n$$ multiparty FHE algorithm where $$n$$ is the number of participants. $$n$$ of $$n$$ means that all participants who were involved in the key generation process must also be involved in the decryption process. Liberate.FHE's MFHE is designed to follow the technique introduced in [this](https://eprint.iacr.org/2011/613) and [this](https://eprint.iacr.org/2020/304) papers. This approach shares similarities with the single-key method in terms of homomorphic operations but differs in the manner keys are generated. All participants have their own secret keys, and a collective public key is generated by the participants' public keys. Therefore the collective encryption key should be generated again whenever there is a change in the member of participants. However, the computation time and the size of a ciphertext are almost identical to that of the single-key encryption. Decryption should be done interactively with the participation of all parties involved. Liberate.FHE offers user-friendly APIs of all the MFHE processes for supporting real-world multiple participants' use cases.

In this tutorial, we will explore the usage of the Multiparty scheme in Liberate.FHE. The structure of this tutorial is as follows:

1. Required Package Import
2. Parameter Configuration & Generate CKKS Engine
3. Generation of keys required for operations
   1. Secret Key
   2. Collective Public Key
   3. Collective Evaluation Key
   4. Collective Rotation Key
4. Encryption
5. Homomorphic Operations
6. Decryption

## Required package import

```python
import numpy as np
from liberate import fhe
from liberate.fhe import presets
```

Import the necessary packages in Liberare.FHE.



## Parameter configuration & generate CKKS engine&#x20;

```python
grade = "silver"
params = presets.params[grade]

engine = fhe.ckks_engine(**params, verbose=True)
```

Set up the homomorphic parameters and create the CKKS engine.



## Generation of keys

### Secret key

```python
# set number of parties
num_parties = 5

# generate each party's secret key
sks = [engine.create_secret_key() for _ in range(num_parties)]
```

Set the number of participants in num\_parties and generate the corresponding secret key.



### Collective public key

```python
# generate collective public key
pks = [engine.create_public_key(sk=sks[0])]
# share the crs every party
crs = engine.multiparty_public_crs(pk=pks[0])
for sk in sks[1:]:
    pks.append(engine.create_public_key(sk=sk, a=crs))

cpk = engine.multiparty_create_collective_public_key(pks=pks)
```

To generate a collective public key, several steps are required. First, the Common Reference String (CRS), which is a value that each party needs to share, is generated. Then, each party generates their public key using their own secret key and the shared CRS. After generating the public key for each party, the collective public key is created using these keys and shared with each party.



### Collective evaluation key

```python
# generate collective evaluation key
evks_share = [engine.create_key_switching_key(sks[0], sks[0])]
# share the crs every party
crs = engine.generate_rotation_crs(evks_share[0])
# generate each party's evk_share
for sk in sks[1:]:
    evks_share.append(engine.multiparty_create_key_switching_key(sk, sk, a=crs))

# summation evk shares
evk_sum = engine.multiparty_sum_evk_share(evks_share)

# share the evk_sum each party
evk_sum_mult = [engine.multiparty_mult_evk_share_sum(evk_sum, sk) for sk in sks]

# summation evk_sum_mult
cevk = engine.multiparty_sum_evk_share_mult(evk_sum_mult)
```

All participants will be shared with the CRS value. Then, each party will generate the secret shared evks\_share using the shared CRS and their own secret key (sk). The generated evks\_share will be used to create evks\_sum and shared with all parties. Each participant will use the shared evks\_sum and their sk to generate evk\_sum\_mult, and then share the generated evk\_sum\_mult to create the collective evaluation key (cevk) and share it with all participants.



### Collective rotation key

```python
# crotk
delta = 5
rotks = [engine.multiparty_create_rotation_key(sk=sks[0], delta=delta)]
crss = engine.generate_rotation_crs(rotks[0])

for sk in sks[1:]:
    rotks.append(engine.multiparty_create_rotation_key(sk=sk, delta=delta, a=crss))
crotk = engine.multiparty_generate_rotation_key(rotks)
```

The process of generating the Collective Rotation Key (crotk) is the same as the process of [cpk](multiparty.md#collective-public-key).



## Encryption

```python
m0 = engine.example(-1, 1)
m1 = engine.example(-10, 10)

ct0 = engine.encorypt(m0, cpk)
ct1 = engine.encorypt(m1, cpk, level=3)
```

Encryption follows the same process as general encryption, except that it uses `cpk`.



## Homomorphic operations

```python
ct_mult = engine.mult(ct0, ct1, cevk)
ct_rot = engine.rotate_single(ct1, rotk=crotk)
```

The Homomorphic Encryption operation is similar to the usual process of Homomorphic Encryption, with the only difference being that it utilizes a collectively generated key by all participants.



## Decryption

```python
##### just decrypt
pcts = [engine.multiparty_decrypt_head(ct0, sks[0])]
for sk in sks[1:]:
    pcts.append(engine.multiparty_decrypt_partial(ct0, sk))
m_ = engine.multiparty_decrypt_fusion(pcts, level=ct0.level)

##### decrypt mult
pcts = [engine.multiparty_decrypt_head(ct_mult, sks[0])]
for sk in sks[1:]:
    pcts.append(engine.multiparty_decrypt_partial(ct_mult, sk))
m_ = engine.multiparty_decrypt_fusion(pcts, level=ct_mult.level)
m_mult = m0 * m1

######## decrypt rotate
pcts = [engine.multiparty_decrypt_head(ct_rot, sks[0])]
for sk in sks[1:]:
    pcts.append(engine.multiparty_decrypt_partial(ct_rot, sk))
m_roll = engine.multiparty_decrypt_fusion(pcts, level=ct_rot.level)
```

The process of decryption consists of three steps. These steps are `multiparty_decrypt_head`, `multiparty_decrypt_partial`, and the final step `multiparty_decrypt_fusion`. One participant uses the `head` function, while the remaining participants use the `partial` function. Then, the participant who wants to see the decrypted result executes the `fusion` function.



The code used in this tutorial is as follows:

```python
import numpy as np
from liberate import fhe
from liberate.fhe import presets

grade = "silver"
params = presets.params[grade]

engine = fhe.ckks_engine(**params, verbose=True)

# set number of parties
num_parties = 5

# generate each party's secret key
sks = [engine.create_secret_key() for _ in range(num_parties)]

###########################################
# generate collective public key
pks = [engine.create_public_key(sk=sks[0])]
# share the crs every party
crs = engine.multiparty_public_crs(pk=pks[0])
for sk in sks[1:]:
    pks.append(engine.create_public_key(sk=sk, a=crs))

cpk = engine.multiparty_create_collective_public_key(pks=pks)

###########################################
# generate collective evaluation key
evks_share = [engine.create_key_switching_key(sks[0], sks[0])]
# share the crs every party
crs = engine.generate_rotation_crs(evks_share[0])
# generate each party's evk_share
for sk in sks[1:]:
    evks_share.append(engine.multiparty_create_key_switching_key(sk, sk, a=crs))

# summation evk shares
evk_sum = engine.multiparty_sum_evk_share(evks_share)

# share the evk_sum each party
evk_sum_mult = [engine.multiparty_mult_evk_share_sum(evk_sum, sk) for sk in sks]

# summation evk_sum_mult
cevk = engine.multiparty_sum_evk_share_mult(evk_sum_mult)

###########################################
# crotk
delta = 5
rotks = [engine.multiparty_create_rotation_key(sk=sks[0], delta=delta)]
crss = engine.generate_rotation_crs(rotks[0])

for sk in sks[1:]:
    rotks.append(engine.multiparty_create_rotation_key(sk=sk, delta=delta, a=crss))
crotk = engine.multiparty_generate_rotation_key(rotks)

###########################################
# cgalk
galks = [engine.create_galois_key(sks[0])]
crss = engine.generate_galois_crs(galks[0])
for sk in sks[1:]:
    galks.append(engine.multiparty_create_galois_key(sk=sk, a=crss))

cgalk = engine.multiparty_generate_galois_key(galks)

##########
m0 = engine.example(-1, 1)
m1 = engine.example(-1, 1)
ct0 = engine.encorypt(m0, cpk)
ct1 = engine.encorypt(m1, cpk, level=3)

####
ct_mult = engine.mult(ct0, ct1, cevk)
ct_rot = engine.rotate_single(ct1, rotk=crotk)

#####
pcts = [engine.multiparty_decrypt_head(ct0, sks[0])]
for sk in sks[1:]:
    pcts.append(engine.multiparty_decrypt_partial(ct0, sk))
m_ = engine.multiparty_decrypt_fusion(pcts, level=ct0.level)

#####
pcts = [engine.multiparty_decrypt_head(ct_mult, sks[0])]
for sk in sks[1:]:
    pcts.append(engine.multiparty_decrypt_partial(ct_mult, sk))
m_ = engine.multiparty_decrypt_fusion(pcts, level=ct_mult.level)
m_mult = m0 * m1

########
pcts = [engine.multiparty_decrypt_head(ct_rot, sks[0])]
for sk in sks[1:]:
    pcts.append(engine.multiparty_decrypt_partial(ct_rot, sk))
m_roll = engine.multiparty_decrypt_fusion(pcts, level=ct_rot.level)

m_roll = np.roll(m1, delta)

print(f">> result decrypt : {np.abs(m_ - m0).max()}")
print(f">> result mult    : {np.abs(m_ - (m_mult)).max()}")
print(f">> result rot     : {np.abs(m_roll - (m_roll)).max()}")
```
