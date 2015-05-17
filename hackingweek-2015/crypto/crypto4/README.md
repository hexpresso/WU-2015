# HackingWeek CTF 2015: Crypto4

**Category:** Crypto |
**Points:** 6536 |
**Solves:** 11 |
**Description:**

> ```
> Session Start: Thu Feb 05 20:49:04 2015
> Session Ident: #mastercsi
> 03[20:49] * Now talking in #mastercsi
> 03[20:49] * Topic is 'http://mastercsi.labri.fr/'
> 03[20:49] * Set by admin!~admin on Sat Nov 22 00:06:50
> 01[20:49]  et j'ai chopé une vieille clé rsa qu'alice utilisait
> 01[20:49]  alice, la alice? tu te fous de moi ?
> 01[20:49]  haha non
> 01[20:49]  mais y'avait juste la moitié, j'ai dû la compléter
>            avec des valeurs au pif pour la faire marcher
> 01[20:49]  ça a l'air de fonctionner quand même en tout cas,
>            si ta un truc à elle à déchiffrer...
> 01[20:49]  attend j'ai sa clé publique qui traîne quelque part,
>            et même un fichier chiffré avec. me suis toujours
>            demandé ce que c'était... 
> 01[20:49]  peut-être que c'est la même clé ?
> 01[20:50]  je t'ai envoyé le truc, jette un coup d'oeil
> 02[21:22] * Disconnected
> Session Close: Thu Feb 05 21:22:11 2015
> ```
>
> La clef de validation est le contenu du message chiffré avec la clef privée d'Alice, reconstituez le en utilisant les fichiers suivants:
> 
> * [`alice.pub`](http://hackingweek.fr/media/Aa0eiHuu/alice.pub)
> * [`mykey.pem`](http://hackingweek.fr/media/Aa0eiHuu/mykey.pem)
> * [`secret`](http://hackingweek.fr/media/Aa0eiHuu/secret)

___

## Write-up

For this task, we were given 3 files.

- _Alice's public key_ :

```
>>> openssl rsa -pubin -in alice.pub -text -noout -modulus
Public-Key: (1024 bit)
Modulus:
    00:c6:c8:35:29:a2:38:8f:14:63:65:c5:f5:fd:4b:
    0d:88:89:61:b9:5d:e1:0f:fa:88:53:a3:c2:cb:ed:
    75:0e:99:59:bd:0f:f8:72:c2:23:2f:6b:ad:32:62:
    4f:35:6a:82:d0:62:75:5e:1e:4f:ed:ae:54:e8:ca:
    24:71:fc:8d:13:ac:70:0e:e2:57:20:d4:d9:08:9f:
    d6:fb:d4:2f:12:e6:a4:1e:1c:1d:e8:1f:57:8c:32:
    13:2a:d0:85:94:e8:51:84:1d:02:39:cd:41:0d:ef:
    11:d1:c1:5e:e7:5b:92:f8:6a:04:f7:c6:c7:f3:6b:
    90:46:b8:fb:2f:e2:95:65:b1
Exponent: 65537 (0x10001)
Modulus=C6C83529A2388F146365C5F5FD4B0D888961B95DE10FFA8853A3C2CBED750E9959BD0FF872C2232F6BAD32624F356A82D062755E1E4FEDAE54E8CA2471FC8D13AC700EE25720D4D9089FD6FBD42F12E6A41E1C1DE81F578C32132AD08594E851841D0239CD410DEF11D1C15EE75B92F86A04F7C6C7F36B9046B8FB2FE29565B1
```
<br>
So we have the exponent `e` and the modulus `N`.

- _mykey.pem_ :

```
>>> openssl rsa -in mykey.pem -text -noout -modulus
Private-Key: (1024 bit)
modulus:
    00:c6:c8:35:29:a2:38:8f:14:63:65:c5:f5:fd:4b:
    0d:88:89:61:b9:5d:e1:0f:fa:88:53:a3:c2:cb:ed:
    75:0e:99:59:bd:0f:f8:72:c2:23:2f:6b:ad:32:62:
    4f:35:6a:82:d0:62:75:5e:1e:4f:ed:ae:54:e8:ca:
    24:71:fc:8d:13:ac:70:0e:e2:57:20:d4:d9:08:9f:
    d6:fb:d4:2f:12:e6:a4:1e:1c:1d:e8:1f:57:8c:32:
    13:2a:d0:85:94:e8:51:84:1d:02:39:cd:41:0d:ef:
    11:d1:c1:5e:e7:5b:92:f8:6a:04:f7:c6:c7:f3:6b:
    90:46:b8:fb:2f:e2:95:65:b1
publicExponent: 3 (0x3)
privateExponent:
    00:84:85:78:c6:6c:25:b4:b8:42:43:d9:4e:a8:dc:
    b3:b0:5b:96:7b:93:eb:5f:fc:5a:e2:6d:2c:87:f3:
    a3:5f:10:e6:7e:0a:a5:a1:d6:c2:1f:9d:1e:21:96:
    df:78:f1:ac:8a:ec:4e:3e:be:df:f3:c9:8d:f0:86:
    c2:f6:a8:5e:0b:ef:c0:ca:19:c5:e2:49:55:49:fe:
    e5:2e:51:3e:7b:e9:f2:22:07:d2:4b:84:7f:bb:0c:
    b5:ba:b7:95:c6:90:05:3e:65:2d:11:53:9a:2d:96:
    0f:ea:de:cb:9b:17:54:87:00:0f:78:12:ce:ac:f5:
    db:83:30:16:06:cc:35:7d:a3
prime1: 245 (0xf5)
prime2: 207 (0xcf)
exponent1: 163 (0xa3)
exponent2: 138 (0x8a)
coefficient: 189 (0xbd)
Modulus=C6C83529A2388F146365C5F5FD4B0D888961B95DE10FFA8853A3C2CBED750E9959BD0FF872C2232F6BAD32624F356A82D062755E1E4FEDAE54E8CA2471FC8D13AC700EE25720D4D9089FD6FBD42F12E6A41E1C1DE81F578C32132AD08594E851841D0239CD410DEF11D1C15EE75B92F86A04F7C6C7F36B9046B8FB2FE29565B1
```
<br>
Here, the most important thing is that the privateExponent `d` is given.

```
>>> 0x00848578c66c25b4b84243d94ea8dcb3b05b967b93eb5ffc5ae26d2c87f3a35f10e67e0aa5a1d6c21f9d1e2196df78f1ac8aec4e3ebedff3c98df086c2f6a85e0befc0ca19c5e2495549fee52e513e7be9f22207d24b847fbb0cb5bab795c690053e652d11539a2d960feadecb9b175487000f7812ceacf5db83301606cc357da3

93059673632372950846760233404528265552269035223116912529822641707683942804264477357459595582862911535897995187130227884549537965505362023024775114219145218863449310278405967527628414837264004662699074903716560546461614167980089864300514695266151435327524971233343815264097726955869839543417933242229553724835
```
<br>
So now, we have `N`, `d` and `e` and can reconstruct the private key : 

```bash
>>> rsatool.py -n 139589510448559426270140350106792398328403552834675368794733962561525914206396716036189393374294367303846992780695341826824306948258043034537162671328717852010658546072889560963675752283945322326005793726927254786393742142055806404861768824642002369852610966461772991591954263592365958281593575065087653537201 -e 3 -d 93059673632372950846760233404528265552269035223116912529822641707683942804264477357459595582862911535897995187130227884549537965505362023024775114219145218863449310278405967527628414837264004662699074903716560546461614167980089864300514695266151435327524971233343815264097726955869839543417933242229553724835

Using (n, d) to initialise RSA instance

n =
c6c83529a2388f146365c5f5fd4b0d888961b95de10ffa8853a3c2cbed750e9959bd0ff872c2232f
6bad32624f356a82d062755e1e4fedae54e8ca2471fc8d13ac700ee25720d4d9089fd6fbd42f12e6
a41e1c1de81f578c32132ad08594e851841d0239cd410def11d1c15ee75b92f86a04f7c6c7f36b90
46b8fb2fe29565b1

e = 3 (0x3)

d =
848578c66c25b4b84243d94ea8dcb3b05b967b93eb5ffc5ae26d2c87f3a35f10e67e0aa5a1d6c21f
9d1e2196df78f1ac8aec4e3ebedff3c98df086c2f6a85e0befc0ca19c5e2495549fee52e513e7be9
f22207d24b847fbb0cb5bab795c690053e652d11539a2d960feadecb9b175487000f7812ceacf5db
83301606cc357da3

p =
cf274700d6f6adea7e5344a45c34a92fb93db8f23f5da7de32decc6fe46b7d6dc43aabf15bb1cc0f
4865112a3d17166682e43cbc2a81e2ed64a75e55c6398a7b

q =
f5a798bad756b8ee9b4e3a91fe1cafd7ffad5770377af0156c23c64d407f92dbe24a92ae7427fd7e
b18c620341a17dc7670986ee676e17d99d497bcfea0b9ec3

```

To finish, we just need to reconstruct the real private key using `p` and `q`.

```bash
>>> rsatool.py -p 10849505326481756291467078773071822372991137432974866960395851121614351820034216452685397471770720115956086308531118673792947647173561074086963379535710843 -q 12865979254173524318205154356956226942340819748396485453570850199275733851574194544096345303446141207553525448737577133880210914025405392588238363787239107 -e 65537 -o real.pem

>>> openssl rsautl -decrypt -in secret -out plaintext -inkey real.pem
>>> cat plaintext 
kenny
```

<br>
Pwned.

__Flag__: kenny

Enjoy,<br>
\- [Notfound](http://www.notfound.ovh)

