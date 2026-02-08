[Volatility](https://github.com/volatilityfoundation/volatility) is an open-source memory forensics framework for incident response and malware analysis. It is used to analyze volatile memory (RAM) from computer systems.

With Volatility, investigators can:

- List running processes, network connections, and open ports to identify suspicious activity. 
- Retrieve hashed passwords, SSL keys, and credentials stored in memory. 
- Detect in-memory malware (fileless malware) that doesn’t write to disk. 
- Extract clipboard contents, screenshots, and command-line history (e.g., from CMD). 
- Analyze registry hives, loaded drivers, and file handles. 
- Reconstruct a forensic timeline of system activity.
- Use YARA rules to scan for known malware signatures. 
- Pull SAM hashes from memory dumps to escalate privileges, even in locked or isolated environments (e.g., ESXi hosts).

Here is a Volatility [CheatSheet](https://blog.onfvp.com/post/volatility-cheatsheet/) based on both version 2 and 3.

We can for example [retrieve the machine's hostname](https://www.aldeid.com/wiki/Volatility/Retrieve-hostname)
