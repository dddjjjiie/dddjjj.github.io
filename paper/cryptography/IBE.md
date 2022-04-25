# Identity-Based Encryption from the Weil Pairing

The original motivation for identity-based encryption was to simplify certificate management in e-mail systems.

## Bilinear Map

Let $G_1$ and $G_2$ be two cyclic groups of order $q$ for some large prime q. A map $e: G_1 \times G_1 \to G_2$ is said to be bilinear if $e(aP, bQ) = e(P, Q)^{ab}$ for all $P, Q \in G_1$ and all $a, b \in Z$.

## MapToPoint

Let $p$ be a prime satisfying $p=2 mod 3$ and $p=6q-1$ for some prime $q>3$. Let $E$ be the elliptic curve $y^2=x^3+1$ over $F_p$. This IBE scheme makes use of a simple algorithm for converting an arbitrary string $ID \in \{0, 1\}^*$ to a point $Q_{ID} \in E/F_p$ of order $q$. We refer to this algorithm as MapToPoint. Let $G$ be a cryptographic hash function $G: \{0, 1\}^* \to F_p$. Algorithm works as follows:

* Compute $y_0 = G(ID)$ and $x_0 = (y_0^2 - 1)^{1/3}$mod $p$.
* Let $Q=(x_0, y_0) \in E/F_p$. Set $Q_{ID} = 6Q$. Then $Q_{ID}$ has order $q$ as required.

## BasicIdent

### Setup

The algorithm works as follows:

* Choose a large k-bit prime $p$ such that $p = 2 mod 3$ and $p=6q-1$ for some $q \gt 3$. Let $E$ be the elliptic curve defined by $y^2 = x^3 + 1$ over $F_p$. Choose an arbitrary $P \in E/F_p$ of order $q$.
* Pick a random $s \in Z_p^*$ and set $P_{pub} = sP$.
* Choose a cryptographic hash function $H: F_{p^2} \to \{0, 1\}^n$ for some n. Choose a cryptographic hash function $G: \{0, 1\}^* -> F_p$.

The message space is $M = \{0, 1\}^n$. The ciphertext space is $C=E/F_p \times \{0, 1\}^n$. The system parameters are $params = <p, n, P, P_{pub}, G, H>$. The master-key is $s \in Z_q$.

### Extract

For a given string $ID \in \{0, 1\}^*$ the algorithm builds a private key $d$ as follows:

* Use MapToPoint to map ID to a point $Q_{ID} \in E/F_p$ of order q.
* Set the private key $d_{ID} = sQ_{ID}$ where s is the master key.

### Encrypt

To encrypt $M$ under the public key ID do the following:

* choose a random $r \in Z_q$ and set the ciphertext to be

$$
C = <rP, M \oplus H(g_{ID}^r) > \text{where } g_{ID}=e(Q_{ID}, P_{pub}) \in F_{p^2}
$$

### Decrypt

Let $C = <U, V>$ be a ciphertext encrypted using the public key ID. If $U \in E/F_p$ is not a point of order $q$ reject the ciphertext. Otherwise, to decrypt $C$ using the private key $d_{ID}$ compute:
$$
V \oplus H(e(d_{ID}, U)) = M \oplus H(e(Q_{ID}, P_{pub})^r) \oplus H(e(d_{ID}, rP))  = M
$$

## IBE with Chosen Ciphertext Security

### Setup

The algorithm works as follows:

* Choose a large k-bit prime $p$ such that $p = 2 mod 3$ and $p=6q-1$ for some $q \gt 3$. Let $E$ be the elliptic curve defined by $y^2 = x^3 + 1$ over $F_p$. Choose an arbitrary $P \in E/F_p$ of order $q$.
* Pick a random $s \in Z_p^*$ and set $P_{pub} = sP$.
* Choose a cryptographic hash function $H_1: \{0, 1\}^n \times \{0, 1\}^n \to F_q$,  $G: \{0, 1\}^n -> \{0, 1\}^n$.

The message space is $M = \{0, 1\}^n$. The ciphertext space is $C=E/F_p \times \{0, 1\}^n$. The system parameters are $params = <p, n, P, P_{pub}, G, H>$. The master-key is $s \in Z_q$.

### Extract

For a given string $ID \in \{0, 1\}^*$ the algorithm builds a private key $d$ as follows:

* Use MapToPoint to map ID to a point $Q_{ID} \in E/F_p$ of order q.
* Set the private key $d_{ID} = sQ_{ID}$ where s is the master key.

### Encrypt

To encrypt M under the public key ID do the following:

* choose a random $\sigma \in \{0, 1\}^n$
* set $r = H_1(\sigma, M)$
* set the ciphertext to be

$$
C = <rP, \sigma \oplus H(g_{ID}^r, M \oplus G_1(\sigma)) \text{ where } g_{ID} = e(Q_{ID}, P_{pub}) \in F_{p^2}
$$

### Decrypt

Let $C = <U, V, W>$ be a ciphertext encrypted using the public key ID. If $U \in E/F_p$ is not a point of order $q$ reject the ciphertext. To decrypt $C$ using the private key $d_{ID}$ do:

* Compute $V \oplus H(e(d_{ID}, U)) = \sigma \oplus H(e(Q_{ID}, P_{pub})^r) \oplus H(e(d_{ID}, rP)) = \sigma$
* Compute $w \oplus G_1(\sigma) = M \oplus G_1(\sigma) \oplus G_1(\sigma) = M$