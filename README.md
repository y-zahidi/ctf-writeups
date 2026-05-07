# ctf-writeups

My walkthroughs of TryHackMe / HackTheBox / boot2root machines and CTF challenges. Methodology over flags — every writeup is built around the same skeleton so the *thinking* is what's reusable, not the specific exploit.

If you're looking for a copy-paste exploit, this isn't the right repo. If you're looking for the reasoning chain that gets you from "open port" to "root shell", read on.

## How I structure a writeup

Every machine here follows the same template ([`TEMPLATE.md`](TEMPLATE.md)):

1. **Box card** — platform, OS, difficulty, release date, tags
2. **Recon** — nmap output and what jumped out
3. **Enumeration** — per-service investigation, with the actual commands I ran
4. **Foothold** — the path from anonymous to first shell
5. **Privilege escalation** — first shell to root
6. **Lessons learned** — the *one thing* I want to remember the next time I see this pattern

The "lessons learned" section is the whole point. I write it last, after I've rooted the box, and I try to express it as a generalisable rule (e.g. "any time SMB null-session works, run `nxc smb -u '' -p '' --shares` *before* anything else") rather than a box-specific note.

## Index

### TryHackMe

| Box | OS | Difficulty | Tags | Writeup |
|---|---|---|---|---|
| [Blue](tryhackme/blue) | Windows 7 | Easy | EternalBlue · MS17-010 · SMB | [`tryhackme/blue/README.md`](tryhackme/blue/README.md) |

### HackTheBox

| Box | OS | Difficulty | Tags | Writeup |
|---|---|---|---|---|
| [Lame](hackthebox/lame) | Linux | Easy | Samba · CVE-2007-2447 · userland-RCE | [`hackthebox/lame/README.md`](hackthebox/lame/README.md) |

> *More boxes will land here as I work through them. The two above are deliberate "first writeup" picks — they're old, they're documented to death, and the point is to demonstrate the writeup format more than the boxes themselves.*

## Conventions

- `$BOX` — IP of the target machine
- `$ATTACKER` — IP of my Kali / attacker box on the lab network (typically `tun0`)
- `<flag>` — redacted flag content (the writeup never publishes the actual flag value)
- Paths in code blocks are absolute when they matter

## Why bother writing these up

Three reasons:

1. **Forces structured thinking.** If I can't articulate why I tried a thing, I probably got there by accident, and I won't get there a second time.
2. **Searchable second brain.** When I see the same Samba banner on a different box six months later, I can grep my own notes faster than the internet.
3. **Honest portfolio signal.** A walkthrough I wrote is verifiable evidence I actually ran the commands.

## Companion repos

- **[pentest-cheatsheet](https://github.com/y-zahidi/pentest-cheatsheet)** — the recipe book the writeups call back to
- **[home-lab-siem](https://github.com/y-zahidi/home-lab-siem)** — the defensive side of the same skill set

## License

[MIT](LICENSE).

## Disclaimer

Every box covered here was rooted on the platform that hosts it (TryHackMe / HackTheBox), with explicit authorisation from that platform. Don't reproduce any of these techniques against systems you don't own or don't have written permission to test.
