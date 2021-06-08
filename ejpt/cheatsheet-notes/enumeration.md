---
description: Information Gathering and Footprinting & Scanning
---

# Enumeration

## whois <a id="whois"></a>

```text
whois foo.bar
```

## subdomains <a id="subdomains"></a>

```text
sublit3r -d foo.bar
```

## ping sweeps <a id="ping-sweeps"></a>

```text
fping -a -g 10.10.10.0/24
```

```text
nmap -sS -n 10.10.10.0/24
```

## Nmap <a id="nmap"></a>

### OS Fingerprinting <a id="os-fingerprinting"></a>

```text
nmap -Pn -A -O 10.10.10.10
```

### Quick scan  <a id="quick-scan"></a>

```text
nmap -sC -sV -A -T4 10.10.10.10 --open
```

### Full scan <a id="full-scan"></a>

```text
nmap -sC -sV -A -T4 -p- 10.10.10.10 --open
```

