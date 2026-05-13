100 pt.

Author: Yahaya Meddy

#### Description

We intercepted a suspicious file from a system, but instead of the password itself, it only contains its SHA-1 hash. Using OSINT techniques, you are provided with personal details about the target. Your task is to leverage this information to generate a custom password list and recover the original password by matching its hash. Download the following files:
[userinfo](https://challenge-files.picoctf.net/c_plain_mesa/3c16fb8e48ddae444f6840e81660a5a8cb2d30b69092b04f619d6ac34676a919/userinfo.txt): Contains the personal details.
[hash](https://challenge-files.picoctf.net/c_plain_mesa/3c16fb8e48ddae444f6840e81660a5a8cb2d30b69092b04f619d6ac34676a919/hash.txt): Contains the SHA-1 hash of the password. [check_password](https://challenge-files.picoctf.net/c_plain_mesa/3c16fb8e48ddae444f6840e81660a5a8cb2d30b69092b04f619d6ac34676a919/check_password.py): Script to test passwords against the hash.

Hint:

1. [CUPP](https://github.com/Mebus/cupp) is a Python tool for generating custom wordlists from personal data.

---

We have the give three files

```bash
└─$ ls
check_password.py  hash.txt  userinfo.txt
```

Having contents

```bash
└─$ echo 'UserInfo: ';cat userinfo.txt; echo '------------------------'; echo 'Hash: ';cat hash.txt
UserInfo:
First Name: Alice
Surname: Johnson
Nickname: AJ
Birthdate: 15-07-1990
Partner's Name: Bob
Child's Name: Charlie

------------------------
Hash:
968c2349040273dd57dc4be7e238c5ac200ceac5
```

And a python script named `check_password.py`

{% raw %}

```python
#!/usr/bin/env python3
import hashlib

HASH_FILE = "hash.txt"
WORDLIST_FILE = "passwords.txt" # wordlist that was generated using CUPP

def load_hash():
    with open(HASH_FILE, "r") as f:
        return f.read().strip()

def crack_password(target_hash):
    with open(WORDLIST_FILE, "r", encoding="utf-8", errors="ignore") as f:
        for password in f:
            password = password.strip()
            if hashlib.sha1(password.encode()).hexdigest() == target_hash:
                return password
    return None

if __name__ == "__main__":
    target_hash = load_hash()
    result = crack_password(target_hash)
    if result:
        print(f"Password found: picoCTF{{{result}}}")
    else:
        print("No match found.")

```

{% endraw %}

So to get the `password.txt` from the given info we will be using CUPP which the challenge ask us to use

```bash
└─$ python3 cupp.py
 ___________
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]

usage: cupp.py [-h] [-i | -w FILENAME | -l | -a | -v] [-q]

Common User Passwords Profiler

options:
  -h, --help         show this help message and exit
  -i, --interactive  Interactive questions for user password profiling
  -w FILENAME        Use this option to improve existing dictionary, or WyD.pl output to make some pwnsauce
  -l                 Download huge wordlists from repository
  -a                 Parse default usernames and passwords directly from Alecto DB. Project Alecto uses purified databases of Phenoelit and CIRT which were merged and enhanced
  -v, --version      Show the version of this program.
  -q, --quiet        Quiet mode (don't print banner)

```

We got `10359` passwords for alice

```bash
└─$ cat alice.txt| wc -l
10359
```

Now let's check for the password (it was pretty quick)

```bash
└─$ python3 check_password.py
Password found: picoCTF{Aj_15901990}
```

Thus the flag is > `picoCTF{Aj_15901990}`
