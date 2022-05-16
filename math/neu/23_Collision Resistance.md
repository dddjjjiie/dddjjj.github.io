 

# Collision Resistance

Let $H: M \to T$ be a hash function ($|M| >> |T|$), A collision for $H$ is a pair $m_0, m_1 \in M$ such that:
$$
H(m_0) = H(m_1) \; \text{and} \; m_0 \neq m_1
$$
A function $H$ is collision resistant if for all efficient algorithms $A$, it's hard to find collisions for this function.
$$
Adv_{CR}[A, H] = Pr[A \; \text{outputs collision for } H] \text{ is negligible}
$$

## MACs from Collision Resistance

Let $I = (S, V)$ be a MAC for short messages over $(K, M, T)$, Let $H:M^{big} \to M$. Define $I^{big} = (S^{big}, V^{big})$ over $(K, M^{big}, T)$ as :
$$
S^{big}(k, m) = S(k, H(m)) \\
V^{big}(k, m, t) = V(k, H(m), t)
$$
**Theorem:** If $I$ is a secure MAC and $H$ is collision resistant then $I^{big}$ is a secure MAC.

Suppose adversary can find $m_0 \neq m_1$ such that: $H(m_0) = H(m_1)$. Then $S^{big}$ is insecure under a 1-chosen message attack:

1. adversary asks for $t \leftarrow S(k, m_0)$.
2. adversary output $(m_1, t)$ as forgery.

## Protecting file Integrity Using Collision Resistance Hash

A user wants to download a software package and he wants to make sure that he really did get a version of the package that he downloaded and it's not some version that the attacker tampered with and modified its contents. So he could refer to a read-only public space that's relatively small. All it has to do is hold small hashes of these software packages. The only requirement is that this space is read-only. And then once he consults this public space, he can very easily compute the hash of a package that he downloaded. Compare it to the value in the public space. And if the two match, then he knows that the version of the package he downloaded is correct.

## MAC VS Hash

|                 | MAC  | Hash |
| --------------- | ---- | ---- |
| Key             | Yes  | No   |
| Read-Only Space | No   | Yes  |

## Generic Attack

Let $H: M \to \{0, 1\}^n$ be a hash function($|M| >> 2^n$). Generic algorithm to find a collision in time $O(2^{n/2})$ hashes:

1. Choose $2^{n/2}$ random message in $M: m_1, ..., m_{2^{n/2}}$.
2. For $i = 1,...,2^{n/2}$ compute $t_i = H(m_i) \in \{0, 1\}^n$.
3. Look for a collision ($t_i = t_j$). If not found, got back to step1.

### Birthday Paradox

Let $r_1, ..., r_n \in \{1, ..., B\}$ be independent identically distributed integers.

**Theorem:** When $n = 1.2 \times B^{1/2}$ then $Pr[\exists i \neq j: r_i = r_j] \geq 1/2$.

Note: Uniform distribution is the worst case for the birthday paradox. In other words, If the distribution from which $r_i$ are sampled from is non-uniform, that in fact fewer than $1.2 \times B^{1/2}$ samples are needed. The uniform distribution is the worst case.

**Proof**:
$$
\begin{align}
Pr[\exists i \neq j: r_i = r_j] & = 1 - Pr[\forall i \neq j: r_i \neq r_j] \\
& = 1 - \frac{B-1}{B} \cdot \frac{B-2}{B} \cdot \cdot \cdot \frac{B-n+1}{B} \\
& = 1 - \prod_{i=1}^{n-1} (1 - \frac{1}{i}) \geq 1 - \prod_{i=1}^{n-1} e^{-i/B} \geq 1 - e^{-n^2/2B}\\
\text{Because } 1-x \leq e^{-x} \\
\end{align}
$$
When $n = 1.2 \times B^{1/2}$, then $n^2/2 = 0.72 \cdot B$, then
$$
1 - e^{-n^2/2B} = 1 - e^{0.72} = 0.53
$$
