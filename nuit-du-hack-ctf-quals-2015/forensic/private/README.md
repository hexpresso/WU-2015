# Nuit du Hack Quals CTF 2015: Private

**Category:** Forensics
**Points:** 100
**Solves:** 114
**Description:** 

> "The quiet you are, the more you are able to ear"
> 
> [http://static.challs.nuitduhack.com/Private.tar.gz](Private.tar.gz)

___

## Write-up

Private 100 was a nice but really easy challenge. Many people found it difficult but it was not.

The first thing to do was to read the network capture to see the content.
We see many things, like STP, ICMP, ARP, CDP…

```bash
$ tshark -r PrivateChannel.pcap.pcapng
1 0.000000000 c2:01:12:a2:f1:02 -> Spanning-tree-(for-bridges)_00 STP Conf. Root = 32768/0/c2:01:12:a2:00:01 Cost = 0 Port = 0x802b
...
 
479 796.186499000 192.168.50.10 -> 192.168.0.50 ICMP Echo (ping) request
480 796.205229000 192.168.0.50 -> 192.168.50.10 ICMP Echo (ping) reply
```
<br><br>
Well, our first though was to check ICMP data : nothing.<br>
After that, we saw that in ICMP pacquet, the ID of IP Layer had values between ASCII code (32-126).

So we have coded a small one liner to check that :

```bash
$ echo -e $(tshark -r PrivateChannel.pcap.pcapng -e ip.id -Tfields  | sed -re 's/0x//;s/(..)/\\x\1/g') | egrep -a -o "[a-z0-9A-Z]+" | tr '\n' ' '
d d e e 0 0 1 1 1 1 l l m m R i h 4 e O r T e a r i s y o u r f l a g 6 J S R 3 c r r 3 t 4 g 3 n t
```
<br><br>

Ho yeah, very interesting !<br>
We can see « is your flag », but the end is quiet fuckedup.<br>
Nvm, we took a pokeball and we called PYTHON !<br>
Be carefull, scapy is capricious and will not accept the pcapng.<br>

To bypass that shit, you can use tcpdump :
```bash
$ tcpdump -r PrivateChannel.pcap.pcapng -w cap.pcap
```
<br><br>

Now, we are ready :)

```python
from scapy.all import *
 
# pk = rdpcap('PrivateChannel.pcap.pcapng')
pk = rdpcap('cap.pcap')
 
buff = ""
for i in range(480, 550):
    try:
         buff += chr(pk[i][1].id)
    except:
        print("YOU JUST LOSE THE GAME NOOB")
 
print("Ok you win ! -> %s" % buff)
```
<br><br>

Execute it :

```bash
$ python icmp.py
YOU JUST LOSE THE GAME NOOB
...
YOU JUST LOSE THE GAME NOOB
Ok you win ! -> here is your flag : S3cr3t4g3nt
```
<br><br>
Pwned,
__Flag__: S3cr3t4g3nt

Enjoy.
<br>
— [Notfound](http://www.notfound.ovh)
