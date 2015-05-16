# Nuit du Hack Quals CTF 2015: faceBox

**Category:** Web
**Points:** 100
**Solves:** 18
**Description:** 

> A shady company decided to write their own software for storing files in the cloud.
> 
> "No no no, this is OUR filebox. We decline any responsability in the usage of our filebox. In any event your files get lost, trashed, stolen or spy on : it's your fault, not ours."
> 
> You are investigating on the security of their cloud storage as it might have disastrous consequences if it were to get hacked by malicious actors.
> 
> <http://prod.facebox.challs.nuitduhack.com/>

___

## Write-up

This task is a web task worth 100 points from the Nuit du Hack qualifications.<br>
Its name should have been `guest fest`.
We are given a website, where you can upload files.<br>
There are already a few files uploaded : confidentials.txt, which is private, and paste01.txt. Both have been uploaded by the same user.

We spent a lot of time on this task :
First of all, someone noticed that the URL was prod.facebox.challs…, however, replacing prod with dev return an error 503 at this time.<br>
Of course, we tried to upload php files, but it did not work.<br>
We also tried to upload files called « credentials.txt » to create a collision in the hash that allows you to view a file. But it did not work either.<br>
We then tried to attack the cookie. It was three base64 strings, joined by a single dot.<br>
It was probably a json, followed by a timestamp-like variable, and a hash to check the cookie is correct.

>    <+elkaluche> Moi j’ai passé 5H à upload des merdes, c’était génial<br>
>    <+elkaluche> ^^<br>
>    <+Arod> et moi 5h à bidouiller le cookie de merde<br>

Remember the domain that was not working earlier, that we thought was pointless ? Well, now it works, and it is far from being pointless.<br>
You need to retrieve as much file as you can from the .git repository.<br>
There are probably better ways to do so, but we created a local git repo, added a file, and tried to download every files.<br>
Once the repo is rebuilt, you can git status to see the missing files, and git checkout $file to recover a file.

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
 
 
def generate_random_filename(user_id,filename):
    dbuser = users.query.filter_by(id=user_id).first()
 
    if dbuser.privkey is not None:
        return md5(str(dbuser.privkey)+filename).hexdigest()
    else:
        privkey = str(randint(10000000,99999999))
        upd = users.query.filter_by(id=user_id).first()
        upd.privkey = privkey
        db.session.commit()
    return md5(str(privkey)+filename).hexdigest()
```


By reading this snippet of code, we figured that each user has a unique privkey.<br>
Since we can read the hash from one file of the admin, we can make bruteforce his privkey.<br>
Tishrom did it, and found 95594864.<br>
Given the salt, we can find the hash of confidentials.txt : md5(str(95594864)+'confidentials.txt').hexdigest()<br>
Flag is therefore at http://prod.facebox.challs.nuitduhack.com/files/view/35e2cb0b2e8bd40347ecd4e32767a060.

__Flag:__ M4x_M4i5_DR

Thanks to
[Arod](https://twitter.com/___Arod),<br>
[kaluche](https://twitter.com/Elkaluche),<br>
[Notfound](http://www.notfound.ovh),<br>
[Tishrom](https://twitter.com/Tishrom),<br>

— [XeR](https://twitter.com/XeR_0x2A)
