---
title: "MEMLABS - LAB05"
date: 2021-06-24
author: "seb"
description: "Writeup du cinquième lab MEMLABS"
type: "post"
---

Cinquième writeup, cinquième challenge MEMLABS ... allons-y pour l'intro :

>We received this memory dump from our client recently. Someone accessed his system when he was not there and he found some rather strange files being accessed. Find those files and they might be useful. I quote his exact statement,
>> The names were not readable. They were composed of alphabets and numbers but I wasn't able to make out what exactly it was.
> Also, he noticed his most loved application that he always used crashed every time he ran it. Was it a virus?
>
> - **Note-1:** This challenge is composed of 3 flags. If you think 2nd flag is the end, it isn't!! :P
> - **Note-2:** There was a small mistake when making this challenge. If you find any string which has the string "L4B_3_D0n3!!" in it, please change it to "L4B_5_D0n3!!" and then proceed.
> - **Note-3:** You'll get the stage 2 flag only when you have the stage 1 flag.

# Process

Comme d'habitude, on commence par un `pslist`. J'ai noté ici les process intéressants :

| Process      | PID  | PPID |
| ------------ | ---- | ---- |
| WinRAR.exe   | 2924 | 1580 |
| notepad.exe  | 2744 | 1580 |
| NOTEPAD.EXE  | 2724 | 1580 |
| WerFault.exe | 2716 | 2632 |
| NOTEPAD.EXE  | 1388 | 1580 |
| WerFault.exe | 780  | 2632 |
| NOTEPAD.EXE  | 2056 | 1580 |
| WerFault.exe | 2168 | 2632 |

Winrar, plusieurs instances du bloc-notes dont certaines en majuscule, et trois instances de `WerFault.exe`, qui est le process lancé par Windows quand une application crash. Ca fait déjà beaucoup de choses à analyser, allons-y pas à pas !

# WerFault

Je n'avais jamais vu le process WerFault avant, et après une petite recherche, je tombe sur [cet article](https://helgeklein.com/blog/2021/03/anatomy-of-werfault-exe-application-crash-error-reporting/) qui nous explique tout ce qui se passe lorsque une application crash et que l'OS lance WerFault. Visiblement, deux instances de WerFaut sont lancées, l'une par `SYSTEM` et l'autre par le process qui a crashé. On nous dit aussi que l'instance lancée par le process ayant crashé est démarrée avec les arguments `-u` (user-mode) et `-p` pour le PID du process en question.

Faisons donc un `cmdlines` sur les WerFault pour y voir plus clair :

```bash
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f MemoryDump_Lab5.raw --profile=Win7SP1x64 cmdline -p 2716,780,2168
Volatility Foundation Volatility Framework 2.6.1
************************************************************************
WerFault.exe pid:   2716
Command line : C:\Windows\SysWOW64\WerFault.exe -u -p 2724 -s 156
************************************************************************
WerFault.exe pid:    780
Command line : C:\Windows\SysWOW64\WerFault.exe -u -p 1388 -s 156
************************************************************************
WerFault.exe pid:   2168
```

Visiblement rien pour `2168`, en revanche `2716` et `780` on respectivement pour l'argument `-p` `2724` et `1388`, soit les deux `NOTEPAD.EXE` chelou en majuscule. J'ai l'impression que cette version de `notepad` n'est pas vraiment légitime, wait and see... En tout cas, j'imagine qu'on a trouvé la fameuse application qui crash au démarrage.

# Notepad - premier flag

Très bien, regardons maintenant les process `notepad`. De la même manière, on va essayer de regarder avec quels arguments les instances ont été ouvertes :

```bash
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f MemoryDump_Lab5.raw --profile=Win7SP1x64 cmdline -p 2744,2724,1388,2056
Volatility Foundation Volatility Framework 2.6.1
************************************************************************
notepad.exe pid:   2744
Command line : "C:\Windows\system32\notepad.exe" 
************************************************************************
NOTEPAD.EXE pid:   2724
Command line : "C:\Users\SmartNet\Videos\NOTEPAD.EXE" 
************************************************************************
NOTEPAD.EXE pid:   1388
************************************************************************
NOTEPAD.EXE pid:   2056
```

Tiens, le `NOTEPAD.EXE` chelou a été lancé depuis `C:\Users\SmartNet\Videos\`, à priori un exe n'a rien à faire là-dedans ! Ca confirme bien l'impression que ce process n'est pas légitime, on va essayer de le dumper et de faire un strings dessus pour y voir plus clair :

```bash
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f MemoryDump_Lab5.raw --profile=Win7SP1x64 procdump -p 2724 -D .
Volatility Foundation Volatility Framework 2.6.1
Process(V)         ImageBase          Name                 Result
------------------ ------------------ -------------------- ------
0xfffffa800108cb30 0x0000000001000000 NOTEPAD.EXE          OK: executable.2724.exe

```

Ok, `strings` ne donne rien d'intéressant, en revanche VirusTotal flag bien le fichier comme malicieux.

A ce stade, j'ai essayé tout ce qui me passait par la tête : regarder les shellbags, la MFT, un `memdump` sur `NOTEPAD.EXE` malicieux, mais rien d'intéressant. 

J'ai essayé le plugin `screenshot`, qui donne un wireframe des fenêtres ouvertes au moment du dump, et là quelque chose d'intéressant est apparu : une fenêtre du photo viewer était ouverte, avec un nom de fichier à rallonge !

![](https://raw.githubusercontent.com/sebescudie/sebescudie.github.io/master/static/images/blog/memlab_05/screenshot.png)

Je suppose qu'il s'agit d'une string en base64, et en commençant à la recopier dans Cyberchef, les premières lettres donnent l'output `flag` ... on est bien en présence d'un flag ! 

Comme j'avais la flemme de recopier la string à la main, j'ai cherché les premiers caractères dans un dump de la MFT que j'avais fait plus tôt, on trouve bien `ZmxhZ3shIV93M0xMX2QwbjNfU3Q0ZzMtMV8wZl9MNEJfM19EMG4zXyEhfQ`, ce qui dans Cyberchef nous donne `flag{!!_w3LL_d0n3_St4g3-1_0f_L4B_3_D0n3_!!}`. L'intro disait qu'il y avait une faute dans la string, on remplace donc `L4B_3` par `L4B_5`.

A ce stade, on ne sait toujours pas vraiment à quoi servait ce `NOTEPAD` vérolé, mais on a notre flag !


# WinRAR - second flag

Une fois encore, un `cmdline` pour voir ce qui s'est passé côté Winrar :

```bash
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f MemoryDump_Lab5.raw --profile=Win7SP1x64 cmdline -p 2924
Volatility Foundation Volatility Framework 2.6.1
************************************************************************
WinRAR.exe pid:   2924
Command line : "C:\Program Files\WinRAR\WinRAR.exe" "C:\Users\SmartNet\Documents\SW1wb3J0YW50.rar"
```

Tiens, un nom en apparence random, ça correspond avec ce que nous disait l'intro. Essayons de le récupérer :

```bash
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f MemoryDump_Lab5.raw --profile=Win7SP1x64 filescan | grep "SW1wb3J0YW50"       
Volatility Foundation Volatility Framework 2.6.1
0x000000003eed56f0      1      0 R--r-- \Device\HarddiskVolume2\Users\SmartNet\Documents\SW1wb3J0YW50.rar
                                                                                                                                         
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS]
└─$ python ~/volatility/volatility-2.6.1/vol.py -f MemoryDump_Lab5.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000003eed56f0 -D .
Volatility Foundation Volatility Framework 2.6.1
DataSectionObject 0x3eed56f0   None   \Device\HarddiskVolume2\Users\SmartNet\Documents\SW1wb3J0YW50.rar
```

Ok, au moment de décompresser l'archive, on voit qu'elle contient un fichier qui s'appelle `Stage2.png`, mais elle est bien sûr protégée par mot de passe. L'intro disait qu'il fallait nécessairement le premier flag pour trouver le second, on va donc l'essayer comme mot de passe : ça fonctionne ! On obtient l'image suivante :

![](https://raw.githubusercontent.com/sebescudie/sebescudie.github.io/master/static/images/blog/memlab_05/stage2_flag.png)

Ceci dit, l'intro nous disait que le challenge était composé de trois flags ... 

# Le bloc note malicieux - dernier flag

J'ai pas envie de lâcher l'affaire avec ce process. J'ai commencé (mais jamais terminé ...) [une room](https://tryhackme.com/room/introtox8664) sur [radare2](https://fr.wikipedia.org/wiki/Radare2) sur TryHackMe, on va donc voir si on peut désassembler le binaire...

Quelque commandes plus tard, on arrive ici :

```bash
┌──(kali㉿kali)-[~/Documents/ctf/MEMLABS/malicious_notepad_procdump]
└─$ r2 NOTEPAD.EXE
[0x0100739d]> aa
[x] Analyze all flags starting with sym. and entry0 (aa)
[0x0100739d]> afl
0x0100739d   15 518  -> 463  entry0
0x010012b4    5 28   -> 31   sym.imp.WINSPOOL.DRV_GetPrinterDriverW
[0x0100739d]> pdf @entry0
            ;-- eip:
┌ 463: entry0 ();
│           ; var int32_t var_38h_2 @ ebp-0x38
│           ; var int32_t var_34h_2 @ ebp-0x34
│           ; var int32_t var_30h_2 @ ebp-0x30
│           ; var int32_t var_2ch_2 @ ebp-0x2c
│           ; var int32_t var_24h_2 @ ebp-0x24
│           ; var int32_t var_20h_2 @ ebp-0x20
│           ; var int32_t var_1ch_2 @ ebp-0x1c
│           ; var int32_t var_18h @ ebp-0x18
│           ; var int32_t var_bp_10h @ ebp-0x10
│           ; var int32_t var_8h_2 @ ebp-0x8
│           ; var int32_t var_4h_2 @ ebp-0x4
│           ; var int32_t var_8h @ ebp+0x14
│           ; var int32_t var_38h @ ebp+0x34
│           ; var int32_t var_34h @ ebp+0x38
│           ; var int32_t var_30h @ ebp+0x3c
│           ; var int32_t var_2ch @ ebp+0x40
│           ; var int32_t var_24h @ ebp+0x48
│           ; var int32_t var_20h @ ebp+0x4c
│           ; var int32_t var_1ch @ ebp+0x50
│           ; var int32_t var_4h @ ebp+0x68
│           ; var int32_t var_10h @ esp+0x20
│           0x0100739d      6a70           push 0x70                   ; 'p' ; 112
│           0x0100739f      6898180001     push 0x1001898
│           0x010073a4      e8bf010000     call 0x1007568
│           0x010073a9      33db           xor ebx, ebx
│           0x010073ab      53             push ebx
│           0x010073ac      8b3dcc100001   mov edi, dword [sym.imp.KERNEL32.dll_GetModuleHandleA] ; [0x10010cc:4]=0x756a1245 reloc.KERNEL32.dll_GetModuleHandleA ; "E\x12ju"
│           0x010073b2      ffd7           call edi
│           0x010073b4      6681384d5a     cmp word [eax], 0x5a4d
│       ┌─< 0x010073b9      751f           jne 0x10073da
│       │   0x010073bb      8b483c         mov ecx, dword [eax + 0x3c]
│       │   0x010073be      03c8           add ecx, eax
│       │   0x010073c0      813950450000   cmp dword [ecx], 0x4550
│      ┌──< 0x010073c6      7512           jne 0x10073da
│      ││   0x010073c8      0fb74118       movzx eax, word [ecx + 0x18]
│      ││   0x010073cc      3d0b010000     cmp eax, 0x10b              ; 267
│     ┌───< 0x010073d1      741f           je 0x10073f2
│     │││   0x010073d3      3d0b020000     cmp eax, 0x20b              ; 523
│    ┌────< 0x010073d8      7405           je 0x10073df
│  ┌┌──└└─> 0x010073da      895de4         mov dword [var_1ch], ebx
│  ╎╎││ ┌─< 0x010073dd      eb27           jmp 0x1007406
│  ╎╎└────> 0x010073df      83b984000000.  cmp dword [ecx + 0x84], 0xe
│  └──────< 0x010073e6      76f2           jbe 0x10073da
│   ╎ │ │   0x010073e8      33c0           xor eax, eax
│   ╎ │ │   0x010073ea      3999f8000000   cmp dword [ecx + 0xf8], ebx
│   ╎ │┌──< 0x010073f0      eb0e           jmp 0x1007400
│   ╎ └───> 0x010073f2      8379740e       cmp dword [ecx + 0x74], 0xe
│   └─────< 0x010073f6      76e2           jbe 0x10073da
│      ││   0x010073f8      33c0           xor eax, eax
│      ││   0x010073fa      3999e8000000   cmp dword [ecx + 0xe8], ebx
│      ││   ; CODE XREF from entry0 @ 0x10073f0
│      └──> 0x01007400      0f95c0         setne al
│       │   0x01007403      8945e4         mov dword [var_1ch_2], eax
│       │   ; CODE XREF from entry0 @ 0x10073dd
│       └─> 0x01007406      895dfc         mov dword [var_4h], ebx
│           0x01007409      6a02           push 2                      ; 2
│           0x0100740b      ff1538130001   call dword [sym.imp.msvcrt.dll___set_app_type] ; 0x1001338
│           0x01007411      59             pop ecx
│           0x01007412      830d9cab0001.  or dword [0x100ab9c], 0xffffffff ; [0x100ab9c:4]=0
│           0x01007419      830da0ab0001.  or dword [0x100aba0], 0xffffffff ; [0x100aba0:4]=0
│           0x01007420      ff1534130001   call dword [sym.imp.msvcrt.dll___p__fmode] ; 0x1001334
│           0x01007426      8b0db89a0001   mov ecx, dword [0x1009ab8]  ; [0x1009ab8:4]=0
│           0x0100742c      8908           mov dword [eax], ecx
│           0x0100742e      ff1530130001   call dword [sym.imp.msvcrt.dll___p__commode] ; 0x1001330
│           0x01007434      8b0db49a0001   mov ecx, dword [0x1009ab4]  ; [0x1009ab4:4]=0
│           0x0100743a      8908           mov dword [eax], ecx
│           0x0100743c      a12c130001     mov eax, dword [sym.imp.msvcrt.dll__adjust_fdiv] ; [0x100132c:4]=0x753732ec reloc.msvcrt.dll__adjust_fdiv
│           0x01007441      8b00           mov eax, dword [eax]
│           0x01007443      a3a4ab0001     mov dword [0x100aba4], eax  ; [0x100aba4:4]=0
│           0x01007448      e8a7010000     call 0x10075f4
│           0x0100744d      391d08960001   cmp dword [0x1009608], ebx  ; [0x1009608:4]=1
│       ┌─< 0x01007453      750c           jne 0x1007461
│       │   0x01007455      68f4750001     push 0x10075f4
│       │   0x0100745a      ff1528130001   call dword [sym.imp.msvcrt.dll___setusermatherr] ; 0x1001328
│       │   0x01007460      59             pop ecx
│       └─> 0x01007461      e877010000     call 0x10075dd
│           0x01007466      6810900001     push 0x1009010
│           0x0100746b      680c900001     push 0x100900c
│           0x01007470      e85d010000     call 0x10075d2
│           0x01007475      a1b09a0001     mov eax, dword [0x1009ab0]  ; [0x1009ab0:4]=0
│           0x0100747a      8945dc         mov dword [var_24h], eax
│           0x0100747d      8d45dc         lea eax, [var_24h]
│           0x01007480      50             push eax
│           0x01007481      ff35ac9a0001   push dword [0x1009aac]
│           0x01007487      8d45d4         lea eax, [var_2ch]
│           0x0100748a      50             push eax
│           0x0100748b      8d45d0         lea eax, [var_30h]
│           0x0100748e      50             push eax
│           0x0100748f      8d45cc         lea eax, [var_34h]
│           0x01007492      50             push eax
│           0x01007493      ff1520130001   call dword [sym.imp.msvcrt.dll___getmainargs] ; 0x1001320
│           0x01007499      8945c8         mov dword [var_38h], eax
│           0x0100749c      6808900001     push 0x1009008
│           0x010074a1      6800900001     push section..data          ; 0x1009000
│           0x010074a6      e827010000     call 0x10075d2
│           0x010074ab      83c424         add esp, 0x24
│           0x010074ae      a11c130001     mov eax, dword [sym.imp.msvcrt.dll__acmdln] ; [0x100131c:4]=0x753704d8 reloc.msvcrt.dll__acmdln
│           0x010074b3      8b30           mov esi, dword [eax]
│           0x010074b5      8975e0         mov dword [var_20h], esi
│           0x010074b8      803e22         cmp byte [esi], 0x22
│           0x010074bb      b862000000     mov eax, 0x62               ; 'b' ; 98
│           0x010074c0      50             push eax
│           0x010074c1      b869000000     mov eax, 0x69               ; 'i' ; 105
│           0x010074c6      50             push eax
│           0x010074c7      b830000000     mov eax, 0x30               ; '0' ; 48
│           0x010074cc      50             push eax
│           0x010074cd      b873000000     mov eax, 0x73               ; 's' ; 115
│           0x010074d2      50             push eax
│           0x010074d3      bb7b000000     mov ebx, 0x7b               ; '{' ; 123
│           0x010074d8      53             push ebx
│           0x010074d9      b94d000000     mov ecx, 0x4d               ; 'M' ; 77
│           0x010074de      51             push ecx
│           0x010074df      bb33000000     mov ebx, 0x33               ; '3' ; 51
│           0x010074e4      53             push ebx
│           0x010074e5      b86d000000     mov eax, 0x6d               ; 'm' ; 109
│           0x010074ea      50             push eax
│           0x010074eb      b85f000000     mov eax, 0x5f               ; '_' ; 95
│           0x010074f0      50             push eax
│           0x010074f1      b86c000000     mov eax, 0x6c               ; 'l' ; 108
│           0x010074f6      50             push eax
│           0x010074f7      b834000000     mov eax, 0x34               ; '4' ; 52
│           0x010074fc      50             push eax
│           0x010074fd      b842000000     mov eax, 0x42               ; 'B' ; 66
│           0x01007502      50             push eax
│           0x01007503      b835000000     mov eax, 0x35               ; '5' ; 53
│           0x01007508      50             push eax
│           0x01007509      b85f000000     mov eax, 0x5f               ; '_' ; 95
│           0x0100750e      50             push eax
│           0x0100750f      b84f000000     mov eax, 0x4f               ; 'O' ; 79
│           0x01007514      50             push eax
│           0x01007515      b856000000     mov eax, 0x56               ; 'V' ; 86
│           0x0100751a      50             push eax
│           0x0100751b      b865000000     mov eax, 0x65               ; 'e' ; 101
│           0x01007520      50             push eax
│           0x01007521      bb52000000     mov ebx, 0x52               ; 'R' ; 82
│           0x01007526      53             push ebx
│           0x01007527      b85f000000     mov eax, 0x5f               ; '_' ; 95
│           0x0100752c      50             push eax
│           0x0100752d      b821000000     mov eax, 0x21               ; '!' ; 33
│           0x01007532      50             push eax
│           0x01007533      b87d000000     mov eax, 0x7d               ; '}' ; 125
│           0x01007538      50             push eax
│       ┌─< 0x01007539      753a           jne 0x1007575
│       │   0x0100753b      59             pop ecx
│       │   0x0100753c      59             pop ecx
│       │   0x0100753d      c3             ret
..
        │   ; CALL XREF from entry0 @ 0x10073a4
│       └─> 0x01007575      44             inc esp
│           0x01007576      2410           and al, 0x10                ; 16
│           0x01007578      896c2410       mov dword [var_10h], ebp
│           0x0100757c      8d6c2410       lea ebp, [var_10h]
│           0x01007580      2be0           sub esp, eax
│           0x01007582      53             push ebx
│           0x01007583      56             push esi
│           0x01007584      57             push edi
│           0x01007585      8b45f8         mov eax, dword [var_8h]
│           0x01007588      8965e8         mov dword [var_18h], esp
│           0x0100758b      50             push eax
│           0x0100758c      8b45fc         mov eax, dword [var_4h_2]
│           0x0100758f      c745fcffffff.  mov dword [var_4h_2], 0xffffffff ; -1
│           0x01007596      8945f8         mov dword [var_8h_2], eax
│           0x01007599      8d45f0         lea eax, [var_bp_10h]
│           0x0100759c      64a300000000   mov dword fs:[0], eax
└           0x010075a2      c3             ret
[0x0100739d]> 
```

On dirait bien qu'un flag se cache là-dedans !

`bi0s{M3m_l4B5_0VeR_!}` est donc notre dernier flag !

# Conclusion

Lab intéressant, j'ai pu apprendre pas mal de choses sur le fonctionnement de `WerFault`, et l'aspect pseudo-RE de la fin était marrant ! Ceci-dit, je trouve les indications un peu contre-intuitives, on s'atend à devoir chercher plusieurs fichiers, alors qu'en fait il n'en est rien.