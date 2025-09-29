# Identify Web Technologies
The very first step is to identify the language used by the backend server
- Using [Wappalyzer](https://addons.mozilla.org/fr/firefox/addon/wappalyzer/) / [BuiltWith](https://addons.mozilla.org/fr/firefox/addon/builtwith/)
- Running web vulnerability scanner (BurpPro / ZAP)
- Visiting the `/index.ext` page, where we would fuzz out `ext` with various common web extensions (`php`, `asp`, `aspx`…) among others, using `burp` or `ffuf` with a [web-extension](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt) wordlist
 
 
 
# Bypassing Filters
## Client-Side
Here we'll try to manipulate the front-end code to disable the validations in place. We'll burp to modify requests. 
- We could modify the `filename="HTB.png"`in the request to `filename="shell.php"`, and the `Content-Type`
- Looking in the `Page Inspector` for front-end validations. Look all the `JavaScript` scripts used to understand the web app, then delete every `check / functions`. Try to grab useful info like the `uploaded` folder if shown
![[FileUpload_ClientSide.png]]
## Blacklist Extensions
Some back-end validations includes a list of blacklisted `disallowed extensions`to prevent uploading web scripts. The validation may also check the `file type` or the `file content` for type matching. 
- Fuzzing extensions w/ `Burp` of `FFUF`using `PayloadsAllTheThings` for [PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) and [.NET](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP), or [SecLists Extensions](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt) and if using Burp `UN-TICK the url ENCODING` option (for the ".")
- In `Windows Servers`, file names are `case insensitive`, so we may try to upload mixed-case like `pHp`
- Once `allowed extensions are found`, we can change the content of the file for a web shell / malicious code
We may try `several allowed extensions` because not all extensions will work w/ all web server configurations
## Whitelist Extensions
A whitelist is generally more secure than a blacklist. Again, the idea is to identify how the web app handle validation.
- Fuzz the upload form with [this Wordlist](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt) to find what extensions are whitelisted by the upload form
- Try `Double Extensions`if the code only tests whether the file name contains the extension (like `shell.jpg.php`)
- Try `Reverse Double Extension` if the code verify only the final extension in file name (like `shell.php.jpg`)
The web application may still deny requests containing `PHP` extensions. Try to fuzz w/ [PHP Wordlist](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) to find blacklisted extensions with the 2 previous `Double Extension` techniques
- Try `Character injection` to input characters before or after the final extension. This code will generate all permutations for file names w/ inputed characters `%20`, `%0a`, `%00`, `%0d0a`, `/`, `.\`, `.`, `...`and `:`
```bash
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```
- Once `allowed extensions are found`, we can change the content of the file for a web shell / malicious code
## Type Filters
Web app also test the `content`of the uploaded file. Two common methods for verifying are `Content-Type Header` and `File Content` (MIME-Type, which contain the [File Signature](https://en.wikipedia.org/wiki/List_of_file_signatures) or [Magic Bytes](https://web.archive.org/web/20240522030920/https://opensource.apple.com/source/file/file-23/file/magic/magic.mime)).
- Fuzz the `Content-Type` header w/ SecLists' [Content-Type Wordlist](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-all-content-types.txt) and focus only on the allowed type. Try both `Content-Type` header (the one for the `attached file` at the bottom and the one for the `full request` at the top)
```bash
# Get only allowed type for make verification faster
cat web-all-content-types.txt | grep 'image/' > image-content-types.txt
```
- Modify the `MIME-Type` of the uploaded file content by adding `GIF8`(GIF) for images files
- Mix all combinations to confuse the web server : `Allowed MIME type with a disallowed Content-Type`, an `Allowed MIME/Content-Type with a disallowed extension`, or a `Disallowed MIME/Content-Type with an allowed extension`, and so on.



# Other Vulns
Some upload forms have secure filters that may not be exploitable with the techniques we discussed, but we could still perform some attacks on the web app.
### XSS
Some file type allow us to introduce `Stored XSS` vulnerability or `CRSF`, for example when we could upload `HTLM` files, on whoever visits the uploaded HTLM page. This is possible when :
- The web app display an `image's metadata` after its upload (in raw text, like the `Comment` or `Artist` Metadata parameters). We could change the image's `MIME-Type` to `text/html` for the web app to show it as an `HTML` document instead of an image, and trigger the `XSS` payload.
```bash
# Set "Comment" Metadata parameters to an image
exiftool -Comment=' "><img src=1 onerror=alert(window.origin)>' HTB.jpg
```
- `XSS` can be triggered with `SVG` images (which are `XML-based`). We can modify the XML data to include an XSS payload. Here's an example of an `HTB.svg` file carrying an `XSS` payload
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1" height="1">
    <rect x="1" y="1" width="1" height="1" fill="green" stroke="black" />
    <script type="text/javascript">alert(window.origin);</script>
</svg>
```
### XXE
- Like w/ the `XSS`, `XXE` can be triggered with `SVG` images (which are `XML-based`). This can be used for an `SVG` image to leak the content of `/etc/passwd`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```
- If the application allows the upload of `.XML` documents, we could try `XXE` attacks
- If it succeded, we could try to read the web application's source files to discover more vulns, or to `locate the upload directory, identify allowed extensions, or find the file naming scheme`. We can use the following payload in our `SVG image`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<svg>&xxe;</svg>
```
- `XML` data is also utilized by many types of documents like `PDF`, `Word Documents`, `PowerPoint Documents`... and in some `document viewer`
- We could use the `XXE` to achieve `SSRF` attack too
### DoS
- Utilize a `Decompression Bomb` with file types that use data compression, like `ZIP` archives, if the web app automatically unzips the ZIP archive
- Another is a `Pixel Flood` attack w/ some image files that utilize image compression, like `JPG` or `PNG`.
- If there is no limit on the file size upload, we could upload an overly large file
### Injection in File Name
If the uploaded `file name is displayed` (reflected), we could inject a command in the file name. 
- We could try to name a file `file$(whoami).jpg` or ``file`whoami`.jpg`` or `file.jpg||whoami`, if the application attempts to move it w/ an OS command (like w/ `mv file /tmp`)
- Similarly, we may use an `XSS` payload in the file name if he's displayed (like `<script>alert(window.origin);</script>`)
- We may also inject an SQL query in the file name (like `file';select+sleep(5);--.jpg`), which may lead to an SQL injection if the file name is insecurely used in an SQL query
### Upload Directory Disclosure
It is required to know the location of our uploaded file to access it and execute the code.
- We may `fuzz` the web app to look for folders
- Read the web app source code using other vulns (`XXE, LFI`) to find where the uploaded files are
- Force `error messages` to hope it will reveal helpful info (upload a file w/ a name that already exist, or w/ an overly long name 5000+ characters)
- [[Insecure Direct Object References (IDOR)]] discusses various methods of finding where files might be and identifying the `file naming scheme`
### Windows-specific Attacks
Those attacks would only work on `Windows` hosts.
- We may use reserved `characters` such as (`|`, `<`, `>`, `*`, or `?`), usually reserved for special uses like wildcards. This will cause an error that might reveal the `upload directory`
- We may use reserved `names` for the uploaded file name like (`CON`, `COM1`, `LPT1`, or `NUL`), which may also cause an error revealing info as the web app will not be allowed to write a file w/ those names
- We may use the Windows [8.3 Filename Convention](https://en.wikipedia.org/wiki/8.3_filename) to overwrite existing files or refer to files that do not exist. `Older versions` of Windows used a Tilde character (`~`) to complete the file name. We can write a file called (`WEB~.CONF`) to overwrite the `web.conf` file or other sensitive system files, leading to information disclosure
### Advanced File Upload Attacks
Any `automatic processing` that occurs to an uploaded file, like `encoding a video, compressing a file, or renaming a file`, may be exploited if not securely coded.
- Some `commonly used libraries` may have public exploits for such vulnerabilities (like the AVI upload vulnerability leading to `XXE` in `ffmpeg`.)



# Exploitation
Once found, we could test the code execution by uploading for example `<?php echo "Hello HTB";?>` to `test.php`, and see if tht `PHP`code gets executed.
- Once confirmed, we could use webshells such as [phpbash](https://github.com/Arrexel/phpbash) for `PHP`. [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Web-Shells) provides a lot of webshells too
- We could use `msfvenom` to generate custom reverse shells, or the [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell) PHP reverse shell

