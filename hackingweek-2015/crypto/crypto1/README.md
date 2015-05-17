# HackingWeek CTF 2015: Crypto 1

<style type="text/css">
    code {
        color: red;
    }
</style>


**Category:** Crypto |
**Points:** 5418 |
**Solves:** 24 |
**Description:** 


> [ADFGVX](http://fr.wikipedia.org/wiki/Chiffre_ADFGVX) était le chiffre utilisé par les Allemands pendant la première guerre mondiale.<br>
> Essayez de vous y frotter en déchiffrant ce message dont le clair est en français:
> 
> ```
> DADVDFDDXDDAXGDFFDXDDDADAXAAGAFGGDAXGADAVAAADVFXFDFDVGFGAAAVFAFGFA
> DADDVDGDVAXVDDVFFADGDGAXGFXXDFXDGGXFAADVGAXDVAVVAVAXDVADXADDDAXGXA
> AVADFVAAFXAGVGAAAFDDGDAGFADFVDAFXDDDDDDADDXXAFFDAGXAVFGFDXFDXD
> ```
> <br>
> _Indice_:<br>
> Le md5sum du mot de passe est <code>a50e38475041f76219748ee22c4377d4</code>.<br>
> Et le _Keysquare_ est: <code>cey1l0zrjshpmof385axib4u92vqwt6gd7nk</code>.<br>

___

## Write-up

This challenge was really fun. I have learn a lot about the ADFGVX encryption.

To solve this challenge, it was necessary to proceed in 2 steps.

* __First step__ : find the plaintext.

```python
#!/usr/bin/env python

from pycipher import ADFGVX
from itertools import permutations


def _possible(a, b):
    for i in range(len(b)):
        for j in range(len(b)):
            if b[i] == b[j]:
                if a[i] != a[j]:
                    return False
            if b[i] != b[j]:
                if a[i] == a[j]:
                    return False
    return True


def _cryptanalyse(cipher):
    if(max(list(cipher), key=lambda x: cipher.count(x)) == cipher[-1:]):
        return cipher
    return False


cipher = """
    DADVDFDDXDDAXGDFFDXDDDADAXAAGAFGGDAXGADAVAAADVFXFDFDVGFGAAAVFAFGFADADDV
    DGDVAXVDDVFFADGDGAXGFXXDFXDGGXFAADVGAXDVAVVAVAXDVADXADDDAXGXAAVADFVAAFXAGV
    GAAAFDDGDAGFADFVDAFXDDDDDDADDXXAFFDAGXAVFGFDXFDXD
    """
crib = "CETTEEPREUVE"

for p in permutations("ABCDEFGHIJ"):
    adfgvx = ADFGVX(key='cey1l0zrjshpmof385axib4u92vqwt6gd7nk',
                    keyword=''.join(p))
    d = adfgvx.decipher(cipher)
    if(_possible(d[len(d)-len(crib):], crib)):
        if(_cryptanalyse(d)):
            print(d + ' <- ' + ''.join(p))
```

So yes, I have guess the end of the plaintext. The word `EPREUVE` was given at a moment, and I guessed that probably be `CETTE EPREUVE`.<br>
That's why I have tried like that. And, fortunately, that was a good choice.

_Output_ : 
```
>>> python adf.py 
TROUVEZLACLEFDECHIFFREMENTDECEMESSAGEETVOUSAUREZLEMOTDEPASSEQUIVOUSPERMETTRADEVALIDERCETTEEPREUVE <- CGBADEFJHI
TR5XVEZC4CLEFDECHIFFREOCNTDCEEMESSAGE02VOUZBUREHCEMOVKEPASSEQUIVOUSR0RMETTRADY2ALIGYRCETTEEPREUVE <- CGBADJFEHI
```

At this point, I was surprised. The plaintext does not give the flag, but tell us to find the key... 
The first thing I did was to try the key I had found. But it failed.
Meanwhile, a hint was added (the `md5sum`).<br>

The second thing I do is to compare my key with the given md5sum.
```
md5(CGBADEFJHI) != a50e38475041f76219748ee22c4377d4
```

Fu. <br><br>

* __Second step__ : Bruteforce the key.

Well, now we have the plaintext and a possible key. 
Knowing the key, we can have the permutation.<br>

`CGBADEFJHI` => `4,3,1,5,6,7,2,9,10,8`<br>

With that, and a little cryptanalysis, it's easy to effectively reduce the bruteforce.
Indeed, if the first letter is a 'D', so the second letter can't be __BEFORE__ 'D', etc.

The permutation is very simple to understand, each letter get an index, and the key is sorted.
For example : 

Key | DABC
---:|:---:
Index | 1234
Permu | `2341`


That's all. Knowing that, it's easy to do a regex which match all the possible keys.<br>
I managed to have this regex :

`D[D-Z][BC][AB][A-Z]{3}`


This regex gave me all possible keys which match the permutation with a `len()` of 7.

Here is my python code :

 ```python
#!/usr/bin/env python

import itertools
import string

for k in itertools.product("bc", repeat=1):
    for l in itertools.product("defghijklmnopqrstuvwxyz", repeat=1):
        for j in itertools.product("ab", repeat=1):
            for i in itertools.product(string.lowercase, repeat=3):
                key = 'd' + ''.join(l) + ''.join(k) + ''.join(j) + ''.join(i)
                if ([i[0] for i in sorted(enumerate(key, 1), key=lambda x: x[1])] == [4, 3, 1, 5, 6, 7, 2]):
                    print(key)
```

I redirected the output into a file [possible_permut.txt](crypto1/possible_permut.txt) to keep them : 

```bash
>>> wc -l possible_permut.txt
37950 possible_permut.txt
```

So now, I just have to bruteforce the 3 last characters. 
With the statement, we passed to `26**10 = 1.4 x 10^14` to `37950 * 26**3 = 6.7 x 10^8`

Here is my final code : 

```python
#!/usr/bin/env python
 
from itertools import product
import hashlib
import string


def _bf_md5(fd):
    for word in fd:
        for i in product(charset, repeat=3):
            key = word.strip() + ''.join(i)
            if hashlib.md5(key).hexdigest() == 'a50e38475041f76219748ee22c4377d4':
                return key

f = open("possible_permut.txt").readlines()
charset = string.lowercase
print("Flag is : %s" % _bf_md5(f))

```

_Output_ :

```
>>> python bf_with_permu.py 
Flag is : docadeitoo
```
<br>
Pwned. <br>
__Flag__ : docadeitoo

Thanks to [XeR](https://twitter.com/XeR_0x2A) for answering my questions.

Enjoy,<br>
\- [Notfound](http://www.notfound.ovh)
