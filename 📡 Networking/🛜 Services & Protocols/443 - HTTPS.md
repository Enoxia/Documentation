# Enumeration
```
# Check Heartbleed CVE-2014-0160
nmap -Pn -n -p 443 -v -T3 --script=ssl-heartbleed,ssl-enum-ciphers,ssl-known-key --script-args vulns.showall -sV --version-intensity=5 <IP>
```
