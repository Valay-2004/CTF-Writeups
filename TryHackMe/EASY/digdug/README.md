---
title: "DigDug - TryHackMe Writeup & Walkthrough"
description: "Identifying and extracting the flag using a DNS server."
permalink: /TryHackMe/EASY/digdug/
---

# DigDug - TryHackMe Writeup

This DNS server hosts a special domain `givemetheflag.com` that contains the flag.

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Easy-blue)](https://tryhackme.com/room/digdug/)
[![Web Exploitation](https://img.shields.io/badge/Category-DNS-brightgreen)](#)

**Task 1: Dig Dug**

Oooh, turns out, this `MACHINE_IP` machine is also a DNS server! If we could `dig` into it, I am sure we could find some interesting records! But... it seems weird, this only responds to a special type of request for a `givemetheflag.com` domain?

**Access this challenge** by deploying both the vulnerable machine (green "Start Lab Machine" button) and the TryHackMe AttackBox (top-right button).

Use common enumeration tools on the AttackBox to get the server on **MACHINE_IP** to respond with the flag.

---

**Answer the questions below**

**Q. Retrieve the flag from the DNS server!**

When I first tried to `dig` the IP, I got the following result:

```shell
└─$ dig -x 10.49.160.255
;; communications error to 10.200.73.100#53: timed out
;; communications error to 10.200.73.100#53: timed out
;; communications error to 10.200.73.100#53: timed out

; <<>> DiG 9.20.22-1-Debian <<>> -x 10.49.160.255
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 49480
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;255.160.49.10.in-addr.arpa.    IN      PTR

;; Query time: 96 msec
;; SERVER: 1.1.1.1#53(1.1.1.1) (UDP)
;; WHEN: Tue Jun 09 08:48:10 EDT 2026
;; MSG SIZE  rcvd: 55
```

The internal resolver timed out, and public DNS returned NXDOMAIN as expected for a private IP.

However, using `nslookup` and directly querying the target DNS server gave us the flag:

```shell
└─$ nslookup
> server 10.49.160.255
Default server: 10.49.160.255
Address: 10.49.160.255#53
> givemetheflag.com
Server:         10.49.160.255
Address:        10.49.160.255#53

givemetheflag.com       text = "flag{0767ccd06e79853318f25aeb08ff83e2}"
givemetheflag.com       text = "flag{0767ccd06e79853318f25aeb08ff83e2}"
>
```

**Flag:** `flag{0767ccd06e79853318f25aeb08ff83e2}`
