# PBKDF2

PBKDF2 is key derivation function with a sliding computational cost, used to reduce vulnerabilities of brute-force attacks.

PBKDF2 applies a pseudorandom function, such as hash-based message authentication code(HMAC)，to the input password or passphrase along with a salt value and repeats the process many times to produce a derived key

## Key derivation process

The PBKDF2 key derivation function has five input parameters:

```js
DK = PBKDF2(PRF, Password, Salt, c, dkLen)
```

where:

* PRF is a pseudorandom function of two parameters with output length hLen
* Password is the master password from which a derived key is generated
* Salt is a sequence of bits
* c is the number of iterations desired
* dkLen is the desired bit-length of the derived key
* DK is the generated derived key

Each hLen-bit block Ti of derived key DK, is computed as follows

```js
DK = T1 || T2 || ... || T(dklen/hlen)
Ti = F(Password, Salt, c, i)
```

The function F is the xor of c iterators of chained PRFS. The first iteration of PRF uses Password as PRF key and Salt concatenated with i encoded as a big-endian 32-bit integer as the input. Subsequent iterations of PRF use Password as the PRF key and the output of the previous PRF coputation as the input:

```js
F(Password, Salt, c, i) = U1 ^ U2 ^ ... ^ Uc
```

where:

```js
U1 = PRF(Password, Salt+INT_32_BE(i))
U2 = PRF(Password, U1)
..
Uc = PRF(Password, Uc-1)
```

For example, WPA2 uses:

```hs
DK = PBKDF2(HMAC-SHA1, passphrase, ssid, 4096, 256)
```

## 参考

[PBKDF2](https://en.wikipedia.org/wiki/PBKDF2)

