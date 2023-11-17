---
description: Information Gathering and Footprinting & Scanning
---

# Enumeration

## whois <a href="#whois" id="whois"></a>

```
whois foo.bar
```

## subdomains <a href="#subdomains" id="subdomains"></a>

```
sublit3r -d foo.bar
```

## ping sweeps <a href="#ping-sweeps" id="ping-sweeps"></a>

```
fping -a -g 10.10.10.0/24
```

```
nmap -sS -n 10.10.10.0/24
```

## Nmap <a href="#nmap" id="nmap"></a>

### OS Fingerprinting <a href="#os-fingerprinting" id="os-fingerprinting"></a>

```
nmap -Pn -A -O 10.10.10.10
```

### Quick scan  <a href="#quick-scan" id="quick-scan"></a>

```
nmap -sC -sV -A -T4 10.10.10.10 --open
```

### Full scan <a href="#full-scan" id="full-scan"></a>

```
nmap -sC -sV -A -T4 -p- 10.10.10.10 --open
```
