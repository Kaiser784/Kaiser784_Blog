---
description: These are condensed notes from the INE course and other blogs and articles.
---

# Cheatsheet/Notes

## Subnet Guide

![TheCyberMentor&apos;s subnet guide](../../../.gitbook/assets/subnet-guide-sheet1.png)

<table>
  <thead>
    <tr>
      <th style="text-align:left">Protocol</th>
      <th style="text-align:left">Port</th>
      <th style="text-align:left">Service</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">TCP</td>
      <td style="text-align:left">21</td>
      <td style="text-align:left">FTP</td>
    </tr>
    <tr>
      <td style="text-align:left">TCP</td>
      <td style="text-align:left">22</td>
      <td style="text-align:left">SSH</td>
    </tr>
    <tr>
      <td style="text-align:left">TCP</td>
      <td style="text-align:left">25</td>
      <td style="text-align:left">SMTP</td>
    </tr>
    <tr>
      <td style="text-align:left">TCP</td>
      <td style="text-align:left">53</td>
      <td style="text-align:left">DNS</td>
    </tr>
    <tr>
      <td style="text-align:left">TCP</td>
      <td style="text-align:left">80, 443</td>
      <td style="text-align:left">HTTP/HTTPS webserver</td>
    </tr>
    <tr>
      <td style="text-align:left">TCP</td>
      <td style="text-align:left">137-139</td>
      <td style="text-align:left">Windows NetBIOS</td>
    </tr>
    <tr>
      <td style="text-align:left">TCP</td>
      <td style="text-align:left">445</td>
      <td style="text-align:left">
        <p>Windows - SMB</p>
        <p>Linux - Samba service</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">TCP</td>
      <td style="text-align:left">1433, 1434</td>
      <td style="text-align:left">MSSQL Database</td>
    </tr>
    <tr>
      <td style="text-align:left">TCP</td>
      <td style="text-align:left">3306</td>
      <td style="text-align:left">MySQL Database</td>
    </tr>
    <tr>
      <td style="text-align:left">TCP</td>
      <td style="text-align:left">8080, 8443</td>
      <td style="text-align:left">HTTP(s) web server, HTTP proxy</td>
    </tr>
  </tbody>
</table>

## Routing

{% hint style="danger" %}
This part is pretty important😉 
{% endhint %}

```text
ip route add 172.10.1.0/24 via 10.10.10.10(VPN Gateway)
```

If you don't know what the VPN gateway is then try it with every host that is up in the network.

If you want external sources for routing/pivoting, you can look at this one I found somewhere in the depths of Reddit \(Don't ask me why Shaq is on it\). 

{% embed url="https://pentest.blog/explore-hidden-networks-with-double-pivoting/" %}

## Interactive Shells

```text
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

| Server | Web Shell |
| :--- | :--- |
| Windows | ASP |
| Apache TomCat | JSP |
| Apache | PHP |

### Reverse Shells

[Pentest Monkey Cheatsheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

## Hacktricks

If you're stuck at vulnerability assessment or don't know how to tackle something you can check here for some kind of push.

{% embed url="https://book.hacktricks.xyz/" %}
