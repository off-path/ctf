 

---

  

## Description

  

> Write Up: offpath<br>

> Créateur: Spipm<br>

> Difficulté: easy<br>

> Points: 150<br>

> Format du flag: brck{.....}<br>

  

### Enoncé

Vous rencontrez un robot en train de méditer dans le parc. Il ouvre ses caméras et commence à parler.
  

"Écoutez la parole de RNGesus. La complexité est l'ennemi de la sécurité. Que votre cryptage soit le plus simple possible, afin de le sécuriser ainsi". Il vous remet un dépliant avec un extrait de code. "Sécurisez chaque message que vous avez avec. Seuls ceux qui voient peuvent entrer."
  

Qu'en penses-tu? Est-ce assez simple pour être sécurisé ?

---

You encounter a bot meditating in the park. He opens his cameras and begins to speak.  
  

"Hear the word of RNGesus. Complexity is the enemy of security. Let your encryption be as simple as possible, as to secure it, thusly". He hands you a flyer with a snippet of code. "Secure every message you have with it. Only those who see can enter."  
  

What do you think? Is it simple enough to be secure?

![[Pasted image 20240223133500.png]]

`nc 0.cloud.chals.io 26265`

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
	char secret[] = "brck{not_the_flag}";
	char *key = NULL;
	size_t read_length, buffer_length = 0;
	
	// Read One Time Key
	FILE *random_bytes = fopen("/dev/urandom", "r");
	read_length = getline(&key, &buffer_length, random_bytes);
	fclose(random_bytes);
	
	// Encrypt
	for (int i = 0; i < strlen(secret); i++)
		secret[i] = secret[i] ^ key[i%read_length];
		
	// Return encrypted secret
	printf("%s", secret);
	free(key);
	return 0;
	
	}
```

L'ennoncé nous parle d'un chiffrement simple, qui s'avere etre un xor dans le code. Dans l'idée, si nous retrouvons la clé, nous pouvons déchiffrer le message.

la commande nc devrait nous envoyer le message chiffré:

```
~/Documents/ctf/breaker > nc 0.cloud.chals.io 26265                                                             INT
��="�LXHc�*&G����&[#�
                     ��Uk-6ѧ�vx�e`
```


On a donc une clé qui vient de urandom, d'une longueur et d'une valeur aléatoire. On sait que le secret commence par `brck{` La clé étant d'une longueur aléatoire, il est possible que celle-ci soit plus petite que la partie que nous connaissons du secret.

Par exemple, pour une clé telle que len(key)=2, on pourrait xor les deux premiers caractères de notre secret connu ("br") avec les deux premiers caractères de notre ciphertext, ce qui nous donnerait les deux premiers nombres de la clé (et dans le cas où la longueur de la clé est de 2, on aurait donc la clé complète).

Une fois la clé complète acquise, on étend la clé sur la longueur du ciphertext, on xor et on a notre plaintext :)

code final:

```
import string

from pwn import *

def xor(text1, text2):
	return bytes([a ^ b for a, b in zip(text1, text2)])
  
known_plaintext1 = b"br"
known_plaintext2 = b"ck"
while True:
  
	#cette partie s'occupe de se connecter au service qui nous envoi le ciphertext

	conn = remote("0.cloud.chals.io", 26265)
	ciphertext = conn.recvS()
	conn.close()

	if isinstance(ciphertext, str):
		ciphertext = ciphertext.encode()
	
	#Dans le cas ou len(key)=1
	key1 = xor(known_plaintext1, ciphertext[0:1])
	key2 = xor(known_plaintext2, ciphertext[1:2])
	key3 = xor(known_plaintext1, ciphertext[2:3])
	key4 = xor(known_plaintext2, ciphertext[3:4])
	key5 = xor(known_plaintext1, ciphertext[4:5])
	
	if key1 == key2 == key3 == key4 == key5:
	
		print("Clé trouvée:", key1)
		# Utiliser la clé trouvée pour déchiffrer le message complet
		full_key = key1 * (len(ciphertext) // len(key1)) + key1[:len(ciphertext) % len(key1)]
		decrypted_message = xor(ciphertext, full_key)
		print("Flag:", decrypted_message)
		break
	
	#Dans le cas ou len(key)=2
	key1 = xor(known_plaintext1, ciphertext[0:2])
	key2 = xor(known_plaintext2, ciphertext[2:4])
	
	if key1 == key2:
		print("Clé trouvée:", key1)
		full_key = key1 * (len(ciphertext) // len(key1)) + key1[:len(ciphertext) % len(key1)]
		decrypted_message = xor(ciphertext, full_key)
		print("Flag:", decrypted_message)
		break
		
	else:
		print('nop')
		continue
```



