---
title: "MEMLABS - LAB01"
date: 2021-06-04
author: "seb"
description: "Writeup du premier lab MEMLABS"
type: "post"
---

# Determiner le profil du dump

D'abord, on va devoir déterminer le profil du dump que l'on cherche à analyser

```bash
python volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw imageinfo
```

Réponse : `Suggested Profile(s) : Win7SP1x64, Win7SP0x64, Win2008R2SP0x64, Win2008R2SP1x64_24000, Win2008R2SP1x64_23418, Win200`

Ok, on va partir sur `Win7SP1x64`. Il faudra spécifier ce profil dans toutes les commandes que l'on va éxecuter par la suite.

# Pendant ce temps à Veracruz

Ensuite, pour avoir une idée de ce qu'il se passait sur la machine au moment du dump, on va lister les process en cours avec `pslist` :

```bash
python volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw --profile=Win7SP1x64 pslist
```

Et la réponse :

```
Offset(V)          Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit                          
------------------ -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0xfffffa8000ca0040 System                    4      0     80      570 ------      0 2019-12-11 13:41:25 UTC+0000                                 
0xfffffa800148f040 smss.exe                248      4      3       37 ------      0 2019-12-11 13:41:25 UTC+0000                                 
0xfffffa800154f740 csrss.exe               320    312      9      457      0      0 2019-12-11 13:41:32 UTC+0000                                 
0xfffffa8000ca81e0 csrss.exe               368    360      7      199      1      0 2019-12-11 13:41:33 UTC+0000                                 
0xfffffa8001c45060 psxss.exe               376    248     18      786      0      0 2019-12-11 13:41:33 UTC+0000                                 
0xfffffa8001c5f060 winlogon.exe            416    360      4      118      1      0 2019-12-11 13:41:34 UTC+0000                                 
0xfffffa8001c5f630 wininit.exe             424    312      3       75      0      0 2019-12-11 13:41:34 UTC+0000                                 
0xfffffa8001c98530 services.exe            484    424     13      219      0      0 2019-12-11 13:41:35 UTC+0000                                 
0xfffffa8001ca0580 lsass.exe               492    424      9      764      0      0 2019-12-11 13:41:35 UTC+0000                                 
0xfffffa8001ca4b30 lsm.exe                 500    424     11      185      0      0 2019-12-11 13:41:35 UTC+0000                                 
0xfffffa8001cf4b30 svchost.exe             588    484     11      358      0      0 2019-12-11 13:41:39 UTC+0000                                 
0xfffffa8001d327c0 VBoxService.ex          652    484     13      137      0      0 2019-12-11 13:41:40 UTC+0000                                 
0xfffffa8001d49b30 svchost.exe             720    484      8      279      0      0 2019-12-11 13:41:41 UTC+0000                                 
0xfffffa8001d8c420 svchost.exe             816    484     23      569      0      0 2019-12-11 13:41:42 UTC+0000                                 
0xfffffa8001da5b30 svchost.exe             852    484     28      542      0      0 2019-12-11 13:41:43 UTC+0000                                 
0xfffffa8001da96c0 svchost.exe             876    484     32      941      0      0 2019-12-11 13:41:43 UTC+0000                                 
0xfffffa8001e1bb30 svchost.exe             472    484     19      476      0      0 2019-12-11 13:41:47 UTC+0000                                 
0xfffffa8001e50b30 svchost.exe            1044    484     14      366      0      0 2019-12-11 13:41:48 UTC+0000                                 
0xfffffa8001eba230 spoolsv.exe            1208    484     13      282      0      0 2019-12-11 13:41:51 UTC+0000                                 
0xfffffa8001eda060 svchost.exe            1248    484     19      313      0      0 2019-12-11 13:41:52 UTC+0000                                 
0xfffffa8001f58890 svchost.exe            1372    484     22      295      0      0 2019-12-11 13:41:54 UTC+0000                                 
0xfffffa8001f91b30 TCPSVCS.EXE            1416    484      4       97      0      0 2019-12-11 13:41:55 UTC+0000                                 
0xfffffa8000d3c400 sppsvc.exe             1508    484      4      141      0      0 2019-12-11 14:16:06 UTC+0000                                 
0xfffffa8001c38580 svchost.exe             948    484     13      322      0      0 2019-12-11 14:16:07 UTC+0000                                 
0xfffffa8002170630 wmpnetwk.exe           1856    484     16      451      0      0 2019-12-11 14:16:08 UTC+0000                                 
0xfffffa8001d376f0 SearchIndexer.          480    484     14      701      0      0 2019-12-11 14:16:09 UTC+0000                                 
0xfffffa8001eb47f0 taskhost.exe            296    484      8      151      1      0 2019-12-11 14:32:24 UTC+0000                                 
0xfffffa8001dfa910 dwm.exe                1988    852      5       72      1      0 2019-12-11 14:32:25 UTC+0000                                 
0xfffffa8002046960 explorer.exe            604   2016     33      927      1      0 2019-12-11 14:32:25 UTC+0000                                 
0xfffffa80021c75d0 VBoxTray.exe           1844    604     11      140      1      0 2019-12-11 14:32:35 UTC+0000                                 
0xfffffa80021da060 audiodg.exe            2064    816      6      131      0      0 2019-12-11 14:32:37 UTC+0000                                 
0xfffffa80022199e0 svchost.exe            2368    484      9      365      0      0 2019-12-11 14:32:51 UTC+0000                                 
0xfffffa8002222780 cmd.exe                1984    604      1       21      1      0 2019-12-11 14:34:54 UTC+0000                                 
0xfffffa8002227140 conhost.exe            2692    368      2       50      1      0 2019-12-11 14:34:54 UTC+0000                                 
0xfffffa80022bab30 mspaint.exe            2424    604      6      128      1      0 2019-12-11 14:35:14 UTC+0000                                 
0xfffffa8000eac770 svchost.exe            2660    484      6      100      0      0 2019-12-11 14:35:14 UTC+0000                                 
0xfffffa8001e68060 csrss.exe              2760   2680      7      172      2      0 2019-12-11 14:37:05 UTC+0000                                 
0xfffffa8000ecbb30 winlogon.exe           2808   2680      4      119      2      0 2019-12-11 14:37:05 UTC+0000                                 
0xfffffa8000f3aab0 taskhost.exe           2908    484      9      158      2      0 2019-12-11 14:37:13 UTC+0000                                 
0xfffffa8000f4db30 dwm.exe                3004    852      5       72      2      0 2019-12-11 14:37:14 UTC+0000                                 
0xfffffa8000f4c670 explorer.exe           2504   3000     34      825      2      0 2019-12-11 14:37:14 UTC+0000                                 
0xfffffa8000f9a4e0 VBoxTray.exe           2304   2504     14      144      2      0 2019-12-11 14:37:14 UTC+0000                                 
0xfffffa8000fff630 SearchProtocol         2524    480      7      226      2      0 2019-12-11 14:37:21 UTC+0000                                 
0xfffffa8000ecea60 SearchFilterHo         1720    480      5       90      0      0 2019-12-11 14:37:21 UTC+0000                                 
0xfffffa8001010b30 WinRAR.exe             1512   2504      6      207      2      0 2019-12-11 14:37:23 UTC+0000                                 
0xfffffa8001020b30 SearchProtocol         2868    480      8      279      0      0 2019-12-11 14:37:23 UTC+0000                                 
0xfffffa8001048060 DumpIt.exe              796    604      2       45      1      1 2019-12-11 14:37:54 UTC+0000                                 
0xfffffa800104a780 conhost.exe            2260    368      2       50      1      0 2019-12-11 14:37:54 UTC+0000 
```

Cool. Plusieurs process peuvent attirer notre attention. Mettons-les de côté avec leur PID pour s'en reservir plus tard :

- `cmd.exe`		1984
- `mspaint.exe`	2424
- `WinRar.exe`	1512

# Qué passa dans le cmd - premier flag

On voit qu'un `cmd.exe` était ouvert, peut-être que quelque chose a été tapé dans la console ? Si on regarde la liste des commandes de volatility (`-h`), on trouve un petit plugin qui pourrait nous être utile :

```
consoles        Extract command history by scanning for _CONSOLE_INFORMATION
```

Super, ça devrait nous donner ce que l'on cherche. Un coup de `python volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw --profile=Win7SP1x64 consoles` plus tard :

```
Volatility Foundation Volatility Framework 2.6.1
**************************************************
ConsoleProcess: conhost.exe Pid: 2692
Console: 0xff756200 CommandHistorySize: 50
HistoryBufferCount: 1 HistoryBufferMax: 4
OriginalTitle: %SystemRoot%\system32\cmd.exe
Title: C:\Windows\system32\cmd.exe - St4G3$1
AttachedProcess: cmd.exe Pid: 1984 Handle: 0x60
----
CommandHistory: 0x1fe9c0 Application: cmd.exe Flags: Allocated, Reset
CommandCount: 1 LastAdded: 0 LastDisplayed: 0
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x60
Cmd #0 at 0x1de3c0: St4G3$1
----
Screen 0x1e0f70 X:80 Y:300
Dump:
Microsoft Windows [Version 6.1.7601]                                            
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.                 
                                                                                
C:\Users\SmartNet>St4G3$1                                                       
ZmxhZ3t0aDFzXzFzX3RoM18xc3Rfc3Q0ZzMhIX0=                                        
Press any key to continue . . .                                                 
**************************************************
ConsoleProcess: conhost.exe Pid: 2260
Console: 0xff756200 CommandHistorySize: 50
HistoryBufferCount: 1 HistoryBufferMax: 4
OriginalTitle: C:\Users\SmartNet\Downloads\DumpIt\DumpIt.exe
Title: C:\Users\SmartNet\Downloads\DumpIt\DumpIt.exe
AttachedProcess: DumpIt.exe Pid: 796 Handle: 0x60
----
CommandHistory: 0x38ea90 Application: DumpIt.exe Flags: Allocated
CommandCount: 0 LastAdded: -1 LastDisplayed: -1
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x60
----
Screen 0x371050 X:80 Y:300
Dump:
  DumpIt - v1.3.2.20110401 - One click memory memory dumper                     
  Copyright (c) 2007 - 2011, Matthieu Suiche <http://www.msuiche.net>           
  Copyright (c) 2010 - 2011, MoonSols <http://www.moonsols.com>                 
                                                                                
                                                                                
    Address space size:        1073676288 bytes (   1023 Mb)                    
    Free space size:          24185389056 bytes (  23064 Mb)                    
                                                                                
    * Destination = \??\C:\Users\SmartNet\Downloads\DumpIt\SMARTNET-PC-20191211-
143755.raw                                                                      
                                                                                
    --> Are you sure you want to continue? [y/n] y                              
    + Processing...                                                         
```

Ok, on peut isoler ça :

```
C:\Users\SmartNet>St4G3$1                                                       
ZmxhZ3t0aDFzXzFzX3RoM18xc3Rfc3Q0ZzMhIX0= 
```

Ca ressemble beaucoup à un truc en base64. Essayons de le décoder ?

```bash
┌──(kali㉿kali)-[~]
└─$ echo ZmxhZ3t0aDFzXzFzX3RoM18xc3Rfc3Q0ZzMhIX0= | base64 --decode                                                                                                              1 ⨯
flag{th1s_1s_th3_1st_st4g3!!} 
```
Boom, premier flag!

# Winrar - troisième flag

Oui, j'ai trouvé le troisième avant le deuxième. On va s'intéresser au process Winrar qu'on a repéré tout à l'heure. Comment pourrait-on savoir ce qu'il s'y passait ? A nouveau, fouillons l'aide de Volatility :


```
cmdline         Display process command-line arguments
```

On a le PID de `Winrar.exe`, on devrait donc pouvoir trouver avec quels args il a été démarré :

```bash
┌──(kali㉿kali)-[~]
└─$ python volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw --profile=Win7SP1x64 cmdline -p 1512
Volatility Foundation Volatility Framework 2.6.1
************************************************************************
WinRAR.exe pid:   1512
Command line : "C:\Program Files\WinRAR\WinRAR.exe" "C:\Users\Alissa Simpson\Documents\Important.rar"
```

Ok, on a donc cherché à décompresser le fichier `Important.rar`. Cet indice d'une grande finesse nous indique que quelque chose se cache certainement dans cette archive. Peut-on l'extraire de la RAM ?

```bash
filescan        Pool scanner for file objects
dumpfiles       Extract memory mapped and cached files
```

On dirait bien que oui ! Commençons par chercher ledit fichier

```bash
┌──(kali㉿kali)-[~]
└─$ python volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw --profile=Win7SP1x64 filescan | grep "Important.rar"                            1 ⨯
Volatility Foundation Volatility Framework 2.6.1
0x000000003fa3ebc0      1      0 R--r-- \Device\HarddiskVolume2\Users\Alissa Simpson\Documents\Important.rar
0x000000003fac3bc0      1      0 R--r-- \Device\HarddiskVolume2\Users\Alissa Simpson\Documents\Important.rar
0x000000003fb48bc0      1      0 R--r-- \Device\HarddiskVolume2\Users\Alissa Simpson\Documents\Important.rar
```

Utilisons maintenant l'offset du fichier pour le dumper avec `dumpfiles` :

```bash
┌──(kali㉿kali)-[~]
└─$ python volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000003fa3ebc0 -D ~/Documents/CTF/MEMLABS/
```

Ok, maintenant si je `cd` vers l'endroit où j'ai dumpé le fichier, j'ai bien quelque chose :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ ll
total 48
-rw-r--r-- 1 kali kali 45056 Jun  4 06:34 file.None.0xfffffa8001034450.dat
-rw-r--r-- 1 kali kali     0 Jun  4 05:51 lab_01.md
````

Renommons ce `file.None.0xfffffa8001034450.dat` en `Important.rar` et essayons de le dé-compresser (j'ai viré les trucs inutiles dans le résultat) :

```bash 
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ unrar x Important.rar 
UNRAR 6.00 freeware      Copyright (c) 1993-2020 Alexander Roshal
Extracting from Important.rar
Password is NTLM hash(in uppercase) of Alissa's account passwd.
Enter password (will not be echoed) for flag3.png: 
```

Aha ! L'archive contient bien notre troisième flag (`flag3.png`), et elle est protégée par mot de passe. Ceci dit, on nous donne une indication sur le mot de passe : il s'agit du hash NTLM du mot de passe d'Alissa. 

Re-re-re-re-regardons la liste de commandes de volatility, est-ce que quelque chose nous intéresse ?

```
hashdump        Dumps passwords hashes (LM/NTLM) from memory
```

Un truc qui dump les hashes NTLM de la mémoire, parfait !

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw --profile=Win7SP1x64 hashdump                                       
Volatility Foundation Volatility Framework 2.6.1
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SmartNet:1001:aad3b435b51404eeaad3b435b51404ee:4943abb39473a6f32c11301f4987e7e0:::
HomeGroupUser$:1002:aad3b435b51404eeaad3b435b51404ee:f0fc3d257814e08fea06e63c5762ebd5:::
Alissa Simpson:1003:aad3b435b51404eeaad3b435b51404ee:f4ff64c8baac57d22f22edc681055ba6:::

```

Un peu de lecture [ici](https://security.stackexchange.com/questions/161889/understanding-windows-local-password-hashes-ntlm) et [là](https://yougottahackthat.com/blog/339/what-is-aad3b435b51404eeaad3b435b51404ee) pour comprendre le format de ces hashes. La partie qui nous intéresse est donc `f4ff64c8baac57d22f22edc681055ba6`.

Pour le style, j'ai cherché comment passer une string en uppercase dans le terminal (normalement je cherche `string to uppercase` dans un moteur de recherche 😎) :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ echo f4ff64c8baac57d22f22edc681055ba6 | tr [a-z] [A-Z]                                                                                                                     127 ⨯
F4FF64C8BAAC57D22F22EDC681055BA6
```

Ok, maintenant essayons de décompresser notre archive avec ce mot de passe. Bingo ! On extrait bien l'image `flag3.png`, qui nous donne notre flag.

# mspaint - flag 2

Bon, la dernière chose à regarder est ce process `mspaint.exe`. Comme le disait la petite histoire, la personne était en train de dessiner un truc au moment où le crash a eu lieu. On imagine donc qu'il va falloir arriver à extraire une image de Paint de notre dump mémoire.

Pour être tout à fait honnête, je n'ai rien fait d'autre que de chercher sur un moteur de recherche comment arriver à faire ça. J'ai essayé le plugin `screenshot` dont le résultat est assez drôle (on obtient un wireframe de toutes les fenêtres ouvertes!), mais ne nous aide pas à voir ce qui était dessiné dans Paint au moment du dump.

Après avoir donc lu [cet article de blog](https://w00tsec.blogspot.com/2015/02/extracting-raw-pictures-from-memory.html), je me lance.

On commence donc par dump le bout de mémoire lié au process mspaint (souvenez-vous, on en avait identifié le PID tout début !) :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw --profile=Win7SP1x64 memdump -p 2424 -D ./mspaint_memdump                   130 ⨯
Volatility Foundation Volatility Framework 2.6.1
************************************************************************
Writing mspaint.exe [  2424] to 2424.dmp

```

Ok, on continue. On renomme ce `2424.dmp` en `2424.data`, et on l'ouvre dans `GIMP` en tant que "Raw Image Data". Et après .. on joue avec les sliders. En essayant d'être attentif aux moments ou un pattern semble se repêter en ajustant les valeurs plus finement quand ça arrive, on commence à tomber sur des trucs. 

J'ai gardé ces valeurs et ai continué à jouer sur l'offset pour à nouveau tomber sur quelque chose qui avait l'air intéressant :

Et en changeant encore un peu les valeurs de width, je tombe enfin sur :

Et boom ! Deuxième flag !