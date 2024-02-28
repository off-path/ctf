
---

  

## Description

  

> Write Up: offpath<br>

> Créateur: Spipm<br>

> Difficulté: easy<br>

> Points: 100

> Format du flag: brck{.....}<br>


"L'un de nos robots adore lire. Nous ne comprenons pas. Pourquoi regarderiez-vous du papier alors que toutes les connaissances appropriées sont partagées par des robots sur Internet ? Dernièrement, elle a même commencé à parler en chiffres. Je pense que le pauvre robot est possédé ! " 


Pouvez-vous comprendre ce qu'elle essayait de dire ?



brck{1746200913432170593.11_1740398198542172490.3_789837700517945346.13}

---

"One of our bots loves to read. We don't get it. Why would you stare at paper when all proper knowledge is shared by bots on the internet? Lately she even started talking in ciphers. I think the poor bot is possessed!"  
  

Can you figure out what she was trying to say?

  

brck{1746200913432170593.11_1740398198542172490.3_789837700517945346.13}

![[Pasted image 20240223133036.png]]


Le thème de base du CTF est tourné autour des robots. L'énoncé parle d'un robot qui aime lire et parler avec des nombres. Le message de fin semble être le flag chiffré qu'il faut donc déchiffrer.

Le format me fait penser au début à une IP, puis je me rends compte que cela ressemble plus à un chiffrement par substitution où chaque nombre pourrait être un caractère ou un mot dans un texte. Après plusieurs heures de recherche dans le vent, je me dis que peut-être que 1746200913432170593.11 signifie que 1746200913432170593 a été chiffré en ROT11, ou en base 11, que 1740398198542172490.3 a été chiffré en ROT3, ou en base 3. Ce n'est peut-être pas ces chiffrements exactement, mais l'idée est que le délimiteur est le "_" et que le ".X" est un indice sur le chiffrement.

---

Il s'est avéré au final que les nombres :

1746200913432170593 1740398198542172490 789837700517945346

étaient en fait des ID de tweet, et que les nombres après les points étaient le mot à prendre dans le tweet. Dans le premier tweet, cela correspond au mot "code", dans le deuxième c'est "is", et le troisième est "everywhere".

Voilà, vous avez votre flag :').