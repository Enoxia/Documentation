# Enumeration
```
nmap -sV -p 873 $IP
```


# Interacting w/ service
```
nc -nv 127.0.0.1 873

(UNKNOWN) [127.0.0.1] 873 (rsync) open
@RSYNCD: 31.0
@RSYNCD: 31.0
#list
dev            	Dev Tools

# Enumerate the found share "dev"
rsync -av --list-only rsync://127.0.0.1/dev

# Sync all files to our machine
rsync -av rsync://127.0.0.1/dev

# If Rsync is configured w/ SSH, we can use the "-e ssh" or "-e "ssh -p2222""
```
[How to Rsync using SSH](https://phoenixnap.com/kb/how-to-rsync-over-ssh)

