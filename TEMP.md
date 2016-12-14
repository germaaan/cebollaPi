https://www.raspberrypi.org/downloads/noobs/

https://downloads.raspberrypi.org/NOOBS_latest



sudo raspi-config
"Advanced options" -> "SSH", "Enable or disable ssh server" -> YES


sudo apt update && sudo sudo apt full-upgrade -y

sudo apt install hostapd isc-dhcp-server tor

sudo nano /etc/default/isc-dhcp-server

  INTERFACES="wlan0"

sudo nano /etc/network/interfaces

#allow-hotplug wlan0
#iface wlan0 inet manual
#    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

iface wlan0 inet static
    address 192.168.1.150
    netmask 255.255.255.0
    broadcast 192.168.1.255
    gateway 192.168.1.1

sudo nano /etc/dhcp/dhcpd.conf

  cambiar
    # option definitions common to all supported networks...
    option domain-name "cebollapi";
    option domain-name-servers 8.8.8.8, 8.8.4.4;

  descomentar
    # If this DHCP server is the official DHCP server for the local
    # network, the authoritative directive should be uncommented.
    authoritative;

  comprobar
    default-lease-time 600;
    max-lease-time 7200;

  añadir
    subnet 192.168.1.0 netmask 255.255.255.0 {
      range 192.168.1.160 192.168.1.170;
      option subnet-mask 255.255.255.0;
      option broadcast-address 192.168.1.255;
      option routers 192.168.1.150;
      option domain-name-servers home;
    }

sudo nano /etc/resolv.conf

nameserver 8.8.8.8
nameserver 8.8.4.4

sudo nano /etc/hostapd/hostapd.conf

# This is the name of the WiFi interface we configured above
interface=wlan0
# Use the nl80211 driver with the brcmfmac driver
driver=nl80211
# This is the name of the network
ssid=SuperNova
# Use the 2.4GHz band
hw_mode=g
# Use channel 6
channel=6
# Accept all MAC addresses
macaddr_acl=0
# Use WPA authentication
auth_algs=1
# Require clients to know the network name
ignore_broadcast_ssid=0
# Use WPA2
wpa=2
# Use a pre-shared key
wpa_key_mgmt=WPA-PSK
# The network passphrase
wpa_passphrase=clavemuysegura
# Use AES, instead of TKIP
wpa_pairwise=TKIP
rsn_pairwise=CCMP

interface=wlan0
driver=nl80211
ssid=Pi_AP
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=Raspberry
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP


sudo nano /etc/default/hostapd
DAEMON_CONF="/etc/hostapd/hostapd.conf"


wget http://www.adafruit.com/downloads/adafruit_hostapd.zip
unzip adafruit_hostapd.zip
sudo mv /usr/sbin/hostapd /usr/sbin/hostapd.ORIG
sudo mv hostapd /usr/sbin
sudo chmod 755 /usr/sbin/hostapd


sudo nano /etc/sysctl.conf
descomentar net.ipv4.ip_forward=1
sudo sysctl -p

sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 22 -j REDIRECT --to-ports 22
sudo iptables -t nat -A PREROUTING -i wlan0 -p udp --dport 53 -j REDIRECT --to-ports 53
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --syn -j REDIRECT --to-ports 9040
sudo sh -c 'iptables-save > /etc/iptables.ipv4.nat'

sudo nano /etc/network/interfaces
up iptables-restore /etc/iptables.ipv4.nat


sudo nano /etc/tor/torrc
despues de
## https://www.torproject.org/docs/faq#torrc
añadir
Log notice file /var/log/tor/notices.log
VirtualAddrNetwork 10.192.0.0/10
AutomapHostsSuffixes .onion,.exit
AutomapHostsOnResolve 1
TransPort 9040
TransListenAddress 192.168.1.1
DNSPort 53
DNSListenAddress 192.168.1.1

sudo touch /var/log/tor/notices.log
sudo chown debian-tor /var/log/tor/notices.log
sudo chmod 644 /var/log/tor/notices.log

sudo service tor start

#CUIDADO
sudo update-rc.d hostapd enable
sudo update-rc.d isc-dhcp-server enable
sudo update-rc.d tor enable

sudo service hostapd start
sudo service isc-dhcp-server start

sudo ifdown wlan0
sudo ifup wlan0
