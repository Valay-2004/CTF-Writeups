---
title: "Cryptosystem - TryHackMe Writeup & Walkthrough"
description: "Breaking a weak RSA implementation using Fermat's Factorization to decrypt the flag in the Cryptosystem room on TryHackMe."
permalink: /TryHackMe/EASY/cryptosystem/
---

# TryHackMe: Cryptosystem Writeup

<img src="https://tryhackme.com/img/THMlogo.png" alt="TryHackMe" width="250" style=" display: block;
  margin-left: auto;
  margin-right: auto;
  width: 50%;">

---

## Challenge Overview

The **Cryptosystem** challenge from TryHackMe (https://tryhackme.com/room/hfb1cryptosystem) is a cryptography puzzle focused on breaking a weak RSA implementation. The challenge provides a Python script that encrypts a flag using RSA, along with the ciphertext (`c`), modulus (`n`), and public exponent (`e`). The goal is to decrypt the ciphertext to retrieve the flag. This writeup details the process of exploiting the RSA vulnerability and recovering the flag.

**Difficulty**: Medium  
**Category**: Cryptography  
**Key Concepts**: RSA, Fermat’s Factorization  
**Flag**: `THM{Just_s0m3_small_amount_of_RSA!}`

---

## Challenge Description

The provided script reveals how the flag was encrypted:

- Two primes, `p` and `q`, are generated. `p` is a 1024-bit prime, and `q` is the next prime after `p` (computed using the `primo` function).
- The modulus is `n = p * q`.
- The public exponent is `e = 0x10001` (65537).
- The private exponent `d` is computed as the modular inverse of `e` modulo `(p-1)*(q-1)`.
- The flag is encoded to a number and encrypted: `c = pow(bytes_to_long(FLAG.encode()), e, n)`.

We are given:

- `c`: The ciphertext (a large number).
- `n`: The modulus (another large number).
- `e`: 65537.

The vulnerability lies in the generation of `q` as the next prime after `p`, making `p` and `q` very close. This allows us to factor `n` using Fermat’s Factorization Method, compute the private key, and decrypt the ciphertext.

---

## Solution Approach

The key to solving this challenge is recognizing that `p` and `q` are close primes, which makes RSA vulnerable to Fermat’s Factorization. Here’s the step-by-step process:

1. **Factorize `n` Using Fermat’s Factorization**:
   - Since `p` and `q` are close, we can use Fermat’s method, which assumes `n = p * q = (a - b)(a + b) = a^2 - b^2`.
   - Start with `a` as the ceiling of the square root of `n`.
   - Increment `a` until `a^2 - n` is a perfect square, yielding `b^2`. Then, `p = a - b` and `q = a + b`.
   - Verify that `p * q == n`.

2. **Compute the Private Key**:
   - Calculate Euler’s totient: `phi = (p-1)*(q-1)`.
   - Compute the private exponent `d` as the modular inverse of `e` modulo `phi`.

3. **Decrypt the Ciphertext**:
   - Decrypt `c` using `m = pow(c, d, n)`.
   - Convert the resulting number `m` to a string using `long_to_bytes` to get the flag.

---

## Solution Script

Below is the Python script used to solve the challenge. It implements Fermat’s Factorization to find `p` and `q`, computes the private key, and decrypts the ciphertext.

### Just Run the script and you will get the flag

```python
# Credit to Grok, for guiding the solution approach
from gmpy2 import isqrt, is_square
from Crypto.Util.number import inverse, long_to_bytes

# Given values
c = 3591116664311986976882299385598135447435246460706500887241769555088416359682787844532414943573794993699976035504884662834956846849863199643104254423886040489307177240200877443325036469020737734735252009890203860703565467027494906178455257487560902599823364571072627673274663460167258994444999732164163413069705603918912918029341906731249618390560631294516460072060282096338188363218018310558256333502075481132593474784272529318141983016684762611853350058135420177436511646593703541994904632405891675848987355444490338162636360806437862679321612136147437578799696630631933277767263530526354532898655937702383789647510
n = 15956250162063169819282947443743274370048643274416742655348817823973383829364700573954709256391245826513107784713930378963551647706777479778285473302665664446406061485616884195924631582130633137574953293367927991283669562895956699807156958071540818023122362163066253240925121801013767660074748021238790391454429710804497432783852601549399523002968004989537717283440868312648042676103745061431799927120153523260328285953425136675794192604406865878795209326998767174918642599709728617452705492122243853548109914399185369813289827342294084203933615645390728890698153490318636544474714700796569746488209438597446475170891
e = 0x10001  # 65537

# Step 1: Fermat's Factorization
def fermat_factorization(n):
    a = isqrt(n)
    if a * a == n:
        return a, a
    while True:
        b_squared = a * a - n
        if is_square(b_squared):
            b = isqrt(b_squared)
            p = a - b
            q = a + b
            if p * q == n:
                return p, q
        a += 1

# Factor n
p, q = fermat_factorization(n)
print(f"p: {p}")
print(f"q: {q}")

# Step 2: Compute private key
phi = (p - 1) * (q - 1)
d = inverse(e, phi)
print(f"d: {d}")

# Step 3: Decrypt ciphertext
m = pow(c, d, n)
flag = long_to_bytes(m)
print(f"Flag: {flag.decode()}")
```

## Installation

`It's always better to use virtual environment for running`

To run the solution script, you need Python 3 and two essential libraries: `gmpy2` for efficient large-number arithmetic and `pycryptodome` for RSA-related functions like `long_to_bytes`. Set up your environment with the following commands:

```bash
# Install Python dependencies
pip install gmpy2 pycryptodome
```

On some systems (e.g., Ubuntu), gmpy2 requires additional system dependencies for its mathematical operations:

```bash
# Install GMP library on Ubuntu/Debian
sudo apt update
sudo apt install libgmp-dev
```

For other operating systems, such as macOS, install the equivalent GMP library (e.g., via Homebrew: brew install gmp). Refer to the gmpy2 documentation if you encounter installation issues.

---

<h2>Happy hacking! Follow my GitHub for more writeups and CTF solutions!
