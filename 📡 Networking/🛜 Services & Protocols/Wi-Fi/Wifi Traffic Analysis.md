Wireshark Filter for Wifi Analysis :

`(wlan.fc.type_subtype== 0x0008) && !(wlan.wfa.ie.wpa.version == 1) && !(wlan.tag.number==48)`

`wlan contains Home_Network` // search packets from SSID Home_Network

`(wlan.ssid contains Amazon) && (wlan.fc.type_subtype == 0x0008)`

`(wlan.ta == e8:de:27:16:87:18) || (wlan.ra == e8:de:27:16:87:18)` // search packets transmited or received by that specific mac address

`wlan contains SecurityTube`

`((wlan.bssid == e8:de:27:16:87:18) && (wlan.fc.type_subtype== 0x0020))`

- **Walkthrough of the lab :**

[https://assets.ine.com/labs/ad-manuals/walkthrough-1141.pdf](https://assets.ine.com/labs/ad-manuals/walkthrough-1141.pdf)

BSSID = @MAC du matériel (point d’accès)

SSID = NomDuRéseau

# Filtering Wifi with Tshark :

`tshark -r WiFi_traffic.pcap -Y 'wlan'` // Filter all WiFi packets

`tshark -r WiFi_traffic.pcap -Y 'wlan.fc.type_subtype==0x000c'` // Deauthentication

`tshark -r WiFi_traffic.pcap -Y 'eapol'` // Display only WPA handshake

`tshark -r WiFi_traffic.pcap -Y 'wlan.fc.type_subtype==8' -Tfields -e wlan.ssid -e wlan.bssid` // Only print SSID and BSSID value of all the beacon frames

`tshark -r WiFi_traffic.pcap -Y 'wlan.ssid==LazyArtists' -Tfields -e wlan.bssid` // Quelle est l’adresse MAC du SSID LazyArtists

`tshark -r WiFi_traffic.pcap -Y 'wlan.ssid==Home_Network' -Tfields -e wlan_radio.channel` // See on which channel is operating the ssid Home_Network

`tshark -r WiFi_traffic.pcap -Y 'wlan.fc.type_subtype==0x000c' -Tfields -e wlan.ra` // See which 2 devices received the Deauthen. messages ?

`tshark -r WiFi_traffic.pcap -Y 'wlan.ta==5c:51:88:31:a0:3b && http' -Tfields -e http.user_agent` // See which device the specified MAC address belong to, using HTTP user_agent

- **Walkthrough of the lab :**

[https://assets.ine.com/labs/ad-manuals/walkthrough-4.pdf](https://assets.ine.com/labs/ad-manuals/walkthrough-4.pdf)