---
title: "Hidden Deep Into My Heart - TryHackMe Writeup & Walkthrough"
description: "Web reconnaissance challenge from Love at First Breach 2026 involving directory enumeration and default credentials."
permalink: /TryHackMe/EASY/Hidden-Deep-Into-My-Heart/
---

# Hidden Deep Into My Heart (LAFB2026E9) - TryHackMe

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Easy-blue)](https://tryhackme.com/room/lafb2026e9)
[![Web Exploitation](https://img.shields.io/badge/Category-Web%20Exploitation-brightgreen)](#)

This is a simple web reconnaissance challenge from the `"Love at First Breach 2026"` event on TryHackMe. The goal is to find a hidden flag in an administrative panel by leveraging directory enumeration and information from standard files.

## Initial Recon: Directory Enumeration

We started by running a dirsearch scan to identify any useful directories on the target.

```bash
└─$ dirsearch -u http://10.49.156.234:5000/
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: ...../reports/http_10.49.156.234_5000/__26-02-20_04-27-20.txt

Target: http://10.49.156.234:5000/

[04:27:20] Starting:
[04:28:13] 400 -  167B  - /console
[04:29:05] 200 -   70B  - /robots.txt

Task Completed
```

This revealed `/robots.txt`, which contained the following:

```plain
User-agent: *
Disallow: /cupids_secret_vault/*

# cupid_arrow_2026!!!
```

The disallowed path `/cupids_secret_vault/` looked promising, and there was a commented-out potential credential or hint: `cupid_arrow_2026!!!`.

![Image of /cupids_secret_vault/ endpoint](./Images/cupids_secret_vault.png)

## Further Enumeration

We ran `dirsearch` again, focusing on the newly discovered path:

```bash
Target: http://10.49.156.234:5000/

[04:48:08] Starting: cupids_secret_vault/
[04:48:52] 200 -    2KB - /cupids_secret_vault/administrator

Task Completed
```

This uncovered `/cupids_secret_vault/administrator`, which presented a login form.

## Exploitation: Admin Login

First, we tried common default credentials:

- Username: `admin`
- Password: `admin`

This failed.

Recalling the comment in `robots.txt`, we tried:

- Username: `admin`
- Password: `cupid_arrow_2026!!!`

![Image of successful login](./Images/Flag.png)

This worked, granting access to the admin panel where the flag was displayed.

## Flag

`THM{l0v3_is_in_th3_r0b0ts_txt}`
