# Automating Recon
These frameworks aim to provide a complete suite of tools for web reconnaissance :
- [FinalRecon](https://github.com/thewhiteh4t/FinalRecon): A Python-based reconnaissance tool offering a range of modules for different tasks like SSL certificate checking, Whois information gathering, header analysis, and crawling. Its modular structure enables easy customisation for specific needs.
- [Recon-ng](https://github.com/lanmaster53/recon-ng): A powerful framework written in Python that offers a modular structure with various modules for different reconnaissance tasks. It can perform DNS enumeration, subdomain discovery, port scanning, web crawling, and even exploit known vulnerabilities.
- [theHarvester](https://github.com/laramies/theHarvester): Specifically designed for gathering email addresses, subdomains, hosts, employee names, open ports, and banners from different public sources like search engines, PGP key servers, and the SHODAN database. It is a command-line tool written in Python.
- [SpiderFoot](https://github.com/smicallef/spiderfoot): An open-source intelligence automation tool that integrates with various data sources to collect information about a target, including IP addresses, domain names, email addresses, and social media profiles. It can perform DNS lookups, web crawling, port scanning, and more.
- [OSINT Framework](https://osintframework.com/): A collection of various tools and resources for open-source intelligence gathering. It covers a wide range of information sources, including social media, search engines, public records, and more.
- [HTTP Observatory](https://developer.mozilla.org/en-US/observatory) : Developed by Mozilla, the ⁨HTTP Observatory⁩ performs an in-depth assessment of a site’s HTTP headers and other key security configurations.
## Focus on FinalRecon
`FinalRecon` offers a wealth of recon information:
- `Header Information`: Reveals server details, technologies used, and potential security misconfigurations.
- `Whois Lookup`: Uncovers domain registration details, including registrant information and contact details.
- `SSL Certificate Information`: Examines the SSL/TLS certificate for validity, issuer, and other relevant details.
- `Crawler`:
    - HTML, CSS, JavaScript: Extracts links, resources, and potential vulnerabilities from these files.
    - Internal/External Links: Maps out the website's structure and identifies connections to other domains.
    - Images, robots.txt, sitemap.xml: Gathers information about allowed/disallowed crawling paths and website structure.
    - Links in JavaScript, Wayback Machine: Uncovers hidden links and historical website data.
- `DNS Enumeration`: Queries over 40 DNS record types, including DMARC records for email security assessment.
- `Subdomain Enumeration`: Leverages multiple data sources ([crt.sh](http://crt.sh), AnubisDB, ThreatMiner, CertSpotter, Facebook API, VirusTotal API, Shodan API, BeVigil API) to discover subdomains.
- `Directory Enumeration`: Supports custom wordlists and file extensions to uncover hidden directories and files.
- `Wayback Machine`: Retrieves URLs from the last five years to analyse website changes and potential vulnerabilities.



# WHOIS
WHOIS is a widely used query and response protocol designed to access databases that store information about registered internet resources. Consider as the "white pages" for domain names. It is a TCP-based transaction-oriented query/response protocol listening on TCP port `43` by default. In simple terms, the Whois database is a searchable list of all domains currently registered worldwide.
Each WHOIS record typically contains the following information :
- `Domain Name`: The domain name itself (e.g., [example.com](http://example.com))
- `Registrar`: The company where the domain was registered (e.g., GoDaddy, Namecheap)
- `Registrant Contact`: The person or organization that registered the domain.
- `Administrative Contact`: The person responsible for managing the domain.
- `Technical Contact`: The person handling technical issues related to the domain.
- `Creation and Expiration Dates`: When the domain was registered and when it's set to expire.
- `Name Servers`: Servers that translate the domain name into an IP address.
[Whois - DomainTools](https://whois.domaintools.com/)
```
whois facebook.com
```



# DNS 
DNS is not merely a technical protocol for translating domain names; it's a critical component of a target's infrastructure that can be leveraged to uncover vulnerabilities and gain access during a penetration test :
- `Uncovering Assets`: DNS records can reveal a wealth of information, including subdomains, mail servers, and name server records. For instance, a `CNAME` record pointing to an outdated server (`dev.example.com` CNAME `oldserver.example.net`) could lead to a vulnerable system.
- `Mapping the Network Infrastructure`: You can create a comprehensive map of the target's network infrastructure by analysing DNS data. For example, identifying the name servers (`NS` records) for a domain can reveal the hosting provider used, while an `A` record for `loadbalancer.example.com` can pinpoint a load balancer. This helps you understand how different systems are connected, identify traffic flow, and pinpoint potential choke points or weaknesses that could be exploited during a penetration test.
- `Monitoring for Changes`: Continuously monitoring DNS records can reveal changes in the target's infrastructure over time. For example, the sudden appearance of a new subdomain (`vpn.example.com`) might indicate a new entry point into the network, while a `TXT` record containing a value like `_1password=...` strongly suggests the organization is using 1Password, which could be leveraged for social engineering attacks or targeted phishing campaigns.
## Digging DNS
DNS reconnaissance involves utilizing specialized tools designed to query DNS servers and extract valuable information. Here are some of the most popular and versatile tools in the arsenal of web recon professionals:

| Tool                       | Key Features                                                                                            | Use Cases                                                                                                                               |
| -------------------------- | ------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `dig`                      | Versatile DNS lookup tool that supports various query types (A, MX, NS, TXT, etc.) and detailed output. | Manual DNS queries, zone transfers (if allowed), troubleshooting DNS issues, and in-depth analysis of DNS records.                      |
| `nslookup`                 | Simpler DNS lookup tool, primarily for A, AAAA, and MX records.                                         | Basic DNS queries, quick checks of domain resolution and mail server records.                                                           |
| `host`                     | Streamlined DNS lookup tool with concise output.                                                        | Quick checks of A, AAAA, and MX records.                                                                                                |
| `dnsenum`                  | Automated DNS enumeration tool, dictionary attacks, brute-forcing, zone transfers (if allowed).         | Discovering subdomains and gathering DNS information efficiently.                                                                       |
| `fierce`                   | DNS reconnaissance and subdomain enumeration tool with recursive search and wildcard detection.         | User-friendly interface for DNS reconnaissance, identifying subdomains and potential targets.                                           |
| `dnsrecon`                 | Combines multiple DNS reconnaissance techniques and supports various output formats.                    | Comprehensive DNS enumeration, identifying subdomains, and gathering DNS records for further analysis.                                  |
| `theHarvester`             | OSINT tool that gathers information from various sources, including DNS records (email addresses).      | Collecting email addresses, employee information, and other data associated with a domain from multiple sources.                        |
| Online DNS Lookup Services | User-friendly interfaces for performing DNS lookups.                                                    | Quick and easy DNS lookups, convenient when command-line tools are not available, checking for domain availability or basic information |
## DIG Commands

|Command|Description|
|---|---|
|`dig domain.com`|Performs a default A record lookup for the domain.|
|`dig domain.com A`|Retrieves the IPv4 address (A record) associated with the domain.|
|`dig domain.com AAAA`|Retrieves the IPv6 address (AAAA record) associated with the domain.|
|`dig domain.com MX`|Finds the mail servers (MX records) responsible for the domain.|
|`dig domain.com NS`|Identifies the authoritative name servers for the domain.|
|`dig domain.com TXT`|Retrieves any TXT records associated with the domain.|
|`dig domain.com CNAME`|Retrieves the canonical name (CNAME) record for the domain.|
|`dig domain.com SOA`|Retrieves the start of authority (SOA) record for the domain.|
|`dig @1.1.1.1 domain.com`|Specifies a specific name server to query; in this case 1.1.1.1|
|`dig +trace domain.com`|Shows the full path of DNS resolution.|
|`dig -x 192.168.1.1`|Performs a reverse lookup on the IP address 192.168.1.1 to find the associated host name. You may need to specify a name server.|
|`dig +short domain.com`|Provides a short, concise answer to the query.|
|`dig +noall +answer domain.com`|Displays only the answer section of the query output.|
|`dig domain.com ANY`|Retrieves all available DNS records for the domain (Note: Many DNS servers ignore `ANY` queries to reduce load and prevent abuse, as per [RFC 8482](https://datatracker.ietf.org/doc/html/rfc8482)).|

Caution: Some servers can detect and block excessive DNS queries. Use caution and respect rate limits. Always obtain permission before performing extensive DNS reconnaissance on a target. More attacks on [[53 - DNS]] 

`To try to discover the real IP behind a WAF`, we could also try to make a `dig` to the victim to see if there is a `CNAME` redirecting to a provider (WAF). 
Once done, we can make `another dig` by adding a `random character` in front of the name. In the case where they're using a wildcard to resolve the names, we could retrieve the `real IP address` of the site, not the WAF's one.


# Subdomains
## Passive Enum
This relies on external sources of information to discover subdomains without directly querying the target's DNS servers.
- One valuable resource is `Certificate Transparency (CT) logs`, public repositories of SSL/TLS certificates.
Sometimes, SSL certificate is used for several domains, and these are most likely still active. We can search using :

|Tool|Key Features|Use Cases|Pros|Cons|
|---|---|---|---|---|
|[crt.sh](https://crt.sh/)|User-friendly web interface, simple search by domain, displays certificate details, SAN entries.|Quick and easy searches, identifying subdomains, checking certificate issuance history.|Free, easy to use, no registration required.|Limited filtering and analysis options.|
|[Censys](https://search.censys.io/)|Powerful search engine for internet-connected devices, advanced filtering by domain, IP, certificate attributes.|In-depth analysis of certificates, identifying misconfigurations, finding related certificates and hosts.|Extensive data and filtering options, API access.|Requires registration (free tier available).|

```bash
# Fetch the JSON output from crt.sh for certificates matching the facebook.com domain, and filter the output selecting only entries where the name_value filed (which contains the domain or subdomain) includes the string "dev", ordering the results alphabetically and removing duplicates.
curl -s "https://crt.sh/?q=facebook.com&output=json" | jq -r '.[]
 | select(.name_value | contains("dev")) | .name_value' | sort -u
```
- Another method involves utilising `search engines` like Google or DuckDuckGo. By employing specialised search operators (e.g., `site:`), you can filter results to show only subdomains related to the target domain.
- [VirusTotal](https://www.virustotal.com/gui/home/upload) maintains its DNS replication service, which is developed by preserving DNS resolutions made when users visit URLs given by them. To receive information about a domain, type the domain name into the search bar and click on the "Relations" tab.
- We can automate this process using [theHarvester](https://github.com/laramies/theHarvester)
## Active Enum
Brute-forcing subdomains breaks down into four steps :
1. `Wordlist Selection`: The process begins with selecting a wordlist containing potential subdomain names. These wordlists can be:
    - `General-Purpose`: Containing a broad range of common subdomain names (e.g., `dev`, `staging`, `blog`, `mail`, `admin`, `test`). This approach is useful when you don't know the target's naming conventions.
    - `Targeted`: Focused on specific industries, technologies, or naming patterns relevant to the target. This approach is more efficient and reduces the chances of false positives.
    - `Custom`: You can create your own wordlist based on specific keywords, patterns, or intelligence gathered from other sources.
2. `Iteration and Querying`: A script or tool iterates through the wordlist, appending each word or phrase to the main domain (e.g., `example.com`) to create potential subdomain names (e.g., `dev.example.com`, `staging.example.com`).
3. `DNS Lookup`: A DNS query is performed for each potential subdomain to check if it resolves to an IP address. This is typically done using the A or AAAA record type.
4. `Filtering and Validation`: If a subdomain resolves successfully, it's added to a list of valid subdomains. Further validation steps might be taken to confirm the subdomain's existence and functionality (e.g., by attempting to access it through a web browser).

Some tools that excel at brute-force enumeration :

|Tool|Description|
|---|---|
|[dnsenum](https://github.com/fwaeytens/dnsenum)|Comprehensive DNS enumeration tool that supports dictionary and brute-force attacks for discovering subdomains.|
|[fierce](https://github.com/mschwager/fierce)|User-friendly tool for recursive subdomain discovery, featuring wildcard detection and an easy-to-use interface.|
|[dnsrecon](https://github.com/darkoperator/dnsrecon)|Versatile tool that combines multiple DNS reconnaissance techniques and offers customisable output formats.|
|[amass](https://github.com/owasp-amass/amass)|Actively maintained tool focused on subdomain discovery, known for its integration with other tools and extensive data sources.|
|[assetfinder](https://github.com/tomnomnom/assetfinder)|Simple yet effective tool for finding subdomains using various techniques, ideal for quick and lightweight scans.|
|[puredns](https://github.com/d3mondev/puredns)|Powerful and flexible DNS brute-forcing tool, capable of resolving and filtering results effectively.|



# Virtual Host
At the core of `virtual hosting` is the ability of web servers to distinguish between multiple websites or applications sharing the same IP address. This is achieved by leveraging the `HTTP Host` header, a piece of information included in every `HTTP` request sent by a web browser.
The key difference between `VHosts` and `subdomains` is their relationship to the `Domain Name System (DNS)` and the web server's configuration.
- `Subdomains`: These are extensions of a main domain name (e.g., `blog.example.com` is a subdomain of `example.com`). `Subdomains` typically have their own `DNS records`, pointing to either the same IP address as the main domain or a different one. They can be used to organise different sections or services of a website.
- `Virtual Hosts` (`VHosts`): Virtual hosts are configurations within a web server that allow multiple websites or applications to be hosted on a single server. They can be associated with top-level domains (e.g., `example.com`) or subdomains (e.g., `dev.example.com`). Each virtual host can have its own separate configuration, enabling precise control over how requests are handled.
Several tools are available to aid in the discovery of virtual hosts:

| Tool                                                 | Description                                                                                                      | Features                                                        |
| ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| [gobuster](https://github.com/OJ/gobuster)           | A multi-purpose tool often used for directory/file brute-forcing, but also effective for virtual host discovery. | Fast, supports multiple HTTP methods, can use custom wordlists. |
| [Feroxbuster](https://github.com/epi052/feroxbuster) | Similar to Gobuster, but with a Rust-based implementation, known for its speed and flexibility.                  | Supports recursion, wildcard discovery, and various filters.    |
| [ffuf](https://github.com/ffuf/ffuf)                 | Another fast web fuzzer that can be used for virtual host discovery by fuzzing the `Host` header.                | Customizable wordlist input and filtering options.              |



# Fingerprinting
This focuses on extracting technical details about the technologies powering a website or web application. This includes banner grabbing, analysing HTTP Headers, probing for specific responses, and analysing page content. A variety of tools exist that automate the fingerprinting process, combining various techniques to identify web servers, operating systems, content management systems, and other technologies :

| Tool         | Description                                                                                                                         | Features                                                                                                     |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `Wappalyzer` | Browser extension and online service for website technology profiling.                                                              | Identifies a wide range of web technologies, including CMSs, frameworks, analytics tools, and more.          |
| `BuiltWith`  | Web technology profiler that provides detailed reports on a website's technology stack.                                             | Offers both free and paid plans with varying levels of detail.                                               |
| `WhatWeb`    | Command-line tool for website fingerprinting.                                                                                       | Uses a vast database of signatures to identify various web technologies.                                     |
| `Nmap`       | Versatile network scanner that can be used for various reconnaissance tasks, including service and OS fingerprinting.               | Can be used with scripts (NSE) to perform more specialised fingerprinting.                                   |
| `Netcraft`   | Offers a range of web security services, including website fingerprinting and security reporting.                                   | Provides detailed reports on a website's technology, hosting provider, and security posture.                 |
| `wafw00f`    | Command-line tool specifically designed for identifying Web Application Firewalls (WAFs).                                           | Helps determine if a WAF is present and, if so, its type and configuration.                                  |
| `Nikto`      | Powerful open-source web server scanner.                                                                                            | Attempt to identify outdated software, insecure files or configurations, and other potential security risks. |
| `Nuclei`     | Best one, includes a vuln scanner.                                                                                                  |                                                                                                              |
| `EyeWitness` | EyeWitness can be used to take screenshots of target web applications, fingerprint them, and identify possible default credentials. |                                                                                                              |
| `BuildWith`  | Firefox extension to see what the site is made of                                                                                   |                                                                                                              |
| `Wappalyzer` | Firefox extension to see what the site is made of                                                                                   |                                                                                                              |
| `HTTrack`    | Allow us to download a website locally to browse files & shares                                                                     |                                                                                                              |
| `Aquatone`   | Automatic and visual inspection of websites across many hosts. Useful when dealing with huge subdomain lists.                       |                                                                                                              |
Some default web server installations :

| **Web Server** | **Default Webroot**    |
| -------------- | ---------------------- |
| `Apache`       | /var/www/html/         |
| `Nginx`        | /usr/local/nginx/html/ |
| `IIS`          | c:\inetpub\wwwroot     |
| `XAMPP`        | C:\xampp\htdocs        |



# Crawling
`Crawling`, often called `spidering`, is the `automated process of systematically browsing the World Wide Web`. Similar to how a spider navigates its web, a web crawler follows links from one page to another, collecting information. 
These crawlers are essentially bots that use pre-defined algorithms to discover and index web pages, making them accessible through search engines or for other purposes like data analysis and web reconnaissance.
The idea is to extract valuable information (links internal & external, comments, metadata, sensitive files..)
## Robots.txt
This file, placed in the root directory of a website, contains instructions in the form of "directives" that tell bots which parts of the website they can and cannot crawl. Common directives include:

|Directive|Description|Example|
|---|---|---|
|`Disallow`|Specifies paths or patterns that the bot should not crawl.|`Disallow: /admin/` (disallow access to the admin directory)|
|`Allow`|Explicitly permits the bot to crawl specific paths or patterns, even if they fall under a broader `Disallow` rule.|`Allow: /public/` (allow access to the public directory)|
|`Crawl-delay`|Sets a delay (in seconds) between successive requests from the bot to avoid overloading the server.|`Crawl-delay: 10` (10-second delay between requests)|
|`Sitemap`|Provides the URL to an XML sitemap for more efficient crawling.|`Sitemap: https://www.example.com/sitemap.xml`|
More info [here](https://academy.hackthebox.com/module/144/section/3077)
## Well-Known URIs
The `.well-known` standard, defined in [RFC 8615](https://datatracker.ietf.org/doc/html/rfc8615), serves as a standardized directory within a website's root domain. This designated location, typically accessible via the `/.well-known/` path on a web server, centralizes a website's critical metadata, including configuration files and information related to its services, protocols, and security mechanisms.
By establishing a consistent location for such data, `.well-known` simplifies the discovery and access process for various stakeholders, including web browsers, applications, and security tools. This streamlined approach enables clients to automatically locate and retrieve specific configuration files by constructing the appropriate URL. For instance, to access a website's security policy, a client would request `https://example.com/.well-known/security.txt`.
The `Internet Assigned Numbers Authority` (`IANA`) maintains a [registry](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml) of `.well-known` URIs, each serving a specific purpose defined by various specifications and standards. Below is a table highlighting a few notable examples:

|URI Suffix|Description|Status|Reference|
|---|---|---|---|
|`security.txt`|Contains contact information for security researchers to report vulnerabilities.|Permanent|RFC 9116|
|`/.well-known/change-password`|Provides a standard URL for directing users to a password change page.|Provisional|https://w3c.github.io/webappsec-change-password-url/#the-change-password-well-known-uri|
|`openid-configuration`|Defines configuration details for OpenID Connect, an identity layer on top of the OAuth 2.0 protocol.|Permanent|http://openid.net/specs/openid-connect-discovery-1_0.html|
|`assetlinks.json`|Used for verifying ownership of digital assets (e.g., apps) associated with a domain.|Permanent|https://github.com/google/digitalassetlinks/blob/master/well-known/specification.md|
|`mta-sts.txt`|Specifies the policy for SMTP MTA Strict Transport Security (MTA-STS) to enhance email security.|Permanent|RFC 8461|
## Popular Web Crawlers
1. `Burp Suite Spider`: Burp Suite, a widely used web application testing platform, includes a powerful active crawler called Spider. Spider excels at mapping out web applications, identifying hidden content, and uncovering potential vulnerabilities.
2. `OWASP ZAP (Zed Attack Proxy)`: ZAP is a free, open-source web application security scanner. It can be used in automated and manual modes and includes a spider component to crawl web applications and identify potential vulnerabilities.
3. `Scrapy (Python Framework)`: Scrapy is a versatile and scalable Python framework for building custom web crawlers. It provides rich features for extracting structured data from websites, handling complex crawling scenarios, and automating data processing. Its flexibility makes it ideal for tailored reconnaissance tasks.
4. `Apache Nutch (Scalable Crawler)`: Nutch is a highly extensible and scalable open-source web crawler written in Java. It's designed to handle massive crawls across the entire web or focus on specific domains. While it requires more technical expertise to set up and configure, its power and flexibility make it a valuable asset for large-scale reconnaissance projects.



# Search Engine
It is like OSINT gathering. The idea is to browse available information about target websites using search engines. It is a crucial component of web reconnaissance for several reasons : 
- `Open Source`: The information gathered is publicly accessible, making it a legal and ethical way to gain insights into a target.
- `Breadth of Information`: Search engines index a vast portion of the web, offering a wide range of potential information sources.
- `Ease of Use`: Search engines are user-friendly and require no specialised technical skills.
- `Cost-Effective`: It's a free and readily available resource for information gathering.
## Search Operators
| Operator                | Operator Description                                         | Example                                             | Example Description                                                                     |
| :---------------------- | :----------------------------------------------------------- | :-------------------------------------------------- | :-------------------------------------------------------------------------------------- |
| `site:`                 | Limits results to a specific website or domain.              | `site:example.com`                                  | Find all publicly accessible pages on example.com.                                      |
| `inurl:`                | Finds pages with a specific term in the URL.                 | `inurl:login`                                       | Search for login pages on any website.                                                  |
| `filetype:`             | Searches for files of a particular type.                     | `filetype:pdf`                                      | Find downloadable PDF documents.                                                        |
| `intitle:`              | Finds pages with a specific term in the title.               | `intitle:"confidential report"`                     | Look for documents titled "confidential report" or similar variations.                  |
| `intext:` or `inbody:`  | Searches for a term within the body text of pages.           | `intext:"password reset"`                           | Identify webpages containing the term “password reset”.                                 |
| `cache:`                | Displays the cached version of a webpage (if available).     | `cache:example.com`                                 | View the cached version of example.com to see its previous content.                     |
| `link:`                 | Finds pages that link to a specific webpage.                 | `link:example.com`                                  | Identify websites linking to example.com.                                               |
| `related:`              | Finds websites related to a specific webpage.                | `related:example.com`                               | Discover websites similar to example.com.                                               |
| `info:`                 | Provides a summary of information about a webpage.           | `info:example.com`                                  | Get basic details about example.com, such as its title and description.                 |
| `define:`               | Provides definitions of a word or phrase.                    | `define:phishing`                                   | Get a definition of "phishing" from various sources.                                    |
| `numrange:`             | Searches for numbers within a specific range.                | `site:example.com numrange:1000-2000`               | Find pages on example.com containing numbers between 1000 and 2000.                     |
| `allintext:`            | Finds pages containing all specified words in the body text. | `allintext:admin password reset`                    | Search for pages containing both "admin" and "password reset" in the body text.         |
| `allinurl:`             | Finds pages containing all specified words in the URL.       | `allinurl:admin panel`                              | Look for pages with "admin" and "panel" in the URL.                                     |
| `allintitle:`           | Finds pages containing all specified words in the title.     | `allintitle:confidential report 2023`               | Search for pages with "confidential," "report," and "2023" in the title.                |
| `AND`                   | Narrows results by requiring all terms to be present.        | `site:example.com AND (inurl:admin OR inurl:login)` | Find admin or login pages specifically on example.com.                                  |
| `OR`                    | Broadens results by including pages with any of the terms.   | `"linux" OR "ubuntu" OR "debian"`                   | Search for webpages mentioning Linux, Ubuntu, or Debian.                                |
| `NOT`                   | Excludes results containing the specified term.              | `site:bank.com NOT inurl:login`                     | Find pages on bank.com excluding login pages.                                           |
| `*` (wildcard)          | Represents any character or word.                            | `site:socialnetwork.com filetype:pdf user* manual`  | Search for user manuals (user guide, user handbook) in PDF format on socialnetwork.com. |
| `..` (range search)     | Finds results within a specified numerical range.            | `site:ecommerce.com "price" 100..500`               | Look for products priced between 100 and 500 on an e-commerce website.                  |
| `" "` (quotation marks) | Searches for exact phrases.                                  | `"information security policy"`                     | Find documents mentioning the exact phrase "information security policy".               |
| `-` (minus sign)        | Excludes terms from the search results.                      | `site:news.com -inurl:sports`                       | Search for news articles on news.com excluding sports-related content.                  |
Here are some common examples of Google Dorks, for more examples, refer to the [Google Hacking Database](https://www.exploit-db.com/google-hacking-database):
```
# Finding Login Pages
site:example.com inurl:login
site:example.com (inurl:login OR inurl:admin)

# Identifying Exposed Files
site:example.com filetype:pdf
site:example.com (filetype:xls OR filetype:docx)

# Uncovering Configuration Files
site:example.com inurl:config.php
site:example.com (ext:conf OR ext:cnf) (searches for extensions commonly used for configuration files)

# Locating Database Backups
site:example.com inurl:backup
site:example.com filetype:sql
```



# Web Archives
The [Wayback Machine](https://web.archive.org/) offers us a unique opportunity to revisit the past and explore the digital footprints of websites as they once were. It's a digital archive of the World Wide Web and other information on the Internet. Founded by the Internet Archive, a non-profit organization, it has been archiving websites since 1996. It captured snapshots of websites at regular intervals automatically. 
![](https://mermaid.ink/svg/pako:eNpNjkEOgjAQRa_SzBou0IUJ4lI3uqQsJu1IG2lLhlZjCHe3YGLc_f9m8vMW0NEQSBgYJyvOVxWarmV8jS4Mvajrgzh2DWvrnhtQ4b_t57ZrtKZ53gBU4Ik9OlMWFxWEUJAseVIgSzTIDwUqrOUPc4q3d9AgE2eqgGMeLMg7jnNpeTKY6OSwaPkfJeNS5MtXePdeP1LGQQs)
It is a treasure trove for web reconnaissance, offering information that can be instrumental in various scenarios :
1. `Uncovering Hidden Assets and Vulnerabilities`: The Wayback Machine allows you to discover old web pages, directories, files, or subdomains that might not be accessible on the current website, potentially exposing sensitive information or security flaws.
2. `Tracking Changes and Identifying Patterns`: By comparing historical snapshots, you can observe how the website has evolved, revealing changes in structure, content, technologies, and potential vulnerabilities.
3. `Gathering Intelligence`: Archived content can be a valuable source of OSINT, providing insights into the target's past activities, marketing strategies, employees, and technology choices.
4. `Stealthy Reconnaissance`: Accessing archived snapshots is a passive activity that doesn't directly interact with the target's infrastructure, making it a less detectable way to gather information.
