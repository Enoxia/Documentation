[https://github.com/utoni/ptunnel-ng](https://github.com/utoni/ptunnel-ng)

ICMP tunneling only works when ping responses are permitted within a firewalled network. It encapsulates traffic within `ICMP packets` containing `echo requests` and `responses`.

When a host within a firewalled network is allowed to ping an external server, it can encapsulate its traffic within the ping echo request and send it to an external server.

The external server can validate this traffic and send an appropriate response, which is extremely useful for data exfiltration and creating pivot tunnels to an external server.

We will use the [ptunnel-ng](https://github.com/utoni/ptunnel-ng) tool to create a tunnel between our Ubuntu server and our attack host. Once a tunnel is created, we will be able to proxy our traffic through the `ptunnel-ng client` , just like `Chisel` does.

We can start the `ptunnel-ng server` on the target pivot host.

### Download Ptunnel-ng
```bash
Bailly@htb[/htb]$ git clone <https://github.com/utoni/ptunnel-ng.git>
```

Then we can run the `autogen.sh` script to get the repo
```bash
Bailly@htb[/htb]$ sudo ./autogen.sh
```

Ptunnel-ng can now be used from the `client and server-side`.

We’ll then need to transfer the repo from our attack host to our pivot host, using `scp -r`

### Starting Ptunnel-ng Server On The Target Host
```bash
ubuntu@WEB01:~/ptunnel-ng/src$ sudo ./src/ptunnel-ng -r10.129.202.64 -R22

[sudo] password for ubuntu: 
./ptunnel-ng: /lib/x86_64-linux-gnu/libselinux.so.1: no version information available (required by ./ptunnel-ng)
[inf]: Starting ptunnel-ng 1.42.
[inf]: (c) 2004-2011 Daniel Stoedle, <daniels@cs.uit.no>
[inf]: (c) 2017-2019 Toni Uhlig,     <matzeton@googlemail.com>
[inf]: Security features by Sebastien Raveau, <sebastien.raveau@epita.fr>
[inf]: Forwarding incoming ping packets over TCP.
[inf]: Ping proxy is listening in privileged mode.
[inf]: Dropping privileges now.
```

The IP address following `-r` should be the IP we want ptunnel-ng to accept connections on.

We would use whatever IP is reachable from our attack host. We would benefit from using this same thinking & consideration during an actual engagement.

### Connecting To Ptunnel-ng Server From Our Attack Host

On the attack host, we can attempt to connect to the ptunnel-ng server (`-p <ipAddressOfPivotHost>`) but ensure this happens through local port 2222 (`-l2222`). Connecting through local port 2222 allows us to send traffic through the ICMP tunnel.
```bash
Bailly@htb[/htb]$ sudo ./src/ptunnel-ng -p10.129.202.64 -l2222 -r10.129.202.64 -R22

[inf]: Starting ptunnel-ng 1.42.
[inf]: (c) 2004-2011 Daniel Stoedle, <daniels@cs.uit.no>
[inf]: (c) 2017-2019 Toni Uhlig,     <matzeton@googlemail.com>
[inf]: Security features by Sebastien Raveau, <sebastien.raveau@epita.fr>
[inf]: Relaying packets from incoming TCP streams.
```

With the ptunnel-ng ICMP tunnel successfully established, we can attempt to connect to the target using SSH through local port 2222 (`-p2222`).

### Tunneling an SSH Connection Through An ICMP Tunnel
```bash
Bailly@htb[/htb]$ ssh -p2222 -lubuntu 127.0.0.1

ubuntu@127.0.0.1's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  <https://help.ubuntu.com>
 * Management:     <https://landscape.canonical.com>
 * Support:        <https://ubuntu.com/advantage>

  System information as of Wed 11 May 2022 03:10:15 PM UTC

  System load:             0.0
  Usage of /:              39.6% of 13.72GB
  Memory usage:            37%
  Swap usage:              0%
  Processes:               183
  Users logged in:         1
  IPv4 address for ens192: 10.129.202.64
  IPv6 address for ens192: dead:beef::250:56ff:feb9:52eb
  IPv4 address for ens224: 172.16.5.129

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   <https://ubuntu.com/blog/microk8s-memory-optimisation>

144 updates can be applied immediately.
97 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

Last login: Wed May 11 14:53:22 2022 from 10.10.14.18
ubuntu@WEB01:~$ 
```

We can view the session logs and traffic statistics on the client & server side of the connection, confirming that the traffic passes through the ICMP tunnel.

- We may also use this tunnel and SSH to perform dynamic port forwarding to allow us to use proxychains in various ways.

### **Enabling Dynamic Port Forwarding over SSH**
```bash
Bailly@htb[/htb]$ ssh -D 9050 -p2222 -lubuntu 127.0.0.1

ubuntu@127.0.0.1's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)
<snip>
```

We could use proxychains with Nmap to scan targets on the internal network `(172.16.5.x)`.

### **Proxychaining through the ICMP Tunnel**
```bash
Bailly@htb[/htb]$ proxychains nmap -sV -sT 172.16.5.19 -p3389

ProxyChains-3.1 (<http://proxychains.sf.net>)
Starting Nmap 7.92 ( <https://nmap.org> ) at 2022-05-11 11:10 EDT
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:80-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
Nmap scan report for 172.16.5.19
Host is up (0.12s latency).

PORT     STATE SERVICE       VERSION
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> .
Nmap done: 1 IP address (1 host up) scanned in 8.78 seconds
```

# Network Traffic Analysis Considerations

It is important that we confirm the tools we are using are performing as advertised and that we have set up & are operating them properly.

In the case of tunneling traffic through different protocols taught in this section with ICMP tunneling, we can benefit from analyzing the traffic we generate with a packet analyzer like `Wireshark`. Take a close look at the short clip below.
![[analyzingTheTraffic.gif]]

- In the first part of this clip, a connection is established over SSH without using ICMP tunneling. We may notice that `TCP` & `SSHv2` traffic is captured.

The command used in the clip: `ssh ubuntu@10.129.202.64`

- In the second part of this clip, a connection is established over SSH using ICMP tunneling. Notice the type of traffic that is captured when this is performed.

Command used in clip: `ssh -p2222 -lubuntu 127.0.0.1`