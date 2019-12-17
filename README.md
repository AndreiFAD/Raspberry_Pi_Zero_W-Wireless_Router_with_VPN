# Raspberry_Pi_Zero_W-Wireless_Router_with_VPN
Raspberry Pi Zero W – Wireless Router with VPN

Prerequisites:
- Raspberry Pi Zero W (link) 
- USB WiFi adapter (link) 
- One old USB cable 
- Micro sd card class 10 (I used 8 GB SanDisk ultra) 
- Raspberry Pi Zero Case (link) 
- Raspbian Buster with desktop (link)

1. Copy the rasbian image to sd card:

Mac: https://www.raspberrypi.org/documentation/installation/installing-images/mac.md
Windows: https://www.raspberrypi.org/documentation/installation/installing-images/windows.md

2. Install packages:

$ sudo apt–get update && sudo apt–get upgrade –y<br>
$ sudo apt–get install hostapd dnsmasq –y<br>

## DHCP Configuration

3. Static IP address for wlan1:

$ sudo nano /etc/dhcpcd.conf<br>
Copy at the end of the file:<br>
interface wlan1<br>
static ip_address=192.168.111.254/24<br>
nohook wpa_supplicant<br>
denyinterfaces wlan1<br>

4. IP range for wlan1:

Create a backup file:<br>
$ sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.default<br>

Create a new file:<br>
$ sudo nano /etc/dnsmasq.conf

Copy this to new file:<br>
interface=wlan1<br>
dhcp–range=192.168.111.1,192.168.111.20,255.255.255.0,24h<br>

## Configuring a Wireless Access Point

5. Configure hostapd:

$ sudo nano /etc/hostapd/hostapd.conf<br>
Copy this to the file:<br>
interface=wlan1<br>
hw_mode=g<br>
channel=1<br>
wmm_enabled=0<br>
macaddr_acl=0<br>
auth_algs=1<br>
ignore_broadcast_ssid=0<br>
wpa=2<br>
wpa_key_mgmt=WPA-PSK<br>
wpa_pairwise=TKIP<br>
rsn_pairwise=CCMP<br>
ssid=PrivateVpn<br>
wpa_passphrase=12345678910<br>
ieee80211n=1<br>

$ sudo nano /etc/default/hostapd<br>
change this line:<br>
DAEMON_CONF=“” to DAEMON_CONF=“/etc/hostapd/hostapd.conf”<br>

Enable and start hostapd:<br>
$ sudo systemctl unmask hostapd.service<br>
$ sudo systemctl enable hostapd.service<br>
$ sudo systemctl start hostapd.service<br>

6. Enabling traffic forwarding and forwarding rule configuration:

$ sudo nano /etc/sysctl.conf<br>
Uncomment this line:<br>
#net.ipv4.ip_forward=1<br>

$ sudo iptables –t nat –A POSTROUTING –o wlan0 –j MASQUERADE<br>
$ sudo iptables –A FORWARD –m conntrack —ctstate RELATED,ESTABLISHED –j ACCEPT<br>
$ sudo iptables –A FORWARD –i wlan1 –o wlan0 –j ACCEPT<br>
$ sudo sh –c “iptables-save > /etc/iptables.ipv4.nat”<br>
$ sudo nano /etc/rc.local<br>

Add this line above “exit 0”:<br>
iptables–restore < /etc/iptables.ipv4.nat<br>

7. Set wifi network for wlan0:

$ sudo nano /etc/wpa_supplicant/wpa_supplicant.conf

Copy this to the file:<br>
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev<br>
update_config=1<br>
country=HU #set your country <br>
network={<br>
        ssid=”home_wifi”<br>
        psk=”passwd”<br>
        id_str=”home”<br>
}<br>
network={<br>
        ssid=”work_wifi”<br>
        scan_ssid=1  #if SSID is hidden<br>
        psk=”passwd”<br>
        id_str=”work”<br>
}<br>

## Configuring a VPN Surfshark

If you sign in with Android or IOS app you can try with 7 days free trial

8. After registration (!) login to website:

https://account.surfshark.com/login

and go to

https://account.surfshark.com/setup/manual

at the bottom of the page:<br>
Get service credentials<br>
These login details are only valid for manual setup: Username and Password<br>

9. How to set up OpenVPN using Linux Terminal:<br>
(it is from here https://support.surfshark.com/hc/en-us/articles/360011051133-How-to-set-up-OpenVPN-using-Linux-Terminal)

Install the necessary packages by entering the command:<br>
$ sudo apt-get install openvpn unzip<br>
  If you are requested to enter your password, please enter your computer’s admin password.<br>
  Navigate to OpenVPN directory by entering:<br>
$ cd /etc/openvpn<br>
  Download Surfshark OpenVPN configuration files:<br>
$ sudo wget https://account.surfshark.com/api/v1/server/configurations<br>
  Extract `configurations.zip`:<br>
$ sudo unzip configurations<br>
  Remove the .zip file which will not be used:<br>
$ sudo rm configurations<br>
  To see the list of all the available servers enter:<br>
$ ls<br>
  Choose one of the servers from the servers list and connect to Surfshark by entering:<br>
$ sudo openvpn [file name]<br>
  For example:<br>
$ sudo openvpn us-dal.prod.surfshark.com_udp.ovpn<br>
$ sudo openvpn /etc/openvpn/us-dal.prod.surfshark.com_udp.ovpn<br>

10. Enabling traffic forwarding and forwarding rule configuration with VPN:

$ sudo nano /etc/sysctl.conf<br>
Uncomment this line:<br>
#net.ipv4.ip_forward=1<br>

$ sudo iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE<br>
$ sudo iptables -A FORWARD -i wlan1 -o tun0 -j ACCEPT<br>
$ sudo iptables -A FORWARD -i tun0 -o wlan1 -m state –state RELATED,ESTABLISHED -j ACCEPT<br>
$ sudo sh -c “iptables-save > /etc/iptables.restore” <br>
$ echo “up iptables-restore < /etc/iptables.restore” | sudo tee –append /etc/network/interfaces<br>

Create file for authentication:<br>
$ sudo nano /etc/openvpn/auth.txt

Add username and password:<br>
useassdasdasd<br>
passsdsaddsas

Run OpenVPN and connect example to USA:<br>
$ sudo openvpn –config “/etc/openvpn/us-dal.prod.surfshark.com_udp.ovpn” –auth-user-pass “/etc/openvpn/auth.txt”

And if “Initialization Sequence Completed” you can using and test on https://whatismycountry.com

## Raspberry Pi Zero W build with USB WIFI adapter and with USB

## References:<br>
https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md<br>
https://www.instructables.com/id/Use-Raspberry-Pi-3-As-Router/<br>
https://www.raspberrypi.org/documentation/configuration/tcpip/README.md<br>
https://www.raspberrypi.org/documentation/installation/installing-images/mac.md<br>
https://www.raspberrypi.org/documentation/installation/installing-images/windows.md<br>
https://support.surfshark.com/



