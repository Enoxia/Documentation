The vulnerability was assigned [CVE-2023-32784](https://www.cve.org/CVERecord?id=CVE-2023-32784) and fixed in [KeePass 2.54](https://keepass.info/news/n230603_2.54.html).

In `KeePass 2.x before 2.54`, it is possible to recover the `cleartext master password` from a `memory dump`, even when a workspace is locked or no longer running. 

The `memory dump` can be a KeePass process dump, swap file (pagefile.sys), hibernation file (hiberfil.sys), or RAM dump of the entire system. The first character cannot be recovered. In 2.54, there is different API usage and/or random string insertion for mitigation.

Here a some useful resources to do this : 
- [Keepass-password-dumper](https://github.com/vdohney/keepass-password-dumper)
- [Keepass-dump-masterkey](https://github.com/matro7sh/keepass-dump-masterkey)
- [Keepass-dump-extractor](https://github.com/JorianWoltjer/keepass-dump-extractor)

