# RSA (Rivest–Shamir–Adleman)

- Rappels d'arithmétique
	* Algorithme d'Euclide étendu
	* Théorème des restes chinois
	* Théorème d'Euler-Fermat
- Cryptographie
	* Symmétrique
	* Asymmétrique
	* RSA
- Cryptanalyse
	* Clés à facteurs communs
  * Facteurs rapprochés
	* Chosen ciphertext attack
	* Håstad's broadcast attack

---

## Algorithme d'Euclide étendu

Soit $a, b \in \mathbb{Z}$, on a $\exists q, r \in \mathbb{Z}: a = qb + r$ avec $r < b$

De plus, sachant que $\gcd(a, b) = \gcd(b, r)$, on peut construire la procédure suivante

```python
def egcd(a, b):
	if a == 0:
		return (b, 0, 1)
	else:
		q, r = divmod(b, a) # quotient et reste de la division Euclidienne
		d, y, x = egcd(r, a)
		return (d, x - q * y, y)
```

On calcule $d, x, y = egcd(a, b)$

Où d est le pgcd(a, b), et $x, y \in \mathbb{Z}$ les coefficients de l'identité de Bézout

$$xa + yb = d$$

---

## Théorème des restes chinois

Soit $p, q \in \mathbb{Z}$ tels que $p \perp q$ et notons $n = pq$

Alors avec $a, b \in \mathbb{Z}$, on a $\exists! c \in \mathbb{Z}_n$ tel que

$$
c \equiv a \pmod p \newline
c \equiv b \pmod q
$$

À l'aide de l'algorithme d'Euclide, on trouve $x, y \in \mathbb{Z}$ les coefficients de Bézout tels que 

$$xp + yq = 1  \tag{car $p \perp q$}$$

Puis on construit la solution c comme suit

$$c \equiv ayq + bxp \pmod n$$

Autrement formulé, on a l'isomorphisme d'anneaux suivant

$$\mathbb{Z}_n \; \tilde{\rightarrow} \; \mathbb{Z}_p \times \mathbb{Z}_q$$

---

## Théorème d'Euler-Fermat

#### Petit théorème de Fermat

Soit $a, p \in \mathbb{Z}$ tels que p est premier, et $a \perp p$, alors

$$a^{p-1} \equiv 1 \pmod p$$

<small>
> "... de quoi je vous envoierois la démonstration, si je n'appréhendois d'être trop long."
</small>

#### Théorème d'Euler (Plus général)

Soit $a, n \in \mathbb{Z}: a \perp n$, alors
$$a^{\varphi(n)} \equiv 1 \pmod n$$

Où $\varphi(n)$ est l'indicatrice d'Euler; l'ordre du groupe $\mathbb{Z}_n^\times$

En particulier, par le théorème des restes chinois, quand `n = pq`, avec p et q premiers
$$\varphi(n) = \varphi(p)\varphi(q) = (p - 1)(q - 1)$$

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

## Fonctionnement de RSA

#### Génération de clé

1. On choisi deux grands nombres premiers p et q.
2. On calcule `n = pq`
3. On choisi e tel que $e \perp \varphi(n)$
 * généralement 3 ou 65537 (petits pour accélérer le chiffrement)
4. On calcule d tel que $ed \equiv 1 \pmod {\varphi(n)}$ (avec l'algorithme d'Euclide étendu)

La <green>clé publique</green> est donc la paire <green>(n, e)</green> et la <red>clé privée</red> est simplement <red>d</red>

#### Chiffrement et déchiffrement

Soit M un entier correspondant au message clair, on calcule le cryptogramme $C_M$

$$C_M \equiv M^e \pmod n$$

et le détenteur de la <red>clé privée</red> peut retrouver le message M comme suit
$$(C_M)^d \equiv M^{ed} \equiv M^{1 + k\varphi(n)} \equiv M \pmod n$$

---

## Facteurs communs

Prenons ici les deux clé publiques $n_1 = 391, n_2 = 667$

Par l'algorithme d'Euclide

On a 

667 = 1 * 391 + 276

391 = 1 * 276 + 115

276 = 1 * 115 + 161

161 = 1 * 115 + 46

115 = 2 * 46 + 23

46 = 2 * 23 + 0

Donc $q = 23 = pgcd(391, 667)$

Et $p_1 = 391 / 23 = 17, p_2 = 667 / 23 = 29$

---

## Facteurs communs

Soient $p_1, p_2, q \in \mathbb{Z}$ premiers, notons

$$n_1 = p_1q \qquad n_2 = p_2q$$

On peut trouver le facteur commun a l'aide de l'algorithme d'Euclide

$$q = \gcd(n_1, n_2)$$

Et retrouver les facteurs manquant de $n_1$ et $n_2$

$$p_1 = \frac{n_1}{q} \qquad p_2 = \frac{n_2}{q}$$

---

## Factorisation de Fermat

Soit n = 5959, on cherche $a, b \in \mathbb{N}$, tels que $n = a^2 - b^2$

Pour $i = 0, 1, ...$ On calcule $a = \lceil\sqrt{n}\rceil + i,\; b^2 = a^2 - n, b = \sqrt{b^2}$

Pour $i = 0$, on a $a = 78,\; b^2 = 125,\; b = 11.18$

Pour $i = 1$, on a $a = 79,\; b^2 = 282,\; b = 16.79$

Pour $i = 2$, on a $a = 80,\; b^2 = 441,\; b = 21$

On a donc 

$$ n = a^2 - b^2 = (a - b) \times (a + b) = (80 - 21) \times (80 + 21) = 59 \times 101 $$

---

## Factorisation de Fermat

La méthode consiste a chercher les $a, b \in \mathbb{N}$ tels que $n = a^2 - b^2$

Ces a, b existent, car si $n = cd$, on constate $n = \(\frac{c+d}{2}\)^2 - \(\frac{c-d}{2}\)^2 = ... = \frac{4cd}{4}$

On a $n = a^2 - b^2 = (a - b) \times (a + b)$

Donc $a \pm b$ sont des facteurs de n


```python
from math import sqrt, isqrt, ceil


def fermat_facto(n):
    a = ceil(sqrt(n))
    b2 = a*a - n
    while isqrt(b2)**2 != b2: # while b not a square
        a += 1
        b2 = a*a - n
    b = isqrt(b2)
    return (a - b, a + b)  # we have n = a*a - b*b
```

---

## Chosen Ciphertext Attack

---

## Chosen Ciphertext Attack

Soit $C_M \equiv M^e \pmod n$, M étant un message secret.

Supposons que l'on peut demander au détenteur de la clé privée de déchiffrer n'importe quel message autre que $C_M$

Prenons $a \in \mathbb{Z}_n^\times$ et $C_a \equiv a^e \pmod n$

On peut calculer $$C_aC_M \equiv a^e M^e \equiv (aM)^e \pmod n$$

On demande a l'oracle de dechiffrer $C_aC_M$

$$(C_aC_M)^d \equiv ((aM)^e)^d \equiv aM \pmod n$$

Ainsi on peut retrouver $M \equiv a^{-1}(C_aC_M)^d$

---

## Håstad's broadcast attack

---

## Håstad's broadcast attack

Soit un message M, chiffré pour 3 clés $n_1, n_2, n_3$ avec $e = 3$

$$
C_1 \equiv M^3 \pmod {n_1} \newline
C_2 \equiv M^3 \pmod {n_2} \newline
C_3 \equiv M^3 \pmod {n_3}
$$

Par le théorème des restes chinois, on peut calculer $C \equiv M^3 \pmod {n_1n_2n_3}$

Comme $M < n_i$, on a $M^3 < n_1n_2n_3$

Ainsi on peut retrouver $M = \sqrt[3]{C}$
