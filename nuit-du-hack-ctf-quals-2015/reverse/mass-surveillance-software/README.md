# Nuit du Hack Quals CTF 2015: Mass Surveillance Software

**Category:** Crackme/Reverse
**Points:** 300
**Solves:** 27
**Description:** 

> CIA : "Our friends at the NSA had a cool song about their capabilities. So we decided to make our own software."
> 
> Here is the song: "Every mail you send. Every call you make. Every file you save. Every thought you save. I'm recording you.
> 
> With anyone you play. anywhere you stay. any place you check. any move you make. I'll be watching you. Oh can't you see. you belong to me. no more privacy. R.I.P democracy.
> 
> Every doc you leak. Every link you click. Every fight you take. Every step you make. We'll be watching you.
> 
> Since you're online i save every trace. You think you're hidden but i can see your face. You look around but you're doomed we're in thge place. You feel so cold and i love to feel your stress. I'm recording, recording, so baby, cheese.
> 
> Oh can't you see. You belong to me. No more privacy. R.I.P. Democracy. Every single word. Picture of your friends. And friends of your friends. All around the world.
> 
> We will spy on you. We will spy on you. We will spy on you. We will spy on you. We will spy on you. We will spy on you. We will spy on you."
> 
> ![](https://www.youtube.com/watch?v=AwWgbxXU5T0)
> 
> [http://static.challs.nuitduhack.com/mass.tar.gz](mass.tar.gz)

___

## Write-up

For this funny challenge, a .exe was given.

- In the first place, we check for packers by using Protection iD :

![Alt text](https://hexpresso.files.wordpress.com/2015/04/re300_1.png)

<br>
Actually we don’t need to unpack it as we’ll just attach Ollydbg to the process.<br>
Then we put a memory breapoint at 0x400000, then just move the window to trigger the breakpoint. (Do not forget to remove the breakpoint then.)<br>
We land in the middle of our target, we can see an interesting call by just scrolling up :

![Alt text](https://hexpresso.files.wordpress.com/2015/04/re300_2.png?w=768)
<br><br>

- Everywhere in the crackme, there is this trick :

![Alt text](https://hexpresso.files.wordpress.com/2015/04/re300_31.png)
![Alt text](https://hexpresso.files.wordpress.com/2015/04/re300_4.png)
<br><br>

You just need to press F7 to get the clear assembly.<br>
You’ll see ecx, eax and edx being set to some values :

![Alt text](https://hexpresso.files.wordpress.com/2015/04/re300_6.png)
<br><br>

- Then two loops :
- Loop for 0x401000 function :

![Alt text](https://hexpresso.files.wordpress.com/2015/04/re300_5.png)
<br><br>

Then the result is moved to ebx.

- Loop for 0x40116E function :

![Alt text](https://hexpresso.files.wordpress.com/2015/04/re300_7.png)
<br><br>

Then the result is moved to ecx.

- We get some conditional jumps :
![Alt text](https://hexpresso.files.wordpress.com/2015/04/re300_8.png)
<br><br>

So, now we know that :

– The serial is 0xe characters.
– Start with M
– ‘d’ in 8th
– 6 characters which gave use 0x8AD3629B through 0x401000
– 6 characters which gave use 0x25B978C2 through 0x40116e
<br><br>


Let’s BF our two missing parts:

__0x401000__ :

```C
#include <stdio.h>
#include <stdlib.h>
 
 
unsigned long lrotl(unsigned long value,unsigned char rotation){
    return (value << rotation) | (value >> (32 - rotation));
}
unsigned long lrotrl(unsigned long value,unsigned char rotation){
    return (value >> rotation) | (value << (32 - rotation));
}
 
 
 
int main()
{
    int sortir = 0;
    char chaine[10] = {0};
    while(!sortir)
    {
        int ecx = 5;
        unsigned int eax = 0xC1AC0FF3;
        unsigned int edx = 0xBADB0155;
         
        int read = fread(chaine,1,7,stdin);
        if(read != 7)
            sortir = 1;
        for(; ecx >= 0 ; ecx--)
        {
            unsigned char al =  eax amp; 0xFF;
            al ^= *(unsigned char*)(chaine+ecx);
            int tempeax = eax amp; 0xFFFFFF00;
            eax = tempeax + al; // XOR AL,BYTE PTR DS:[ECX+403009]
            eax = lrotl(eax,7); // ROL eax,7
            eax *= edx;
            eax =~ eax;
            eax *= edx;
            eax = lrotrl(eax,2);
        }
        if(eax == 0x8AD3629B)
            printf("%s",chaine);
    }
 
 
}
```
<br><br>


__0x40116E__ :

```C
#include <stdio.h>
#include <stdlib.h>
 

unsigned long lrotl(unsigned long value,unsigned char rotation){
    return (value << rotation) | (value >> (32 - rotation));
}
unsigned long lrotrl(unsigned long value,unsigned char rotation){
    return (value >> rotation) | (value << (32 - rotation));
}
 
 
int main()
{
 
    int sortir = 0;
    char chaine[10] = {0};
    while(!sortir)
    {
        int ecx = 5;
        unsigned int eax = 0x17A5F4CC;
        unsigned int edx = 0x0B194553;
         
        int read = fread(chaine,1,7,stdin);
        if(read != 7)
            sortir = 1;
        for(; ecx >= 0 ; ecx--)
        {
            unsigned char al =  eax amp; 0xFF;
            al ^= *(unsigned char*)(chaine+ecx);
            int tempeax = eax amp; 0xFFFFFF00;
            eax = tempeax + al;
            eax = lrotrl(eax,2);
            eax *= edx;
            eax =~ eax;
            eax = lrotl(eax,7);
        }
        if(eax == 0x25B978C2)
            printf("%s",chaine);
    }
 
 
 
}
```
<br>


We use [http://sourceforge.net/projects/crunch-wordlist/](http://sourceforge.net/projects/crunch-wordlist/) to generate combinations :

```bash
$ ./crunch 6 6 abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ| ./soluce
bqXVdd
einHan <=
eGsZCm
fSYDes
tJluIA
wLkbna
DaJnzm
[...]
```
<br>

We have selected this collision among others because it makes `MeinHand`.

Handy is a cellphone and it was making sense with the challenge statement.<br>
So we can get a more optimized bruteforce for the second part :

```bash
$ ./crunch 6 6 -t y,@@@@ | ./soluce2
yKaput
```
<br>

That gives us the valid serial MeinHandyKaput

__Flag:__ MeinHandyKaput

<br>

Enjoy.
— [Tishrom](https://twitter.com/Tishrom)
