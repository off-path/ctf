 

---

  

## Description

  

> Write Up: offpath<br>

> Créateur: Spipm<br>

> Difficulté: easy - medium<br>

> Points: 150<br>

> Format du flag: brck{.....}<br>


"Monsieur, monsieur ! C'est un chantier de construction." Vous regardez ce que vous pensiez être un bâtiment en construction, mais vous réalisez qu’il s’agit d’un robot de construction. "Monsieur, s'il vous plaît, écartez-vous. Je devais mettre ces blocs en ordre depuis la semaine dernière, mais un robot de construction débutant les a mélangés." "Je peux m'écarter", dites-vous au robot, "mais je pourrai peut-être vous aider." 


Pouvez-vous aider le robot à mettre les blocs en ordre ?

---
  
"Sir, sir! This is a construction site." You look up at what you thought was a building being constructed, but you realize it is a construction bot. "Sir please move aside. I had to have these blocks in order since last week, but some newbie construction bot shuffled them." "I can move aside, " you tell the bot, "but I might be able to help you out."  
  

Can you help the bot get the blocks in order?

![[Pasted image 20240223133816.png]]

```
import binascii
from Crypto.Cipher import AES
from os import urandom
from string import printable
import random
from time import time

flag = "brk{...}"
key = urandom(32)

def encrypt(raw):
	cipher = AES.new(key, AES.MODE_ECB)
	return binascii.hexlify(cipher.encrypt(raw.encode()))

# Generate random bytes
random.seed(int(time()))
rand_printable = [x for x in printable]
random.shuffle(rand_printable)

# Generate ciphertext
with open('ciphertext','w') as fout:
	for x in flag:
		for y in rand_printable:
			# add random padding to block and encrypt
			fout.write(encrypt(x + (y*31)).decode())
```

On a également un fichier ciphertext contenant notre texte chiffré via le programme.


Il y a plusieurs points notables dans ce code:

- Le chiffrement est en AES.CBC avec une key venant de urandom.

- Chaque caractère du flag est chiffré avec 31 cractères de padding, padding qui vient de 
	```from string import printable``` qui contient 100 carctères affichables (alpĥabet en min et en maj, les 10 chiffres et quelque caractères spéciaux)

-  La table printable a été mélangé, mais un seed basé sur la fonction time() régule le mélange aléatoir de la table



Dans l'idée, le code fait ceci:

On prends le premier caractère du flag, on ajoute 31 autre caractères, et on chiffres le tout.
Ce qui veut dire que si le premier caractère est ```b``` et que le padding ajouté est ```u*31``` 
Le chiffrement donnerait quelque chose comme:

```20e2b5f9404ebf52719c78e2b9f86b45420e2b5f9404ebf52179c78e2b9f86b454```

Sauf que dans le code, on boucle sur chacun des éléments du tableau printable, cela veut donc dire qu'il y a forcement un moment ou le programme chiffre le carctère ```b``` avec un padding de ```b*31``` 
Le chiffrement donnerait donc quelque chose comme:

```2024ebf9b4027588ee954b7f95e2b1472024ebf9b4027588ee954b7f95e2b147```

Et on remarque qu'ici, la première moitié est la même que la seconde. 
Ce qui est normal étant donnée qu'on a chiffré une chaine avec de l'AES CBC (qui fonctionne par trnache de 16), et que notre chaine chiffré equivaut à ```32*b```

On pourrait donc faire une boucle, qui récupère tout les blocs ou la première moitié est égale à la deuxieme, et qui nous renverrait son index ```i%100``` (modulo 100 car on a 100 carctères dans la table printable, et on renvoit son index car si l'index est égale a 4, cela veut dire que pour un certains caractères du flag, celui ci etait égale au 4ème élément de la table printable soit ```d```)

Sauf que le soucis, c'est que cette table a été mélangée. Donc le 4ème caractère peut très bien être un ```u``` comme un ```e``` comme un ```_``` ou  un ```2```

On doit donc trouver le seed, ou une approximation pour qu'on puisse la bruteforce.

Si on fait exiftool du fichier ciphertext, on obtient une date convertible en timestamp unix:
```1708556400```

Dans l'ennoncé, le robot nous dit quelque chose d'interessant:
```I had to have these blocks in order since last week,```
On peut surement retirer une semaine a notre timestamp afin de gagner du temps sur notre brutforce de seed.

On a tout les ingrédients, et la recette, it's time to coook ^^

Code final:

```
import random
from string import printable

#lire le fichier contenant le ciphertext
def lire_fichier_chiffre(chemin_fichier):
	with open(chemin_fichier, 'r') as fichier:
		texte_chiffre = fichier.read()
	return texte_chiffre

#shuffle la table printable en fonction du seed
def shuffle_printable(seed):
	rand_printable = [x for x in printable]
	random.seed(seed)
	random.shuffle(rand_printable)
	return rand_printable 

chemin_fichier = 'ciphertext'
texte_chiffre = lire_fichier_chiffre(chemin_fichier)

taille_bloc = 64
blocs = [texte_chiffre[i:i+taille_bloc] for i in range(0, len(texte_chiffre), taille_bloc)]

seed_approx = 1708556400
une_semaine_en_secondes = 604800
seed_debut = seed_approx - une_semaine_en_secondes
  
while True:
	caracteres_correspondants = []
	rand_printable = shuffle_printable(seed_approx)
	
	for i, bloc in enumerate(blocs):
		moitie1, moitie2 = bloc[:len(bloc)//2], bloc[len(bloc)//2:]
		if moitie1 == moitie2:
			indice = i % len(rand_printable)
			caracteres_correspondants.append(rand_printable[indice])
	
	flag = ''.join(caracteres_correspondants)
	if flag.startswith('brck{'):
		print("Flag :", flag)
		break
	else:
		print(seed_approx)
		seed_approx -= 1
```