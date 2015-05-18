# HackingWeek CTF 2015: Forensic 2

**Category:** forensic |
**Points:** 3440 |
**Solves:** 48 |
**Description:**


> L'image mémoire fournie a été capturée sur une machine compromise à vous de l'analyser pour répondre aux questions (il s'agit de la même image pour les quatre épreuves de forensic, inutile de la télécharger plusieurs fois).<br>
> <br> 
> L'un des utilisateurs de la machine venait de consulter plusieurs sites Web à propos d'un incident qui impliquait une personnalité de showbiz. La clef de validation est le <code>PrenomNom</code> de cette personnalité.
> 
> * <code>dump.gz</code> (md5sum:1273931ce359f59bce95ce4507e1f4bf)


___

## Write-up

This task was also quite easy, here is my oneliner :<br>
```bash
strings dump | grep "actu"
 
; String0 = CPUManufacturer ,  Numeric0 = Unused.
; Sometimes the BIOS does not report the memory it is actually using.
; String Args: (CPUManufacturer, Op, Op, Op)
; String Args: (CPUManufacturer, CPUFamilyOp, CPUModelOp, CPUSteppingOp,
; String Args: (CPUManufacturer, Op, Op, Op)
Previous packet was actually BOOTP!
A       https://fr.cinema.yahoo.com/actualite/crash-avion-harrison-ford-blesse-070221[...]
 
HarrisonFord
```
<br>
__Flag__: `HarrisonFord`
<br>
Enjoy,<br>
\- [Rawger](https://twitter.com/_rawger) 
