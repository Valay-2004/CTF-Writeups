Author: Janice He

#### Description

Oops! Someone accidentally sent an important file to a network printer—can you retrieve it from the print server?

Additional details will be available after launching your challenge instance.

Hints:

1. knowing how SMB protocol works would be helpful!
2. smbclient and smbutil are good tools

---

Okay so we check for `smb` shares with `smbclient`

```bash
└─$ smbclient -L //mysterious-sea.picoctf.net -p 60694 -N

        Sharename       Type      Comment
        ---------       ----      -------
        shares          Disk      Public Share With Guests
        IPC$            IPC       IPC Service (Samba 4.19.5-Ubuntu)
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to mysterious-sea.picoctf.net failed (Error NT_STATUS_CONNECTION_REFUSED)
Unable to connect with SMB1 -- no workgroup available

```

Then we get into `/shares` as no other `printer` related share can be found

```bash
└─$ smbclient //mysterious-sea.picoctf.net/shares -p 60694 -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Mar  6 15:25:45 2026
  ..                                  D        0  Fri Mar  6 15:25:45 2026
  dummy.txt                           N     1142  Wed Feb  4 16:22:17 2026
  flag.txt                            N       37  Fri Mar  6 15:25:45 2026

                65536 blocks of size 1024. 58696 blocks available
```

And thus we can fetch flag on our system

```bash
smb: \> get flag.txt
getting file \flag.txt of size 37 as flag.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
smb: \> get dummy.txt
getting file \dummy.txt of size 1142 as dummy.txt (0.7 KiloBytes/sec) (average 0.4 KiloBytes/sec)
smb: \>
```

Thus our flag is

```bash
└─$ cat flag.txt
picoCTF{5mb_pr1nter_5h4re5_b3f2f855}
```

And our `dummy.txt`

```bash
└─$ cat dummy.txt
Far far away, behind the word mountains, far from the countries Vokalia and Consonantia, there live the blind texts. Separated they live in Bookmarksgrove right at the coast of the Semantics, a large language ocean. A small river named Duden flows by their place and supplies it with the necessary regelialia. It is a paradisematic country, in which roasted parts of sentences fly into your mouth. Even the all-powerful Pointing has no control about the blind texts it is an almost unorthographic life One day however a small line of blind text by the name of Lorem Ipsum decided to leave for the far World of Grammar. The Big Oxmox advised her not to do so, because there were thousands of bad Commas, wild Question Marks and devious Semikoli, but the Little Blind Text didn’t listen. She packed her seven versalia, put her initial into the belt and made herself on the way. When she reached the first hills of the Italic Mountains, she had a last view back on the skyline of her hometown Bookmarksgrove, the headline of Alphabet Village and the subline of her own road, the Line Lane. Pityful a rethoric question ran over her cheek, then
```
