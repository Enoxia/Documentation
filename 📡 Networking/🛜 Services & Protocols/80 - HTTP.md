# Enumeration & Checks
```
nmap -sVC -p80 --script=http-* $IP

# Metasploit modules
msf6> use auxiliary/scanner/http/http_*

# IIS
nmap -Pn -n -T3 -v -sV --version-intensity=5 -Pn -p 80 --script=http-iis-webdav-vuln <IP>

# JBOSS (CVE-2010-0738)
nmap -Pn -n -T3 -v -sV --version-intensity=5 -Pn -p 80 --script=http-vuln-cve2010-0738 <IP>

# PHP-CGI (CVE-2012-1823)
nmap -Pn -n -T3 -v -sV --version-intensity=5 -Pn -p 80 --script=http-vuln-cve2012-1823 <IP>

# RCE Ruby on Rails (CVE-2013-0156)
nmap -Pn -n -T3 -v -sV --version-intensity=5 -Pn -p 80 --script=http-vuln-cve2013-0156 <IP>

# WAF Detection
nmap -Pn -n -T3 -v -sV --version-intensity=5 -Pn -p 80 --script=http-waf-detect,http-waf-fingerprint <IP>

whatweb $IP
```

# Others things to do
```
# View from browser in terminal if no GUI
browsh --startup-url http://$IP/
lynx http://$IP/
```