# Enumeration 
```
# Trying zone transfer
nmap -sVC -p53 --script=dns-transfer-zone $IP

# DNS Zone Transfer using dig
dig axfr domain.htb @IP

# Using DNSenum
dnsenum 10.10.10.100

# DNSenum + bruteforce potential usernames
dnsenum --dnsserver @IP --enum -p 0 -s 0 -o subdomains.txt -f /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt domain.htb

# Using Fierce
fierce --domain zonetransfer.me
```

What is a [zone transfer](https://academy.hackthebox.com/module/144/section/1255)


# Footprinting using dig :

`dig ns inlanefreight.htb @10.129.14.128` // Get the known nameservers of the domain

`dig CH TXT version.bind 10.129.120.85` // Query the DNS server’s version

`dig any inlanefreight.htb @10.129.14.128` // View all available records

`dig axfr inlanefreight.htb @10.129.14.128` // Perform a zone transfer

`dig axfr internal.inlanefreight.htb @10.129.14.128` // Same using a subnet (may show internal IP and hostnames)

`for sub in $(cat /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\\|SOA' | sed -r '/^\\s*$/d' | grep $sub | tee -a subdomains.txt;done` // Try to bruteforce hostnames


# Querying Records :

### **Querying: A Records**

`export TARGET="facebook.com"`

`nslookup $TARGET` // Ask DNS servers for information about the host

`dig facebook.com @1.1.1.1` // We can here specify a precise DNS server using dig

```
;; ANSWER SECTION:
facebook.com.           169     IN      A       31.13.92.36
```

The entry may be held in the cache for `169` seconds before the information must be requested again. The class is understandably the Internet (`IN`).

### **Querying: A Records for a Subdomain**

`export TARGET=www.facebook.com`

`nslookup -query=A $TARGET $IP`

`dig a www.facebook.com @1.1.1.1`

```
;; ANSWER SECTION:
www.facebook.com.       3585    IN      CNAME   star-mini.c10r.facebook.com.
star-mini.c10r.facebook.com. 45 IN      A       31.13.92.36
```

### **Querying: PTR Records for an IP Address (reverse lookup)**

`nslookup -query=PTR 31.13.92.36`

`dig -x 31.13.92.36 @1.1.1.1`

```
;; ANSWER SECTION:
36.92.13.31.in-addr.arpa. 1028  IN      PTR     edge-star-mini-shv-01-frt3.facebook.com.
```

### **Querying: TXT Records**

`export TARGET="facebook.com"`

`nslookup -query=TXT $TARGET $IP`

`dig txt facebook.com @1.1.1.1`

```
;; ANSWER SECTION:
facebook.com.           86400   IN      TXT     "v=spf1 redirect=_spf.facebook.com"
facebook.com.           7200    IN      TXT     "google-site-verification=A2WZWCNQHrGV_TWwKh6KHY90tY0SHZo_RnyMJoDaG0s"
facebook.com.           7200    IN      TXT     "google-site-verification=wdH5DTJTc9AYNwVunSVFeK0hYDGUIEOGb-RReU6pJlY"
```

### **Querying: MX Records**

`export TARGET="facebook.com"`

`nslookup -query=MX $TARGET $IP`

`dig mx facebook.com @1.1.1.1` `dig mx facebook.com | grep "MX" | grep -v ";"` `host -t MX facebook.com`

```
;; ANSWER SECTION:
facebook.com.           3600    IN      MX      10 smtpin.vvv.facebook.com.
```

Then we can retrive the IP of the new found mail server using `A Records` :

```bash
host -t A smtpin.vvv.facebook.com
smtpin.vvv.facebook.com has address 10.129.14.128
```

### **Querying: ANY Existing Records**

`export TARGET="google.com"`

`nslookup -query=ANY $TARGET $IP`

`dig any google.com @8.8.8.8`

```
;; ANSWER SECTION:
google.com.             249     IN      A       142.250.184.206
google.com.             249     IN      AAAA    2a00:1450:4001:830::200e
google.com.             549     IN      MX      10 aspmx.l.google.com.
google.com.             3549    IN      TXT     "apple-domain-verification=30afIBcvSuDV2PLX"
google.com.             3549    IN      TXT     "facebook-domain-verification=22rm551cu4k0ab0bxsw536tlds4h95"
google.com.             549     IN      MX      20 alt1.aspmx.l.google.com.
google.com.             3549    IN      TXT     "docusign=1b0a6754-49b1-4db5-8540-d2c12664b289"
google.com.             3549    IN      TXT     "v=spf1 include:_spf.google.com ~all"
google.com.             3549    IN      TXT     "globalsign-smime-dv=CDYX+XFHUw2wml6/Gb8+59BsH31KzUr6c1l2BPvqKX8="
google.com.             3549    IN      TXT     "google-site-verification=wD8N7i1JTNTkezJ49swvWW48f8_9xveREV4oB-0Hf5o"
google.com.             9       IN      SOA     ns1.google.com. dns-admin.google.com. 403730046 900 900 1800 60
google.com.             21549   IN      NS      ns1.google.com.
google.com.             21549   IN      NS      ns3.google.com.
google.com.             549     IN      MX      50 alt4.aspmx.l.google.com.
google.com.             3549    IN      TXT     "docusign=05958488-4752-4ef2-95eb-aa7ba8a3bd0e"
google.com.             549     IN      MX      30 alt2.aspmx.l.google.com.
google.com.             21549   IN      NS      ns2.google.com.
google.com.             21549   IN      NS      ns4.google.com.
google.com.             549     IN      MX      40 alt3.aspmx.l.google.com.
google.com.             3549    IN      TXT     "MS=E4A68B9AB2BB9670BCE15412F62916164C0B20BB"
google.com.             3549    IN      TXT     "google-site-verification=TV9-DBe4R80X4v0M4U_bd_J9cpOJM0nikft0jAgjmsQ"
google.com.             21549   IN      CAA     0 issue "pki.goog"
```

The more recent [RFC8482](https://tools.ietf.org/html/rfc8482) specified that `ANY` DNS requests be abolished. Therefore, we may not receive a response to our `ANY` request from the DNS server or get a reference to the said RFC8482.

```
;; ANSWER SECTION:
cloudflare.com.         2747    IN      HINFO   "RFC8482" ""
```

### **Querying: Determine if our target uses hosting providers with the `WHOIS` database**

`export TARGET="facebook.com"`

`nslookup $TARGET`

```
Non-authoritative answer:
Name:	facebook.com
Address: 157.240.199.35
Name:	facebook.com
Address: 2a03:2880:f15e:83:face:b00c:0:25de
```

`whois 157.240.199.35`

### Using DNSRECON:

```bash
Bailly@htb[/htb]$ dnsrecon -d inlanefreight.com
[*] std: Performing General Enumeration against: inlanefreight.com...
[-] DNSSEC is not configured for inlanefreight.com
[*]      SOA ns-161.awsdns-20.com 205.251.192.161
[*]      SOA ns-161.awsdns-20.com 2600:9000:5300:a100::1
[*]      NS ns1.inlanefreight.com 178.128.39.165
[*]      NS ns2.inlanefreight.com 206.189.119.186
[*]      A inlanefreight.com 134.209.24.248
[*]      AAAA inlanefreight.com 2a03:b0c0:1:e0::32c:b001
[*]      SPF v=spf1 include:_spf.google.com include:mail1.inlanefreight.com include:google.com ~all
[*]      TXT inlanefreight.com HTB{5Fz6UPNUFFzqjdg0AzXyxCjMZ}
[*]      TXT _dmarc.inlanefreight.com v=DMARC1; p=reject; rua=mailto:master@inlanefreight.com; ruf=mailto:master@inlanefreight.com; fo=1;
[*] Enumerating SRV Records
[+] 0 Records Found
```

### Using DNSdumpster

The website `DNSdumpster` allowing us to have graphs mapping of the domain :

[DNSdumpster.com - dns recon and research, find and lookup dns records](https://dnsdumpster.com/)

# Domain Takeovers & Subdomain Enumeration

`Domain takeover` is registering a non-existent domain name to gain control over another domain. If attackers find an expired domain, they can claim that domain to perform further attacks such as hosting malicious content on a website or sending a phishing email leveraging the claimed domain.

Domain takeover is also possible with subdomains called `subdomain takeover`. A DNS's canonical name (`CNAME`) record is used to map different domains to a parent domain. Many organizations use third-party services like AWS, GitHub, Akamai, Fastly, and other content delivery networks (CDNs) to host their content. In this case, they usually create a subdomain and make it point to those services. For example :

```
sub.target.com.   60   IN   CNAME   anotherdomain.com
```

The domain name (e.g., `sub.target.com`) uses a CNAME record to another domain (e.g., `anotherdomain.com`). Suppose the `anotherdomain.com` expires and is available for anyone to claim the domain since the `target.com`'s DNS server has the `CNAME` record. In that case, anyone who registers `anotherdomain.com` will have complete control over `sub.target.com` until the DNS record is updated

### Subdomain Enumeration

```shell-session
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done
```

Before performing a subdomain takeover, we should enumerate subdomains for a target domain using tools like [Subfinder](https://github.com/projectdiscovery/subfinder). This tool can scrape subdomains from open sources like [DNSdumpster](https://dnsdumpster.com/). Other tools like [Sublist3r](https://github.com/aboul3la/Sublist3r) can also be used to brute-force subdomains by supplying a pre-generated wordlist :

```bash
# Need Internet Access
./subfinder -d inlanefreight.com -v       
...snip...
[bufferover] Source took 2.193235338s for enumeration
ns2.inlanefreight.com
www.inlanefreight.com
ns1.inlanefreight.com
support.inlanefreight.com
[INF] Found 4 subdomains for inlanefreight.com in 20 seconds 11 milliseconds
```

- An excellent alternative is a tool called [Subbrute](https://github.com/TheRook/subbrute). This tool allows us to use self-defined resolvers and perform pure DNS brute-forcing attacks during internal penetration tests on hosts that do not have Internet access

```bash
Bailly@htb[/htb]$ git clone https://github.com/TheRook/subbrute.git >> /dev/null 2>&1
Bailly@htb[/htb]$ cd subbrute
Bailly@htb[/htb]$ echo "ns1.inlanefreight.com" > ./resolvers.txt
Bailly@htb[/htb]$ ./subbrute inlanefreight.com -s ./names.txt -r ./resolvers.txt

Warning: Fewer than 16 resolvers per process, consider adding more nameservers to resolvers.txt.
inlanefreight.com
ns2.inlanefreight.com
www.inlanefreight.com
ms1.inlanefreight.com
support.inlanefreight.com

<SNIP>
```

The tool has found four subdomains associated with `inlanefreight.com`. Using the `nslookup` or `host` command, we can enumerate the `CNAME` records for those subdomains :

```bash
Bailly@htb[/htb]$ host support.inlanefreight.com

support.inlanefreight.com is an alias for inlanefreight.s3.amazonaws.com
```

The `support` subdomain has an alias record pointing to an AWS S3 bucket. However, the URL `https://support.inlanefreight.com` shows a `NoSuchBucket` error indicating that the subdomain is potentially vulnerable to a subdomain takeover.
Now, we can take over the subdomain by creating an AWS S3 bucket with the same subdomain name.

![[s3.webp]]

The [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz) repository is also an excellent reference for a subdomain takeover vulnerability. It shows whether the target services are vulnerable to a subdomain takeover and provides guidelines on assessing the vulnerability.

# DNS Spoofing :

DNS spoofing = DNS Cache Poisoning. This attack involves altering legitimate DNS records with false information so that they can be used to redirect online traffic to a fraudulent website.

- An attacker could intercept the communication between a user and a DNS server to route the user to a fraudulent destination instead of a legitimate one by performing a Man-in-the-Middle (`MITM`) attack.
- Exploiting a vulnerability found in a DNS server could yield control over the server by an attacker to modify the DNS records.

### Local DNS Cache Poisoning

We can perform DNS Cache Poisoning using MITM tools like [Ettercap](https://www.ettercap-project.org/) or [Bettercap](https://www.bettercap.org/).

To exploit the DNS cache poisoning via `Ettercap`, we should first edit the `/etc/ettercap/etter.dns` file to map the target domain name (e.g., `inlanefreight.com`) that they want to spoof and the attacker's IP address (e.g., `192.168.225.110`) that they want to redirect a user to :

```bash
Bailly@htb[/htb]$ cat /etc/ettercap/etter.dns

inlanefreight.com      A   192.168.225.110
*.inlanefreight.com    A   192.168.225.110
```

Next, start the `Ettercap` tool and scan for live hosts within the network by navigating to `Hosts > Scan for Hosts`. Once completed, add the target IP address (e.g., `192.168.152.129`) to Target1 and add a default gateway IP (e.g., `192.168.152.2`) to Target2.

![[target.webp]]

Activate `dns_spoof` attack by navigating to `Plugins > Manage Plugins`. This sends the target machine with fake DNS responses that will resolve `inlanefreight.com` to IP address `192.168.225.110`:

![[etter_plug.webp]]

After a successful DNS spoof attack, if a victim user coming from the target machine `192.168.152.129` visits the `inlanefreight.com` domain on a web browser, they will be redirected to a `Fake page` that is hosted on IP address `192.168.225.110`:

![[etter_site.webp]]

In addition, if the victim ping the domain `inlanefreight.htb` the ip `192.168.225.110` should respond.

# Dangerous Settings :

[DNS Most popular attacks](https://securitytrails.com/blog/most-popular-types-dns-attacks)

`/etc/bind/named.conf.local`

|Dangerous settings|**Description**|
|---|---|
|`allow-query`|Defines which hosts are allowed to send requests to the DNS server.|
|`allow-recursion`|Defines which hosts are allowed to send recursive requests to the DNS server.|
|`allow-transfer`|Defines which hosts are allowed to receive zone transfers from the DNS server.|
|`zone-statistics`|Collects statistical data of zones.|

Don’t forget to use `DNS over TLS` (`DoT`) or `DNS over HTTPS` (`DoH`) when it’s possible

# DNS Records

| **DNS Record** | **Description**                                                                                                                                                                                                                                   |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `A`            | Returns an IPv4 address of the requested domain as a result.                                                                                                                                                                                      |
| `AAAA`         | Returns an IPv6 address of the requested domain.                                                                                                                                                                                                  |
| `MX`           | Returns the responsible mail servers as a result.                                                                                                                                                                                                 |
| `NS`           | Returns the DNS servers (nameservers) of the domain.                                                                                                                                                                                              |
| `TXT`          | This record can contain various information. The all-rounder can be used, e.g., to validate the Google Search Console or validate SSL certificates. In addition, SPF and DMARC entries are set to validate mail traffic and protect it from spam. |
| `CNAME`        | This record serves as an alias. If the domain [www.hackthebox.eu](http://www.hackthebox.eu) should point to the same IP, and we create an A record for one and a CNAME record for the other.                                                      |
| `PTR`          | The PTR record works the other way around (reverse lookup). It converts IP addresses into valid domain names.                                                                                                                                     |
| `SOA`          | Provides information about the corresponding DNS zone and email address of the administrative contact. The at sign (@) is replaced by a dot (.) in the email address                                                                              |
| `SRV`          | Service records                                                                                                                                                                                                                                   |
| `HINFO`        | Host information                                                                                                                                                                                                                                  |