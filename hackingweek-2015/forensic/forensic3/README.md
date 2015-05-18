# HackingWeek CTF 2015: Forensic 3

**Category:** forensic |
**Points:** 4300 |
**Solves:** 37 |
**Description:**

> L'image mémoire fournie a été capturée sur une machine compromise à vous de l'analyser pour répondre aux questions (il s'agit de la même image pour les quatre épreuves de forensic, inutile de la télécharger plusieurs fois).<br>
> <br> 
> Trouvez le mot de passe de l'utilisateur <code>admin</code> qui se trouve quelque part en mémoire.
> 
> * <code>dump.gz</code> (md5sum:1273931ce359f59bce95ce4507e1f4bf)

___

## Write-up

I fount the method Wdiget here :
http://articles.forensicfocus.com/2014/04/28/windows-logon-password-get-windows-logon-password-using-wdigest-in-memory-dump/

```bash
$ vol -f /root/Téléchargements/hackweek/dump --profile=Win7SP1x86 logon
 
Volatility Foundation Volatility Framework 2.3.1
[!] lsass.exe(480) dump start!
[!] lsass.exe(480) dump is complete!
[!] wdigest.dll dump start!
[!] wdigest.dll dump is complete!
[!] lsasrv.dll dump start!
[!] lsasrv.dll dump is complete!
[!] lsass.exe(480) memmap dump start!
[!] lsass.exe(480) memmap dump is complete!
 
[!] Encrypt type is <3DES>
[!] Password decrypt success!
 
[=] Username :  admin
[=] Password :  cV[5g@2I
```
<br>
__Flag__: `cV[5g@2I`
<br>
Enjoy,<br>
\- [Rawger](https://twitter.com/_rawger)
