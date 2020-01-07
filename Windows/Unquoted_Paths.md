# Unquoted paths

If we find a service running with an unquoted path and spaces in the path we can hijack the path and use it to elevate privileges.

Find vulnerable services using the command:

```
$ wmic service get name,displayname,pathname,startmode |findstr /i "Auto" |findstr /i /v "C:\Windows\\" |findstr /i /v """
```

For example:

```
C:\Program Files\something\xampp.exe
```

We could place our payload with any of the following paths:

```
C:\Program.exe

C:\Program Files.exe

```

In some case, we might able to overwrite the service. So check out permissions:

```
icacls "C:\Program Files (x86)\Program Folder"
```

If wmic is not available we can use sc:

```
sc qc xampp
```

We need to see BUILTIN\Users permissions with (F) / (C) / (M) for our group.
