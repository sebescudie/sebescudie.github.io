---
title: "Defcon DFIR 2019 CTF"
date: 2021-10-19
author: "seb"
description: "Writeup du CTF Memory Forensics Defcon 2019"
type: "post"
---

## Bonjour

Ça faisait longtemps! En cherchant un CTF orienté analyse mémoire, je suis tombé sur le [CTF DFIR du Defcon 2019](https://defcon2019.ctfd.io/). Allons-y donc pour la partie Memory Forensics! J'ai choisi cette fois-ci de masquer les réponses.

### get your volatility on

_What is the SHA1 hash of triage.mem?_

On nous demande pour commencer de calculer le `sha1` du dump mémoire. Et voilà :

```bash
┌──(kali㉿kali)-[~/Documents/volatility/2.6]
└─$ sha1sum ~/Downloads/Triage-Memory.mem          
****************************************  /home/kali/Downloads/Triage-Memory.mem
```

### pr0file

_What profile is the most appropriate for this machine? (ex: Win10x86_14393)_

Maintenant, on nous demande d'identifier le profil du dump. Il suffit de demander à `imageinfo` :

```bash
┌──(kali㉿kali)-[~/Documents/volatility/2.6]
└─$ python vol.py -f ~/Downloads/Triage-Memory.mem imageinfo
Volatility Foundation Volatility Framework 2.6
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : *******, *******, *******, *******, *******, *******
[...]
```

### hey, write this down

_What was the process ID of notepad.exe?_

Ici, on nous demande le PID de `notepad.exe`. Ce qui est tout à fait trivial grâce à `pslist` :

```bash
┌──(kali㉿kali)-[~/Documents/volatility/2.6]
└─$ python vol.py -f ~/Downloads/Triage-Memory.mem --profile=Win7SP1x64 pslist | grep "notepad"                                                                             130 ⨯
Volatility Foundation Volatility Framework 2.6
0xfffffa80054f9060 notepad.exe            ****   ****      1       60      1      0 2019-03-22 05:32:22 UTC+0000  
```

### wscript can haz children

_Name the child processes of wscript.exe._

On nous demande maintenant de trouver les process enfants de `wscript.exe`. J'ai commencé par faire un `pslist` pour trouver le PID dudit `wscript`, et ensuite re-`pslist` en faisant un `grep` sur le PID précédemment trouvé. Ça nous sort un seul process qui est bien notre flag!

```bash
┌──(kali㉿kali)-[~/Documents/volatility/2.6]
└─$ python vol.py -f ~/Downloads/Triage-Memory.mem --profile=Win7SP1x64 pslist | grep "****"   
Volatility Foundation Volatility Framework 2.6
0xfffffa8005a80060 wscript.exe            ****   ****      8      312      1      1 2019-03-22 05:35:32 UTC+0000                                 
0xfffffa8005a1d9e0 **********.exe         ****   ****      5      109      1      1 2019-03-22 05:35:33 UTC+0000 
```

### tcpip settings

_What was the IP address of the machine at the time the RAM dump was created?_

Hop, gogo `netscan` :

```bash
──(kali㉿kali)-[~/Documents/volatility/2.6]
└─$ python vol.py -f ~/Downloads/Triage-Memory.mem --profile=Win7SP1x64 netscan 
Volatility Foundation Volatility Framework 2.6
Offset(P)          Proto    Local Address                  Foreign Address      State            Pid      Owner          Created
0x13e057300        UDPv4    ***.***.***.***                *:*                                   2888     svchost.exe    2019-03-22 05:32:20 UTC+0000
0x13e05b4f0        UDPv6    ***.***.***.***                *:*                                   2888     svchost.exe    2019-03-22 05:32:20 UTC+0000
0x13e05b790        UDPv6    ***.***.***.***                *:*                                   2888     svchost.exe    2019-03-22 05:32:20 UTC+0000
[...]
```

J'ai coupé les résultats, mais une adresse locale semble sortir du lot : il s'agit de notre flag.

### intel

_Based on the answer regarding to the infected PID, can you determine what the IP of the attacker was?_

La question fait référence à la question sur le process enfant : on y avait identifié un process avec un nom pour le moins étrange. Quand on regarde les résultats du `netscan` fait plus, haut on retrouve ledit process connecté à une IP sur le port `4444` (Metasploit ?). Cet IP est notre flag.

### i <3 windows dependencies 

_What process name is VCRUNTIME140.dll associated with?_

Ok, ici on cherche à savoir quel process utilise `VCRUNTIME140.dll`. 

Pour cela, un petit tour dans l'aide de `vol` nous oriente vers `dlllist` qui sert à nous lister les dll chargées par chaque process. Le résultat étant très (très) long, on peut essayer de faire un `grep` sur `VCRUNTIME140.dll`. 

Sauf que le plugin affiche sur 3 lignes des infos de chaque process (nom, PID, etc), et seulement après les dll chargées ligne par ligne. J'ai donc ajouté les arguments `-B` et `-A` à `grep` pour sortir les `n` lignes avant et après le match. Je ne sais pas si c'est la bonne façon de faire (c'est un peu bourrin) mais après avoir tâtonné, ça fonctionne !

```bash
┌──(kali㉿kali)-[~/Documents/volatility/2.6]
└─$ python vol.py -f ~/Downloads/Triage-Memory.mem --profile=Win7SP1x64 dlllist | grep "VCRUNTIME140.dll" -B 50 -A 50
[...]
************************************************************************
Blablablabla   pid:   ****
Command line : "C:\Program Files\***\****.exe"
Service Pack 1

Base                             Size          LoadCount Path
------------------ ------------------ ------------------ ----
0x000000013f420000           0xa9d000             0xffff "C:\Program Files\***\****.exe"
[...]
```

### mal-ware-are-you

_What is the md5 hash value the potential malware on the system?_

On avait réussi à identifier le nom d'un process qui semblait malicieux dans une question précédente. J'ai cherché ledit process avec `filescan` et ai dumpé le fichier, puis calculé son MD5, mais ce n'est pas la bonne réponse.

J'ai donc fait un `cmdline` sur ce process pour voir comment il avait été lancé, mais rien de concluant non-plus (ceci dit j'ai trouvé quelque chose qui nous servira plus tard).

Je passe donc cette question pour l'instant !

### lm-get bobs hash

_What is the LM hash of bobs account?_

On nous demande de trouver le hash LM de Bob. Il se trouve que `vol` vient de base avec le plugin `hashdump` qui sert justement à ça :

```bash
┌──(kali㉿kali)-[~/Documents/volatility/2.6]
└─$ python vol.py -f ~/Downloads/Triage-Memory.mem --profile=Win7SP1x64 hashdump
Volatility Foundation Volatility Framework 2.6
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Bob:1000:******************************:******************************:::
```

### vad the impaler

_What protections does the VAD node at 0xfffffa800577ba10 have?_

J'ai déjà entendu parler des VAD mais n'ai aucune idée de ce dont il s'agit. [Cet article](https://imphash.medium.com/windows-process-internals-a-few-concepts-to-know-before-jumping-on-memory-forensics-part-4-16c47b89e826) sur Medium semble donner une bonne définition du sujet.

On nous donne donc l'adresse d'un VAD et on nous demande son niveau de protection. Essayons de faire un `grep "0xfffffa800577ba10"` sur `vadinfo` :

```bash
┌──(kali㉿kali)-[~/Documents/volatility/2.6]
└─$ python vol.py -f ~/Downloads/Triage-Memory.mem --profile=Win7SP1x64 vadinfo | grep "0xfffffa800577ba10" -A 5
Volatility Foundation Volatility Framework 2.6
VAD node @ 0xfffffa800577ba10 Start 0x0000000000030000 End 0x0000000000033fff Tag Vad 
Flags: NoChange: 1, Protection: 1
Protection: ****
Vad Type: VadNone
ControlArea @fffffa8005687a50 Segment fffff8a000c4f870
NumberOfSectionReferences:          1 NumberOfPfnReferences:           0
```

J'avais fait un premier essai sans `-A` mais n'ai bien sûr rien eu d'intéressant. Et la valeur du champ `Protection` ici est bien notre flag !

### more vads?!

_What protections did the VAD starting at 0x00000000033c0000 and ending at 0x00000000033dffff have?_

Cette fois ci, on nous demande le niveau de protection d'un VAD en nous donnant ses adresses de début et de fin. Est-ce qu'on pourrait s'en tirer de la même manière qu'au dessus ?

```bash
┌──(kali㉿kali)-[~/Documents/volatility/2.6]
└─$ python vol.py -f ~/Downloads/Triage-Memory.mem --profile=Win7SP1x64 vadinfo | grep "Start 0x00000000033c0000 End 0x00000000033dffff" -A 5        
Volatility Foundation Volatility Framework 2.6
VAD node @ 0xfffffa80052652b0 Start 0x00000000033c0000 End 0x00000000033dffff Tag VadS
Flags: CommitCharge: 32, PrivateMemory: 1, Protection: 24
Protection: ****
Vad Type: VadNone
[...]
```

On dirait bien que oui.

### vacation bible school 

_There was a VBS script run on the machine. What is the name of the script? (submit without file extension)_

Quelques questions plus haut j'avais cherché à savoir comment notre process malicieux avait été lancé, et il se trouve que `cmdline` nous montrait un fichier `.vbs`. C'est bien notre flag!

### thx microsoft

_An application was run at 2019-03-07 23:06:58 UTC, what is the name of the program? (Include extension)_

Je ne sais pas trop par où commencer là dessus. Comme ça, ça me fait penser à l'[AMCache](https://www.ssi.gouv.fr/en/publication/amcache-analysis/), mais le plugin `amcache` ne s'avère pas concluant :

```bash
┌──(kali㉿kali)-[~/Documents/volatility/2.6]
└─$ python vol.py -f ~/Downloads/Triage-Memory.mem --profile=Win7SP1x64 amcache
Volatility Foundation Volatility Framework 2.6
The requested key could not be found in the hive(s) searched
```

En fouillant sur l'internet, je suis tombé sur [cet article](https://www.andreafortuna.org/2017/10/16/amcache-and-shimcache-in-forensic-analysis/) qui nous dit que "Amcache and Shimcache can provide a timeline of which program was executed and when it was first run and last modified". Et je me souvenais avoir croisé dans l'aide de `vol` un plugin `shimcache`. Après un premier run pour voir à quoi ressemblait l'output, j'ai fait un `grep` sur la date indiquée dans l’énoncé pour arriver au flag!

```bash
┌──(kali㉿kali)-[~/Documents/volatility/2.6]
└─$ python vol.py -f ~/Downloads/Triage-Memory.mem --profile=Win7SP1x64 shimcache | grep "2019-03-07 23:06:58"
Volatility Foundation Volatility Framework 2.6
2019-03-07 23:06:58 UTC+0000   \??\C:\Program Files (x86)\***\****\*******.exe
```

### lightbulb moment

_What was written in notepad.exe in the time of the memory dump?_

Comme d'habitude, le plugin `notepad` ne fonctionne pas, je vais donc regarder du côté de `pslist` quel est le PID de `notepad.exe` et essayer de voir avec quelle `cmdline` il a été lancé, voir si on ne peut pas récupérer un ficher... et bien non. `notepad.exe` a été lancé sans aucun argument.

Peut-être dumper la zone mémoire et faire un strings dessus ? C'est trop fastidieux, il doit bien y avoir un autre moyen.

Après quelques recherches sur l'internet, je tombe sur [cet article](https://www.andreafortuna.org/2018/03/02/volatility-tips-extract-text-typed-in-a-notepad-window-from-a-windows-memory-dump/) qui nous parle justement de comment extraire du texte du bloc note depuis un dump mémoire. Il y est fait une précision très intéressante : "the `-e l` switch is needed because notepad stores text in 16-bit little-endian:". Aha! Effectivement, si on fait un `strings` sur le dump avec cette option, le contenu est bien plus digeste ! Comme je n'ai aucune idée de quoi chercher, je scroll un peu puis fait un `CTRL + F` sur "flag", et c'est le bingo, notre flag est bien là !

### 8675309

_What is the shortname of the file at file record 59045?_

Là comme ça, ça me fait penser à la MFT.. donc allons-y avec `mftparser` et `grep` : 

```bash
┌──(kali㉿kali)-[~/Documents/volatility/2.6]
└─$ python vol.py -f ~/Downloads/Triage-Memory.mem --profile=Win7SP1x64 mftparser | grep "59045" -A 30
```

Je ne donne pas les résultats de la commande ici car ils sont un peu longs, mais le flag s'y trouve bien!

### whats-a-metasploit?

_This box was exploited and is running meterpreter. What PID was infected?_

En parcourant les résultats de `netscan` plus tôt, j'ai vu passer un port qui ressemble beaucoup au port par défaut de Metasploit. Et si l'on regarde le PID associé au process connecté à ce port, on a bien notre flag !

## Conclusion

Intéressant, j'ai pu en apprendre un peu plus sur le Shimcache et les VAD ! En revanche, à la différence des CTF Memlabs, j'ai trouvé que tout était très centré sur Volatility, là où Memlabs peut demander des trucs un peu plus drôles comme désassembler un binaire ou passer un memdump dans GIMP :)

Je suis allé lire un writeup pour avoir la solution à l'histoire du hash MD5, et je n'étais en fait pas loin du but : il fallait faire un `procdump` et non pas un `dumpfiles` sur le process chelou. A ce stade je suis pas sûr de comprendre la différence entre dumper un fichier execultable avec `filescan`/`dumpfiles` et `procdump`. J'imagine que `dumpfiles` n'est tout simplement pas fait pour les executables, à la différence de `procdump`. Bon à savoir !