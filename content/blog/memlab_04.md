---
title: "MEMLABS - LAB04"
date: 2021-06-23
author: "seb"
description: "Writeup du quatrième lab MEMLABS"
type: "post"
---

Encore un writeup, encore MEMLABS, pour le quatrième challenge cette fois-ci. Commençons par l'intro :

> My system was recently compromised. The Hacker stole a lot of information but he also deleted a very important file of mine. I have no idea on how to recover it. The only evidence we have, at this point of time is this memory dump. Please help me.

Visiblement, il va falloir rechercher un fichier supprimé ... c'est parti !

#  Process

Histoire de se simplifier la vie, on va sauvegarder le résultat de `pslist` dans un fichier texte pour pouvoir s'y référer plus facilement. Voilà ce que j'ai noté d'intéressant :

| Process        | PID  |
| -------------- | ---- |
| StickyNot.exe  | 2438 |
| GoogleCrashHan | 2272 |
| GoogleCrashHan | 2248 |

Visiblement, on a le widget Sticky Notes d'ouvert. J'ai déjà vu dans un autre CTF des informations intéressantes (--> des mots de passe) notés là-dedans. Le contenu des notes est stocké quelque part dans le système de fichiers, on ira donc voir ce qui s'y cache !
Aussi, deux process GoogleCrashHandler. Est-ce qu'un Chrome aurait planté ? On va regarder ça aussi.

# Sticky notes

Selon [cet article](https://www.pcworld.com/article/2090765/where-sticky-notes-are-stored-and-how-you-can-recover-them.html) de PCWorld : 

> Windows stores your sticky notes in a special appdata folder, which is probably C:\Users\logon\AppData\Roaming\Microsoft\Sticky Notes—with logon being the name with which you log onto your PC. You’ll find only one file in that folder, StickyNotes.snt, which contains all your notes". 

Ok, essayons de chercher ce fichier :

```bash
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f MemoryDump_Lab4.raw --profile=Win7SP1x64 filescan | grep "StickyNotes.snt"
Volatility Foundation Volatility Framework 2.6.1
0x000000003fd40910     17      1 RW-r-- \Device\HarddiskVolume2\Users\SlimShady\AppData\Roaming\Microsoft\Sticky Notes\StickyNotes.snt
```

Cool, maintenant, comment va-t-on faire pour en extraire le contenu ? Après une petite recherche, je tombe sur [ce post SuperUser](https://superuser.com/a/518786), qui semble indiquer qu'il s'agit en fait d'une archive que l'on peut ouvrir avec 7Zip. Et effectivement, après un `7z x StickyNotes.snt`, on obtient bien trois fichiers, dont le premier est un RTF avec le texte suivant : 

```
The clipboard plugin works well but it doesn't give the flag :P
```

Ok, ça troll. Essayons le plugin clipboard, par curiosité ?

```bash
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f MemoryDump_Lab4.raw --profile=Win7SP1x64 clipboard                           
Volatility Foundation Volatility Framework 2.6.1
Session    WindowStation Format                         Handle Object             Data                                              
---------- ------------- ------------------ ------------------ ------------------ --------------------------------------------------
         2 WinSta0       CF_UNICODETEXT                    0x0 ------------------                                                   
         2 WinSta0       0x0L                             0x10 ------------------                                                   
         2 WinSta0       0x100ffL               0x200000000000 ------------------                                                   
         2 WinSta0       CF_TEXT                           0x1 ------------------                                                   
         1 WinSta0       CF_UNICODETEXT                    0x0 ------------------                                                   
         1 WinSta0       0x0L                             0x10 ------------------                                                   
         1 WinSta0       0x1010bL               0x200000000000 ------------------                                                   
         1 WinSta0       CF_TEXT                           0x1 ------------------                                                   
         1 ------------- ------------------            0x1010b 0xfffff900c1eb75c0                                                   
         2 ------------- ------------------            0x100ff 0xfffff900c1acd390 
```

Ok, je ne vois pas trop quoi faire de ces informations pour l'instant. En lisant [la doc de clipboard sur le repo de Volatility](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference-Gui#clipboard), j'ai vu qu'ils mentionnaient le fait qu'on pouvait voir le path d'un fichier qui aurait été copié-collé avec la commande `clipboard`. Dans leur exemple, on peut voir qu'un fichier a été copié sur le bureau, ça m'a donné l'idée de regarder si on ne trouvait pas quelque chose d'intéressant ici :

```bash
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f MemoryDump_Lab4.raw --profile=Win7SP1x64 filescan | grep "Desktop"            
Volatility Foundation Volatility Framework 2.6.1
0x000000003e8ad250     14      0 R--r-- \Device\HarddiskVolume2\Users\eminem\Desktop\galf.jpeg
0x000000003e8d19e0     16      0 R--r-- \Device\HarddiskVolume2\Users\eminem\Desktop\Screenshot1.png
0x000000003fc398d0     16      0 R--rw- \Device\HarddiskVolume2\Users\SlimShady\Desktop\Important.txt
```

En effet, y'a du monde (j'ai supprimé les lignes qui n'étaient pas intéressantes) ! Note pour plus tard : toujours regarder ce qui se trouve sur le bureau dans un CTF :)

J'ai bien réussi à dumper les deux images (`galf.jpeg` et `Screenshot1.png`). En revanche, rien ne se passe lorsque j'essaye de dumper `Important.txt`. Après une petite recherche, j'apprends dans [cette issue](https://github.com/volatilityfoundation/volatility/issues/588) que 

> "its possible the file's content isn't cached anymore by the cache manager (its likely to get removed after the process closes the file handle). That doesn't mean the file's content is entirely wiped out of RAM however...it could just mean the association between the file object and the file data is broken (i.e. pointers).". 

Peut-être qu'il s'agit de notre fichier supprimé ? Gardons ça sous le coude pour plus tard.

Je commence par faire un `steghide` sur l'image `galf.jpeg`, et à nouveau, ça troll : on obtient un fichier `LMAO.txt` avec le contenu suivant : 

```
Move on bro, there is nothing here :). 
```
Super.

L'autre fichier, `Screenshot1.png`, est juste un screenshot de Google. Rien ne semble caché dedans, steganographiquement parlant.

J'ai par la suite essayé les plugins permettant d'extraire des artefacts de Chrome, mais sans succès.

# Fichier supprimé : le flag

Bon, au final on diverge beaucoup mais on trouve pas grand chose. Je décide donc de faire un tour sur internet pour chercher comment recover un fichier supprimé via volatility. Et là, je tombe sur un writeup d'un autre CTF créé par stuxn3t, où il est également question de recover un fichier nommé `Important.txt`. Et là, la réponse tombe toute cuite, il suffit d'utiliser le plugin `mftparser` pour afficher la Master File Table et avoir accès au fichier, a condition qu'il n'ait pas été override.

Et là, sans surprise, si on envoie le résultat de la commande dans in fichier texte, on obtient :

```bash
MFT entry found at offset 0x3bd8ac00
Attribute: In Use & File
Record Number: 60583
Link count: 2


$STANDARD_INFORMATION
Creation                       Modified                       MFT Altered                    Access Date                    Type
------------------------------ ------------------------------ ------------------------------ ------------------------------ ----
2019-06-27 13:14:13 UTC+0000 2019-06-27 13:26:12 UTC+0000   2019-06-27 13:26:12 UTC+0000   2019-06-27 13:14:13 UTC+0000   Archive

$FILE_NAME
Creation                       Modified                       MFT Altered                    Access Date                    Name/Path
------------------------------ ------------------------------ ------------------------------ ------------------------------ ---------
2019-06-27 13:14:13 UTC+0000 2019-06-27 13:14:13 UTC+0000   2019-06-27 13:14:13 UTC+0000   2019-06-27 13:14:13 UTC+0000   Users\SlimShady\Desktop\IMPORT~1.TXT

$FILE_NAME
Creation                       Modified                       MFT Altered                    Access Date                    Name/Path
------------------------------ ------------------------------ ------------------------------ ------------------------------ ---------
2019-06-27 13:14:13 UTC+0000 2019-06-27 13:14:13 UTC+0000   2019-06-27 13:14:13 UTC+0000   2019-06-27 13:14:13 UTC+0000   Users\SlimShady\Desktop\Important.txt

$OBJECT_ID
Object ID: 7726a550-d498-e911-9cc1-0800275e72bc
Birth Volume ID: 80000000-b800-0000-0000-180000000100
Birth Object ID: 99000000-1800-0000-690d-0a0d0a0d0a6e
Birth Domain ID: 0d0a0d0a-0d0a-6374-0d0a-0d0a0d0a0d0a

$DATA
0000000000: 69 0d 0a 0d 0a 0d 0a 6e 0d 0a 0d 0a 0d 0a 63 74   i......n......ct
0000000010: 0d 0a 0d 0a 0d 0a 0d 0a 66 7b 31 0d 0a 0d 0a 0d   ........f{1.....
0000000020: 0a 5f 69 73 0d 0a 0d 0a 0d 0a 5f 6e 30 74 0d 0a   ._is......_n0t..
0000000030: 0d 0a 0d 0a 0d 0a 5f 45 51 75 34 6c 0d 0a 0d 0a   ......_EQu4l....
0000000040: 0d 0a 0d 0a 5f 37 6f 5f 32 5f 62 55 74 0d 0a 0d   ...._7o_2_bUt...
0000000050: 0a 0d 0a 0d 0a 0d 0a 0d 0a 0d 0a 5f 74 68 31 73   ..........._th1s
0000000060: 5f 64 30 73 33 6e 74 0d 0a 0d 0a 0d 0a 0d 0a 5f   _d0s3nt........_
0000000070: 6d 34 6b 65 0d 0a 0d 0a 0d 0a 5f 73 33 6e 0d 0a   m4ke......_s3n..
0000000080: 0d 0a 0d 0a 0d 0a 73 33 7d 0d 0a 0d 0a 47 6f 6f   ......s3}....Goo
0000000090: 64 20 77 6f 72 6b 20 3a 50                        d.work.:P
```

Le contenu de `$DATA` semble bien être un flag : `ctf{1_is_n0t_EQu4l_7o_2_bUt_th1s_d0s3nt_m4ke_s3ns3}`, suivi d'un `Good work :P`.

Alors bon, je n'avais pas connaissance de ce plugin-là, ni-même de la MFT en soi, on va donc faire de petites recherches sur le sujet histoire de ramener quelque chose de ce CTF :)

# Was ist MFT ?

On commence donc par lire [la doc Microsoft sur la MFT](https://docs.microsoft.com/en-us/windows/win32/fileio/master-file-table). Elle nous dit que 

> [all] information about a file, including its size, time and date stamps, permissions, and data content, is stored either in MFT entries, or in space outside the MFT that is described by MFT entries.
> As files are added to an NTFS file system volume, more entries are added to the MFT and the MFT increases in size. When files are deleted from an NTFS file system volume, their MFT entries are marked as free and may be reused. However, disk space that has been allocated for these entries is not reallocated, and the size of the MFT does not decrease

Ok, donc pour chaque fichier sur notre machine, on aurait une entrée dans la MFT qui nous donnerait sa taille, ses dates de création/modification, etc. Selon [l'article Wikipedia](https://fr.wikipedia.org/wiki/Master_File_Table), si un fichier est de petite taille (700 à 800 octets visiblement), le fichier est sotcké directement dans la MFT.

Aussi, je suis tombé sur [un fil Reddit plutôt intéressant](https://www.reddit.com/r/computerforensics/comments/jook6y/when_does_an_mft_record_get_deleted/) qui parle de ce qu'il se passe lorsque l'on supprime un fichier, et comment les entrées de la MFT sont modifiées en fonction.

# Conclusion

J'aurais peut-être pu chercher à recover le fichier supprimé dès le début, mais le fait de tâtonner m'a donné l'occasion de me pencher sur les Sticky Notes qui peuvent être un artefact intéressant, et aussi de me souvenir de toujours faire un tour sur le Bureau pour trouver des choses (parfois) pertinentes !

Aussi, j'ai pu apprendre les bases du fonctionnement de la MFT, et tester `mftparser` qui pourra je pense s'avérer vachement pratique dans de futurs CTF.