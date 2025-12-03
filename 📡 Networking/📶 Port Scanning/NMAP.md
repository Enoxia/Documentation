# Scanning Options

| **Nmap Option**                                   | **Description**                                                                                                                                        |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `-p- --min-rate=10000 -T4 -A $IP`                 | Default agressive CTF Linux scan                                                                                                                       |
| `-p- -nv -Pn --min-rate=10000 -sV -sC -T4 -A $IP` | Default agressive CTF Windows scan                                                                                                                     |
| `-sn 10.10.10.0/24`                               | Target network range.                                                                                                                                  |
| `-sn 10.10.10.1-100`                              | Target IP range (from 10.10.10.1 to 10.10.10.100)                                                                                                      |
| `-sn`                                             | Disables port scanning.                                                                                                                                |
| `-Pn`                                             | Disables ICMP Echo Requests                                                                                                                            |
| `-n`                                              | Disables DNS Resolution.                                                                                                                               |
| `-PE`                                             | Performs the ping scan by using ICMP Echo Requests against the target.                                                                                 |
| `--packet-trace`                                  | Shows all packets sent and received.                                                                                                                   |
| `--reason`                                        | Displays the reason for a specific result.                                                                                                             |
| `--disable-arp-ping`                              | Disables ARP Ping Requests.                                                                                                                            |
| `--top-ports=<num>`                               | Scans the specified top ports that have been defined as most frequent.                                                                                 |
| `-p-`                                             | Scan all ports.                                                                                                                                        |
| `-p22-110`                                        | Scan all ports between 22 and 110.                                                                                                                     |
| `-p22,25`                                         | Scans only the specified ports 22 and 25.                                                                                                              |
| `-F`                                              | Fast port scan, scans top 100 ports.                                                                                                                   |
| `-sS`                                             | Performs an TCP SYN-Scan.                                                                                                                              |
| `-sA`                                             | Performs an TCP ACK-Scan, better for IDS / IPS evasion.                                                                                                |
| `-sU`                                             | Performs an UDP Scan.                                                                                                                                  |
| `-sV`                                             | Scans the discovered services for their versions.                                                                                                      |
| `-sC`                                             | Perform a Script Scan with scripts that are categorized as "default".                                                                                  |
| `-sT`                                             | TCP Connect Scan, using three-way handshake. Most polite and stealthy scan (don’t disturb other services) for not being seen by IDS                    |
| `sudo nmap --script-updatedb`                     | Update the database of NSE scripts                                                                                                                     |
| `--script <script>`                               | Performs a Script Scan by using the specified scripts.                                                                                                 |
| `--script-trace`                                  | Trace the progress of NSE scripts at the network level                                                                                                 |
| `-O`                                              | Performs an OS Detection Scan to determine the OS of the target.                                                                                       |
| `-A`                                              | Performs OS Detection, Service Detection, and traceroute scans.                                                                                        |
| `-D RND:5`                                        | Used to generate random IP addresses for the scan instead of using our own (in case some IPS block IP). Here it sets the number of random Decoys at 5. |
| `-e`                                              | Specifies the network interface that is used for the scan.                                                                                             |
| `-S 10.10.10.200`                                 | Specifies the source IP address for the scan. The idea is to set an IP which is in the same network as the target                                      |
| `-g`                                              | Specifies the source port for the scan.                                                                                                                |
| `--dns-server <ns>`                               | DNS resolution is performed by using a specified name server.                                                                                          |
| `--packet-trace -n --disable-arp-ping -Pn`        | Track how a packet send is handled, and get more details                                                                                               |
| `--source-port 53`                                | Performs the scans from the specified source port 53 (to be trust by firewalls and bypass IPS)                                                         |


# Output Options

|**Nmap Option**|**Description**|
|---|---|
|`-oA filename`|Stores the results in all available formats starting with the name of "filename".|
|`-oN filename`|Stores the results in normal format with the name "filename".|
|`-oG filename`|Stores the results in "grepable" format with the name of "filename".|
|`-oX filename`|Stores the results in XML format with the name of "filename".|


# Performance Options

|**Nmap Option**|**Description**|
|---|---|
|`--max-retries <num>`|Sets the number of retries for scans of specific ports.|
|`--stats-every=5s`|Displays scan's status every 5 seconds.|
|`-v/-vv`|Displays verbose output during the scan.|
|`--initial-rtt-timeout 50ms`|Sets the specified time value as initial RTT timeout.|
|`--max-rtt-timeout 100ms`|Sets the specified time value as maximum RTT timeout.|
|`--min-rate 300`|Sets the number of packets that will be sent simultaneously.|
|`-T <0-5>`|Specifies the specific timing template.|
|`/usr/share/nmap/scripts/`|Default location of scripts|
|`-iL`|Use a list of hosts in a file|


# Network Discovering
```bash
# Using netdiscover
sudo netdiscover -i eth0 -r 10.10.10.0/24

# Using bash, we'll get all subnets 
for i in $(seq 254); do ping 192.168.$i.254 -c1 -W1 & done | grep from | awk '{print $4}' | sed 's/://' | sort -u > all_networks

# We can retrieve all alive hosts from different subnets
while read ip; do base=$(echo $ip | awk -F. '{print $1"."$2"."$3}'); for i in $(seq 1 254); do ping -c1 -W1 $base.$i &>/dev/null && echo $base.$i & done; wait; done < all_networks | sort -u > all_hosts
```

# Hosts Discovering
We could use bash to do a ping sweep :
```bash
# This will scan for all available hosts in the network, and save all ip addresses that responded to the ping request to the alive_hosts file
for i in $(seq 254); do ping 10.10.10.$i -c1 -W1 & done | grep from | awk '{print $4}' | sed 's/://' | sort -u > alive_hosts```
```

# Scan Alive Hosts
If we’re during a pentest and the client provided us a list of hosts, we can use it directly with nmap :
```bash
cat alive_hosts
10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
10.129.2.19
10.129.2.20
10.129.2.28

sudo nmap -sn -oA tnet -iL hosts.lst | grep for | cut -d" " -f5

10.129.2.18
10.129.2.19
10.129.2.20
```

Remember, this may mean that the other hosts ignore the default **ICMP echo requests** because of their firewall configurations. Since `Nmap` does not receive a response, it marks those hosts as inactive.

Other host discovery techniques :

[Putting It All Together: Host Discovery Strategies | Nmap Network Scanning](https://nmap.org/book/host-discovery-strategies.html)


# Don’t forget to scan for UDP Ports !

`nmap -sUV`
