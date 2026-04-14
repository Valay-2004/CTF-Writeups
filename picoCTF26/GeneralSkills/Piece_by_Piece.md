Author: Yahaya Meddy

#### Description

After logging in, you will find multiple file parts in your home directory. These parts need to be combined and extracted to reveal the flag.

Additional details will be available after launching your challenge instance.

---
So I got into the machine via given `ssh` credentials
and this was the output of `ls -lah`
```bash
ctf-player@pico-chall$ ls -lah
total 28K
drwxr-xr-x 1 ctf-player ctf-player  20 Mar 10 11:59 .
drwxr-xr-x 1 root       root        24 Feb  4 22:40 ..
drwx------ 2 ctf-player ctf-player  34 Mar 10 11:59 .cache
-rw-r--r-- 1 root       root        67 Feb  4 22:40 .profile
-rw-r--r-- 1 ctf-player ctf-player 282 Feb  4 21:22 instructions.txt
-rw-r--r-- 1 ctf-player ctf-player  51 Feb  4 22:40 part_aa
-rw-r--r-- 1 ctf-player ctf-player  51 Feb  4 22:40 part_ab
-rw-r--r-- 1 ctf-player ctf-player  51 Feb  4 22:40 part_ac
-rw-r--r-- 1 ctf-player ctf-player  51 Feb  4 22:40 part_ad
-rw-r--r-- 1 ctf-player ctf-player  35 Feb  4 22:40 part_ae
ctf-player@pico-chall$ 

```


Let's see what's in `instruction.txt`
```bash
ctf-player@pico-chall$ cat instructions.txt 
Hint:

- The flag is split into multiple parts as a zipped file.
- Use Linux commands to combine the parts into one file.
- The zip file is password protected. Use this "supersecret" password to extract the zip file.
- After unzipping, check the extracted text file for the flag.

```

So let's merge and check for the "SUPERSECRET" text file for flag

```bash
ctf-player@pico-chall$ cat part_* > flag.zip
ctf-player@pico-chall$ ls 
flag.zip  instructions.txt  part_aa  part_ab  part_ac  part_ad  part_ae
ctf-player@pico-chall$ unzip flag.zip 
Archive:  flag.zip
[flag.zip] flag.txt password: 
 extracting: flag.txt                
ctf-player@pico-chall$ ls
flag.txt  flag.zip  instructions.txt  part_aa  part_ab  part_ac  part_ad  part_ae
ctf-player@pico-chall$ cat flag.
cat: flag.: No such file or directory
ctf-player@pico-chall$ cat flag.txt 
picoCTF{z1p_and_spl1t_f1l3s_4r3_fun_28d309dc}
```

Thus our flag is > `picoCTF{z1p_and_spl1t_f1l3s_4r3_fun_28d309dc}`
