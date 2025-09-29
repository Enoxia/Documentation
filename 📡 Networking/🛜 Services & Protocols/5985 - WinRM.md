# Enumeration
```
nmap -sVC $IP -p5985,5986 --disable-arp-ping -n
```


# Interacting w/ service
```
# Establish WinRM session from Windows using PSCredential Object
PS C:\htb> $password = ConvertTo-SecureString "Klmcargo2" -AsPlainText -Force
PS C:\htb> $cred = new-object System.Management.Automation.PSCredential ("INLANEFREIGHT\forend", $password)
PS C:\htb> Enter-PSSession -ComputerName ACADEMY-EA-MS01 -Credential $cred

# Establish WinRM session from Linux w/ Evil-winrm
evil-winrm -i 10.129.201.234 -u forend -p P455w0rD! -e "/workspace"
evil-winrm -i 10.129.201.234 -u forend -H HASH -e "/workspace"
```


# Bruteforce
```
netexec winrm $IP -u user.list -p password.list
```