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

