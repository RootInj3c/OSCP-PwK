# OSCP Prep Notes

Welcome to my learning notes for OSCP by OffensiveSecurity. I will update the metrials here during my 30 days course time.

## Taking Proof files

### Windows

```
hostname && whoami && type C:\Documents and Settings\Administrator\Desktop\proof.txt && ipconfig /all
```

### Linux

```
hostname && whoami && cat /root/proof.txt && /sbin/ifconfig
```

## Methodology

1) Ports & Services Enumeration using NMAP

```
nmap -sV -sC -p- <IP>
nmap -sU --top-ports=50 <IP>
nmap --script smb-vuln-* <IP>
nmap -sV -T4 -p- <IP> (Quick method)
```

2) Enumerate web application

```
gobuster dir -u <url> -w /usr/share/wordlist/directory-2.3-meduim.txt
nikito -u <url>
```

3) Post Exploition

After gaining inital acsses to shell and prompt TTY shell, then running one of the following script to enumrate:

Windows:

- Powerup - https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc
- JAWS - https://github.com/411Hall/JAWS

Linux:

- LinEnum.sh - https://github.com/rebootuser/LinEnum
- LSE - https://github.com/diego-treitos/linux-smart-enumeration

If none of them works, move to manully search.

4) Priviliage esclation

Depend on the attack vector preform priviliage esclation.
