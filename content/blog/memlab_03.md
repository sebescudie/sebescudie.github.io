---
title: "MEMLABS - LAB03"
date: 2021-06-16
author: "seb"
description: "Writeup du troisième lab MEMLABS"
type: "post"
---

Troisième post, troisième writeup MEMLABS ! Cette fois-ci je vais moins rentrer dans les détails de chaque commande. C'est parti pour la description du challenge :

>A malicious script encrypted a very secret piece of information I had on my system. Can you recover the information for me please?
>Note-1: This challenge is composed of only 1 flag. The flag split into 2 parts.
>Note-2: You'll need the first half of the flag to get the second.
>You will need this additional tool to solve the challenge

```bash
$ sudo apt install steghide
```

# Liste des process

Comme d'habitude, on commence par un `pslist` pour voir ce qui se passait sur notre machine. Parmi les process, on peut noter deux instances de `notepad.exe`, dont on va noter les PID :

- 3736
- 3432

# Première moitié du flag

Regardons donc ce qui se passait dans le bloc note. Une fois encore, le plugin `notepad` ne fonctionne pas avec ce profil, on va donc essayer `cmdline` pour voir quels fichiers ont été ouverts :

```bash
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/ctf/MEMLABS/MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 cmdline -p 3736,3432
Volatility Foundation Volatility Framework 2.6.1
************************************************************************
notepad.exe pid:   3736
Command line : "C:\Windows\system32\NOTEPAD.EXE" C:\Users\hello\Desktop\evilscript.py
************************************************************************
notepad.exe pid:   3432
Command line : "C:\Windows\system32\NOTEPAD.EXE" C:\Users\hello\Desktop\vip.txt
```

Wow, rien que ça. Un script python semblerait-il malicieux, et un fichier `vip.txt`. Un `filescan`, `dumpfiles` et renommage plus tard, on a bien les deux fichiers sur notre machine.

Voyons ce que contient `vip.txt` :

```bash
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS]
└─$ cat vip.txt                                      
am1gd2V4M20wXGs3b2U=
```
On dirait une string en base64 .. en la décodant, on obtient : ``jm`wex3m0\k7oe``. Pour l'instant je ne sais pas trop quoi faire de cette information, mais je garde ça sous le coude pour plus tard. Il s'agit peut-être d'un bout du flag ?

Regardons maintenant notre `evilscript.py`

```python
import sys
import string

def xor(s):
        a = ''.join(chr(ord(i)^3) for i in s)
        return a
def encoder(x):
        return x.encode("base64")
if __name__ == "__main__":
        f = open("C:\\Users\\hello\\Desktop\\vip.txt", "w")
        arr = sys.argv[1]
        arr = encoder(xor(arr))
        f.write(arr)
        f.close()
```

Effectivement c'est plus clair, il s'agit du script qui a servi à encoder notre fichier `vip.txt` ! Visiblement, le script va prendre une string en parametre, `xor` chacun de ses caractères avec 3 et encoder le tout en base64. Essayons d'écrire un script qui va reverse ça (ici sur python 3.9) :

```python
import base64

def xor(s):
    a = ''.join(chr(ord(i)^3) for i in s)
    return a

def decoder(x):
    return base64.b64decode(x).decode('utf-8')

if __name__ == "__main__":
    print(xor(decoder("am1gd2V4M20wXGs3b2U=")))
```

Quand on run ça, on obtient `inctf{0n3_h4lf`, ce qui ressemble bien à une première moitié de flag !

J'ai découvert il y a peu l'outil en ligne [Cyberchef](https://gchq.github.io/CyberChef/), qui permet de faire facilement une ou plusieurs opérations sur une chaine de caractères avec une GUI dans le navigateur. Voilà comment on aurait pu faire la même chose avec :

![](https://raw.githubusercontent.com/sebescudie/sebescudie.github.io/master/static/images/blog/memlab_03/cyberchef.png)


# Second flag

Et maintenant, il faut chercher. Rien ne m'a sauté aux yeux dans `pstree`, rien de particulièrement fou avec `screenshot` ... j'ai donc regardé si d'autres fichiers intéressants n'étaient pas aussi sur le bureau, comme les deux que l'on a extrait tout à l'heure. Et effectivement ...

```bash
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/ctf/MEMLABS/MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 filescan | grep "Desktop"
Volatility Foundation Volatility Framework 2.6.1
0x0000000004f34148      2      0 RW---- \Device\HarddiskVolume2\Users\hello\Desktop\suspision1.jpeg
[...]
```

Un fichier jpeg avec un nom marrant. Ca semble coller avec l'indication en début de CTF qui demandait d'installer `steghide`. On rapatrie donc le fichier sur notre disque dur, il s'agit d'une image shutterstock :

![](https://raw.githubusercontent.com/sebescudie/sebescudie.github.io/master/static/images/blog/memlab_03/suspision1.jpeg)

Ok, donc si essaye de passer l'image dans `steghide`, on nous demande un mot de passe :

```bash
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS]
└─$ steghide --extract -sf suspision1.jpeg
Enter passphrase: 
```

L'intro du CTF disait qu'on avait besoin de la première moitié du flag pour trouver le deuxième, essayons ça en guise de mot de passe ...

```bash
wrote extracted data to "secret text".
```

Excellent. Et quand on affiche `secret text` :

```bash
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS]
└─$ cat secret\ text
_1s_n0t_3n0ugh}
```

On trouve bien la deuxième moitié du flag !