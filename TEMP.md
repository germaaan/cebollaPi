



#CUIDADO
sudo update-rc.d isc-dhcp-server enable
sudo update-rc.d hostapd enable
sudo update-rc.d tor enable

sudo service isc-dhcp-server start
sudo service hostapd start
sudo service tor start



sudo ifdown wlan0
sudo ifup wlan0
