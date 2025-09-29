IDOR vulnerabilities occur when a web application exposes a direct reference to an object, like a file or a database resource, which the end-user can directly control to obtain access to other similar objects. If a link reference to a file using this type of URL `download.php?file_id=1`, an attacker might try to access to other file located at `download.php?file_id=2`.



# IDOR Exploitation
## Identifying Direct Object References
- Study the `HTTP` request to look for `URL parameters` or `APIs` w/ an object reference. We can look in `HTTP headers` too, like `cookies`.
- We could identify `unused parameters or APIs` in front-end `JavaScript AJAX calls`. Some web applications may insecurely place all function calls on the front-end. We can do the same with back-end code if we have access to it (open-source web applications)
- To perform more advanced IDOR attacks, we could `register multiple users` and compare their `HTTP request` & object references
## IDOR Enumeration
- `Fuzz` all `predictable naming pattern` when discovering files (like `/documents/Invoice_2_08_2020.pdf`). We can use a bash `for` loop over the discovered `uid` parameter to find other documents.
```bash
#!/bin/bash

url="http://SERVER_IP:PORT"

for i in {1..10}; do
        for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
                wget -q $url/$link
        done
done
```
- The same can be done w/ `Burp Pro Intruder` by replaying the downloading request
## Encoded References
Some references will be hashed or encoded (`base64` among others) and we could decode them to get the original object reference.
- We can attempt to hash various values, like `uid`, `username`, `filename`, and many others, and see if any of them match the value we have
-  We may utilize `Burp Comparer` to fuzz various values and then compare each to our hash to see if we find any matches
- Once identified, we could `calculate the hash for each user`, and make a request on the download page that would download all the files
```bash
#!/bin/bash

for i in {1..10}; do
    for hash in $(echo -n $i | base64 -w 0 | md5sum | tr -d ' -'); do
        curl -sOJ -X POST -d "contract=$hash" http://SERVER_IP:PORT/download.php
    done
done
```
## Function Disclosure
- Search for all `front-end functions` that gets executed to further understand how the web app works
## Insecure APIs
IDOR vulnerabilities may also exist in `function calls and APIs`, and exploiting them would allow us to `perform various actions as other users` like `changing another user's private information, reset a password, buy an item using another payment information, change email to reset password...`
`PUT` requests are usually used in APIs to update item details, while `POST` is used to create new items, `DELETE` to delete items, and `GET` to retrieve item details.
- Look for requests being made at `API` endpoint like `/profile/api.php/profile/1`
- Look for any interesting parameters sent (like `role, email, uid...`) or in client's HTTP request (`Cookie: role=`)
- Try to `fuzz` all available parameters (`change uid to another user's uid, create new user w/ arbitrary details, change role or privileges...`). Be careful to change the request method (`POST` when creating, `PUT` when updating...)
- Use the `API` to read other users' details, to leak sensitive information (using `GET` request w/ another `uid`) to further exploit the `IDOR` (leak info we couldn't calculate before like `uuid` or `hashes`)
- Another attack would be to place an `XSS payload` in a text-field ("about", "comment"...) that would get executed when the user visits their page