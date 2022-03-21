# Primecoin: Cryptocurrency with Prime Number Proof-of-Work

## Problems

One of the disadvantages of Bitcoin is that its mining algorithm has little-real-world value.

## Efficient Verification of PoW

For the primecoin design three types of prime chains are accepted as proof-of-work: Cunningham chain of first kind, Cunningham chain of second kind, and bi-twin chain. The primes in the prime chain are subject to a maximum size protocol in order to ensure efficient verification on all nodes(require the primes not to be too large).

The classical Fermat test of base 2 is used together with Euler-Lagrange-Lifchitz test to verify probable primality for prime chains.

```js
bi-twin chain:5 7 11 13
the first Cunningham chain:1531 3061 6121 12241 24481
the second Cunningham chain:2 5 11 23 47
```

## Non-Reusability of PoW

For the purposes of Primecoin, the "origin" of a bi-twin is defined as the average of the first pair, and for single Cunningham chains the origin is what the average of the first pair would be if the Cunningham chain's twin also existed. For example, the origins of the two single Cunningham chains given above are 1530 and 3, respectively.

The restriction is that the origin of a prime chain must be divisible by the hash of the block. The quotient of the division then becomes the proof-of-work certificate. Block hash, the value that is embedded in the child block, is derived from hashing the header together with the proof-of-work certificate.

```js
block hash = H(block header || certificate)
certificate = block hash / origin
```

## Difficulty Adjustability of PoW

The remainder of Fermat test could be used to construct a relatively linear continuous difficulty curve for a given prime chain length.

Let k be the prime chain length. The prime chain is P0, P1, ..., Pk-1. Let r be the Fermat test remainder of the next number in chain Pk. Now pk / r is used to measure the difficulty of the chain. The prime chain length is then computed with a fractional length part:

```js
d = k + (pk - r) / pk
```

Length target is stepped up or down through integral boundaries during length target adjustment, at fixed step-up/step-down threshold of 225/256 - 1.

## Reference

[论文链接](../../articles/blockchain/consensus/Primecoin.pdf)

[PRIMECOIN: THE CRYPTOCURRENCY WHOSE MINING IS ACTUALLY USEFUL](https://bitcoinmagazine.com/business/primecoin-the-cryptocurrency-whose-mining-is-actually-useful-1373298534)

