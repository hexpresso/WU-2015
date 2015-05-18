# HackingWeek CTF 2015: Reverse 1

**Category:** reverse |
**Points:** 2322 |
**Solves:** 60 |
**Description:**


> Le clef est le mot de passe du programme Linux suivant:
> 
> * [crackme](http://hackingweek.fr/media/Aa0eiHuu/crackme-01)

___

## Write-up

* First step : Let's have a look with gdb.
```
=> 0x804849f:   cmp    ebx,0xd
   0x80484a2:   je     0x80484d2
```

Well, `0xd => 13` so the expected `len()` for the password is `12 + '\n'`.

* Second step : Give a 12-length password.

```
$ ltrace ./crackme-01
fwrite("Welcome to this crackme !!!\n\nPas"..., 1, 39, 0xb7728ac0Welcome to this crackme !!!
...
getline(0xbfb3cc28, 0xbfb3cc2c, 0xb7728c20, 0xb7728ac0Password: thegame_____
...
fprintf(0xb7728ac0, "thegame_____\n"thegame_____
...
strncmp("oy-aiG", "thegame_____\n", 6)
```
<br>
Okay, the first part is `oy-aiG`.

```
$ ltrace ./crackme-01 
fwrite("Welcome to this crackme !!!\n\nPas"..., 1, 39, 0xb777eac0Welcome to this crackme !!!
...
getline(0xbf97f418, 0xbf97f41c, 0xb777ec20, 0xb777eac0Password: oy-aiGthegam
...
fprintf(0xb777eac0, "oy-aiGthegam\n"oy-aiGthegam
...
strncmp("oy-aiG", "oy-aiGthegam\n", 6)
strncmp("thegam\n", "\\xu8Ae", 6)
```

The second part is compared with `\xu8Ae`.
The password must be `oy-aiG || \xu8Ae`.

Let's try this :

```
./crackme-01 
Welcome to this crackme !!!

Password: oy-aiG\xu8Ae
oy-aiG\xu8Ae
Excellent, you succeed !
```
<br>
__Flag__: `oy-aiG\xu8Ae`
<br>
Enjoy,<br>
\- [Notfound](http://www.notfound.ovh)
