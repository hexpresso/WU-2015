# Nuit du Hack Quals CTF 2015: Crackme Prime

**Category:** Crackme/Reverse
**Points:** 150
**Solves:** 90
**Description:** 

> "I am Optimus Prime, and I send this message to any surviving Autobots taking refuge among the stars. We are here, we are waiting."
> 
> Keygen me, I'm the Prime.
> 
> Validate your serial here : <http://crackmeprime.challs.nuitduhack.com/>
> 
> [http://static.challs.nuitduhack.com/prime.tar.gz](prime.tar.gz)

___

## Write-up

This task is a Crackme/Reverse task worth 150 points from the Nuit du Hack qualifications.
We are given an ELF :

```
crackme: ELF 32-bit LSB executable, Intel 80386, version 1 (GNU/Linux), statically linked, for GNU/Linux 2.6.32, BuildID[sha1]=0x92d632c664b683dc98873fe1c785d1e6928e7272, not stripped
```

This task was not particularly hard, but we solved it in an interesting way.<br>
As usual, we start by opening the crackme in IDA.<br>
We quickly notice that it expects a serial number in argv.<br>
There is a strlen instruction right after :

```asm
.text:08048E2A                 push    eax
.text:08048E2B                 call    strlen
.text:08048E30                 add     esp, 10h
.text:08048E33                 cmp     eax, 29
.text:08048E36                 jnz     badboy_format
```


If the serial’s length is not 29, we jump to an error.

```asm
.text:08048E47                 push    '0'             ; c
.text:08048E49                 push    eax             ; s
.text:08048E4A                 call    _strchr
.text:08048E4F                 add     esp, 10h
.text:08048E52                 test    eax, eax
.text:08048E54                 jz      short goodboy_noZeroes
```

We can see that it expects the serial to have no zeroes in it.

There is a huge block with memsets, strncpy, strtol, and « c1″ (a custom function).<br>
By looking at the strncpy, we realize that the format is supposed to be formated this way :

`([0-9a-fA-F]{4}.){5}[0-9a-fA-F]{4}`

(Understand : 6 blocs of hex shorts, separated by any char, because strtol’s 3rd parameter is base 16.)

The c1 function is a custom function that uses AES.<br>
It seems to decrypt a routine, and jump to it, since there is a call eax after the AES calls.<br>
By setting a breakpoint on the call eax, we can see what the function was :

```asm
[heap]:093664B0 ; =============== S U B R O U T I N E =======================================
[heap]:093664B0
[heap]:093664B0 ; Attributes: bp-based frame
[heap]:093664B0
[heap]:093664B0 isPrime proc near
[heap]:093664B0
[heap]:093664B0 divisors= dword ptr -8
[heap]:093664B0 i= dword ptr -4
[heap]:093664B0 n= dword ptr  8
[heap]:093664B0
[heap]:093664B0 push    ebp
[heap]:093664B1 mov     ebp, esp
[heap]:093664B3 sub     esp, 10h
[heap]:093664B6 mov     [ebp+divisors], 0
[heap]:093664BD mov     [ebp+i], 1
[heap]:093664C4 jmp     short loc_93664E3
[heap]:093664C6 ; ---------------------------------------------------------------------------
[heap]:093664C6
[heap]:093664C6 loc_93664C6:                            ; CODE XREF: isPrime+39j
[heap]:093664C6 mov     eax, [ebp+n]
[heap]:093664C9 cdq
[heap]:093664CA idiv    [ebp+i]
[heap]:093664CD mov     eax, edx
[heap]:093664CF test    eax, eax
[heap]:093664D1 jnz     short loc_93664DF
[heap]:093664D3 add     [ebp+divisors], 1
[heap]:093664D7 cmp     [ebp+divisors], 2
[heap]:093664DB jle     short loc_93664DF
[heap]:093664DD jmp     short loc_93664EB
[heap]:093664DF ; ---------------------------------------------------------------------------
[heap]:093664DF
[heap]:093664DF loc_93664DF:                            ; CODE XREF: isPrime+21j
[heap]:093664DF                                         ; isPrime+2Bj
[heap]:093664DF add     [ebp+i], 1
[heap]:093664E3
[heap]:093664E3 loc_93664E3:                            ; CODE XREF: isPrime+14j
[heap]:093664E3 mov     eax, [ebp+i]
[heap]:093664E6 cmp     eax, [ebp+n]
[heap]:093664E9 jle     short loc_93664C6
[heap]:093664EB
[heap]:093664EB loc_93664EB:                            ; CODE XREF: isPrime+2Dj
[heap]:093664EB cmp     [ebp+divisors], 2
[heap]:093664EF jnz     short loc_93664F8
[heap]:093664F1 mov     eax, 1
[heap]:093664F6 jmp     short locret_93664FD
[heap]:093664F8 ; ---------------------------------------------------------------------------
[heap]:093664F8
[heap]:093664F8 loc_93664F8:                            ; CODE XREF: isPrime+3Fj
[heap]:093664F8 mov     eax, 0
[heap]:093664FD
[heap]:093664FD locret_93664FD:                         ; CODE XREF: isPrime+46j
[heap]:093664FD leave
[heap]:093664FE retn
[heap]:093664FE isPrime endp
[heap]:093664FE
[heap]:093664FE ; ---------------------------------------------------------------------------
```


It checks how many numbers can divide n, from 1 to n.<br>
It is used to check the primality of a number : if a number n can be divided by only 2 (1, n) numbers, it is prime.<br>
The crackme uses this function to check if every hex numbers are prime.<br>
It then adds the 5 first chunks with each other, and check it is still prime modulo the 6th chunk.

Summary :

– Length = 29
– No 0 in the string
– 6 chunks of short hex (4 chars), separated by any char
– Every chunk is a prime number
– (c1 + c2 + c3 + c4 + c5) % c6 is prime.


Most teams wrote a bruteforce script. We used our secret weapon :
![Alt text](https://i0.wp.com/s15.postimg.org/4se3gog23/36_standback.png)
_Stand back, I’m going to try science !_

There are couple of prime numbers that we call `twin primes`.<br>
They are somewhat special, because (p1, p2) are prime, and `p2 – p1 == 2`, which is prime as well.<br>
Also, `(n + k) % n = k % n`.<br>
So, `(n + n + n + n + k) % n = k % n`. Do you see where we’re getting now ?<br>
All we had to do is to find twin primes with no 0 in it.<br>
We took a list of the first prime numbers, until 65537 (RSA, anyone ?), and looked for twin numbers.<br>
We found (65447, 65449) or (0xFFA7, 0xFFA9) in hex.<br>
We tried to input the serial FFA7-FFA7-FFA7-FFA7-FFA9-FFA7, and… bingo !<br>
The page [http://crackmeprime.challs.nuitduhack.com/validate](http://crackmeprime.challs.nuitduhack.com/validate) traded us our serial for the flag.

Congratulation! The flag is : WowThatWasEasyAES

__Flag:__ WowThatWasEasyAES

— [XeR](https://twitter.com/XeR_0x2A)
