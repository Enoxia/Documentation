Acropalypse is the latest critical vulnerability affecting the Windows Snipping Tool! Microsoft has just released an emergency update. Here is the rundown.

Now tracked as [CVE-2023-28303](https://www.it-connect.fr/acropalypse-microsoft-deploie-une-mise-a-jour-pour-cette-faille-critique-dans-loutil-capture-de-windows/), the Acropalypse vulnerability found in the Windows Snipping Tool can leak sensitive data and compromise the confidentiality of your screenshots.
So, how does it work?

Essentially, when you take a screenshot using this built-in Windows tool, you might edit the capture (specifically by cropping it) to hide sensitive details—such as an IP address, a username, or an account number—before sharing it. However, due to this security flaw dubbed 'Acropalypse', it is possible to revert these edits and reveal the hidden information within the image !

[SnipRecover](https://github.com/m31r0n/SnipRecover-CLI) is a tool that can be used to do that. It can **detect PNG files** modified by the Windows Snipping Tool vulnerability (CVE‑2023‑28303), and **restore the original** image by recovering compressed data appended after the IEND chunk.
