| **Category**                                                                                                               | **Applications**                                                       |
| -------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| [Web Content Management](https://enlyft.com/tech/web-content-management)                                                   | Joomla, Drupal, WordPress, DotNetNuke, etc.                            |
| [Application Servers](https://enlyft.com/tech/application-servers)                                                         | Apache Tomcat, Phusion Passenger, Oracle WebLogic, IBM WebSphere, etc. |
| [Security Information and Event Management (SIEM)](https://enlyft.com/tech/security-information-and-event-management-siem) | Splunk, Trustwave, LogRhythm, etc.                                     |
| [Network Management](https://enlyft.com/tech/network-management)                                                           | PRTG Network Monitor, ManageEngine Opmanger, etc.                      |
| [IT Management](https://enlyft.com/tech/it-management-software)                                                            | Nagios, Puppet, Zabbix, ManageEngine ServiceDesk Plus, etc.            |
| [Software Frameworks](https://enlyft.com/tech/software-frameworks)                                                         | JBoss, Axis2, etc.                                                     |
| [Customer Service Management](https://enlyft.com/tech/customer-service-management)                                         | osTicket, Zendesk, etc.                                                |
| [Search Engines](https://enlyft.com/tech/search-engines)                                                                   | Elasticsearch, Apache Solr, etc.                                       |
| [Software Configuration Management](https://enlyft.com/tech/software-configuration-management)                             | Atlassian JIRA, GitHub, GitLab, Bugzilla, Bugsnag, Bitbucket, etc.     |
| [Software Development Tools](https://enlyft.com/tech/software-development-tools)                                           | Jenkins, Atlassian Confluence, phpMyAdmin, etc.                        |
| [Enterprise Application Integration](https://enlyft.com/tech/enterprise-application-integration)                           | Oracle Fusion Middleware, BizTalk Server, Apache ActiveMQ, etc.        |
# Common Applications

|**Application**|**Description**|
|---|---|
|WordPress|[WordPress](https://wordpress.org/) is an open-source Content Management System (CMS) that can be used for multiple purposes. It's often used to host blogs and forums. WordPress is highly customizable as well as SEO friendly, which makes it popular among companies. However, its customizability and extensible nature make it prone to vulnerabilities through third-party themes and plugins. WordPress is written in PHP and usually runs on Apache with MySQL as the backend.|
|Drupal|[Drupal](https://www.drupal.org/) is another open-source CMS that is popular among companies and developers. Drupal is written in PHP and supports using MySQL or PostgreSQL for the backend. Additionally, SQLite can be used if there's no DBMS installed. Like WordPress, Drupal allows users to enhance their websites through the use of themes and modules.|
|Joomla|[Joomla](https://www.joomla.org/) is yet another open-source CMS written in PHP that typically uses MySQL but can be made to run with PostgreSQL or SQLite. Joomla can be used for blogs, discussion forums, e-commerce, and more. Joomla can be customized heavily with themes and extensions and is estimated to be the third most used CMS on the internet after WordPress and Shopify.|
|Tomcat|[Apache Tomcat](https://tomcat.apache.org/) is an open-source web server that hosts applications written in Java. Tomcat was initially designed to run Java Servlets and Java Server Pages (JSP) scripts. However, its popularity increased with Java-based frameworks and is now widely used by frameworks such as Spring and tools such as Gradle.|
|Jenkins|[Jenkins](https://jenkins.io/) is an open-source automation server written in Java that helps developers build and test their software projects continuously. It is a server-based system that runs in servlet containers such as Tomcat. Over the years, researchers have uncovered various vulnerabilities in Jenkins, including some that allow for remote code execution without requiring authentication.|
|Splunk|Splunk is a log analytics tool used to gather, analyze and visualize data. Though not originally intended to be a SIEM tool, Splunk is often used for security monitoring and business analytics. Splunk deployments are often used to house sensitive data and could provide a wealth of information for an attacker if compromised. Historically, Splunk has not suffered from a considerable amount of known vulnerabilities aside from an information disclosure vulnerability ([CVE-2018-11409](https://nvd.nist.gov/vuln/detail/CVE-2018-11409)), and an authenticated remote code execution vulnerability in very old versions ([CVE-2011-4642](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-4642)).|
|PRTG Network Monitor|[PRTG Network Monitor](https://www.paessler.com/prtg) is an agentless network monitoring system that can be used to monitor metrics such as uptime, bandwidth usage, and more from a variety of devices such as routers, switches, servers, etc. It utilizes an auto-discovery mode to scan a network and then leverages protocols such as ICMP, WMI, SNMP, and NetFlow to communicate with and gather data from discovered devices. PRTG is written in [Delphi](https://en.wikipedia.org/wiki/Delphi_(software)).|
|osTicket|[osTicket](https://osticket.com/) is a widely-used open-source support ticketing system. It can be used to manage customer service tickets received via email, phone, and the web interface. osTicket is written in PHP and can run on Apache or IIS with MySQL as the backend.|
|GitLab|[GitLab](https://about.gitlab.com/) is an open-source software development platform with a Git repository manager, version control, issue tracking, code review, continuous integration and deployment, and more. It was originally written in Ruby but now utilizes Ruby on Rails, Go, and Vue.js. GitLab offers both community (free) and enterprises versions of the software.|
# Discovery & Enumeration
```bash
nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list
```
Those tools can be used to quickly inspect all hosts running web applications and take screenshots of each, using `Nmap XML scan output`
# Example of Organisation’s Note Taking
`External Penetration Test - <Client Name>`

- `Scope` (including in-scope IP addresses/ranges, URLs, any fragile hosts, testing timeframes, and any limitations or other relative information we need handy)
- `Client Points of Contact`
- `Credentials`
- `Discovery/Enumeration`
    - `Scans`
    - `Live hosts`
- `Application Discovery`
    - `Scans`
    - `Interesting/Notable Hosts`
- `Exploitation`
    - `<Hostname or IP>`
    - `<Hostname or IP>`
- `Post-Exploitation`
    - `<Hostname or IP>`
    - `<<Hostname or IP>`
    We can put all our `URL` to test in a scope file :  Don’t forget to add all the hosts in the `/etc/hosts` file !
```bash
# Add scope URL to /etc/hosts
IP=10.129.42.195
printf "%s\\t%s\\n\\n" "$IP" "app.inlanefreight.local dev.inlanefreight.local blog.inlanefreight.local" | sudo tee -a /etc/hosts
   
# Scope list
cat scope_list 
   
app.inlanefreight.local
dev.inlanefreight.local
drupal-dev.inlanefreight.local
drupal-qa.inlanefreight.local
drupal-acc.inlanefreight.local
drupal.inlanefreight.local
blog-dev.inlanefreight.local
blog.inlanefreight.local
app-dev.inlanefreight.local
jenkins-dev.inlanefreight.local
jenkins.inlanefreight.local
web01.inlanefreight.local
gitlab-dev.inlanefreight.local
gitlab.inlanefreight.local
support-dev.inlanefreight.local

# Nmap Scan on IP
sudo nmap --open -sV 10.129.201.50
```
# Using EyeWitness
[EyeWitness](https://github.com/FortyNorthSecurity/EyeWitness) can take the `XML` output from both `Nmap` and `Nessus` and create a report with screenshots of each web application present on the various ports using Selenium.
```bash
# Install Eyewitness
sudo apt install eyewitness

# Run Eyewitness to take screenshots using the Nmap XML output
eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness
```
# Using Aquatone
[aquatone](https://github.com/michenriksen/aquatone) can take screenshots when provided a `.txt` file of hosts or an Nmap `.xml` file with the `-nmap` flag.
```bash
wget https://github.com/michenriksen/aquatone/releases/download/v1.7.0/aquatone_linux_amd64_1.7.0.zip
unzip aquatone_linux_amd64_1.7.0.zip
```
We can move it to a location in our `$PATH` such as `/usr/local/bin` to be able to call the tool from anywhere. Here we provide the tool the same `web_discovery.xml` Nmap output specifying the `-nmap` flag :
```bash
 cat web_discovery.xml | ./aquatone -nmap
```
# Using Nuclei
[Nuclei](https://github.com/projectdiscovery/nuclei) is a modern, high-performance vulnerability scanner that leverages simple YAML-based templates. It empowers you to design custom vulnerability detection scenarios that mimic real-world conditions, leading to zero false positives.
```bash
# Default rapid scan
nuclei -target https://example.com
```
