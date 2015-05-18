# HackingWeen CTF 2015: Forensic 1

**Category:** forensic | 
**Points:** 2580 |
**Solves:** 58 |
**Description:**


> L'image mémoire fournie a été capturée sur une machine compromise à vous de l'analyser pour répondre aux questions (il s'agit de la même image pour les quatre épreuves de forensic, inutile de la télécharger plusieurs fois).<br>
> <br>
> Pour obtenir la clef de validation de cette épreuve est donnée par le PID, le PPID et le nombre de threads du programme <code>Solitaire</code>. Mettez-le au format suivant <code>PID:PPID:nThreads</code>.
> 
> * `dump.gz` (md5sum:1273931ce359f59bce95ce4507e1f4bf)


___

## Write-up

This task was quite simple :<br>
```bash
vol -f /root/Téléchargements/hackweek/dump --profile=Win7SP1x86 pslist | grep Solitaire | awk '{printf("%d:%d:%d\n",$3,$4,$5)}'
```
<br>
_Output_: 
```
Volatility Foundation Volatility Framework 2.3.1
2992:1312:8
```

__Flag__: `2992:1312:8`
<br>
Enjoy,<br>
\- [Rawger](https://twitter.com/_rawger)
