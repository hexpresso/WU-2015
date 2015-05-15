# Nuit du Hack Quals CTF 2015: Game Of Life

**Category:** Crypto
**Points:** 150
**Solves:** 178
**Description:** 

> "We're born alone, we live alone, we die alone. Only through our love and friendship can we create the illusion for the moment that we're not alone." (Orson Orwell)
> 
> Cells cells cells, the basis of life. Don't let them die and tell us their secret.
> 
> [http://static.challs.nuitduhack.com/GOL.tar.gz](GOL.tar.gz)

## Write-up

I wrote this writeup because I have seen some guys doing this challenge by using XORTOOL, but without understanding …

> mister X | python xortool.py cipher.txt; cat xortool_out/000.out
> mister X | So hardcore

That’s why I’ll try to give you a real explaination of « why xortool have worked »
So, for this challenge, we were given a .tar.gz.

```sh
$ ls
GOL.tar.gz
$ tar zxvf GOL.tar.gz
liv_GOL/
liv_GOL/consignes
liv_GOL/jdlv.py
liv_GOL/cipher.txt
```

Well, we had a setpoint, a cipher.txt and a python script.

The setpoint learned us this :

```sh
$ cat consignes
[ Game of life ]
[+] The above text has been encoded using the game of life rules on a 8x8 array.
```

Seems legit, the given ciphertext has been encoded with the python script.

Let’s have a quick look on the ciphertext :

```
11000100
00010000
01000111
…
}T]Q_YVTBDUUEECQSUDB:qEDTEB
\_\RBUTETBPW_^:jX^U
```

Okay, a range of 8-grouped bits, one byte per line so.
And after, this encrypted message.

At this point, we looked the program coded in python.
Functions :

```python
def creerGrille():
def genKey(key):
def initGrille(grille,seed):
def tourSuivant(grille):
def genBitstream(grille,key):
def xor(ent1,ent2):
def wrapper():
```

The first function called is the wrapper().

```python
def wrapper():
    key=sys.argv[1]
    fichier=open(sys.argv[2],'rb')
    encfile=''
    bitstream=''
    grille=initGrille(creerGrille(),genKey(key))
    for i in fichier.readlines():
        bitstream=genBitstream(grille,key)
        encfile+=xor(i,bitstream)
        tourSuivant(grille)
    print encfile
```

Well, this function do :
* initialize the variable « key » with the first argument
* open the file given in the second argument
* create a grid
* for each line, generate a bitstream with the grid and the key
* encrypt the line using XOR and the bitstream
* print the encrypted file

So, the main point is about the XOR encryption.
Firstly, XOR is reversible and it’s not a good encryption even if the key has the same length of the plain. To clarify

    (I had some echoes of people do not agree with my statements. To clarify the situation, and angry people who said that I need the review the basic [maybe true], XOR is NOT SECURE except in special circumstances, like OTP + size(key) >= size(plaintext). Without this, XOR is NOT SECURE, particularly when it used in a CTF hahaha (often repeated small key) … But, the discussion is open and I’m not mean, instead of thrashing me on IRC on channel where I’m not present … :p )

Secondly, the bitstream is init by the genBistream function, let’s have a look :

```python
def genBitstream(grille, key):
    bitstream = ''
    for j in range(8):
        bitstream += grille[j][7]
    print bitstream
    return bitstream
```

(Lol, the « key » is not used …)

So, the plaintext is XORed by a bitstream, looks like (example) :

```
01010101 ^ line1
00001111 ^ line2
...
nnnnnnnn ^ linen
```

Here, we see that the challenge is fuckedup. Why ?
Because, in the ciphertext, the bistream is given. So, we though to use the bistream to decrypt (not tried). But if you look closer, you can see that the bitstream (generate by the game of life) is spotted very early … The key used by the creator has completly break the challenge ..

The given bitstream was :

```
11000100
00010000
01000111
...
11001100
00000001
10000000
00000000
00000000
00000000
*
00000000
```

That’s to say, at the start, we must have the same key but after, noneed.

Try to explain with an example by printing the bitstream :

```sh
$ cat file
you loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou loseyou
...
you lose
```

```sh
$ python pouet.py key_fuck_gol__ file
10000110
11000000
...
01000000 <-- END OF GOL
...
00000000
00000000
*
00000000  <-- THE END
```

Ok, the key given in example is good. It stop the GOL early.

```sh
$ python pouet.py key_fuck_gol__ file > enc
$ python pouet.py $RANDOM enc
...
you lose
```

HO YEAH. So, by giving a RANDOM key to the program and having a short GOL, we are able to decrypt.

Let’s try with the given ciphertext :

```sh
$ python pouet.py $RANDOM cipher.txt
...
A bientôt peut-être sur un toit ou dans une autre vie.
 
Flag : ToBeAndToLast
```

PWNED :)


To go further, if RANDOM gave us a bad key which make lonk GOL :

```sh
KEY=$RANDOM ; python pouet.py $KEY cipher.txt ; echo $KEY
Fmag!: UoBdAndTnLart
12024
```

So, 12024 do not decrypt the ciphertext properly.
For fun, we check his GOL :

```
11011100 <-- START
01010000
11010000
11100000
...
01010000
11010100
00101110 <-- END
```

Indeed, the key generate the GOL which is long enough.

Thanks to this challenge, Notfound makes the firstblood of the quals.

![Alt text](http://i.imgur.com/PimPN0B.png "Output")
![Alt text](https://i2.wp.com/bilder.hifi-forum.de/max/390852/bitch-please-kopie-300x238_175397.jpg "Output")

Enjoy,
[Notfound](http://www.notfound.ovh)
