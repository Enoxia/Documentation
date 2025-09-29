# Content Discovery
```bash
# Default FFUF command for files discovery
ffuf -c -recursion -recursion-depth 1 -w /opt/useful/SecLists/Discovery/Web-Content/big.txt -u http://IP:PORT/FUZZ -t 200

# Extension Fuzzing
# Note : The wordlist used already contains a dot (.), so no need to add one in the command !
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/web-extensions.txt:FUZZ -u <http://IP>:PORT/indexFUZZ

# Page fuzzing
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u <http://IP>:PORT/FUZZ.php
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u <http://IP>:PORT/FUZZ -recursion -recursion-depth 1 -e .php

# HTTP Get Request Fuzzing
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx

# HTTP Post Request Fuzzing
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx

# Value Fuzzing
for i in $(seq 1 1000); do echo $i >> ids.txt; done
ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```
## FFUF Options
```
MATCHER OPTIONS:
  -mc                 Match HTTP status codes, or "all" for everything. (default: 200,204,301,302,307,401,403,405)
  -ml                 Match amount of lines in response
  -mr                 Match regexp
  -ms                 Match HTTP response size
  -mw                 Match amount of words in response

FILTER (exclude) OPTIONS:
  -fc                 Filter HTTP status codes from response. Comma separated list of codes and ranges
  -fl                 Filter by amount of lines in response. Comma separated list of line counts and ranges
  -fr                 Filter regexp
  -fs                 Filter (exclude) HTTP response size. Comma separated list of sizes and ranges
  -fw                 Filter by amount of words in response. Comma separated list of word counts and ranges

OTHERS:
  -c                  Colored output
  -recursion          Activates the recursive scan (to search into discovered folders)
  -recursion-depth    Specified the maximum depth (folders) to scan
  -t                  Set threads number (may cause DoS)
  -e .php             Will search extension files, here .php
```



# Subdomains Fuzzing
Don't forget to add the host to `/etc/hosts` ! Don't forget to test either via `vHosts` or `DNS` mode (HTTP header requests for subdomains under websites that are not public vs DNS request to find public subdomains using public DNS records). The difference between `VHosts` and `subdomains` is that a VHost is basically a "subdomain" served on the same server and has the same IP, such that a single IP could be serving two or more different websites. A subdomain could have a different IP than the original domain.
```bash
# Add the host to the /etc/hosts file
echo "10.10.10.10 academy.htb" >> /etc/hosts

# Subdomains Fuzzing
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.inlanefreight.com

# Vhosts Fuzzing
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://IP:PORT/ -H "Host: FUZZ.academy.htb" -fs 10918

# Subdomains Fuzzing w/ Gobuster
gobuster dns -d inlanefreight.com -w /usr/share/SecLists/Discovery/DNS/namelist.txt

# Vhosts Fuzzing w/ Gobuster
gobuster vhost -u http://host.com -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```
## OSINT Enumeration
- [Sudomy](https://github.com/screetsec/Sudomy) is a subdomain enumeration tool to collect subdomains and analyzing domains performing advanced automated reconnaissance, that can be used for OSINT (Open-source intelligence) activities.
- [Sublist3r](https://github.com/aboul3la/Sublist3r) is a python tool designed to enumerate subdomains of websites using OSINT. It enumerates subdomains using many search engines such as Google, Yahoo, Bing, Baidu and Ask. Sublist3r also enumerates subdomains using Netcraft, Virustotal, ThreatCrowd, DNSdumpster and ReverseDNS.
## DNS Enumeration
DNSenum is a comprehensive toolkit for DNS reconnaissance, allowing us to enumerate DNS records, perform zone transfer, bruteforce subdomains, google scraping, reverse lookup, and whois lookup. More info in [[53 - DNS]]
```bash
# Enumerating domain & subdomains bruteforcing 
dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -r

# Request zone transfer
dig axfr @nsztm1.digi.ninja zonetransfer.me
```
