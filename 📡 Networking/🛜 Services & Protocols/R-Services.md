Suite of services hosted to enable remote access or issue commands between Unix hosts over TCP/IP. Default(s) port(s) `512, 513, 514`

It transmit information from client to server (and vice versa) over the network in an unencrypted format (MITM)

R-services are only accessible through a suite of programs known as `r-commands`.

The [R-commands](https://en.wikipedia.org/wiki/Berkeley_r-commands) suite consists of the following programs:

- rcp (`remote copy`)
- rexec (`remote execution`)
- rlogin (`remote login`)
- rsh (`remote shell`)
- rstat
- ruptime
- rwho (`remote who`)

Each command has its intended functionality; however, we will only cover the most commonly abused `r-commands`. The table below will provide a quick overview of the most frequently abused commands, including the service daemon they interact with, over what port and transport method to which they can be accessed, and a brief description of each.

|**Command**|**Service Daemon**|**Port**|**Transport Protocol**|**Description**|
|---|---|---|---|---|
|`rcp`|`rshd`|514|TCP|Copy a file or directory bidirectionally from the local system to the remote system (or vice versa) or from one remote system to another. It works like the `cp` command on Linux but provides `no warning to the user for overwriting existing files on a system`.|
|`rsh`|`rshd`|514|TCP|Opens a shell on a remote machine without a login procedure. Relies upon the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files for validation.|
|`rexec`|`rexecd`|512|TCP|Enables a user to run shell commands on a remote machine. Requires authentication through the use of a `username` and `password` through an unencrypted network socket. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files.|
|`rlogin`|`rlogind`|513|TCP|Enables a user to log in to a remote host over the network. It works similarly to `telnet` but can only connect to Unix-like hosts. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files.|

The `/etc/hosts.equiv` file contains a list of trusted hosts and is used to grant access to other systems on the network. When users on one of these hosts attempt to access the system, they are automatically granted access without further authentication.

```bash
Bailly@htb[/htb]$ cat /etc/hosts.equiv

# <hostname> <local username>
pwnbox cry0l1t3
```

# Footprinting :
`sudo nmap -sV -p 512,513,514 10.0.17.2`

R-services rely on trusted information sent from the remote client to the host machine they are attempting to authenticate to. We can bypass this authentication through the use of the `/etc/hosts.equiv` and `.rhosts` files on the system. Those files contains a list of hosts (`IPs` or `Hostnames`) and users that are `trusted` by the local host when a connection attempt is made using `r-commands`. Entries in either file can appear like the following :

```bash
Bailly@htb[/htb]$ cat .rhosts

htb-student     10.0.17.5
+               10.0.17.10
+               +
```

The `+` modifier can be used within these files as a wildcard to specify anything. In this example, the `+` modifier allows any external user to access r-commands from the `htb-student` user account via the host with the IP address `10.0.17.10`.

Misconfigurations in either of these files can allow an attacker to authenticate as another user without credentials, with the potential for gaining code execution.

```bash
Bailly@htb[/htb]$ rlogin 10.0.17.2 -l htb-student

Last login: Fri Dec  2 16:11:21 from localhost

[htb-student@localhost ~]$
```

We have successfully logged in under the `htb-student` account on the remote host due to the misconfigurations in the `.rhosts` file. Once successfully logged in, we can also abuse the `rwho` command to list all interactive sessions on the local network by sending requests to the UDP port 513.

```
Bailly@htb[/htb]$ rwho

root     web01:pts/0 Dec  2 21:34
htb-student     workstn01:tty1  Dec  2 19:57  2:25
```

To provide additional information in conjunction with `rwho`, we can issue the `rusers` command.

```bash
Bailly@htb[/htb]$ rusers -al 10.0.17.5

htb-student     10.0.17.5:console          Dec 2 19:57     2:25
```