Author: Yahaya Meddy

#### Description

What’s behind the numeric gate? You only get access if you enter the _right_ kind of number. You can download the program file [here](https://challenge-files.picoctf.net/c_green_hill/5baff00695e30be7ca9d1202abff4cde6d9262a62282c40c491d250b66fa5678/gatekeeper). Reverse the program behavior and find what input grants access to the secret flag. `nc green-hill.picoctf.net 61972`.

Hints:

- Tools like `Ghidra`, IDA Free, or Radare2 can analyze the binary’s logic.
- The program’s output isn’t straightforward; reversing the string and cleaning out extra text may help you recover the flag.

---

### Step 0 – what I tried first and failed miserably 💀

Ran the binary locally:

```
Enter a numeric code (must be > 999 ): 1000
Access Denied.

Enter a numeric code (must be > 999 ): 9999
Too high.

Enter a numeric code (must be > 999 ): 1337
Access Denied.
```

Tried 000–999 → too small  
Tried 10000+ → too high or denied  
Tried every 10xx number → still denied

Conclusion: pure decimal 4-digit numbers = cope

---

### Step 1 – actually look at the decomp (the important part)

Opened in IDA / Ghidra → main is doing this:

```assembly
 =============== S U B R O U T I N E =======================================
.text:000055555555557F
.text:000055555555557F ; Attributes: bp-based frame
.text:000055555555557F
.text:000055555555557F ; int __fastcall main(int argc, const char **argv, const char **envp)
.text:000055555555557F                 public main
.text:000055555555557F main            proc near               ; DATA XREF: _start+21↑o
.text:000055555555557F
.text:000055555555557F var_38          = dword ptr -38h
.text:000055555555557F var_34          = dword ptr -34h
.text:000055555555557F s               = byte ptr -30h
.text:000055555555557F var_8           = qword ptr -8
.text:000055555555557F
.text:000055555555557F ; __unwind { // dword_555555554000
.text:000055555555557F                 endbr64
.text:0000555555555583                 push    rbp
.text:0000555555555584                 mov     rbp, rsp
.text:0000555555555587                 sub     rsp, 40h
.text:000055555555558B                 mov     rax, fs:28h
.text:0000555555555594                 mov     [rbp+var_8], rax
.text:0000555555555598                 xor     eax, eax
.text:000055555555559A                 lea     rdi, aEnterANumericC ; "Enter a numeric code (must be > 999 ): "
.text:00005555555555A1                 mov     eax, 0
.text:00005555555555A6                 call    _printf
.text:00005555555555AB                 mov     rax, cs:stdout@@GLIBC_2_2_5
.text:00005555555555B2                 mov     rdi, rax        ; stream
.text:00005555555555B5                 call    _fflush
.text:00005555555555BA                 lea     rax, [rbp+s]
.text:00005555555555BE                 mov     rsi, rax
.text:00005555555555C1                 lea     rdi, a31s       ; "%31s"
.text:00005555555555C8                 mov     eax, 0
.text:00005555555555CD                 call    ___isoc99_scanf
.text:00005555555555D2                 mov     [rbp+var_38], 0FFFFFFFFh
.text:00005555555555D9                 lea     rax, [rbp+s]
.text:00005555555555DD                 mov     rdi, rax        ; s
.text:00005555555555E0                 call    _strlen
.text:00005555555555E5                 mov     [rbp+var_34], eax
.text:00005555555555E8                 lea     rax, [rbp+s]
.text:00005555555555EC                 mov     rdi, rax
.text:00005555555555EF                 call    is_valid_decimal
.text:00005555555555F4                 test    eax, eax
.text:00005555555555F6                 jz      short loc_555555555609
.text:00005555555555F8                 lea     rax, [rbp+s]
.text:00005555555555FC                 mov     rdi, rax        ; nptr
.text:00005555555555FF                 call    _atoi
.text:0000555555555604                 mov     [rbp+var_38], eax
.text:0000555555555607                 jmp     short loc_555555555647
.text:0000555555555609 ; ---------------------------------------------------------------------------
.text:0000555555555609
.text:0000555555555609 loc_555555555609:                       ; CODE XREF: main+77↑j
.text:0000555555555609                 lea     rax, [rbp+s]
.text:000055555555560D                 mov     rdi, rax
.text:0000555555555610                 call    is_valid_hex
.text:0000555555555615                 test    eax, eax
.text:0000555555555617                 jz      short loc_555555555634
.text:0000555555555619                 lea     rax, [rbp+s]
.text:000055555555561D                 mov     edx, 10h        ; base
.text:0000555555555622                 mov     esi, 0          ; endptr
.text:0000555555555627                 mov     rdi, rax        ; nptr
.text:000055555555562A                 call    _strtol
.text:000055555555562F                 mov     [rbp+var_38], eax
.text:0000555555555632                 jmp     short loc_555555555647
.text:0000555555555634 ; ---------------------------------------------------------------------------
.text:0000555555555634
.text:0000555555555634 loc_555555555634:                       ; CODE XREF: main+98↑j
.text:0000555555555634                 lea     rdi, aInvalidInput ; "Invalid input."
.text:000055555555563B                 call    _puts
.text:0000555555555640                 mov     eax, 1
.text:0000555555555645                 jmp     short loc_555555555698
.text:0000555555555647 ; ---------------------------------------------------------------------------
.text:0000555555555647
.text:0000555555555647 loc_555555555647:                       ; CODE XREF: main+88↑j
.text:0000555555555647                                         ; main+B3↑j
.text:0000555555555647                 cmp     [rbp+var_38], 3E7h
.text:000055555555564E                 jg      short loc_55555555565E
.text:0000555555555650                 lea     rdi, aTooSmall  ; "Too small."
.text:0000555555555657                 call    _puts
.text:000055555555565C                 jmp     short loc_555555555693
.text:000055555555565E ; ---------------------------------------------------------------------------
.text:000055555555565E
.text:000055555555565E loc_55555555565E:                       ; CODE XREF: main+CF↑j
.text:000055555555565E                 cmp     [rbp+var_38], 270Fh
.text:0000555555555665                 jle     short loc_555555555675
.text:0000555555555667                 lea     rdi, aTooHigh   ; "Too high."
.text:000055555555566E                 call    _puts
.text:0000555555555673                 jmp     short loc_555555555693
.text:0000555555555675 ; ---------------------------------------------------------------------------
.text:0000555555555675
.text:0000555555555675 loc_555555555675:                       ; CODE XREF: main+E6↑j
.text:0000555555555675                 cmp     [rbp+var_34], 3
.text:0000555555555679                 jnz     short loc_555555555687
.text:000055555555567B                 mov     eax, 0
.text:0000555555555680                 call    reveal_flag
.text:0000555555555685                 jmp     short loc_555555555693
.text:0000555555555687 ; ---------------------------------------------------------------------------
.text:0000555555555687
.text:0000555555555687 loc_555555555687:                       ; CODE XREF: main+FA↑j
.text:0000555555555687                 lea     rdi, aAccessDenied ; "Access Denied."
.text:000055555555568E                 call    _puts
.text:0000555555555693
.text:0000555555555693 loc_555555555693:                       ; CODE XREF: main+DD↑j
.text:0000555555555693                                         ; main+F4↑j ...
.text:0000555555555693                 mov     eax, 0
.text:0000555555555698
.text:0000555555555698 loc_555555555698:                       ; CODE XREF: main+C6↑j
.text:0000555555555698                 mov     rcx, [rbp+var_8]
.text:000055555555569C                 xor     rcx, fs:28h
.text:00005555555556A5                 jz      short locret_5555555556AC
.text:00005555555556A7                 call    ___stack_chk_fail
.text:00005555555556AC ; ---------------------------------------------------------------------------
.text:00005555555556AC
.text:00005555555556AC locret_5555555556AC:                    ; CODE XREF: main+126↑j
.text:00005555555556AC                 leave
.text:00005555555556AD                 retn
.text:00005555555556AD ; } // starts at 55555555557F
.text:00005555555556AD main            endp
.text:00005555555556AD
```

- `scanf("%31s")` → reads string
- checks `is_valid_decimal()` → if yes → atoi()
- else checks `is_valid_hex()` → if yes → strtol(..., 16)
- then: value must be > 999 (0x3E7) **and** ≤ 9999 (0x270F)
- **AND** — the input string length must be **exactly 3** → only then calls `reveal_flag()`

> **Key moment**: length == 3 **and** hex value in [1000–9999]

So decimal numbers with 3 digits = too small  
4+ digits = length wrong → Access Denied  
→ we need **3-character hex input** that converts to 1000–9999 decimal

---

### Step 2 – find a working input (the moment of enlightenment)

Some easy ones:

- `3E8` → hex 3E8 = 1000 decimal → perfect lowest
- `400` → 1024
- `7FF` → 2047
- `999` → 2457 (still decimal valid but we want hex path anyway)
- `FFF` → 4095

I just threw

```
3E8
```

at the nc server and…

```
Access granted: }af5ftc_oc_ip7128ftc_oc_ipf_99ftc_oc_ip9_TGftc_oc_ip_xehftc_oc_ip_tigftc_oc_ipid_3ftc_oc_ip{FTCftc_oc_ipocipftc_oc_ip
```

Bruh what is this cursed string 😭

---

### Step 3 – reverse + clean = flag (the hint was literal)

Just reverse the whole garbage:

```bash
echo '}af5ftc_oc_ip7128ftc_oc_ipf_99ftc_oc_ip9_TGftc_oc_ip_xehftc_oc_ip_tigftc_oc_ipid_3ftc_oc_ip{FTCftc_oc_ipocipftc_oc_ip' | rev
```

Got:

```
picoctf{3_digit_pi_co_ctfhex_pi_co_ctfGT_9pi_co_ctf99_fpi_co_ctf8217pi_co_ctf5fa}
```

Then remove every `pi_co_ctf` (they're just noise added by `reveal_flag` probably)

Final cleaned:

```
picoctf{3_digit_hex_GT_999_f82175fa}
```

---

> **Callout – quick summary for people who skip to the end**

Input that works: **any 3-char hex** that == 1000–9999 decimal  
Examples: `3E8`, `400`, `7D0`, `AAA`, `FFF`  
Then: take the cursed output → reverse it → delete all `pi_co_ctf` → profit

> **Callout – pro tip**

You don’t even need Ghidra/IDA if you just notice:

- it wants length == 3
- it accepts hex
- range 1000–9999

Just try `3E8` and you’re basically done in 20 seconds lol

---

Flag:  
**`picoctf{3_digit_hex_GT_999_f82175fa}`**

thanks to the bro who said 3E8 first ..... legend 🙏  
see you in the next reversing hell
