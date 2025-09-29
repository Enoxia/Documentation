Obfuscation is a technique used to make a script more difficult to read by humans but allows it to function the same from a technical point of view, though performance may be slower. This is usually achieved automatically by using an obfuscation tool, which takes code as an input, and attempts to re-write the code in a way that is much more difficult to read, depending on its design.
There are many reasons why developers may consider obfuscating their code. One common reason is to hide the original code and its functions to prevent it from being reused or copied without the developer's permission, making it more difficult to reverse engineer the code's original functionality. Another reason is to provide a security layer when dealing with authentication or encryption to prevent attacks on vulnerabilities that may be found within the code.

The most common usage of obfuscation, however, is for malicious actions. It is common for attackers and malicious actors to obfuscate their malicious scripts to prevent Intrusion Detection and Prevention systems from detecting their scripts.

[Obfuscator](https://obfuscator.io/)

JavaScript scripts are found running on the client-side, and can be accessible through the source code of the page. 

# Deobfuscation Websites

| **Website**                                 |
| ------------------------------------------- |
| [JS Console](https://jsconsole.com/)        |
| [Prettier](https://prettier.io/playground/) |
| [Beautifier](https://beautifier.io/)        |
| [JSNice](http://www.jsnice.org/)            |



# Browse front-end scripts
- Alway look in the `Inspector` in the `Debugger` tab, and browse each interesting script
- [Jxscout](https://github.com/francisconeves97/jxscout) is a tool designed to help security researchers analyze and find vulnerabilities in JavaScript code. It works with your favorite proxy (Burp or Caido), capturing requests and saving optimized versions locally for easy analysis in your preferred code editor.



# Encoding & Decodeding references
Others types of encoding can existsSome tools can help us automatically determine the type of encoding, like [Cipher Identifier](https://www.boxentriq.com/code-breaking/cipher-identifier).

|                                                  |               |
| ------------------------------------------------ | ------------- |
| echo hackthebox \| base64                        | base64 encode |
| echo ENCODED_B64 \| base64 -d                    | base64 decode |
| echo hackthebox \| xxd -p                        | hex encode    |
| echo ENCODED_HEX \| xxd -p -r                    | hex decode    |
| echo hackthebox \| tr 'A-Za-z' 'N-ZA-Mn-za-m'    | rot13 encode  |
| echo ENCODED_ROT13 \| tr 'A-Za-z' 'N-ZA-Mn-za-m' | rot13 decode  |
