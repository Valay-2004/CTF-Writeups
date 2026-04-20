---
title: "TryHeartMe - TryHackMe Writeup & Walkthrough"
description: "A detailed guide on exploiting TryHeartMe using JWT tampering and privilege escalation."
permalink: /TryHackMe/EASY/TryHeartMe/
---

# TryHeartMe - TryHackMe Writeup

Exploiting a Valentine's-themed web application via JWT cookie tampering to gain admin privileges and purchase the hidden "Valenflag".

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Easy-blue)](https://tryhackme.com/room/lafb2026e5/)
[![Web Exploitation](https://img.shields.io/badge/Category-Web%20Exploitation-brightgreen)](#)

**Key Vulnerabilities / Concepts:**

- JWT (JSON Web Token) Identification
- Broken Authentication (Missing Signature Verification)
- Client-Side State Manipulation (Role & Credits)
- Privilege Escalation

## Table of Contents

- [Enumeration](#enumeration)
  - [Nmap Scan](#nmap-scan)
  - [Initial Web Exploration](#initial-web-exploration)
- [Initial Access / Analysis](#initial-access--analysis)
  - [Identifying the JWT](#identifying-the-jwt)
- [Exploitation](#exploitation)
  - [JWT Tampering](#jwt-tampering)
  - [Gaining Admin Access](#gaining-admin-access)
- [Final Capture](#final-capture)

---

## Enumeration

### Nmap Scan

We begin with an Nmap scan to identify open ports and services:

```bash
# Nmap scan initiated
# nmap -sC -sV -p- <TARGET_IP>
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
5000/tcp open  http    Gunicorn
```

The scan reveals:

- **Port 22**: SSH (Standard).
- **Port 5000**: A web service running on Gunicorn. This will be our primary target.

### Initial Web Exploration

Accessing the website on port `5000` reveals a Valentine's-themed gift shop.

![Shop Dashboard](images/shop_dashboard.png)

The site allows users to **Signup** and **Login**. After creating an account, we are presented with a dashboard where we have **0 credits**. Looking through the shop, there is a mention of a hidden "Valenflag," but it's nowhere to be found for regular users.

---

## Initial Access / Analysis

### Identifying the JWT

Since the application tracks user credits and roles, the first place to look is the browser's storage.

1. Open DevTools (**F12**).
2. Navigate to the **Storage** (or Application) tab.
3. Check the **Cookies** section.

![Browser Cookies](images/cookies.png)

We find a cookie named `token` or `jwt`. For this walkthrough, let's use the one we found:

```jwt
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImhhY2szckBoYWNrLmNvbSIsInJvbGUiOiJ1c2VyIiwiY3JlZGl0cyI6MCwiaWF0IjoxNzcxNzMxMjYyLCJ0aGVtZSI6InZhbGVudGluZSJ9.D6YfcWVTFtWqkj-Uc78fu3PegPT0eBgj2gBwa0BbvCU
```

> [!NOTE]
> JWTs consist of three parts separated by dots: **Header**, **Payload**, and **Signature**. They are Base64Url encoded.

Decoding this on [jwt.io](https://jwt.io) reveals:

**Payload:**

```json
{
  "email": "hack3r@hack.com",
  "role": "user",
  "credits": 0,
  "iat": 1771731262,
  "theme": "valentine"
}
```

---

## Exploitation

### JWT Tampering

The presence of `role` and `credits` in the JWT payload is a massive red flag. If the server doesn't verify the signature, we can simply modify these values.

> [!IMPORTANT]
> In a secure implementation, the server checks the signature against a secret key. If the signature is missing or verification is skipped, any user can impersonate an admin.

We modify the payload as follows:

- **role**: `user` → `admin`
- **credits**: `0` → `554443` (To ensure we can buy anything)

![JWT Manipulation](images/jwt_token.png)

**Updated Payload:**

```json
{
  "email": "hack3r@hack.com",
  "role": "admin",
  "credits": 554443,
  "iat": 1771733075,
  "theme": "valentine"
}
```

### Gaining Admin Access

After updating the payload, we generate the new JWT:

```jwt
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImhhY2szckBoYWNrLmNvbSIsInJvbGUiOiJhZG1pbiIsImNyZWRpdHMiOjU1NDQ0MywiaWF0IjoxNzcxNzMzMDc1LCJ0aGVtZSI6InZhbGVudGluZSJ9.S7cmXw2rCBbnnJQRXA4O4iJ4HWgcHYTW6456kWvR3ls
```

> [!WARNING]
> The signature part (`S7cmXw...`) may change with each refresh or encoding tool. However, since the server skips verification, the exact signature often doesn't matter as long as the format is valid.

1. Replace your current cookie in DevTools with this new string.
2. Refresh the page.

---

## Final Capture

Upon refreshing, we are redirected to or can now access the **Admin Panel**. The shop dashboard updates to show our custom credit amount, and the **Valenflag** item is now visible!

![Admin Panel](images/admin_panel.png)

The Valenflag costs **777 credits**. Since we have plenty, we "purchase" it.

![Flag Found](images/flag.png)

**Flag:** `THM{v4l3nt1n3_jwt_c00k13_t4mp3r_4dm1n_sh0p}`

### Key Takeaways

- **Never trust client-side data**: Values like `role` or `balance` should be stored and validated on the server.
- **Enforce JWT Signatures**: Always use strong secrets and ensure the server rejects tokens with invalid or missing signatures.
- **Identity check**: Periodically re-validate user permissions on the backend for every sensitive action (like purchasing or accessing admin panels).

Happy hacking! ❤️🧑‍💻
