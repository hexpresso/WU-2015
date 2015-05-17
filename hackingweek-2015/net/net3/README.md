# HackingWeek CTF 2015: Net 3

**Category:** net |
**Points:** 3784 |
**Solves:** 43 |
**Description:**


> Le fichier `pcap` suivant contient une capture réseau d'un transfert via le protocole SMB2. Il s'agit de retrouver un fichier zippé qui transité sur le réseau et d'en extraire la clef qu'il contient.
> 
> * [smb-capture.pcap.gz](http://hackingweek.fr/media/Aa0eiHuu/smb-capture.pcap.gz)

___

## Write-up

For this challenge, it was written that we had to find a zipped file.<br>
The first thing I did was to use foremost searching for .zip :

```bash
$ foremost -t zip smb-capture.pcap
Processing: smb-capture.pcap
|foundat=secret/
foundat=secret/flag.txt
```
<br><br>
Well, foremost have found some .zip, let's see.

```bash
$ ls output/zip/
00007967.zip  00007968.zip  00028015.zip  00028131.zip  00029940.zip  00029941.zip  00077631.zip  00079582.zip  00079583.zip
```
<br><br>
When I tried to unzip one, I had this output :

```bash
unzip 00007967.zip 
Archive:  00007967.zip
   skipping: secret/flag.txt         need PK compat. v5.1 (can do v4.6)
   creating: secret/
```
<br><br>
This is because two things :
- The file is password-protected.
- The zipped file is a 7z.

```bash
$ 7z e 00007967.zip

7-Zip 9.20  Copyright (c) 1999-2010 Igor Pavlov  2010-11-18
p7zip Version 9.20 (locale=fr_FR.utf8,Utf16=on,HugeFiles=on,4 CPUs)

Processing archive: 00007967.zip

Extracting  secret
Extracting  secret/flag.txt
Enter password (will not be echoed) : the_game
     Data Error in encrypted file. Wrong password?

Sub items Errors: 1
```
<br><br>

Hm, we need to find the password.<br>
I did the bruteforce using john2zip and john :

```bash
$ zip2john 00007967.zip
00007967.zip->secret/ is not encrypted!
00007967.zip->secret/flag.txt is using AES encryption, extrafield_length is 11
00007967.zip:$zip$*0*1*a2b728db72200be0*a089

$ echo '$zip$*0*1*a2b728db72200be0*a089' > pass
$ john pass 
Loaded 1 password hash (WinZip PBKDF2-HMAC-SHA-1 [32/32])
dawn             (?)
```
<br><br>

Oh, a possible password ... Let's try it : 
```bash
7z e 00007967.zip -pdawn

7-Zip 9.20  Copyright (c) 1999-2010 Igor Pavlov  2010-11-18
p7zip Version 9.20 (locale=fr_FR.utf8,Utf16=on,HugeFiles=on,4 CPUs)

Processing archive: 00007967.zip

Extracting  secret
Extracting  secret/flag.txt

Everything is Ok

Folders: 1
Files: 1
Size:       25
Compressed: 296
```
<br><br>

All seem good. 

```bash
cat flag.txt 
Le flag est : AsHTwn1xw2
```
<br><br>

Pwned.
<br>
__Flag__: AsHTwn1xw2
<br>
Enjoy,<br>
\- [Notfound](http://www.notfound.ovh)
