# Pass-The-Hash Windows

In most case of leathral movment, we usually don't have password. 
This technique, allows us to authenticate to a remote target by using a valid username and NTLM/LM hash rather than username and password.

Windows hash format:

```
user:group:id:ntlmpassword::
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


Our final NTLM hash for attack will be *aad3c435b514a4eeaad3b935b51304fe*
