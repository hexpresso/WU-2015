# 0CTF 2015 Quals CTF: Treasure

**Category:** Misc
**Points:** 50
**Solves:** 45
**Description:** 

> Romors say that something is buried in treasure.ctf.0ops.sjtu.cn, happy treasure hunting. :)

___

## Write-up

First of all, we do a DNS request :

```bash
$ host -a treasure.ctf.0ops.sjtu.cn
treasure.ctf.0ops.sjtu.cn. 299 IN AAAA 2001:470:d:b28::40:1
treasure.ctf.0ops.sjtu.cn. 299 IN A 127.0.0.1
```
<br>

Well, we see that the IPv4 pointing on localhost, but the IPv6 is more interesting.

Let’s try a ping6 on it:
```bash
$ ping6 2001:470:d:b28::40:1
PING 2001:470:d:b28::40:1(2001:470:d:b28::40:1) 56 data bytes
64 bytes from 2001:470:d:b28::40:1: icmp_seq=1 ttl=14 time=175 ms
```
<br>

Well, it works :)

We decide to traceroute6 on it :
```bash
$ traceroute6 2001:470:d:b28::40:1
1 ...
2 ...
//...//
20 2001:470:d:b28::9:2 (2001:470:d:b28::9:2) 172.610 ms 171.792 ms 172.313 ms
21 2001:470:d:b28::10:2 (2001:470:d:b28::10:2) 171.797 ms 171.699 ms 171.371 ms
22 2001:470:d:b28::11:2 (2001:470:d:b28::11:2) 173.071 ms 171.519 ms 171.780 ms
23 2001:470:d:b28::12:2 (2001:470:d:b28::12:2) 172.427 ms 173.171 ms 171.872 ms
24 2001:470:d:b28::13:2 (2001:470:d:b28::13:2) 172.574 ms 172.949 ms 171.313 ms
25 0000000110001101110000000 (2001:470:d:b28::14:2) 173.872 ms 171.953 ms 171.219 ms
26 0111110111100111110111110 (2001:470:d:b28::15:2) 172.954 ms 172.866 ms 171.901 ms
27 0100010110001100110100010 (2001:470:d:b28::16:2) 172.798 ms 173.038 ms 171.559 ms
28 0100010101000011010100010 (2001:470:d:b28::17:2) 173.087 ms 173.022 ms 173.842 ms
29 0100010101010101110100010 (2001:470:d:b28::18:2) 172.429 ms 174.906 ms 175.403 ms
30 0111110110011011010111110 (2001:470:d:b28::19:2) 174.955 ms 175.574 ms 171.454 ms
```
<br>

Ok, we see some bits.
Our first thought was to translate this into ascii (group by 7 or 8)
But this idea was a big fail :D
We also try XOR, etc.

Notfound said `It’s look like a header of a QRCode, but I think my mind is burning lol.`

But he was right.

Based on this, I had the idea to test with the ipv6 2001:470:d:b28::20:2 :
```bash
$ host 2001:470:d:b28::20:2
2.0.0.0.0.2.0.0.0.0.0.0.0.0.0.0.8.2.b.0.d.0.0.0.0.7.4.0.1.0.0.2.ip6.arpa domain name pointer 0000000101010101010000000.
```
<br>

__BINGO !__

So, I have used a small bash oneliner done by notfound that give us all bits :
```bash
$ for i in {14..38}; do host 2001:470:d:b28::$i:2 | awk '{print $5}'; done
 
0000000110001101110000000.
0111110111100111110111110.
0100010110001100110100010.
0100010101000011010100010.
0100010101010101110100010.
0111110110011011010111110.
0000000101010101010000000.
1111111110111100111111111.
0011100010001010011100111.
0100011011001101101000000.
0101010000111110110010100.
0011111011010110011010101.
1001010100000111010010000.
0001111100000101001010110.
0110110100110010110100000.
0100101001101111101000010.
0110100101100000000001010.
1111111100111011011101001.
0000000101101110010101100.
0111110101111100011100110.
0100010110011010000001101.
0100010111011101000011000.
0100010110010110111010010.
0111110100101111000010110.
0000000100000010010100110.
```
<br>

The last to do was to convert this to an image.<br>
I have used a well-known format, the portable pixmap (.ppm).
```bash
echo "P1" > flag.ppm
echo "25 $((38-14+1))" >> flag.ppm
for i in {14..38}; do host 2001:470:d:b28::$i:2 | gawk '{T=$5 ; sub(/.$/, "", T); print T}' >> flag.ppm; done
convert flag.ppm -negate flag.png
gimp flag.png
```
<br>
![Alt text](https://hexpresso.files.wordpress.com/2015/03/flag_treasure.png)<br>
_flag_treasure_

__Flag__: 0CTF{Reverse DNS is so FUN!}
<br><br>
Enjoy!<br>
[Sorcier](https://twitter.com/_sorcier_) & [Notfound](https://twitter.com/Notfound404__)
