# [Box name] — [Platform]

| Field | Value |
|---|---|
| Platform | TryHackMe / HackTheBox / VulnHub / OffSec |
| OS | Linux / Windows |
| Difficulty | Easy / Medium / Hard / Insane |
| Release | YYYY-MM-DD |
| Tags | `web` `ad` `lfi` `kerberoast` `cve-XXXX-YYYY` |
| Rooted on | YYYY-MM-DD |

## TL;DR

Two-line summary of the path: *anonymous → user → root*.

## 1. Recon

```bash
sudo nmap -sCV -p- --min-rate 1000 -oN nmap.txt $BOX
```

```
PORT     STATE SERVICE
...
```

What jumps out:

- Service X on port Y because …
- The version banner of Z is older than the public CVE for it, look that up

## 2. Enumeration

### Service X (port Y)

What I tried, in order:

```bash
# command 1
# command 2
```

What I saw, what I learned, what I crossed off the list.

### Service Z (port W)

(repeat per service)

## 3. Foothold

The exploit chain that got the first shell. Be explicit about *why* this exploit worked here vs. why I picked it over alternatives.

```bash
# the actual command(s)
```

User flag:

```
<flag>
```

## 4. Privilege escalation

```bash
# linpeas / winPEAS / manual checks
```

The privesc primitive (sudo misconfig, SUID, kernel CVE, AD ACL, …), why it's exploitable, and the exact steps.

Root flag:

```
<flag>
```

## 5. Lessons learned

> **Generalisable rule** — express the takeaway as a pattern, not as a box-specific fact. *"Any time `nmap` shows X, immediately do Y before anything else."*

- Lesson 1
- Lesson 2

## Tools used

| Tool | Phase | One-liner |
|---|---|---|
| nmap | Recon | `nmap -sCV -p-` |
| ffuf | Enum | `ffuf -u http://$BOX/FUZZ -w …` |
| … | … | … |

## References

- [Official write-up by …](#)
- [HackTricks page on …](#)
- [CVE-XXXX-YYYY](#)
