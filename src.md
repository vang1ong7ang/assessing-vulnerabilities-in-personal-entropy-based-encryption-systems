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
4. if at least $m$ questions were answered correctly, the secret $s$ can be reconstructed by *[shamir's secret sharing scheme](https://en.wikipedia.org/wiki/Shamir%27s_secret_sharing)* with the correct $m$ answers by $s = \sum_{i \in \mathcal{M}}{y_i \prod_{\mathcal{M} \setminus \lbrace i \rbrace}{\frac{i}{j - i}}} \mod p$ where $\mathcal{M} \subseteq [1, n]$ contains the indices of the $m$ correct answers

## problem definition

the problem is to reconstruct the secret key $s$ by the random number $c$ and the encrpted shares $\mathbf{z}$ as well as the common knowledges denoted previously without asking user any questions

# the solution

the solution is essentially a meet in the middle attack

1. construct the hash matrix $\mathbf{K}$ where $k_{i,j} = H(\langle q_i, j, c \rangle)$
2. decrypt all possible shares $\mathbf{Y}$ where $y_{i,j} = E_{k_{i, j}}^{-1}(z_i)$
3. select $\mathcal{U}, \mathcal{V} \subset [1, n]$ where $\mathcal{U} \cap \mathcal{V} = \emptyset$ and $|\mathcal{U} \cup \mathcal{V}| \ge m$
4. construct *[lagrange basis polynomials](https://en.wikipedia.org/wiki/Lagrange_polynomial)* $L_i(x) = \prod_{j \in \mathcal{U} \cup \mathcal{V} \setminus \lbrace i \rbrace }{\frac{x - i}{i - j}}$
5. calculate the *[lagrange interpolation polynomial](https://en.wikipedia.org/wiki/Lagrange_polynomial)* matrix $\mathbf{L}$ where $L_{i, j} = y_{i,j} L_i(x)$
6. generate the set of possible combinations of *[lagrange interpolation polynomials](https://en.wikipedia.org/wiki/Lagrange_polynomial)* over $\mathcal{U}$ and $\mathcal{V}$: $\mathcal{L_U} = \lbrace \sum_{i \in \mathcal{U}}{L_{i, j}} | j \in [1, o_i] \rbrace$ and $\mathcal{L_V} = \lbrace \sum_{i \in \mathcal{V}}{L_{i, j}} | j \in [1, o_i] \rbrace$
7. look for $L_{\mathcal{U}} \in \mathcal{L_U}$ and $L_{\mathcal{V}} \in \mathcal{L_V}$ where $\deg (L_{\mathcal{U}} + L_{\mathcal{V}} \mod p) \le m$
8. reveal the secret key by $s = L_{\mathcal{U}}(0) + L_{\mathcal{V}}(0) \mod p$

# the analysis

TODO

# the experiment

TODO

# the conclusion

TODO
