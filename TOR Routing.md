# The Onion Router on Raspberry pi(iot)

### PLEASE GO THROUGH README.MD



**Just Follow these Step when you setuped the RPI**
``
export LANGUAGE=en_US.UTF-8
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
locale-en_US.UTF-8
sudo dpkg-reconfigure locales
sudo apt-get update
sudo apt-get install hostapd isc-dhcp-server
sudo apt-get install iptables-persistent
sudo nano /etc/dhcp/dhcpd.conf
``
**Find the lines**

option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;

**and change them to add a # in the beginning so they say**
#option domain-name "example.org";
#option domain-name-servers ns1.example.org, ns2.example.org;


**Find the lines that say**
#If this DHCP server is the official DHCP server for the local
#network, the authoritative directive should be uncommented.
#authoritative;

**and remove the # so it says**

authoritative;

**Then scroll down to the bottom and add the following lines**
``
subnet 192.168.42.0 netmask 255.255.255.0 {
	range 192.168.42.10 192.168.42.50;
	option broadcast-address 192.168.42.255;
	option routers 192.168.42.1;
	default-lease-time 600;
	max-lease-time 7200;
	option domain-name "local";
	option domain-name-servers 8.8.8.8, 8.8.4.4;
}
``
Ctrl+x to save;
``
sudo nano /etc/default/isc-dhcp-server
``

****scroll down to INTERFACES="" and update it to say INTERFACES="wlan0"


`
sudo ifconfig  wlan0 down
sudo nano /etc/network/interfaces
`

**Find the line auto wlan0 and add a # in front of the line, and in front of every line afterwards.
If you don't have that line, just make sure it looks like the screenshot below in the end!
Basically just remove any old wlan0 configuration settings, we'll be changing them up**

`
auto lo

iface lo inet loopback 
iface usb0 inet dhcp

allow-hotplug wlan0

iface wlan0 inet static
 address 192.168.42.1
 netmask 255.255.255.0

#iface wlan0 inet manual
#wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
#iface default inet dhcp
`
ctrl+x to save;

``
sudo ifconfig wlan0 192.168.42.1
sudo nano /etc/hostapd/hostapd.conf

interface=wlan0
#driver=rtl871xdrv
ssid=TORNet
country_code=US
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=Raspberry
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP
wpa_group_rekey=86400
ieee80211n=1
wme_enabled=1

sudo nano /etc/default/hostapd
``
**Find the line #DAEMON_CONF="" and edit it so it says DAEMON_CONF="/etc/hostapd/hostapd.conf"
**
``
sudo nano /etc/sysctl.conf
``
**Scroll to the bottom and uncomment 
**
`net.ipv4.ip_forward=1`

``
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i usb0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o usb0 -j ACCEPT
sudo sh -c "iptables-save > /etc/iptables/rules.v4"

sudo /usr/sbin/hostapd /etc/hostapd/hostapd.conf
``
**SHOULD BE ABLE TO SEE IF NETWORK IS UP TEST
**
``
sudo mv /usr/share/dbus-1/system-services/fi.epitest.hostap.WPASupplicant.service ~/

sudo reboot 
sudo /usr/sbin/hostapd /etc/hostapd/hostapd.conf
sudo service hostapd start 
sudo service isc-dhcp-server start
sudo update-rc.d hostapd enable 
sudo update-rc.d isc-dhcp-server enable
sudo ifup usb0
sudo ifup wlan0(if neccesary)
``
**CHECK TO SEE IF ITS UP AND WORKING
**
``
sudo service isc-dhcp-server status
sudo service hostapd status
``

**Yeah  the router has successfully configured ;)**
**Now we need to route oru Traffic through TOR Network**

**TOR Access Point**

``
sudo apt-get update
sudo apt-get install tor
sudo nano /etc/tor/torrc
``
**and copy and paste the text into the top of the file, right below the the FAQ notice.**

``

Log notice file /var/log/tor/notices.log
VirtualAddrNetwork 10.192.0.0/10
AutomapHostsSuffixes .onion,.exit
AutomapHostsOnResolve 1
TransPort 192.168.42.1:9040
TransPort 127.0.0.1:9040
TransListenAddress 192.168.42.1
DNSPort 127.0.0.1:53
DNSListenAddress 192.168.42.1
``

``
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 22 -j REDIRECT --to-ports 22
sudo iptables -t nat -A PREROUTING -i wlan0 -p udp --dport 53 -j REDIRECT --to-ports 53
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --syn -j REDIRECT --to-ports 9040
sudo iptables -t nat -L
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
``

** if needed  | **
``
sudo touch /var/log/tor/notices.log
sudo chown debian-tor /var/log/tor/notices.log
sudo chmod 644 /var/log/tor/notices.log
ls -l /var/log/tor
}
sudo service tor start
sudo service tor status
sudo update-rc.d tor enable
``

**On Wakeup **
```
sudo ifup usb0
sudo service isc-dhcp-server status
sudo service hostapd status

sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 22 -j REDIRECT --to-ports 22
sudo iptables -t nat -A PREROUTING -i wlan0 -p udp --dport 53 -j REDIRECT --to-ports 53
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --syn -j REDIRECT --to-ports 9040
sudo iptables -t nat -L
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"


sudo service tor start
```
