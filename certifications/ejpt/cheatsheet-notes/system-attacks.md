---
description: Hashes and Passwords
---

# System Attacks

## John the Ripper

### Unshadow

`passwd` is the **/etc/passwd** file \
`shadow` is the **/etc/shadow** file

```
unshadow passwd shadow > hash
```

### Hash Cracking

```
john --wordlist=/etc/john/rockyou.txt hash
```

​
