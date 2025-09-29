# Enumeration
```
nmap -sU --script ipmi-version -p 623 $IP

# Using Metasploit
use auxiliary/scanner/ipmi/ipmi_version

# Dumping hashes
use auxiliary/scanner/ipmi/ipmi_dumphashes
hashcat -m 7300 hash wordlist.txt
```


# Default BMCs credentials
During internal penetration tests, we often find BMCs where the administrators have not changed the default password.
Some unique default passwords to keep in our cheatsheets include :

|**Product**|**Username**|**Password**|
|---|---|---|
|Dell iDRAC|root|calvin|
|HP iLO|Administrator|randomized 8-character string consisting of numbers and uppercase letters|
|Supermicro IPMI|ADMIN|ADMIN|

# Dangerous Settings
If default credentials do not work to access a BMC, we can turn to a [flaw](http://fish2.com/ipmi/remote-pw-cracking.html) in the RAKP protocol in IPMI 2.0. During the authentication process, the server sends a salted SHA1 or MD5 hash of the user's password to the client before authentication takes place. This can be leveraged to obtain the password hash for ANY valid user account on the BMC. These password hashes can then be cracked offline using a dictionary attack using `Hashcat` mode `7300`. In the event of an HP iLO using a factory default password, we can use this Hashcat mask attack command `hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u` which tries all combinations of upper case letters and numbers for an eight-character password.

There is no direct "fix" to this issue because the flaw is a critical component of the IPMI specification. Clients can opt for very long, difficult to crack passwords or implement network segmentation rules to restrict the direct access to the BMCs.

When cracking a hash, we could think of password re-use and gain SSH access