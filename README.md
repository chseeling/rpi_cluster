# rpi_cluster
Hadoop, Spark, MPI


Documentation of
* Construction of a 5-node Raspberry PI 3B+ cluster.
* Installation of Hadoop File System, Spark, Thrift Server, MPI

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
