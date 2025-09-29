
# FPING

```
# Discover hosts on a network
fping -I eth0 -g 10.10.10.0/24 -a 2>/dev/null

Bien comparer les résultats, car on peut trouver une IP avec un `arp-scan` qui serait pas présente dans le `ping` car n’y réponds pas (ex: Windows)
```


# ZENMAP

GUI comes by default on Kali.
Set the IP address of the network in “Target” and then clic on “Scan”

And we can get a typology (graph) of all the discovered hosts