# the introduction

conventional *[encryption](https://en.wikipedia.org/wiki/Encryption)* technology often relies on users to protect a secret key by selecting a *[password](https://en.wikipedia.org/wiki/Password)* or *[passphrase](https://en.wikipedia.org/wiki/Passphrase)*

while a strong *[passphrase](https://en.wikipedia.org/wiki/Passphrase)* is known only to the user, it also has the critical drawback that it must be remembered exactly to recover the secret key

over time, the user's ability to recall the passphrase may diminish, leading to the potential loss of access to the secret key

in their article, *["Protecting Secret Keys with Personal Entropy," Ellison et al. (2000)](https://www.researchgate.net/publication/223831757_Protecting_secret_keys_with_personal_entropy)* propose a scheme where users can protect a secret key by leveraging personal entropy, encrypting the passphrase using answers to several personal questions

for instance, the authors suggest using 22 questions, each with an *[entropy](https://en.wikipedia.org/wiki/Entropy_(information_theory))* of about 8 bits, to secure a 112 bits secret key and any 14 correct answers would be sufficient to recover the secret key

the authors posit that the most effective attack against this secret sharing scheme would be a *[brute force](https://en.wikipedia.org/wiki/Brute-force_attack)* approach targeting the minimum *[entropy](https://en.wikipedia.org/wiki/Entropy_(information_theory))* set of questions to reach the threshold

however, this scheme may not be as robust as initially described and can be compromised by a *[meet in the middle attack](https://en.wikipedia.org/wiki/Meet-in-the-middle_attack)*

this article provides a detailed security analysis of the *[personal entropy encryption scheme](https://www.researchgate.net/publication/223831757_Protecting_secret_keys_with_personal_entropy)*
