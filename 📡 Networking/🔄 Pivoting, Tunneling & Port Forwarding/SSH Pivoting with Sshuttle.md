[SSHuttle](<[https://github.com/sshuttle/sshuttle](https://github.com/sshuttle/sshuttle)>) 

A python written tool which removes the need to configure proxychains.

However, this tool only works for pivoting over SSH and does not provide other options for pivoting over TOR or HTTPS proxy servers.

So port 22 need to be open on the machine !

`Sshuttle` can be extremely useful for automating the execution of iptables and adding pivot rules for the remote host. We can configure the Ubuntu server as a pivot point and route all of Nmap's network traffic with sshuttle.

We can install it and start it using the option `-r` to connect to the remote machine with a username and password. Then we need to include the network or IP we want to route through the pivot host, in our case, is the network `172.16.5.0/23`.

```bash
Bailly@htb[/htb]$ sudo apt-get install sshuttle -y
Bailly@htb[/htb]$ sudo sshuttle -r ubuntu@10.129.202.64 172.16.5.0/23 -v 
```

With this command, sshuttle creates an entry in our `iptables` to redirect all traffic to the `172.16.5.0/23` network through the pivot Ubuntu host :

```bash
Bailly@htb[/htb]$ nmap -v -sV -p3389 172.16.5.19 -A -Pn
<SNIP>
PORT     STATE SERVICE       VERSION
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: inlanefreight.local
|   DNS_Computer_Name: DC01.inlanefreight.local
|   Product_Version: 10.0.17763
|_  System_Time: 2022-08-14T02:58:25+00:00
|_ssl-date: 2022-08-14T02:58:25+00:00; +7s from scanner time.
| ssl-cert: Subject: commonName=DC01.inlanefreight.local
| Issuer: commonName=DC01.inlanefreight.local
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-08-13T02:51:48
| Not valid after:  2023-02-12T02:51:48
| MD5:   58a1 27de 5f06 fea6 0e18 9a02 f0de 982b
|_SHA-1: f490 dc7d 3387 9962 745a 9ef8 8c15 d20e 477f 88cb
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 6s, deviation: 0s, median: 6s
Completed NSE at 11:16, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> .
Nmap done: 1 IP address (1 host up) scanned in 4.07 seconds
```

We can now use any tool directly without using proxychains.