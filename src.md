# the introduction

conventional *[encryption](https://en.wikipedia.org/wiki/Encryption)* technology often relies on users to protect a secret key by selecting a *[password](https://en.wikipedia.org/wiki/Password)* or *[passphrase](https://en.wikipedia.org/wiki/Passphrase)*

while a strong *[passphrase](https://en.wikipedia.org/wiki/Passphrase)* is known only to the user, it also has the critical drawback that it must be remembered exactly to recover the secret key

over time, the user's ability to recall the passphrase may diminish, leading to the potential loss of access to the secret key

in their article, *["Protecting Secret Keys with Personal Entropy," Ellison et al. (2000)](https://www.researchgate.net/publication/223831757_Protecting_secret_keys_with_personal_entropy)* propose a scheme where users can protect a secret key by leveraging personal entropy, encrypting the passphrase using answers to several personal questions

for instance, the authors suggest using 22 questions, each with an *[entropy](https://en.wikipedia.org/wiki/Entropy_(information_theory))* of about 8 bits, to secure a 112 bits secret key and any 14 correct answers would be sufficient to recover the secret key

the authors posit that the most effective attack against this secret sharing scheme would be a *[brute force](https://en.wikipedia.org/wiki/Brute-force_attack)* approach targeting the minimum *[entropy](https://en.wikipedia.org/wiki/Entropy_(information_theory))* set of questions to reach the threshold

however, this scheme may not be as robust as initially described and can be compromised by a *[meet in the middle attack](https://en.wikipedia.org/wiki/Meet-in-the-middle_attack)*

this article provides a detailed security analysis of the *[personal entropy encryption scheme](https://www.researchgate.net/publication/223831757_Protecting_secret_keys_with_personal_entropy)*

# the problem

firstly the original method of encrypting the secret key using personal entropy will be explained

then the process for recovering the secret key under normal circumstances will be shown

finally let's address the main problem: how to crack the secret key without knowing the answers to the user's personal questions

some common knowledges will be denoted here at the begining

- the list of $n$ questions $\mathbf{q}$ and the list of numbers of choices $\mathbf{o}$ for each question $q_i$
- the *[hash function](https://en.wikipedia.org/wiki/Hash_function)* $H$
- the number of shares $n$, the threshold $m$ and the *[finite field](https://en.wikipedia.org/wiki/Finite_field)* modular $p$ used in *[shamir's secret sharing scheme](https://en.wikipedia.org/wiki/Shamir%27s_secret_sharing)*
- the encryption function $E$ and the decryption function $E^{-1}$ *[symmetric encryption scheme](https://en.wikipedia.org/wiki/Symmetric-key_algorithm)*

## encrypting

1. the user select an answer $a_i \in [1, o_i]$ for each question $q_i$ to generate the list of $n$ answers $\mathbf{a}$
2. generate a random number $c$ to be used as *[salt](https://en.wikipedia.org/wiki/Salt_(cryptography))*
3. compute a list of $n$ hashes $\mathbf{k}$ where $k_i = H(\langle q_i, a_i, c \rangle)$
4. split the secret message $s$ into a list of $n$ secret shares $\mathbf{y}$ by *[shamir's secret sharing scheme](https://en.wikipedia.org/wiki/Shamir%27s_secret_sharing)*: each share $y_i = \sum_{j \in [1,m]}{i^j w_j} \mod p$, where $\mathbf{w}$ is a list of constant random numbers with $w_0 = s$
5. encrypt each share $y_i$ to obtain $\mathbf{z}$ where $z_i = E_{k_i}(y_i)$
6. retain the salt $c$ and the encrypted shares $\mathbf{z}$

## reconstructing

1. ask the user the same questions $\mathbf{q}$ to generate answers $\mathbf{a}$
2. compute a list of $n$ hashes $\mathbf{k}$ where $k_i = H(\langle q_i, a_i, c \rangle)$
3. decrypt the shares $\mathbf{y}$ using the equation $y_i = E_{k_i}^{-1}(z_i)$
4. if at least $m$ questions were answered correctly, the secret $s$ can be reconstructed by *[shamir's secret sharing scheme](https://en.wikipedia.org/wiki/Shamir%27s_secret_sharing)* with the correct $m$ answers by $s = \sum_{i \in \mathcal{M}}{y_i \prod_{x \in \mathcal{M} \setminus \lbrace i \rbrace}{\frac{x}{x - i}}} \mod p$ where $\mathcal{M} \subseteq [1, n]$ contains the indices of the $m$ correct answers

## problem definition

the problem is to reconstruct the secret key $s$ by the random number $c$ and the encrpted shares $\mathbf{z}$ as well as the common knowledges denoted previously without asking user any questions

# the solution

the solution is essentially a meet in the middle attack

1. construct the hash matrix $\mathbf{K}$ where $k_{i,j} = H(\langle q_i, j, c \rangle)$
2. decrypt all possible shares $\mathbf{Y}$ where $y_{i,j} = E_{k_{i, j}}^{-1}(z_i)$
3. select $\mathcal{U}, \mathcal{V} \subset [1, n]$ where $\mathcal{U} \cap \mathcal{V} = \emptyset$ and $|\mathcal{U} \cup \mathcal{V}| \ge m$
4. construct *[lagrange basis polynomials](https://en.wikipedia.org/wiki/Lagrange_polynomial)* $L_i(x) = \prod_{j \in \mathcal{U} \cup \mathcal{V} \setminus \lbrace i \rbrace }{\frac{x - j}{i - j}}$
5. calculate the *[lagrange interpolation polynomial](https://en.wikipedia.org/wiki/Lagrange_polynomial)* matrix $\mathbf{L}$ where $L_{i, j} = y_{i,j} L_i(x)$
6. conbime the *[lagrange interpolation polynomials](https://en.wikipedia.org/wiki/Lagrange_polynomial)* over $\mathcal{U}$ and $\mathcal{V}$: $L_{\mathcal{U},\mathbf{j}} = \sum_{i \in \mathcal{U}}{L_{i, j_i}}$ and $L_{\mathcal{V},\mathbf{j}} = \sum_{i \in \mathcal{V}}{L_{i, j_i}}$
7. look for $\mathbf{j}$ satisfying $\deg (L_{\mathcal{U},\mathbf{j}} + L_{\mathcal{V},\mathbf{j}}) \le m$
8. reveal the secret key $s = \sum_{i \in \mathcal{M}}{y_{i, j_i} \prod_{x \in \mathcal{M} \setminus \lbrace i \rbrace}{\frac{x}{x - i}}} \mod p$ where $\mathcal{M} \subseteq [1, n]$ contains any $m$ indices

# the analysis

TODO

# the experiment

## proof of concept

a proof of concept written in Python will be shown below:

- the *[SHA-256](https://en.wikipedia.org/wiki/SHA-2)* based *[hash function](https://en.wikipedia.org/wiki/Hash_function)* `hash` will be implemented as $H$
- the functions `sssenc` and `sssdec` represent the encryption and reconstruction functions of *[shamir's secret sharing scheme](https://en.wikipedia.org/wiki/Shamir%27s_secret_sharing)*
- the functions `modenc` and `moddec` are the encryption function $E$ and decryption function $E^{-1}$ of the *[symmetric encryption scheme](https://en.wikipedia.org/wiki/Symmetric-key_algorithm)* using a basic *[caesar cipher](https://en.wikipedia.org/wiki/Caesar_cipher)* for simplicity

with the predefined functions above, the proof of concept can be implemented:

- the function `peenc` is the encryption function of the personal entropy encryption scheme
- the function `pedec` is the reconstruction function of the personal entropy encryption scheme
- the function `pemitm` reconstructs the secret key using a *[meet in the middle attack](https://en.wikipedia.org/wiki/Meet-in-the-middle_attack)*

```python
from random import randrange
from hashlib import sha256
from sympy import prod
from sympy import mod_inverse
from sympy import poly
from sympy import abc
from itertools import product

n = 10
m = 7
p = 65537

Q = [i for i in range(n)]
O = [10 for _ in range(n)]


def hash(q: int, a: int, c: int) -> int: return int.from_bytes(sha256(f'{q}:{a}:{c}'.encode()).digest(), 'big') % p
def sssenc(W: list[int]) -> list[int]: return [sum(v * x ** i for i, v in enumerate(W)) % p for x in range(1, n+1)]
def sssdec(Y: list[int]) -> int: return sum(y*prod((j+1)*mod_inverse(j-i, p) for j, y in enumerate(Y) if y is not None and j != i) for i, y in enumerate(Y) if y is not None) % p
def modenc(y: int, k: int) -> int: return (k+y) % p
def moddec(z: int, k: int) -> int: return (z-k) % p


def peenc(s: int, A: list[int]) -> tuple[int, list[int]]:
    c = randrange(p)
    K = [hash(q, a, c) for q, a in zip(Q, A)]
    Y = sssenc([s] + [randrange(p) for _ in range(1, m)])
    return c, [modenc(y, k) for k, y in zip(K, Y)]


def pedec(c: int, Z: int, A: list[int]) -> int:
    K = [None if a is None else hash(q, a, c) for q, a in zip(Q, A)]
    Y = [None if k is None else moddec(z, k) for k, z in zip(K, Z)]
    return sssdec(Y)


def pemitm(c: int, Z: int, U: list[int], V: list[int]) -> list[int]:
    W = U+V
    KK = [[hash(Q[i], j+1, c) for j in range(O[i])] for i in range(n)]
    YY = [[moddec(Z[i], k) for k in KK[i]]for i in range(n)]
    L = [list(reversed(poly(prod((abc.x-j-1)*mod_inverse(i-j, p) for j in U+V if j != i)).all_coeffs()))[m:] for i in range(n)]
    LL = [[[YY[i][j]*v % p for v in L[i]] for j in range(O[i])] for i in range(n)]
    LU = [LL[v] for v in U]
    LV = [LL[v] for v in V]
    DU = {I: tuple(sum(LU[i][I[i]][j] for i in range(len(U))) % p for j in range(len(W)-m)) for I in product(*[range(O[i]) for i in U])}
    DV = {I: tuple(-sum(LV[i][I[i]][j] for i in range(len(U))) % p for j in range(len(W)-m)) for I in product(*[range(O[i]) for i in V])}
    S = set([v for v in DU.values()]).intersection(set([v for v in DV.values()]))
    P = [([i for i in DU if DU[i] == v], [i for i in DV if DV[i] == v]) for v in S]
    II = [Iuu+Ivv for Iu, Iv in P for Iuu, Ivv in product(Iu, Iv)]
    DX = [{W[v]: YY[W[v]][I[v]] for v in range(m)} for I in II]
    LY = [[X[v] if v in X else None for v in range(n)] for X in DX]
    return [sssdec(Y) for Y in LY]


if __name__ == '__main__':
    print(f'n={n}; m={m}; p={p}; Q={Q}; O={O};')
    a = [1, 2, 3, 4, 3, 2, 1, 1, 2, 2]
    s = 25
    c, Z = peenc(s, a)
    print(f'personal_entropy_encryption({s}, {a}) = {c} {Z}')
    s = pedec(c, Z, [1, 2, 3, 4, 3, 2, None, None, None, 2])
    print(f'personal_entropy_reconstruction({c}, {Z}) = {s}')
    S = pemitm(c, Z, [0, 1, 2, 3, 4], [5, 6, 7, 8, 9])
    print(f'using meet in the middle attack: secret is one of {S}')
```

the function `pemitm` successfully reveals the secret key in this experiment

# the conclusion

TODO
