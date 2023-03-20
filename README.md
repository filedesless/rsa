# RSA (Rivest–Shamir–Adleman)

- Differents types de chiffrement étudiés
	* symmétrique
	* asymmétrique
- Rappels d'arithmétique
	* Algorithme d'Euclide étendu
	* Théorème des restes chinois
	* Théorème d'Euler-Fermat
- Fonctionnement de RSA
	* Génération de clé
	* Chiffrement et déchiffrement
- Limitations connues de RSA
	* common factor attack (bad key)
	* CCA on textbook RSA (oracle decrypts chosen cipher, no padding)
	* CRT optimization with single failure (computation failed without verification)
	* Håstad's broadcast attack (same m, crypted for 3 n, e = 3, no padding)
	* Bleichenbacher attack (TODO, CCA2 deterministic padding pkcs-15)

---

## Chiffrement symmétrique

- Alice et Bob doivent au préalable échanger en secret une clé
- Celle-ci permet de chiffrer et déchiffrer des messages

**Ex: Avec clé 3, le chiffre de César suivant permet de chiffrer et déchiffrer du texte facilement en effectuant un décalage**
 <center>
`A B C D E F G H I J K L M N O P Q R S T U V W X Y Z`<br>
`D E F G H I J K L M N O P Q R S T U V W X Y Z A B C`
</center>

Ainsi, le message 'WE ATTACK AT DAWN' devient 'ZH DWWDFN DW GDZQ'

Les systèmes de ce genre ont comme défaut qu'ils assument que les utilisateurs
peuvent échanger une clé en toute confidentialité

---

## Chiffrement asymmétrique

- Alice et Bob génèrent chacun une paire de clés
 * Une <green>clé publique</green>, qu'ils peuvent distribuer
 * Une <red>clé privée</red>, qu'ils gardent chacun pour eux

- Alice peut utiliser la <green>clé publique</green> de Bob pour lui chiffrer un message
 * Bob déchiffre le message avec sa <red>clé privée</red>

- Alice peut aussi utiliser sa <red>clé privée</red> pour signer un message
 * Bob vérifie la signature avec la <green>clé publique</green> de Alice

**RSA est un example de chiffrement asymmétrique**

L'avantage de ces systèmes est qu'ils ne nécéssitent pas d'échange préalable de secret

---

## Algorithme d'Euclide étendu

Soit $a, b \in \mathbb{Z}$, $\exists q, r \in \mathbb{Z}: a = bq + r$

Aussi, on a $\gcd(a, b) = \gcd(b, r)$

```python
def egcd(a, b):
	if a == 0:
		return (b, 0, 1)
	else:
		q, r = divmod(b, a) # quotient et reste de la division Euclidienne
		d, y, x = egcd(r, a)
		return (d, x - q * y, y)
```

Calcule $d = \gcd(a, b)$, ainsi que les coefficient $x, y \in \mathbb{Z}$ de l'identité de Bézout

$$xa + yb = d$$

---

## Théorème des restes chinois

Soit $p, q \in \mathbb{Z}: p \perp q$ et $n = pq$

Alors avec $a, b \in \mathbb{Z}$, le système suivant a une solution

$$
x \equiv a \pmod p \newline
x \equiv b \pmod q
$$

Et pour toutes deux solutions $x_1, x_2$ on a $x_1 \equiv x_2 \pmod n$

Ainsi il existe une unique solution inférieure à n

Autremet dit, on a l'isomorphisme d'anneaux suivant:

$$
\mathbb{Z}/n\mathbb{Z} \; \tilde{\rightarrow} \; \mathbb{Z}/p\mathbb{Z} \times \mathbb{Z}/q\mathbb{Z}
$$

---

## Théorème d'Euler-Fermat

#### Petit théorème de Fermat

Soit $a, p \in \mathbb{Z}$ tels que p est premier, et $p \nmid a$, alors
$$a^{p-1} \equiv 1 \pmod p$$

#### Théorème d'Euler (Plus général)

Soit $a, n \in \mathbb{Z}: a \perp n$, alors
$$a^{\varphi(n)} \equiv 1 \pmod n$$

Où $\varphi(n)$ est l'indicatrice d'Euler; l'ordre du groupe multiplicatif $\mathbb{Z}/n\mathbb{Z}$

En particulier, par le théorème des restes chinois, quand `n = pq`, avec p et q premiers
$$\varphi(n) = \varphi(p)\varphi(q) = (p - 1)(q - 1)$$

---

## Fonctionnement de RSA

#### Génération de clé

1. On choisi deux grands nombres premiers p et q.
2. On calcule `n = pq`
3. On choisi e tel que $e \perp \varphi(n)$
 * généralement 3 ou 65537 (petits pour accélérer le chiffrement)
4. On calcule d tel que $ed \equiv 1 \pmod {\varphi(n)}$ (avec l'algorithme d'Euclide étendu)

La clé publique est donc la paire (n, e) et la clé privée d

#### Chiffrement et déchiffrement

Soit M un entier correspondant au message à chiffrer, on calcule

$$M^e \equiv C \pmod n$$

et le détenteur de la clé privée peut retrouver le message comme suit
$$C^d \equiv M^{ed} \equiv M^{1 + k\varphi(n)} \equiv M \pmod n$$

---

## Problème de factorization première

La sécurité du cryptosystème RSA repose sur la difficulté de retrouver le message originel M,
connaissant seulement le message chiffré C, l'exposant e et le moduli n.

La manière connue la plus rapide de faire est de décomposer n en ses facteurs premiers,
afin de calculer `ɸ(n)` et l'inverse modulaire de e en `ɸ(n)`. Calculer cette décomposition
est jugé suffisament difficile pour des n suffisament grands (habituellement écrit sur 2048 bits de nos jours)

---

## Common Factor Attack

Soient $n_1 = p_1q$ et $n_2 = p_2q$

Alors on a $\gcd(n_1, n_2) = q$ et $p_1 = \frac{n_1}{q}, p_2 = \frac{n_2}{q}$

---

## Chosen Ciphertext Attack

Soit $C \equiv M^e \pmod n$

Prenons $C_a \equiv 2^e \pmod n$, et $C_b = C_aC$

$$ 
\begin{equation}
\begin{split}
C_b & \equiv 2^e M^e \pmod n \newline
    & \equiv (2M)^e
\end{split}
\end{equation}
$$

On demande a l'oracle de dechiffrer $C_b$

$$
\begin{equation}
\begin{split}
(C_b)^d & \equiv ((2M)^e)^d \pmod n \newline
      & \equiv 2M
\end{split}
\end{equation}
$$

---

  * CRT optimization with single failure (computation failed without verification)
	* Håstad's broadcast attack (same m, crypted for 3 n, e = 3, no padding)
	* Bleichenbacher attack (TODO, CCA2 deterministic padding pkcs-15)
