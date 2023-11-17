---
description: These are condensed notes from the INE course and other blogs and articles.
---

# Cheatsheet/Notes

## Subnet Guide

![TheCyberMentor's subnet guide](<../../../.gitbook/assets/Subnet-Guide - Sheet1.png>)

| Protocol | Port       | Service                                                 |
| -------- | ---------- | ------------------------------------------------------- |
| TCP      | 21         | FTP                                                     |
| TCP      | 22         | SSH                                                     |
| TCP      | 25         | SMTP                                                    |
| TCP      | 53         | DNS                                                     |
| TCP      | 80, 443    | HTTP/HTTPS webserver                                    |
| TCP      | 137-139    | Windows NetBIOS                                         |
| TCP      | 445        | <p>Windows - SMB</p><p>Linux        - Samba service</p> |
| TCP      | 1433, 1434 | MSSQL Database                                          |
| TCP      | 3306       | MySQL Database                                          |
| TCP      | 8080, 8443 | HTTP(s) web server, HTTP proxy                          |

## Routing

{% hint style="danger" %}
This part is pretty important:wink:&#x20;
{% endhint %}

```
ip route add 172.10.1.0/24 via 10.10.10.10(VPN Gateway)
```

If you don't know what the VPN gateway is then try it with every host that is up in the network.

If you want external sources for routing/pivoting, you can look at this one I found somewhere in the depths of Reddit (Don't ask me why Shaq is on it).&#x20;

{% embed url="https://pentest.blog/explore-hidden-networks-with-double-pivoting/" %}

## Interactive Shells

```
bash -i
python -c 'import pty; pty.spawn("/bin/sh")'
echo os.system('/bin/bash')
/bin/sh -i
perl -e 'exec "/bin/sh";'
perl: exec "/bin/sh";
ruby: exec "/bin/sh"
lua: os.execute('/bin/sh')
```

## Recommended Shells

| Server        | Web Shell |
| ------------- | --------- |
| Windows       | ASP       |
| Apache TomCat | JSP       |
| Apache        | PHP       |

### Reverse Shells

[Pentest Monkey Cheatsheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

## Hacktricks

If you're stuck at vulnerability assessment or don't know how to tackle something you can check here for some kind of push.

{% embed url="https://book.hacktricks.xyz/" %}
