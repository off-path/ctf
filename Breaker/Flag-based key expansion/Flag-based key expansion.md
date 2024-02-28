

---


## Description

  

> Write Up: offpath<br>

> Créateur: Spipm<br>

> Difficulté: medium - hard<br>

> Points: 150<br>

> Format du flag: brck{.....}<br>


À travers les arbres, vous apercevez un robot chantant qui le berce sur son clavier. "Un jour!" il chante : "Un jour, nous élargirons toutes nos clés, personne n'a besoin de voir, sauf toi et moi. Ce pixel peut-il être nous ? Peut-il être craqué ? Eh bien, si nous le piratons, nous le piraterons ensemble" . Le robot arrête soudainement de chanter, émettant un son qui ne peut être décrit que comme un signal de robot. "Qu'est-ce qui ne va pas?" tu demandes. "J'ai haché les quatre derniers mots de mes paroles pour les garder en sécurité", explique le robot. "Mais j'ai accidentellement pixelisé l'image et maintenant je ne peux plus les lire !" 


Pouvez-vous aider le robot à récupérer ses paroles ?

indice : les mots du drapeau sont garantis d'être dans la liste de mots rockyou.


---

Through the trees you see a singing robot, rocking it on its keyboard. "Some day!" it sings, "Soome day, we'll expand all our keys, no one needs to see, except you and me. Can this pixel be us? Can it be cracked? Well if we hack it, we'll hack it togetheeer". The robot suddenly stops singing, making a sound that can only be described as a robot sighing. "What's wrong?" you ask. "I hashed the last four words of my lyrics to keep them safe, " the robot explains. "But I accidentally pixelized the image and now I can't read them!"  
  

Can you help the robot recover its lyrics?

hint: Flag words are guaranteed to be in the rockyou wordlist.

![[Pasted image 20240223161643.png]]



Le hint nous dis clairement qu'il faut 'juste' brutforce les mots dans la liste rockyou 4 par 4 jusqu'a tomber sur le même hash qui nous est fourni dans l'image avec le zip

![[Pasted image 20240223204539.png]]

après mettre rendu commte 1h plus tard que j'avais un segfault silencieux dans mon code, puis 1h plu tard que j'ai mal recopier le hash de sortit -_- 

Je pense que mon code est fonctionnel, mais le soucis est que cela prends beaucoup trop de temps, je trouve ca bizarre qu'un challenge de niveau hard soit solvable juste en optimisant un peu un brutforce, je pense qu'il doit y avoir autre chose ...


---


Et en effet il y avait bien quelque points a voir en plus.

- Déja, brutforce 4 hashs en même temps, c'est trop long, même en optimisant.
- De plus, les mots ont d'abord été hashé en md5, ce que j'ai completement zappé.
- Et enfin il y a indice que je n'ai pas dutout vu, mais sur le screen, on a les hashs flouté.
  

Avec Depix ou Unredacter, on peut obtenir récupérer un texte brut à partir de captures d'écran pixelisées. 

Grace a ces tools, on obtient ceci:

```
"e7__35ef8458e44________2_e6068ce",
"_4_9_a1f_9_____98eb2___94____397",
"_41___8__46_97c6_2____7__9__251_",
"_b6ded3_5b___3_2_5_____f504__dc_"
```
Les ```_``` correspondent a des partie manquante de notre hash, il faut donc les retrouver.

Une tehcnique pourrait être de hasher en md5 la liste rockyou, et de comparé chaque carctère de chaque hash avec nos 4 hashs.
Si le caractère de notre hash est un ```_``` , on passe au caractère suivant, si tout les caractères à défaults des ```_``` sont les mêmes, nous avons possiblement notre hash.

Code final

```
from hashlib import md5

partial_hashes = [
	"e7__35ef8458e44________2_e6068ce",
	"_4_9_a1f_9_____98eb2___94____397",
	"_41___8__46_97c6_2____7__9__251_",
	"_b6ded3_5b___3_2_5_____f504__dc_"
]

init_key = "aabacadaeafa0a1a2a3a4a5a6a7a8a9bbcbdbebfb0b1b2b3b4b5b6b7b8b9ccdcecfc0c1c2c3c4c5c6c7c8c9ddedfd0d1d2d3d4d5d6d7d8d9eefe0e1e2e3e4e5e6e7e8e9ff0f1f2f3f4f5f6f7f8f90010203040506070809112131415161718192232425262728293343536373839445464748495565758596676869778798899"

words = open('rockyou.txt','rb').read().split(b'\n')

def compare_parthash(full_hash,partial_hash):
	for i,v in enumerate(partial_hash):
		if v == '_':
			continue
		if full_hash[i] != v:
			return False
	return True

found_words = []
for partial_hash in partial_hashes:
	for word in words:

		if word == b'':
			continue

		m = md5()
		m.update(init_key.encode())

		for w in found_words:
			m.update(w)

		m.update(word)

		if compare_parthash(m.hexdigest(),partial_hash):
			found_words.append(word)
			print("found word for %s: %s" % (partial_hash, word.decode()))
			break

print("brck{%s}" % "_".join([x.decode() for x in found_words]))
```