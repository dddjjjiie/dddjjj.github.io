# Linear Congruential Generators

LGC us a source of random numbers. The form of generator is
$$
X_{i+1} = (aX_i + c) \; mod \; m, with \; 0 \le X_i \le m
$$
m is called modulus, $X_0$, a, and c are known as the seed, multiplier, and the increment respectively

For example, consider m=31, a=7, c=0 and begin with $X_0 = 19$, The next integers in the sequence are

```js
9, 1, 7, 18, 2, 14, 5, 4, 28, 10, 8, 25, 20, 16, 19,
```

**The Period**

The period can be no greater than m, A full period generator is one in which the period is m, and it is obtained if:

* c is relatively prime to m
* (a-1) is a multiple of q, for each prime factor q of m
* (a-1) is a multiple of 4, if m is

**The Increment**

if c > 0, we can achieve a full period by such:

* m = $2^b$
* set (a-1) as a multiple of 4
* c should be odd-valued

if c = 0, a full period is not possible. A maximum number of random variables can be achieved by such:

* A maximum period generator which period is m-1，is one in which a is primitive element(本原元) modulo m，if m is prime
* m is often set to the largest prime number less than $2^b$.The most commonly used modulus is $2^31 - 1$

## 参考

[Linear congruential generator](https://en.wikipedia.org/wiki/Linear_congruential_generator)