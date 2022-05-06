# DES(Data Encryption Standard)

## Feistel Network

Given functions $f_1,..., f_d: \{0, 1\}^n \to \{0, 1\}^n$.

**Goal**: Build an invertible function out of $d$ functions $F: \{0, 1\}^{2n} \to \{0, 1\}^{2n}$.

There are two blocks of n bits. In other words, the input is actually 2n bits. The R and L stand for right and left. If we follow the R inputs, it gets copied into the L output without any change at all. However, the L inputs is changed somewhat. What happens is, the R inputs is fit into the function $f_1$ and the result is then XORed with $L_0$ and that becomes the new $R_1$.

![1651839810354](../../img/1651839810354.png)
$$
L_i = R_{i-1} \\
R_i = f_i(R_{i-1}) \oplus L_0
$$
**Claim**: for all $f_1,..., f_d: \{0, 1\}^n \to \{0, 1\}^n$, Feistel network $F: \{0, 1\}^{2n} \to \{0, 1\}^{2n}$ is invertible.

**Proof**: Let's look at one round of a Feistel network. Suppose the input that we're given is $R_{i+1}, L_{i+1}$ and we want to compute $R_i, L_i$. So we want to compute the round in the reverse direction.

![1651840387516](../../img/1651840387516.png)
$$
R_{i_1} = L_i \\
L_{i-1} = R_i \oplus f_i(L_i)
$$
**Theorem**: Let $f: K \times \{0, 1\}^n \to \{0, 1\}^n$ a secure PRF. So it's indistinguishable from random. Then it turns out that if you use this function in three rounds of a Feistel network, what you end up with is a secure pseudo random permutation. In other words, what you end up with is an invertible function that is indistinguishable from a truly random invertible function.

## DES

DES ia a 16 round Feistel network, so there are functions $f_1, ..., f_{16}: \{0, 1\}^{32} \to \{0, 1\}^{32}$ that map 32 bits to 32 bits. And as a result, the DES itself acts on 64 bit blocks, $2 \times 32$. All the sixteen round function in DES are actually all derived from a single function $F$. Just used with different keys.
$$
f_i(x) = F(k_i, x)
$$
Suppose you have a 64 bit input, the first thing it does is this initial permutation that just permutes the 64 bits around. This is not for security reasons, this is just specified in the standard. Then we go into the sixteen round Feistel network which uses the function $f_1$ to $f_{16}$ as specified before. Then we have another permutation called the final permutation, which just the inverse of the initial permutation. There's a key expansion step, the 56-bit DES key is expanded into these rounds keys $k_1, ..., k_{16}$, where each round key is 48 bits.

![1651841730960](../../img/1651841730960.png)

### Function F

It takes 32 bit value $x$ as input, remember this is $R_0, ..., R_{14}$ and so on. And then it takes also a 48 bit round key $k_i$. The first thing it does is goes through an expansion box, which takes 32 bits and maps them into 48 bits. All the expansion box does is just replicate some bits and move other bits around. The next thing we do is we compute an XOR with the round key. The 48 bits are broken into 8 groups of 6 bits, and they go into what are called S boxes. The S box is a map, 6 bits to 4 bits. So the output of the S boxes are 32 bits. And finally this is fed into another permutation which just maps the bits around. So it just permutes the 32 bits around and that's the final 32 bit output of this $F$ function.

![1651842486134](../../img/1651842486134.png)

### S box

These S boxes are just functions from 6 bits to 4 bits. They are just implemented as a look up table. For example, if you want to look at the output that corresponds to "011011".  Then you look at the first and last bit "01", and then look at the middle 4 bits "1101", and you see that the output is 1001.

![1651842692520](../../img/1651842692520.png)