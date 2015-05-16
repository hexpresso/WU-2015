# Nuit du Hack Quals CTF 2015: PDFCeption

**Category:** Forensics
**Points:** 500
**Solves:** 7
**Description:** 

> "What is the most resilient parasite? Bacteria? A virus? An intestinal worm? An idea. Resilient... highly contagious. Once an idea has taken hold of the brain it's almost impossible to eradicate. An idea that is fully formed - fully understood - that sticks; right in there somewhere. " (Cobb)
> 
> Where is that idea located? Can you tell us what it is?
> 
> <http://static.challs.nuitduhack.com/PDFCeption.tar.gz>

___

## Write-up

Well, we don’t validate this challenge at time, but few secondes after the end …

We just explain a quick way to do because this shitty challenge does not deserve a nice writeup.<br>
We were given a .vdi and a file called `lastdump`.<br>
This file was supposed to be encrypted (LUKS) but it was not the case …

So we have extracted the data :

```bash
$ foremost -t pdf lastdump
# or
$ photorec <on the lastdump>
```
<br><br>

It was like … 3AM and we already got the PDF with the logo.<br>
We spend many many MANY times on it, searching for a PDF in the PDF (PDFCeption …), playing with ascii85decode to decode stream, etc.

At 10AM, Notfound asked to the author, an embittered person (yggdrasil):

> 2015-04-04 10:02:19 Notfound_ is the logo important ?

No reponse …
A shitty hint was given :

> PDFCeption -> Hint: find the difference http://bit.ly/1avCLaQ

Just before midnight, we decided to try LSB on the logo (for a MISC500, yeah LSB, seems legit).<br>
Indeed, the surprise was huge. The logo is stegano !!!!

![Alt text](https://hexpresso.files.wordpress.com/2015/04/lel.png)
_lel_

Last step, find what kind : LSB BGR.

__The flag is__: DaddyDontTouchMeThere

— [Notfound](http://www.notfound.ovh)
