# HackingWeek CTF 2015: Net 2

**Category:** net |
**Points:** 1720 |
**Solves:** 67 |
**Description:**



> Le fichier `pcap` suivant contient une capture réseau d'un scan `nmap`, retrouvez les services qui ont été trouvés sur la machine cible. La clef de validation sera `service1:service2:service3:...` en mettant les services par ordre de ports croissants.
> 
> * [nmap-scan.pcap](http://hackingweek.fr/media/Aa0eiHuu/nmap-scan.pcap)

___

## Write-up

This challenge was fun but quite simple.

Indeed, when a service is enable on a machine, the host replies SYN/ACK to a SYN. That is the `three-way handshake`.

![Alt text](http://www.georgecoding.com/wp-content/uploads/2013/04/handshake.gif)

Knowing that, it's easy to output using tshark all the SYN/ACK received !

Here is my code :

```bash
$ tshark -r nmap-scan.pcap | awk  '/\[SYN, ACK\]/{print $8}'
```
<br>
So, to have the expected answer, using `/etc/services`
```bash
$ for PORT in $(tshark -r nmap-scan.pcap | awk  '/\[SYN, ACK\]/{print $8}'|sort -n); do 
> awk '/\t'"${PORT%→*}"'\/tcp/{printf "%s:", $1}' /etc/services
> done
ftp:ssh:telnet:http:https:
```
<br>
Pwned.

__Flag__: ftp:ssh:telnet:http:https

Enjoy,<br>
\- [Notfound](http://www.notfound.ovh)
