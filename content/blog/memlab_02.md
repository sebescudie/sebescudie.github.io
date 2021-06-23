---
title: "MEMLABS - LAB02"
date: 2021-06-16
author: "seb"
description: "Writeup du second lab MEMLABS"
type: "post"
---

Encore un writeup, cette fois-ci sur le second lab du repo [MEMLABS](https://github.com/stuxnet999/MemLabs) ! Sans plus attendre, la description du challenge :

> One of the clients of our company, lost the access to his system due to an unknown error. He is supposedly a very popular "environmental" activist. As a part of the investigation, he told us that his go to applications are browsers, his password managers etc. We hope that you can dig into this memory dump and find his important stuff and give it back to us.

Quelles sont nos pistes ? Visiblement, on va avoir affaire à :

- Un browser
- Un gestionnaire de mots de passe
- Le "environmental" entre guillemets peut-être un indice indiquant qu'on va devoir regarder du côté des variables d'environnement

# Profil

C'est parti, commençons par déterminer le profil de notre sample.

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/memory_samples/MemoryDump_Lab2.raw imageinfo
Volatility Foundation Volatility Framework 2.6.1
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win7SP1x64, Win7SP0x64, Win2008R2SP0x64, Win2008R2SP1x64_24000, Win2008R2SP1x64_23418, Win2008R2SP1x64, Win7SP1x64_24000, Win7SP1x64_23418
[...]
```

Ok, `Win7SP1x64` it is.

# CTRL+SHIFT+ESC

Regardons maintenant ce qui se passait sur la machine au moment du dump avec `pslist` :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/memory_samples/MemoryDump_Lab2.raw --profile=Win7SP1x64 pslist
Volatility Foundation Volatility Framework 2.6.1
Offset(V)          Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit                          
------------------ -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0xfffffa8000ca0040 System                    4      0     80      541 ------      0 2019-12-14 10:35:21 UTC+0000                                 
0xfffffa80014976c0 smss.exe                248      4      3       37 ------      0 2019-12-14 10:35:21 UTC+0000                                 
0xfffffa80014fdb30 csrss.exe               320    312     10      446      0      0 2019-12-14 10:35:27 UTC+0000                                 
0xfffffa8001c40060 csrss.exe               368    360      8      237      1      0 2019-12-14 10:35:28 UTC+0000                                 
0xfffffa8000ca8840 psxss.exe               376    248     18      786      0      0 2019-12-14 10:35:28 UTC+0000                                 
0xfffffa8001c5a700 winlogon.exe            416    360      6      112      1      0 2019-12-14 10:35:30 UTC+0000                                 
0xfffffa8001c5b2b0 wininit.exe             424    312      3       75      0      0 2019-12-14 10:35:30 UTC+0000                                 
0xfffffa8001c95320 services.exe            484    424      8      206      0      0 2019-12-14 10:35:31 UTC+0000                                 
0xfffffa8001c9d910 lsass.exe               492    424      8      546      0      0 2019-12-14 10:35:31 UTC+0000                                 
0xfffffa8001c9e2d0 lsm.exe                 500    424     10      181      0      0 2019-12-14 10:35:31 UTC+0000                                 
0xfffffa8001cec790 svchost.exe             588    484     12      354      0      0 2019-12-14 10:35:35 UTC+0000                                 
0xfffffa8001d13060 VBoxService.ex          652    484     14      135      0      0 2019-12-14 10:35:36 UTC+0000                                 
0xfffffa8001d4ab30 svchost.exe             720    484      7      275      0      0 2019-12-14 10:35:37 UTC+0000                                 
0xfffffa8001d76320 svchost.exe             812    484     21      474      0      0 2019-12-14 10:35:38 UTC+0000                                 
0xfffffa8001da6930 svchost.exe             852    484     20      417      0      0 2019-12-14 10:35:38 UTC+0000                                 
0xfffffa8001dacb30 svchost.exe             876    484     39      962      0      0 2019-12-14 10:35:38 UTC+0000                                 
0xfffffa8001df65f0 audiodg.exe             268    812      7      131      0      0 2019-12-14 10:35:41 UTC+0000                                 
0xfffffa8001e1eb30 svchost.exe             472    484     12      301      0      0 2019-12-14 10:35:42 UTC+0000                                 
0xfffffa8001e47740 svchost.exe            1044    484     16      361      0      0 2019-12-14 10:35:43 UTC+0000                                 
0xfffffa8000cf9220 spoolsv.exe            1208    484     13      279      0      0 2019-12-14 10:35:47 UTC+0000                                 
0xfffffa8001ed8b30 svchost.exe            1248    484     18      303      0      0 2019-12-14 10:35:49 UTC+0000                                 
0xfffffa8001f4eb30 svchost.exe            1368    484     23      314      0      0 2019-12-14 10:35:51 UTC+0000                                 
0xfffffa8001f7c060 TCPSVCS.EXE            1412    484      4       97      0      0 2019-12-14 10:35:53 UTC+0000                                 
0xfffffa80020f2b30 taskhost.exe           1928    484      9      154      1      0 2019-12-14 10:36:04 UTC+0000                                 
0xfffffa8002105b30 taskeng.exe            1996    876      5       79      0      0 2019-12-14 10:36:04 UTC+0000                                 
0xfffffa800211e7c0 dwm.exe                2012    852      4       72      1      0 2019-12-14 10:36:04 UTC+0000                                 
0xfffffa8002131340 explorer.exe           1064   2004     37      989      1      0 2019-12-14 10:36:05 UTC+0000                                 
0xfffffa80021bb240 SearchIndexer.         1836    484     12      628      0      0 2019-12-14 10:36:11 UTC+0000                                 
0xfffffa80021c0b30 VBoxTray.exe           1896   1064     13      138      1      0 2019-12-14 10:36:13 UTC+0000                                 
0xfffffa80022a3b30 csrss.exe              2308   2300      9      246      2      0 2019-12-14 10:36:24 UTC+0000                                 
0xfffffa80022a73e0 winlogon.exe           2336   2300      4      109      2      0 2019-12-14 10:36:24 UTC+0000                                 
0xfffffa8000e76060 taskhost.exe           2604    484      9      154      2      0 2019-12-14 10:36:29 UTC+0000                                 
0xfffffa8000e95060 dwm.exe                2652    852      3       71      2      0 2019-12-14 10:36:29 UTC+0000                                 
0xfffffa8000e9a110 explorer.exe           2664   2632     19      632      2      0 2019-12-14 10:36:29 UTC+0000                                 
0xfffffa8000edcb30 VBoxTray.exe           2792   2664     12      139      2      0 2019-12-14 10:36:30 UTC+0000                                 
0xfffffa80022e5950 cmd.exe                2096   2664      1       19      2      0 2019-12-14 10:36:35 UTC+0000                                 
0xfffffa8000e63060 conhost.exe            2068   2308      2       50      2      0 2019-12-14 10:36:35 UTC+0000                                 
0xfffffa8002109b30 chrome.exe             2296   2664     27      658      2      0 2019-12-14 10:36:45 UTC+0000                                 
0xfffffa8001cc7a90 chrome.exe             2304   2296      8       71      2      0 2019-12-14 10:36:45 UTC+0000                                 
0xfffffa8000eea7a0 chrome.exe             2476   2296      2       55      2      0 2019-12-14 10:36:46 UTC+0000                                 
0xfffffa8000ea2b30 chrome.exe             2964   2296     13      295      2      0 2019-12-14 10:36:47 UTC+0000                                 
0xfffffa8000fae6a0 chrome.exe             2572   2296      8      177      2      0 2019-12-14 10:36:56 UTC+0000                                 
0xfffffa800105c060 WmiPrvSE.exe           2636    588     12      293      0      0 2019-12-14 10:37:02 UTC+0000                                 
0xfffffa800100c060 WmiApSrv.exe           2004    484      6      115      0      0 2019-12-14 10:37:05 UTC+0000                                 
0xfffffa800230eb30 chrome.exe             1632   2296     14      219      2      0 2019-12-14 10:37:12 UTC+0000                                 
0xfffffa800101e640 dllhost.exe            2376    588      9      250      1      0 2019-12-14 10:37:40 UTC+0000                                 
0xfffffa800224a8c0 KeePass.exe            3008   1064     12      316      1      0 2019-12-14 10:37:56 UTC+0000                                 
0xfffffa8002230b30 sppsvc.exe             2764    484      5      151      0      0 2019-12-14 10:38:00 UTC+0000                                 
0xfffffa80010e5b30 svchost.exe            1076    484     17      337      0      0 2019-12-14 10:38:02 UTC+0000                                 
0xfffffa80010f44a0 wmpnetwk.exe            928    484     18      523      0      0 2019-12-14 10:38:03 UTC+0000                                 
0xfffffa80011956a0 notepad.exe            3260   3180      1       61      1      0 2019-12-14 10:38:20 UTC+0000                                 
0xfffffa80011aa060 DumpIt.exe             3844   1064      2       45      1      1 2019-12-14 10:38:43 UTC+0000                                 
0xfffffa8001194570 conhost.exe            3852    368      2       52      1      0 2019-12-14 10:38:43 UTC+0000                                 
0xfffffa8001189b30 WmiPrvSE.exe           4004    588      9  1572864 ------      0 2019-12-14 10:39:00 UTC+0000                                 
```

On peut ici isoler quelques processes interessants. Notons leurs PID pour plus tard!

| Process     | PID  |
|-------------|------|
| KeePass.exe | 3008 |
| chrome.exe  | 1632 |
| notepad.exe | 3260 |

# Notepad - premier flag

Je me souviens avoir croisé dans l'aide de volatility un plugin qui concernait notepad :

```
notepad         List currently displayed notepad text
```

Essayons le sur notre sample, voir si on arrive à extraire quelque chose d'intéressant :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/memory_samples/MemoryDump_Lab2.raw --profile=Win7SP1x64 notepad
Volatility Foundation Volatility Framework 2.6.1
ERROR   : volatility.debug    : This command does not support the profile Win7SP1x64
```

Bon, visiblement c'est non.

Peut-être qu'on peut regarder avec quels args le process a été lancé ?

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/memory_samples/MemoryDump_Lab2.raw --profile=Win7SP1x64 cmdline -p 3260                                          1 ⨯
Volatility Foundation Volatility Framework 2.6.1
************************************************************************
notepad.exe pid:   3260
Command line : "C:\Windows\system32\NOTEPAD.EXE" C:\Users\SmartNet\Secrets\Hidden.kdbx
```

Un fichier Keepass (mais pourquoi avoir cherché à l'ouvrir avec notepad ?!) ! Essayons de le récupérer :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/memory_samples/MemoryDump_Lab2.raw --profile=Win7SP1x64 filescan | grep "Hidden.kdbx"
Volatility Foundation Volatility Framework 2.6.1
0x000000003fb112a0     16      0 R--r-- \Device\HarddiskVolume2\Users\SmartNet\Secrets\Hidden.kdbx
```

Il est bien là.


```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/memory_samples/MemoryDump_Lab2.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000003fb112a0 -D .                     1 ⨯
Volatility Foundation Volatility Framework 2.6.1
DataSectionObject 0x3fb112a0   None   \Device\HarddiskVolume2\Users\SmartNet\Secrets\Hidden.kdbx
```

Super. Renommons le fichier dumpé en `secret.kdbx` et essayons de l'ouvrir avec KeepassXC.. evidement on nous demande un mot de passe. L'intro du CTF mentionnait "environment", peut-être que volatility nous permet de lister les varibles d'environnement de la machine ?

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/memory_samples/MemoryDump_Lab2.raw --profile=Win7SP1x64 envars -p 3008
Volatility Foundation Volatility Framework 2.6.1
Pid      Process              Block              Variable                       Value
-------- -------------------- ------------------ ------------------------------ -----
    3008 KeePass.exe          0x0000000000161320 ALLUSERSPROFILE                C:\ProgramData
    3008 KeePass.exe          0x0000000000161320 APPDATA                        C:\Users\Alissa Simpson\AppData\Roaming
    3008 KeePass.exe          0x0000000000161320 CommonProgramFiles             C:\Program Files\Common Files
    3008 KeePass.exe          0x0000000000161320 CommonProgramFiles(x86)        C:\Program Files (x86)\Common Files
    3008 KeePass.exe          0x0000000000161320 CommonProgramW6432             C:\Program Files\Common Files
    3008 KeePass.exe          0x0000000000161320 COMPUTERNAME                   SMARTNET-PC
    3008 KeePass.exe          0x0000000000161320 ComSpec                        C:\Windows\system32\cmd.exe
    3008 KeePass.exe          0x0000000000161320 FP_NO_HOST_CHECK               NO
    3008 KeePass.exe          0x0000000000161320 HOMEDRIVE                      C:
    3008 KeePass.exe          0x0000000000161320 HOMEPATH                       \Users\Alissa Simpson
    3008 KeePass.exe          0x0000000000161320 LOCALAPPDATA                   C:\Users\Alissa Simpson\AppData\Local
    3008 KeePass.exe          0x0000000000161320 LOGONSERVER                    \\SMARTNET-PC
    3008 KeePass.exe          0x0000000000161320 NEW_TMP                        C:\Windows\ZmxhZ3t3M2xjMG0zX1QwXyRUNGczXyFfT2ZfTDRCXzJ9
    3008 KeePass.exe          0x0000000000161320 NUMBER_OF_PROCESSORS           1
    3008 KeePass.exe          0x0000000000161320 OS                             Windows_NT
    3008 KeePass.exe          0x0000000000161320 Path                           C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\
    3008 KeePass.exe          0x0000000000161320 PATHEXT                        .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
    3008 KeePass.exe          0x0000000000161320 PROCESSOR_ARCHITECTURE         AMD64
    3008 KeePass.exe          0x0000000000161320 PROCESSOR_IDENTIFIER           Intel64 Family 6 Model 142 Stepping 9, GenuineIntel
    3008 KeePass.exe          0x0000000000161320 PROCESSOR_LEVEL                6
    3008 KeePass.exe          0x0000000000161320 PROCESSOR_REVISION             8e09
    3008 KeePass.exe          0x0000000000161320 ProgramData                    C:\ProgramData
    3008 KeePass.exe          0x0000000000161320 ProgramFiles                   C:\Program Files
    3008 KeePass.exe          0x0000000000161320 ProgramFiles(x86)              C:\Program Files (x86)
    3008 KeePass.exe          0x0000000000161320 ProgramW6432                   C:\Program Files
    3008 KeePass.exe          0x0000000000161320 PSModulePath                   C:\Windows\system32\WindowsPowerShell\v1.0\Modules\
    3008 KeePass.exe          0x0000000000161320 PUBLIC                         C:\Users\Public
    3008 KeePass.exe          0x0000000000161320 SESSIONNAME                    Console
    3008 KeePass.exe          0x0000000000161320 SystemDrive                    C:
    3008 KeePass.exe          0x0000000000161320 SystemRoot                     C:\Windows
    3008 KeePass.exe          0x0000000000161320 TEMP                           C:\Users\ALISSA~1\AppData\Local\Temp
    3008 KeePass.exe          0x0000000000161320 TMP                            C:\Users\ALISSA~1\AppData\Local\Temp
    3008 KeePass.exe          0x0000000000161320 USERDOMAIN                     SmartNet-PC
    3008 KeePass.exe          0x0000000000161320 USERNAME                       Alissa Simpson
    3008 KeePass.exe          0x0000000000161320 USERPROFILE                    C:\Users\Alissa Simpson
    3008 KeePass.exe          0x0000000000161320 windir                         C:\Windows
    3008 KeePass.exe          0x0000000000161320 windows_tracing_flags          3
    3008 KeePass.exe          0x0000000000161320 windows_tracing_logfile        C:\BVTBin\Tests\installpackage\csilogfile.log
```

Mh, la variable `NEW_TMP` est étrange : `C:\Windows\ZmxhZ3t3M2xjMG0zX1QwXyRUNGczXyFfT2ZfTDRCXzJ9`. On dirait une string en base64, essayons de la décoder :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ echo ZmxhZ3t3M2xjMG0zX1QwXyRUNGczXyFfT2ZfTDRCXzJ9 | base64 --decode
flag{w3lc0m3_T0_$T4g3_!_Of_L4B_2}
```

Et hop, premier flag ! Ceci dit, ce n'est pas le mot de passe de notre fichier keepas. J'imagine qu'on trouvera l'info plus tard.

Ici j'ai d'abord lancé `envars` sans PID, mais devant la masse de résultats, j'ai filtré avec celui de Keepass, pour voir si quelque chose s'en dégageait. On a bien trouvé notre flag mais j'avoue ne pas trop comprendre pourquoi cette variable est rattachée au PID en question...

Par acquis de conscience, j'ai vérifié le résultat de `cmdline` sur le PID de keepass, on trouve bien le même fichier :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/memory_samples/MemoryDump_Lab2.raw --profile=Win7SP1x64 cmdline -p 3008  
Volatility Foundation Volatility Framework 2.6.1
************************************************************************
KeePass.exe pid:   3008
Command line : "C:\Program Files (x86)\KeePass Password Safe 2\KeePass.exe" "C:\Users\SmartNet\Secrets\Hidden.kdbx"
```

# Chrome - troisième flag

Il était fait mention dans l'intro d'un navigateur. Dans la liste des process, on tombe sur un `chrome.exe`. De base, volatility ne dispose d'aucun plugin pour extraire des renseignements facilement d'un browser (historique de navigation, cookies..), on devrait faire ça à la main. Ceci dit, après une petite recherche, on tombe sur le repo [volatility-plugins](https://github.com/superponible/volatility-plugins) de [superponible](https://github.com/superponible), qui propose entre autres un plugin `chromehistory`, qui nous donne accès à des tas de choses intéressantes sur le browser (historique, cookies, termes de recherche, etc.). Et en effet, c'est très simple à utiliser :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/memory_samples/MemoryDump_Lab2.raw --profile=Win7SP1x64 chromehistory  
Volatility Foundation Volatility Framework 2.6.1
Index  URL                                                                              Title                                                                            Visits Typed Last Visit Time            Hidden Favicon ID
------ -------------------------------------------------------------------------------- -------------------------------------------------------------------------------- ------ ----- -------------------------- ------ ----------
    34 https://bi0s.in/                                                                 Amrita Bios                                                                           1     1 2019-12-14 10:37:11.596681        N/A       
    33 http://bi0s.in/                                                                  Amrita Bios                                                                           1     0 2019-12-14 10:37:11.596681        N/A       
    32 https://mega.nz/#F!TrgSQQTS!H0ZrUzF0B-ZKNM3y9E76lg                               MEGA                                                                                  2     0 2019-12-14 10:21:39.602970        N/A       
    31 https://www.ndtv.com/                                                            NDTV: Latest News, India News, Breaking...s, Bollywood, Cricket, Videos & Photos      1     1 2019-12-14 10:18:09.449115        N/A       
    30 http://ndtv.com/                                                                 NDTV: Latest News, India News, Breaking...s, Bollywood, Cricket, Videos & Photos      1     0 2019-12-14 10:18:09.449115        N/A       
    28 http://blog.bi0s.in/                                                             bi0s                                                                                  1     0 2019-12-14 09:41:52.269568        N/A       
    29 https://blog.bi0s.in/                                                            bi0s                                                                                  1     1 2019-12-14 10:18:12.073607        N/A       
    27 https://r3xnation.wordpress.com/about/                                           About – R3xNation                                                                   1     0 2019-12-14 10:07:31.296539        N/A       
    26 https://www.youtube.com/                                                         YouTube                                                                               1     1 2019-12-14 10:04:59.173510        N/A       
    24 http://in.yahoo.com/                                                             Yahoo India | News, Finance, Cricket, Lifestyle and Entertainment                     1     0 2019-12-14 09:33:25.210345        N/A       
    23 http://yahoo.in/                                                                 Yahoo India | News, Finance, Cricket, Lifestyle and Entertainment                     1     1 2019-12-14 09:33:25.210345        N/A       
    25 https://in.yahoo.com/                                                            Yahoo India | News, Finance, Cricket, Lifestyle and Entertainment                     2     0 2019-12-14 09:33:32.266003        N/A       
    21 https://www.bbc.com/sport/football/50780855                                      Jurgen Klopp signs new Liverpool deal until 2024 - BBC Sport                          1     0 2019-12-14 09:31:35.842850        N/A       
    19 https://bbc.com/                                                                 BBC - Homepage                                                                        1     1 2019-12-14 09:30:55.836868        N/A       
    18 http://bbc.com/                                                                  BBC - Homepage                                                                        1     0 2019-12-14 09:30:55.836868        N/A       
    20 https://www.bbc.com/                                                             BBC - Homepage                                                                        1     0 2019-12-14 09:30:55.836868        N/A       
    17 https://volatilevirus.home.blog/blog-posts/                                      Blog Posts – Abhiram's Blog                                                         1     0 2019-12-14 10:07:35.236223        N/A       
    16 https://ashutosh1206.github.io/writeups/                                         Writeups | Ashutosh                                                                   1     0 2019-12-14 10:07:32.324863        N/A       
    15 https://www.india.com/                                                           Latest India News, Breaking News, Entertainment News | India.com News                 1     1 2019-12-14 09:30:08.206258        N/A       
    14 https://www.onlinesbi.com/                                                       State Bank of India                                                                   1     1 2019-12-14 09:29:37.802253        N/A       
    12 http://ashutosh1206.github.io/                                                   Home | Ashutosh                                                                       1     0 2019-12-14 09:29:33.876790        N/A       
    13 https://ashutosh1206.github.io/                                                  Home | Ashutosh                                                                       1     1 2019-12-14 09:29:33.876790        N/A       
    10 http://r3xnation.wordpress.com/                                                  R3xNation – Free Flowing passions                                                   1     0 2019-12-14 09:29:17.212089        N/A       
     9 https://volatilevirus.home.blog/                                                 Abhiram's Blog – Dying Is The Day Worth Living For!!                                1     1 2019-12-14 09:27:31.877522        N/A       
     8 http://volatilevirus.home.blog/                                                  Abhiram's Blog – Dying Is The Day Worth Living For!!                                1     0 2019-12-14 09:27:31.877522        N/A       
    11 https://r3xnation.wordpress.com/                                                 R3xNation – Free Flowing passions                                                   1     1 2019-12-14 09:29:17.212089        N/A       
     7 https://www.facebook.com/                                                        Facebook – log in or sign up                                                        3     1 2019-12-14 09:33:15.814086        N/A       
     4 http://bing.com/                                                                 Bing                                                                                  1     0 2019-12-14 09:16:18.118193        N/A       
     6 https://www.bing.com/?toWww=1&redig=2BBD701F84AA44D2A71D870534D085AE             Bing                                                                                  1     0 2019-12-14 09:33:00.366479        N/A       
     5 https://bing.com/                                                                Bing                                                                                  1     1 2019-12-14 09:16:18.118193        N/A       
     3 https://www.google.com/                                                          Google                                                                                2     1 2019-12-14 09:32:52.147284        N/A       
     2 https://chrome.google.com/webstore/category/extensions?hl=en                     Chrome Web Store - Extensions                                                         1     0 2019-12-14 09:32:53.844597        N/A       
     1 https://chrome.google.com/webstore?hl=en                                         Chrome Web Store - Extensions                                                         1     0 2019-12-14 09:16:05.724461        N/A       
```

Le lien MegaUpload saute directement aux yeux. Voyons ce qui se cache derrière.. 

![](https://raw.githubusercontent.com/sebescudie/sebescudie.github.io/master/static/images/blog/memlab_02/megaupload.png)

Une archive `Important.zip` ! Après l'avoir téléchargée, essayons de la déziper :

```bash
┌──(kali㉿kali)-[~/Downloads]
└─$ 7z x Important.zip   

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
[...]
Comment = Password is SHA1(stage-3-FLAG) from Lab-1. Password is in lowercase.
```

Evidement elle est protégée par mot de passe, et on nous indique qu'il s'agit du SHA1 du troisième flag du premier lab. Si on s'en réfère au [writeup publié sur ce même blog](https://sebescudie.github.io/blog/memlab_01/), le troisième flag était `flag{w3ll_3rd_stage_was_easy`.

Calculons le SHA1 du flag :

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ echo -n flag{w3ll_3rd_stage_was_easy} | sha1sum                                                                                                                                                                         127 ⨯
6045dd90029719a039fd2d2ebcca718439dd100a  -
```

On arrive bien à dézipper l'archive en fournissant ce mot de passe, et on obtient le troisième flag !

![](https://raw.githubusercontent.com/sebescudie/sebescudie.github.io/master/static/images/blog/memlab_02/flag3.png)

Il nous manque maintenant le deuxième ...

# Keepass

Après quelques recherches sur la manière d'extraire un master password Keypass d'un dump mémoire, je tombe sur [le writeup d'un autre CTF](https://blog.bi0s.in/2020/02/09/Forensics/HackTM-FindMyPass/) qui donne des informations intéressantes sur le sujet. 
Comme le fait remarquer l'auteur du writeup (qui semble d'ailleurs être la personne qui a créé le CTF que nous sommes en train de faire !), la base de données keepass était ouverte au moment du dump (on avait bien un process `keepass.exe` en train de tourner, ayant été lancé avec le fichier `kdbx` qu'on a dumpé plus tôt). Ce qui signifie que le master password est probablement quelque part dans la RAM.

L'auteur suggère ensuite de dumper le process en question, et faire un `strings` dessus pour y trouver le mot de passe. Et bien allons-y !

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/memory_samples/MemoryDump_Lab2.raw --profile=Win7SP1x64 memdump -p 3008 -D ./memdump 
Volatility Foundation Volatility Framework 2.6.1
************************************************************************
Writing KeePass.exe [  3008] to 3008.dmp
```

On maintenant notre dump, voyons ce que nous dit `strings`.

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ strings ./memdump/3008.dmp > strings.txt
````

Après dix siècles de scroll à la recherche d'un indice, rien d'intéressant. J'abandonne la piste du `memdump` (d'autant plus que je ne maitrise pas des masses le sujet) pour tester quelque chose de plus débile : et si notre environmental activist avait noté son mot de passe quelque part sur son ordinateur ? Essayons un `filescan` avec `password` ...

```bash
┌──(kali㉿kali)-[~/Documents/CTF/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f ~/Documents/memory_samples/MemoryDump_Lab2.raw --profile=Win7SP1x64 filescan | grep -i "password"
Volatility Foundation Volatility Framework 2.6.1
0x000000003e868370     16      0 R--r-d \Device\HarddiskVolume2\Program Files (x86)\KeePass Password Safe 2\KeePass.exe.config
0x000000003e873070      8      0 R--r-d \Device\HarddiskVolume2\Program Files (x86)\KeePass Password Safe 2\KeePass.exe
0x000000003e8ef2d0     13      0 R--r-d \Device\HarddiskVolume2\Program Files (x86)\KeePass Password Safe 2\KeePass.exe
0x000000003e8f0360      4      0 R--r-d \Device\HarddiskVolume2\Program Files (x86)\KeePass Password Safe 2\KeePass.XmlSerializers.dll
0x000000003eaf7880     15      1 R--r-d \Device\HarddiskVolume2\Program Files (x86)\KeePass Password Safe 2\KeePass.XmlSerializers.dll
0x000000003fb0abc0     10      0 R--r-d \Device\HarddiskVolume2\Program Files (x86)\KeePass Password Safe 2\KeePassLibC64.dll
0x000000003fce1c70      1      0 R--r-d \Device\HarddiskVolume2\Users\Alissa Simpson\Pictures\Password.png
0x000000003fd62f20      2      0 R--r-- \Device\HarddiskVolume2\Program Files (x86)\KeePass Password Safe 2\KeePass.config.xml
0x000000003fecf820     15      0 R--r-d \Device\HarddiskVolume2\Program Files (x86)\KeePass Password Safe 2\unins000.exe
```

Tiens `Password.png`... Une fois encore, essayons d'extraire le fichier (je vous fait grâce de la commande cette fois-ci) .. ça fonctionne ! L'image nous donne bien le mot de passe, ceci-dit il faudra un tout peut peu chercher dedans ;)

On fouille maintenant la base données, rien d'intéressant à première vue, quelques mots de passe randoms .. Regardons dans la corbeille de la DB Keepass

![](https://raw.githubusercontent.com/sebescudie/sebescudie.github.io/master/static/images/blog/memlab_02/keepass_01.png)

Quand on double-click sur cette entrée et qu'on révèle le mot de passe, bingo, on a notre second flag !

![](https://raw.githubusercontent.com/sebescudie/sebescudie.github.io/master/static/images/blog/memlab_02/keepass_02.png)