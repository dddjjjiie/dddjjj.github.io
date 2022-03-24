# MemoHash

## SeqMemoHash

Let H be one-way compression function. Let D be a hash digest size. Let s be the master-secret whose size equals D. Let M\[i\] for $0 \le i \lt N$ be the memory array, where the memory cell holds D bytes. Let R be the number of rounds of the function. The output is the memory array M.

```python
SeqMemoHash(s, R, N)
(1) Set M[0] = s
(2) For i=1 to N-1 do set M[i] = H(M[i-1])
(3) For r=1 to R do
	(a) For b = 0 to N-1 do
    	(i) M(b) = H(M[(b-1+N) % N]) || M[b]
```

if we set $R \ge N+1$, then SeqMemoHash is strict memory hard.

## RandMemoHash

The inputs of each hashing step of SeqMemoHash are known in advance, so it is possible to use an optimal register allocation algorithm to reuse as many intermediate results as possible. So we can propose a slight modification of previous function that prevents any register allocation algorithm to know in advance which registers will be needed in the future.

```js
RandMemoHash(s, R, N)
(1) Set M[0]  s
(2) For i=1 to N-1 do set M[i] = H(M(i-1))
(3) For r=1 to R do
    (a) For b=0 to N-1 do
        (i) p = (b-1+N) % N
	   (ii) q = AsInteger(M[p]) % (N-1)
	  (iii) j = (b+q) % N
	   (iv) M[b] = H(M[p] || M[j])
```

## Reference

[STRICT MEMORY HARD HASHING FUNCTIONS](http://www.hashcash.org/papers/memohash.pdf)

