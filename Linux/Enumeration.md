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





