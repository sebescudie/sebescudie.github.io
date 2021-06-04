---
title: "MEMLABS - LAB01"
date: 2021-06-04
author: "seb"
description: "Writeup du premier lab MEMLABS"
type: "post"
---
Premier article du blog, et c'est un writeup ! On va résoudre pas à pas le premier CTF du repo [MEMLABS](https://github.com/stuxnet999/MemLabs), qui propose de petits CTF centrés sur l'analyse de dumps mémoire.

Le but ici n'est pas de faire une introduction [volatility](https://www.volatilityfoundation.org/), mais le writeup devrait être (je l'espère !) suffisament documenté/expliqué pour qu'une personne familière avec la ligne de commande et curieuse puisse se faire une idée du fonctionnement du framework.

Commençons par la description du challenge :


>My sister's computer crashed. We were very fortunate to recover this memory dump. Your job is get all her important files from the system. From what we remember, we suddenly saw a black window pop up with some thing being executed. When the crash happened, she was trying to draw something. Thats all we remember from the time of crash.

Ok, donc visiblement une fenêtre `cmd` est apparue pendant que la personne dessinait quelque chose. C'est parti ! 


# Determiner le profil du dump

D'abord, on va devoir déterminer le profil du dump que l'on cherche à analyser. Volatility fait très bien ça pour nous avec le plugin `imageinfo` : 

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw imageinfo                                                
Volatility Foundation Volatility Framework 2.6.1
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win7SP1x64, Win7SP0x64, Win2008R2SP0x64, Win2008R2SP1x64_24000, Win2008R2SP1x64_23418, Win2008R2SP1x64, Win7SP1x64_24000, Win7SP1x64_23418
                     AS Layer1 : WindowsAMD64PagedMemory (Kernel AS)
                     AS Layer2 : FileAddressSpace (/home/kali/Documents/Memory Samples/MemoryDump_Lab1.raw)
                      PAE type : No PAE
                           DTB : 0x187000L
                          KDBG : 0xf800028100a0L
          Number of Processors : 1
     Image Type (Service Pack) : 1
                KPCR for CPU 0 : 0xfffff80002811d00L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2019-12-11 14:38:00 UTC+0000
     Image local date and time : 2019-12-11 20:08:00 +0530

```

Ok, on va partir sur `Win7SP1x64`. Il faudra spécifier ce profil dans toutes les commandes que l'on va éxecuter par la suite.

# Pendant ce temps à Veracruz

Ensuite, pour avoir une idée de ce qu'il se passait sur la machine au moment du dump, on va lister les process en cours avec `pslist` :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw --profile=Win7SP1x64 pslist
Volatility Foundation Volatility Framework 2.6.1
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

| Process       | PID    |
|---------------|--------|
| `cmd.exe`     | `1984` |
| `mspaint.exe` | `2424` |
| `winrar.exe`  | `1512` |

Rien ne nous parlait de Winrar dans l'intro du CTF. Ceci dit, il y a peut-être quelque chose y trouver...

# Qué passa dans le cmd - premier flag

L'intro parlait d'un `cmd` qui s'ouvre juste avant le crash. Peut-être qu'on pourrait voir ce qu'il s'y est passé ? Si on regarde la liste des plugins de volatility (`-h`), on trouve quelque chose qui pourrait nous être utile :

```
consoles        Extract command history by scanning for _CONSOLE_INFORMATION
```

Cool, ça a l'air de faire ce qu'on cherche :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw --profile=Win7SP1x64 consoles
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

Ok, ces deux lignes ont l'air étrange, et semblent nous indiquer qu'il s'agit du premier flag !

```cmd
C:\Users\SmartNet>St4G3$1                                                       
ZmxhZ3t0aDFzXzFzX3RoM18xc3Rfc3Q0ZzMhIX0= 
```

Ca ressemble beaucoup à un truc en base64. Essayons de le décoder ?

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ echo ZmxhZ3t0aDFzXzFzX3RoM18xc3Rfc3Q0ZzMhIX0= | base64 --decode                                                                                                              1 ⨯
flag{th1s_1s_th3_1st_st4g3!!} 
```
Boom, premier flag !

# Winrar - troisième flag

Oui, j'ai trouvé le troisième avant le deuxième. On va s'intéresser au process Winrar qu'on a repéré tout à l'heure. Comment pourrait-on savoir ce qu'il s'y passait ? A nouveau, fouillons l'aide de Volatility :


```
cmdline         Display process command-line arguments
```

On a le PID de `winrar.exe`, on devrait donc pouvoir trouver avec quels args il a été démarré :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw --profile=Win7SP1x64 cmdline -p 1512
Volatility Foundation Volatility Framework 2.6.1
************************************************************************
WinRAR.exe pid:   1512
Command line : "C:\Program Files\WinRAR\WinRAR.exe" "C:\Users\Alissa Simpson\Documents\Important.rar"
```

Ok, on a donc cherché à décompresser le fichier `Important.rar`. Cet indice d'une grande finesse nous indique que quelque chose se cache certainement dans cette archive. Peut-on l'extraire de la RAM ? Une fois encore, un petit tour dans l'aide de `vol`...

```
filescan        Pool scanner for file objects
dumpfiles       Extract memory mapped and cached files
```

On dirait bien que oui ! Commençons par chercher ledit fichier

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw --profile=Win7SP1x64 filescan | grep "Important.rar"
Volatility Foundation Volatility Framework 2.6.1
0x000000003fa3ebc0      1      0 R--r-- \Device\HarddiskVolume2\Users\Alissa Simpson\Documents\Important.rar
```

Nickel, le fichier est bien présent dans notre dump. Utilisons maintenant son offset pour le dumper avec `dumpfiles` :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000003fa3ebc0 -D . 
Volatility Foundation Volatility Framework 2.6.1
DataSectionObject 0x3fa3ebc0   None   \Device\HarddiskVolume2\Users\Alissa Simpson\Documents\Important.rar
```

Ok, vérifions maintenant le contenu de notre repertoire :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ ll
total 176
-rw-r--r-- 1 kali kali 45056 Jun  4 09:29 file.None.0xfffffa8001034450.dat
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

Aha ! L'archive contient bien notre troisième flag (`flag3.png`), mais elle est protégée par mot de passe. Ceci dit, on nous donne une indication pour le trouver : il s'agit du hash NTLM du mot de passe d'Alissa. 

Re-re-re-re-regardons la liste de commandes de volatility, est-ce que quelque chose peut nous intéresser ?

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

![](https://raw.githubusercontent.com/sebescudie/sebescudie.github.io/master/static/images/blog/memlab_01/memlab_flag3.png)

# mspaint - flag 2

La dernière chose à regarder est ce process `mspaint.exe`. Comme le disait la petite histoire, la personne était en train de dessiner un truc au moment où le crash a eu lieu. On imagine donc qu'il va falloir arriver à extraire une image de Paint de notre dump mémoire.

Pour être tout à fait honnête, je n'ai rien fait d'autre que de chercher sur un moteur de recherche comment arriver à faire ça. J'ai essayé le plugin `screenshot` dont le résultat est assez drôle (on obtient un wireframe de toutes les fenêtres ouvertes!), mais ne nous aide pas à voir ce qui était dessiné dans Paint au moment du dump.

![](https://raw.githubusercontent.com/sebescudie/sebescudie.github.io/master/static/images/blog/memlab_01/memlab_screenshot_01.png)

Après avoir donc lu [cet article de blog](https://w00tsec.blogspot.com/2015/02/extracting-raw-pictures-from-memory.html), je me lance.

On commence donc par dump le bout de mémoire lié au process mspaint (souvenez-vous, on en avait identifié le PID tout début !) :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/Memory\ Samples/MemoryDump_Lab1.raw --profile=Win7SP1x64 memdump -p 2424 -D ./mspaint_memdump                   130 ⨯
Volatility Foundation Volatility Framework 2.6.1
************************************************************************
Writing mspaint.exe [  2424] to 2424.dmp

```

Ok, on continue. On renomme ce `2424.dmp` en `2424.data`, et on l'ouvre dans `GIMP` en tant que "Raw Image Data". Et après .. on joue avec les sliders. En essayant d'être attentif aux moments ou un pattern semble se repêter et en ajustant les valeurs plus finement quand ça arrive, on commence à tomber sur des choses sympa :

![](https://raw.githubusercontent.com/sebescudie/sebescudie.github.io/master/static/images/blog/memlab_01/memlab_raw_01.png)

Incroyable de se dire que tout ça traine dans la RAM !

J'ai gardé ces valeurs et ai continué à jouer sur l'offset pour à nouveau tomber sur quelque chose qui avait l'air intéressant :

![](https://raw.githubusercontent.com/sebescudie/sebescudie.github.io/master/static/images/blog/memlab_01/memlab_raw_02.png)

Et en changeant encore un peu les valeurs de width, je tombe enfin sur :

![](https://raw.githubusercontent.com/sebescudie/sebescudie.github.io/master/static/images/blog/memlab_01/memlab_raw_03.png)

Et boom ! Deuxième flag !

# Conclusion

Et bien voilà, c'était très intéressant ! Je compte bien faire d'autres CTFs du repo MEMLAB dans le futur, et le fait de les documenter ici m'oblige à être consciencieux et à prendre des notes sur ce que je fais, ce qui devrait vraiment m'aider à apprendre !

J'espère que vous aurez trouvé le sujet intéressant :) 