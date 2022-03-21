# Salsa20

Salsa20 is stream cipher, it is built on a pseudorandom function based on add-rotate-XOR operations——32-bit addition, bitwise addition(XOR) and rotation operations. The core function maps a 256-bit key, a 64-bit nonce, and a 64-bit counter to a 512-bit block of the key stream.

## Stucture

Internally, the cipher uses bitwise addition$\bigoplus$, 32-bit addition mod $2^{32}$ ⊞, and constant-distance rotation operations(<<<, 循环移位) on an internal state of sixteen 32-bit words. The internal state is made of sixteen 32-bit words arranged as a 4$\times$4 matrix.

| 0    | 1    | 2    | 3    |
| ---- | ---- | ---- | ---- |
| 4    | 5    | 6    | 7    |
| 8    | 9    | 10   | 11   |
| 12   | 13   | 14   | 15   |

The initial state is made of eight words of key, two words of stream position, two words of nonce and four fixed words.

<table class="wikitable">
<caption>Initial state of Salsa20
</caption>
<tbody><tr>
<td data-sort-value="" style="background: #ececec; color: #2C2C2C; vertical-align: middle; text-align: center;" class="table-na">"expa"
</td>
<td style="background:#0099ff">Key
</td>
<td style="background:#0099ff">Key
</td>
<td style="background:#0099ff">Key
</td></tr>
<tr>
<td style="background:#0099ff">Key
</td>
<td data-sort-value="" style="background: #ececec; color: #2C2C2C; vertical-align: middle; text-align: center;" class="table-na">"nd 3"
</td>
<td style="background:#93db83">Nonce
</td>
<td style="background:#93db83">Nonce
</td></tr>
<tr>
<td style="background:#ff7474"><abbr title="Position">Pos.</abbr>
</td>
<td style="background:#ff7474"><abbr title="Position">Pos.</abbr>
</td>
<td data-sort-value="" style="background: #ececec; color: #2C2C2C; vertical-align: middle; text-align: center;" class="table-na">"2-by"
</td>
<td style="background:#0099ff">Key
</td></tr>
<tr>
<td style="background:#0099ff">Key
</td>
<td style="background:#0099ff">Key
</td>
<td style="background:#0099ff">Key
</td>
<td data-sort-value="" style="background: #ececec; color: #2C2C2C; vertical-align: middle; text-align: center;" class="table-na">"te k"
</td></tr></tbody></table>

The constant words spell "expand 32-byte k" in ASCII (i.e. the 4 words are "expa", "nd 3", "2-by", and "te k"). For example, a key is (1, 2, 3, ...., 32)，nonce is (3, 1, 4, 1, 5, 9, 2, 6), and block number is 7, the original matrix is:

```js
0x61707865, 0x04030201, 0x08070605, 0x0c0b0a09,
0x100f0e0d, 0x3320646e, 0x01040103, 0x06020905,
0x00000007, 0x00000000, 0x79622d32, 0x14131211,
0x18171615, 0x1c1b1a19, 0x201f1e1d, 0x6b206574.
```

The core operation in Salsa20 is the quater-round QR(a, b, c, d) that takes a four-word input and produces a four-word output:

```js
b ^= (a + d) <<< 7;
c ^= (b + a) <<< 9;
d ^= (c + b) <<< 13;
a ^= (d + c) <<< 18;
```

Odd-numbered rounds apply QR(a, b, c, d) to each of the four columns in the 4$\times$4 matrix, and even-numbered rounds apply it to each of the four rows. Two consecutive rounds(column-round and row-round) together are called a double-round:

```js
// Odd round
QR( 0,  4,  8, 12)	// column 1
QR( 5,  9, 13,  1)	// column 2
QR(10, 14,  2,  6)	// column 3
QR(15,  3,  7, 11)	// column 4
// Even round
QR( 0,  1,  2,  3)	// row 1
QR( 5,  6,  7,  4)	// row 2
QR(10, 11,  8,  9)	// row 3
QR(15, 12, 13, 14)	// row 4
```

An implementation in C/C++ appears below

```c++
#include <stdint.h>
#define ROTL(a,b) (((a) << (b)) | ((a) >> (32 - (b))))
#define QR(a, b, c, d)(		\
	b ^= ROTL(a + d, 7),	\
	c ^= ROTL(b + a, 9),	\
	d ^= ROTL(c + b,13),	\
	a ^= ROTL(d + c,18))
#define ROUNDS 20
 
void salsa20_block(uint32_t out[16], uint32_t const in[16])
{
	int i;
	uint32_t x[16];

	for (i = 0; i < 16; ++i)
		x[i] = in[i];
	// 10 loops × 2 rounds/loop = 20 rounds
	for (i = 0; i < ROUNDS; i += 2) {
		// Odd round
		QR(x[ 0], x[ 4], x[ 8], x[12]);	// column 1
		QR(x[ 5], x[ 9], x[13], x[ 1]);	// column 2
		QR(x[10], x[14], x[ 2], x[ 6]);	// column 3
		QR(x[15], x[ 3], x[ 7], x[11]);	// column 4
		// Even round
		QR(x[ 0], x[ 1], x[ 2], x[ 3]);	// row 1
		QR(x[ 5], x[ 6], x[ 7], x[ 4]);	// row 2
		QR(x[10], x[11], x[ 8], x[ 9]);	// row 3
		QR(x[15], x[12], x[13], x[14]);	// row 4
	}
	for (i = 0; i < 16; ++i)
		out[i] = x[i] + in[i];
}
```



