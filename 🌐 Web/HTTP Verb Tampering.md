The `HTTP` protocol works by accepting various HTTP methods as `verbs` at the beginning of an HTTP request. An HTTP Verb Tampering attack exploits web servers that accept many `HTTP verbs and methods`. This can be exploited by sending malicious requests using unexpected methods.
HTTP has [9 different verbs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) that can be accepted as HTTP methods by web servers.

| Verb      | Description                                                                                         |
| --------- | --------------------------------------------------------------------------------------------------- |
| `HEAD`    | Identical to a GET request, but its response only contains the `headers`, without the response body |
| `PUT`     | Writes the request payload to the specified location                                                |
| `DELETE`  | Deletes the resource at the specified location                                                      |
| `OPTIONS` | Shows different options accepted by a web server, like accepted HTTP verbs                          |
| `PATCH`   | Apply partial modifications to the resource at the specified location                               |

Very sensitive functionalities can be done like writing (`PUT`) or deleting (`DELETE`) files to the webroot directory on the back-end server.

# Bypassing Basic Authentication
We could try to bypass basic `HTTP Auth` prompt
![](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_reset.jpg)
For this, we'll need to identify `which pages / folders` are restricted by the authentication, and which `HTTP request method` is used.
```bash
# See what HTTP methods are accepted
curl -i -X OPTIONS http://SERVER_IP:PORT/
```
- To change a `request method in Burp`, we can click on the request and select `Change Request Method`



# Bypassing Security Filters
If a security filter was being used to detect `injection` vulnerabilities and `only checked` for injections in `POST` parameters, it may be possible to bypass it by simply changing the request method to `GET`.
- Security filters (to prevent malicious upload, or command injection) can be bypassed using `HTTP Verb Tampering` vulnerability to achieve other vulns, like `Command Injection`. 
