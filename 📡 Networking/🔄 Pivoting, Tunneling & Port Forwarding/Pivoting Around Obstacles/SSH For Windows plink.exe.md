Plink (PuTTY link), is a Windows command-line SSH tool that comes as a part of the PuTTY package when installed.

If we land on a Windows host and need to use it to pivot, we can use what is already there. If the host is older and PuTTY is present (or we can find a copy on a file share), we can use it to create our pivot and potentially avoid detection a little longer. It is useful if we cannot import our own tools without being exposed.

That is just one potential scenario where Plink could be beneficial. We could also use Plink if we use a Windows system as our primary attack host instead of a Linux-based system.

In the image below, we have a Windows-based attack host.

![[66.webp]]

- We can start a `dynamic port forward` on the `Ubuntu server` using :

```bash
plink -ssh -D 9050 ubuntu@10.129.15.50
```

- Another Windows-based tool called [Proxifier](https://www.proxifier.com/) can be used to start a SOCKS tunnel via the SSH session we created. Proxifier is a Windows tool that creates a tunneled network for desktop client applications and allows it to operate through a SOCKS or HTTPS proxy and allows for proxy chaining. It is possible to create a profile where we can provide the configuration for our SOCKS server started by Plink on port 9050.

![[reverse_shell_9.webp]]

After configuring the SOCKS server for `127.0.0.1` and port 9050, we can directly start `mstsc.exe` to start an RDP session with a Windows target that allows RDP connections.