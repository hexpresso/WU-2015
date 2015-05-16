# Nuit du Hack Quals CTF 2015: Superman

**Category:** Crackme/Reverse
**Points:** 500
**Solves:** 49
**Description:** 

> "You will give the people of Earth an ideal to strive towards. They will race behind you, they will stumble, they will fall. But in time, they will join you in the sun, Kal. In time, you will help them accomplish wonders." (Jor-El)
> 
> Use your reversing super-powers to uncover the flag, be a superman.
> 
> [http://static.challs.nuitduhack.com/superman.tar.gz](superman.tar.gz)

___

## Write-up


This task is a Crackme/Reverse task worth 500 points from the Nuit du Hack qualifications.

We were given an ELF :

```
superman: ELF 32-bit LSB executable, Intel 80386, invalid version (SYSV), for GNU/Linux 2.6.24, dynamically linked, interpreter 04, corrupted section header size
```


This task is very similar to the Clark Kent. Except that there is a LOT of instructions that seems to be important on the first place.<br>
RageYL found that those instructions are nothing but a huge neg of a whole chunk of the memory.

By checking the imports, we can see that it makes a call to ptrace.<br>
This function is often used to check for debuggers.<br>
So we started looking around it.

```
LOAD:080487EE                 call    ptrace
LOAD:080487F3                 cmp     eax, 0FFFFFFFFh
LOAD:080487F8                 jnz     loc_8048823
LOAD:080487FE                 lea     eax, aBoohDonTDebugM ; "Booh! Don't debug me!\n"
LOAD:08048804                 mov     [esp], eax      ; format
LOAD:08048807                 call    printf
LOAD:0804880C                 mov     ecx, 1
LOAD:08048811                 mov     dword ptr [esp], 1 ; status
LOAD:08048818                 mov     [ebp+var_34], eax
LOAD:0804881B                 mov     [ebp+var_38], ecx
LOAD:0804881E                 call    exit
LOAD:08048823 ; -------------------------------------------------------------
LOAD:08048823
LOAD:08048823 loc_8048823:                            ; CODE XREF: sub_80487A0+58j
LOAD:08048823                 lea     eax, aWelcomeToNdh2k ; "Welcome to NDH2k15\n"
LOAD:08048829                 mov     [esp], eax      ; format
LOAD:0804882C                 call    printf
```


However, we cannot patch it, as there is a routine that checks for alterations.<br>
We cannot put a regular breakpoint as well, since putting a breakpoint means overwriting a code with instruction INT3 (0xCC).<br>
Therefore, we decided to put a hardware breakpoint : this kind of breakpoint doesn’t overwrite the code.

Using IDA, (and gdbserver), we can set an hardware breakpoint that does not break, but sets eax to 0.
It saves us time, since we don’t have to edit eax, and press F9.

![Alt text](http://i.imgur.com/YQvvo8V.png)<br>
_Setting a hardware breakpoint with IDA_

We also put a breakpoint to :

```
LOAD:0804913F                 call    [ebp+addr]
```

to keep debugging the program.

It branches to a routine that starts by checking our uid.<br>
If uid is not 0, it exits and asks for super user power.

![Alt text](https://i0.wp.com/gifatron.com/wp-content/uploads/2013/03/RRnhhqW.gif) 
_if(getuid()) exit();_

Of course, instead of runing as root, we used the same trick that we used earlier to bypass the uid0 check.<br>
A few lines under, there is a reboot syscall. Only fools would run a crackme as root, right ? ;)

```
<Mastho> désolé étape 2, hbreak sur le call 
<Mastho> et tu vires le syscall reboot
<Mastho> tu le lances en root le process ? 
<XeR> J'étais en user
<Mastho> k
<XeR> Donc j'ai mis eax à 0 après le getuid
<Mastho> moi j'ai testé les deux
<Mastho> ;)
<XeR> Et au niveau du xor esi, esi, j'ai sauté après le reboot
<Mastho> (et j'ai fail une fois, ça m'a reboot)
```


It is important to skip the whole reboot block :

```
MEMORY:08077080 xor     esi, esi        ; magic4
MEMORY:08077082 mov     edx, offset unk_1234567 ; magic3
MEMORY:08077087 mov     ecx, offset unk_28121969 ; magic2
MEMORY:0807708C mov     ebx, offset unk_FEE1DEAD ; magic1
MEMORY:08077091 mov     eax, 58h ; 'X'
MEMORY:08077096 int     80h             ; LINUX - sys_reboot
```


Otherwise, the xor esi, esi instruction will make the program to segfault.
We used even more IDA magic to skip over it, and break at the same time.

![Alt text](http://i.imgur.com/0Loi69Y.png)
_IDA is love, IDA is life_

The crackme reads user input later with a read syscall. We can safely ignore every function calls before it.<br>
However, the functions after are important, since they may alter/read the user input.<br>
It calls memset twice, to clean two differents buffers (that we will call buffer1, and buffer2).<br>
The next function is short. It is a `for()` loop, with 2 conditions on i : `for(i = 0; i < a && i < b; i++)`<br>
It will in fact decipher a 3rd buffer, that we will refer as « opcodes ». (He he he.)<br>
The next function is a virtual machine.<br>
By checking quickly at the comparisions it makes, we figured quickly that it was in fact a BrainFuck VM.<br>

![Alt text](http://i.imgur.com/BGUep6M.png)
_Cthulhu.png_

There are two kind of VM : those who checks the user input, and those who generate a flag.<br>
This one is of the second kind : it generates a flag from opcode, and put it in buffer1, and buffer2.<br>
Thankfully, we checked it before digging inside the VM. (Step over it, and check the arguments)

```
MEMORY:FFFFCC98                 db  4Fh ; O
MEMORY:FFFFCC9C                 db  4Dh ; M
MEMORY:FFFFCCA0                 db  47h ; G
MEMORY:FFFFCCA4                 db  54h ; T
MEMORY:FFFFCCA8                 db  68h ; h
MEMORY:FFFFCCAC                 db  34h ; 4
MEMORY:FFFFCCB0                 db  74h ; t
MEMORY:FFFFCCB4                 db  57h ; W
MEMORY:FFFFCCB8                 db  34h ; 4
MEMORY:FFFFCCBC                 db  73h ; s
MEMORY:FFFFCCC0                 db  34h ; 4
MEMORY:FFFFCCC4                 db  48h ; H
MEMORY:FFFFCCC8                 db  30h ; 0
MEMORY:FFFFCCCC                 db  72h ; r
MEMORY:FFFFCCD0                 db  72h ; r
MEMORY:FFFFCCD4                 db  31h ; 1
MEMORY:FFFFCCD8                 db  62h ; b
MEMORY:FFFFCCDC                 db  6Ch ; l
MEMORY:FFFFCCE0                 db  33h ; 3
MEMORY:FFFFCCE4                 db  43h ; C
MEMORY:FFFFCCE8                 db  68h ; h
MEMORY:FFFFCCEC                 db  34h ; 4
MEMORY:FFFFCCF0                 db  6Ch ; l
MEMORY:FFFFCCF4                 db  6Ch ; l
MEMORY:FFFFCCF8                 db  33h ; 3
MEMORY:FFFFCCFC                 db  6Eh ; n
MEMORY:FFFFCD00                 db  67h ; g
MEMORY:FFFFCD04                 db  33h ; 3
MEMORY:FFFFCD08                 db  21h ; !
```

__Flag:__ OMGTh4tW4s4H0rr1bl3Ch4ll3ng3!

Enjoy.

Thanks to
RageYL for helping me through this task. :)<br>
— [XeR](https://twitter.com/XeR_0x2A)
