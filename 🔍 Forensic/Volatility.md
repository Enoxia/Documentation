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

Here is a useful [Volatility CheatSheet](https://repository.root-me.org/Forensic/EN%20-%20Volatility%20cheatsheet%20v2.4.pdf) from Root-Me.

Here is another Volatility [CheatSheet](https://blog.onfvp.com/post/volatility-cheatsheet/) based on both version 2 and 3.

We can for example [retrieve the machine's hostname](https://www.aldeid.com/wiki/Volatility/Retrieve-hostname)

Here are the output of the `volatility2 -h` output : 
| Commande | Description |
| :--- | :--- |
| **amcache** | Print AmCache information |
| **apihooks** | Detect API hooks in process and kernel memory |
| **atoms** | Print session and window station atom tables |
| **atomscan** | Pool scanner for atom tables |
| **auditpol** | Prints out the Audit Policies from HKLM\SECURITY\Policy\PolAdtEv |
| **bigpools** | Dump the big page pools using BigPagePoolScanner |
| **bioskbd** | Reads the keyboard buffer from Real Mode memory |
| **cachedump** | Dumps cached domain hashes from memory |
| **callbacks** | Print system-wide notification routines |
| **clipboard** | Extract the contents of the windows clipboard |
| **cmdline** | Display process command-line arguments |
| **cmdscan** | Extract command history by scanning for _COMMAND_HISTORY |
| **connections** | Print list of open connections [Windows XP and 2003 Only] |
| **connscan** | Pool scanner for tcp connections |
| **consoles** | Extract command history by scanning for _CONSOLE_INFORMATION |
| **crashinfo** | Dump crash-dump information |
| **deskscan** | Poolscaner for tagDESKTOP (desktops) |
| **devicetree** | Show device tree |
| **dlldump** | Dump DLLs from a process address space |
| **dlllist** | Print list of loaded dlls for each process |
| **driverirp** | Driver IRP hook detection |
| **drivermodule** | Associate driver objects to kernel modules |
| **driverscan** | Pool scanner for driver objects |
| **dumpcerts** | Dump RSA private and public SSL keys |
| **dumpfiles** | Extract memory mapped and cached files |
| **dumpregistry** | Dumps registry files out to disk |
| **editbox** | Displays information about Edit controls. (Listbox experimental.) |
| **envars** | Display process environment variables |
| **eventhooks** | Print details on windows event hooks |
| **evtlogs** | Extract Windows Event Logs (XP/2003 only) |
| **filescan** | Pool scanner for file objects |
| **gahti** | Dump the USER handle type information |
| **gditimers** | Print installed GDI timers and callbacks |
| **gdt** | Display Global Descriptor Table |
| **getservicesids** | Get the names of services in the Registry and return Calculated SID |
| **getsids** | Print the SIDs owning each process |
| **handles** | Print list of open handles for each process |
| **hashdump** | Dumps passwords hashes (LM/NTLM) from memory |
| **hibinfo** | Dump hibernation file information |
| **hivedump** | Prints out a hive |
| **hivelist** | Print list of registry hives. |
| **hivescan** | Pool scanner for registry hives |
| **hpakextract** | Extract physical memory from an HPAK file |
| **hpakinfo** | Info on an HPAK file |
| **idt** | Display Interrupt Descriptor Table |
| **iehistory** | Reconstruct Internet Explorer cache / history |
| **imagecopy** | Copies a physical address space out as a raw DD image |
| **imageinfo** | Identify information for the image |
| **impscan** | Scan for calls to imported functions |
| **joblinks** | Print process job link information |
| **kdbgscan** | Search for and dump potential KDBG values |
| **kpcrscan** | Search for and dump potential KPCR values |
| **ldrmodules** | Detect unlinked DLLs |
| **lsadump** | Dump (decrypted) LSA secrets from the registry |
| **machoinfo** | Dump Mach-O file format information |
| **malfind** | Find hidden and injected code |
| **mbrparser** | Scans for and parses potential Master Boot Records (MBRs) |
| **memdump** | Dump the addressable memory for a process |
| **memmap** | Print the memory map |
| **messagehooks** | List desktop and thread window message hooks |
| **mftparser** | Scans for and parses potential MFT entries |
| **moddump** | Dump a kernel driver to an executable file sample |
| **modscan** | Pool scanner for kernel modules |
| **modules** | Print list of loaded modules |
| **multiscan** | Scan for various objects at once |
| **mutantscan** | Pool scanner for mutex objects |
| **notepad** | List currently displayed notepad text |
| **objtypescan** | Scan for Windows object type objects |
| **patcher** | Patches memory based on page scans |
| **poolpeek** | Configurable pool scanner plugin |
| **printkey** | Print a registry key, and its subkeys and values |
| **privs** | Display process privileges |
| **procdump** | Dump a process to an executable file sample |
| **pslist** | Print all running processes by following the EPROCESS lists |
| **psscan** | Pool scanner for process objects |
| **pstree** | Print process list as a tree |
| **psxview** | Find hidden processes with various process listings |
| **qemuinfo** | Dump Qemu information |
| **raw2dmp** | Converts a physical memory sample to a windbg crash dump |
| **screenshot** | Save a pseudo-screenshot based on GDI windows |
| **servicediff** | List Windows services (ala Plugx) |
| **sessions** | List details on _MM_SESSION_SPACE (user logon sessions) |
| **shellbags** | Prints ShellBags info |
| **shimcache** | Parses the Application Compatibility Shim Cache registry key |
| **shutdowntime** | Print ShutdownTime of machine from registry |
| **sockets** | Print list of open sockets |
| **sockscan** | Pool scanner for tcp socket objects |
| **ssdt** | Display SSDT entries |
| **strings** | Match physical offsets to virtual addresses (may take a while, VERY verbose) |
| **svcscan** | Scan for Windows services |
| **symlinkscan** | Pool scanner for symlink objects |
| **thrdscan** | Pool scanner for thread objects |
| **threads** | Investigate _ETHREAD and _KTHREADs |
| **timeliner** | Creates a timeline from various artifacts in memory |
| **timers** | Print kernel timers and associated module DPCs |
| **truecryptmaster** | Recover TrueCrypt 7.1a Master Keys |
| **truecryptpassphrase** | TrueCrypt Cached Passphrase Finder |
| **truecryptsummary** | TrueCrypt Summary |
| **unloadedmodules** | Print list of unloaded modules |
| **userassist** | Print userassist registry keys and information |
| **userhandles** | Dump the USER handle tables |
| **vaddump** | Dumps out the vad sections to a file |
| **vadinfo** | Dump the VAD info |
| **vadtree** | Walk the VAD tree and display in tree format |
| **vadwalk** | Walk the VAD tree |
| **vboxinfo** | Dump virtualbox information |
| **verinfo** | Prints out the version information from PE images |
| **vmwareinfo** | Dump VMware VMSS/VMSN information |
| **volshell** | Shell in the memory image |
| **windows** | Print Desktop Windows (verbose details) |
| **wintree** | Print Z-Order Desktop Windows Tree |
| **wndscan** | Pool scanner for window stations |
| **yarascan** | Scan process or kernel memory with Yara signatures |
