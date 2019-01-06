# rpi_cluster
Hadoop, Spark, MPI


Documentation of
* Construction of a 5-node Raspberry PI 3B+ cluster.
* Installation of Hadoop File System, Spark, Thrift Server, MPI

The cluster will have 1 master node (clustpi01) and 4 slave nodes (clustpi02-05).
Only the master node wll be connected to the home wifi router.
The slaves nodes and the master node are connected via a switch and they form a subnet with the master being a DHCP server and gateway.

**Important Note: Instructions are just what worked from me, they are not optimised, nor can I guarantee that they will work for you. You must do your own research and apply what suits your setuup and requirements.**

![Assembled cluster](https://github.com/chseeling/rpi_cluster/blob/master/images/20190106_rpi_cluster.jpg)

## Part List

5x
* Raspberry Pi B+
* 64MB SD Card
* USB-C to USB-A cable
* Ethernet Patch Lan Cable (1 foot)

1x
* Edimax ES-5500G V3 5-Port Gigabit Switch
* Comsol 6 Port Desktop USB Charger  (60W, 12A, output 2.4A per port)
* Clear Acrylic 5 Layer Cluster Case Shelf 


Optional
* USB SSD or USB stick for NFS

## Tools
Laptop or Desktop (I used a Windows 10 laptop)
Putty
Access to wifi router.

## Preparing SSD cards

Download [Raspbian Stretch](https://www.raspberrypi.org/downloads/raspbian/) image. I used

    2018-11-13-raspbian-stretch-full.zip

Flash image to SSD card using [Etcher](https://www.balena.io/etcher/) or equivalent. After completion add two files to the boot partition:
* ssh
* wpa_supplicant.conf  containing
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
network={
    ssid="YOUR_NETWORK_NAME"
    psk="YOUR_PASSWORD"
    key_mgmt=WPA-PSK
}
```
## Booting and configuring Master node

Insert SSD card and power up node that will become master node.
Find the IP address of the new node by checking the wifi router (there are many other ways to find the IP).
Use putty and ssh into node using IP, user:pi, password: raspberry

    sudo raspi-config

* change node name to clustpi01
* expand file system

Reboot.

### Assign static IP address to wlan0
Choose whatever IP address is free on your router. I chose 192.168.0.27
```
sudo nano /etc/dhcpcd.conf
#adding lines
interface wlan0
static ip_address=192.168.0.27
static routers=192.168.0.1
static domain_name_servers=8.8.8.8
```
sudo reboot

### Set up DHCP and wifi bridge on master
I followed https://pimylifeup.com/raspberry-pi-wifi-bridge/

My /etc/dnsmasq.conf  looks like this (xx need reflect your actual mac address)

```
# General configuration
interface=eth0
listen-address=192.168.50.1
bind-interfaces
domain-needed
bogus-priv
domain=cluster
dhcp-range=192.168.50.10,192.168.50.254,48h
dhcp-authoritative
dhcp-host=b8:27:eb:xx:xx:xx,192.168.50.12
dhcp-host=b8:27:eb:xx:xx:xx,192.169.50.13
dhcp-host=b8:27:eb:xx:xx:xx,192,169.50.14
dhcp-host=b8:27:eb:xx:xx:xx,192.169.50.15
```

## Set up each slave node
The setup is similar to the master node, except for the DHCP server setup, which is not required.
The wifi is only needed temporarily to find the mac address.

I also added the following to /etc/hosts file of each node (this may be redundant):
```
192.168.50.1    clustpi01.local
192.168.50.1    clustpi01
192.168.50.12   clustpi02.local
192.168.50.12   clustpi02
192.168.50.13   clustpi03.local
192.168.50.13   clustpi03
192.168.50.14   clustpi04.local
192.168.50.14   clustpi04
192.168.50.15   clustpi05.local
192.168.50.15   clustpi05
```

Once network setup is complete, check with:
```
$ nmap -sn 192.168.50.0/24
Starting Nmap 7.40 ( https://nmap.org ) at 2019-01-06 11:31 GMT
Nmap scan report for clustpi01.local (192.168.50.1)
Host is up (0.0019s latency).
Nmap scan report for clustpi02.local (192.168.50.12)
Host is up (0.0023s latency).
Nmap scan report for clustpi03.local (192.168.50.13)
Host is up (0.00071s latency).
Nmap scan report for clustpi04.local (192.168.50.14)
Host is up (0.00066s latency).
Nmap scan report for clustpi05.local (192.168.50.15)
Host is up (0.0021s latency).
Nmap done: 256 IP addresses (5 hosts up) scanned in 2.98 seconds
```
