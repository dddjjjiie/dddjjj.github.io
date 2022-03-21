# Scrypt

Scrypt is a password-based key derivation. The algorithm was specifically designed to make it costly to perform large-scale custom hardware attacks by requiring large amounts of momory.

The scrypt function is designed to hinders such attempts by raising the source demands of the algorithm. Specifically, the algorithm is designed to use a large amount of memory campared to other password-based KDFs, making the size and cost of a hardware implementation much more expensive, and therefore limiting the amount of parallelism an attacker can use, for a given amount of financial resources.

## Overview

The large memory requirements of scrypt come from a large vector of pseudorandom bit  strings that are generated as part of the algorithm. Once the vector is generated, the elements of it are accessed in a pseudo-random order and combined to produce the derived key. A straightforward implementation would need to keep the entire vector in RAM so that it can be accessed as needed.

## Algorithm

```js
Function scrypt
   Inputs: This algorithm includes the following parameters:
      Passphrase:                Bytes    string of characters to be hashed
      Salt:                      Bytes    string of random characters that modifies the hash to protect against Rainbow table attacks
      CostFactor (N):            Integer  CPU/memory cost parameter – Must be a power of 2 (e.g. 1024)
      BlockSizeFactor (r):       Integer  blocksize parameter, which fine-tunes sequential memory read size and performance. (8 is commonly used)
      ParallelizationFactor (p): Integer  Parallelization parameter. (1 .. 232-1 * hLen/MFlen)
      DesiredKeyLen (dkLen):     Integer  Desired key length in bytes (Intended output length in octets of the derived key; a positive integer satisfying dkLen ≤ (232− 1) * hLen.)
      hLen:                      Integer  The length in octets of the hash function (32 for SHA256).
      MFlen:                     Integer  The length in octets of the output of the mixing function (SMix below). Defined as r * 128 in RFC7914.
   Output:
      DerivedKey:                Bytes    array of bytes, DesiredKeyLen long

   Step 1. Generate expensive salt
   blockSize ← 128*BlockSizeFactor  // Length (in bytes) of the SMix mixing function output (e.g. 128*8 = 1024 bytes)

   Use PBKDF2 to generate initial 128*BlockSizeFactor*p bytes of data (e.g. 128*8*3 = 3072 bytes)
   Treat the result as an array of p elements, each entry being blocksize bytes (e.g. 3 elements, each 1024 bytes)
   [B0...Bp−1] ← PBKDF2HMAC-SHA256(Passphrase, Salt, 1, blockSize*ParallelizationFactor)

   Mix each block in B Costfactor times using ROMix function (each block can be mixed in parallel)
   for i ← 0 to p-1 do
      Bi ← ROMix(Bi, CostFactor)

   All the elements of B is our new "expensive" salt
   expensiveSalt ← B0∥B1∥B2∥ ... ∥Bp-1  // where ∥ is concatenation
 
   Step 2. Use PBKDF2 to generate the desired number of bytes, but using the expensive salt we just generated
   return PBKDF2HMAC-SHA256(Passphrase, expensiveSalt, 1, DesiredKeyLen);
```

where:

* Passphrase、Salt、BlockSizeFactor、ParallelizationFactor is the input of function of the first PBKDF2，the output of this function is divided into a array with a length of p
* For each Bi，take it and CostFactor as parameters of function ROMix，use the return value of ROMix subsitute the original value of Bi
* Final，concate each element in array B as the expensiveSalt parameter and call the PBKDF2 function again

```js
Function ROMix(Block, Iterations)

   Create Iterations copies of X
   X ← Block
   for i ← 0 to Iterations−1 do
      Vi ← X
      X ← BlockMix(X)

   for i ← 0 to Iterations−1 do
      j ← Integerify(X) mod Iterations 
      X ← BlockMix(X xor Vj)

   return X
```

where:

* Block is the element of array B，Iteration is equivalent to CostFactor
* The first loop uses Bi to generate a large vector
* The second loop use the vector to generate a new value to replace Bi

```js
Function BlockMix(B):

    The block B is r 128-byte chunks (which is equivalent of 2r 64-byte chunks)
    r ← Length(B) / 128;

    Treat B as an array of 2r 64-byte chunks
    [B0...B2r-1] ← B

    X ← B2r−1
    for i ← 0 to 2r−1 do
        X ← Salsa20/8(X xor Bi)  // Salsa20/8 hashes from 64-bytes to 64-bytes
        Yi ← X

    return ← Y0∥Y2∥...∥Y2r−2 ∥ Y1∥Y3∥...∥Y2r−1
```

where:

* first split B into an array of 64 bytes each，the length of B is 128 * BlockSizeFactor(r)，so the length of the new array is 2r
* second，use a for loop to encry the result of XOR between the previous encryption result and Bi
* third，return the value of  each encryption result connection

## References

[scrypt](https://en.wikipedia.org/wiki/Scrypt)

