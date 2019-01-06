# rpi_cluster
Hadoop, Spark, MPI



Documentation of
* Construction of a 5-node Raspberry PI 3B+ cluster.
* Installation of Hadoop File System, Spark, Thrift Server, MPI

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

    sudo raspi-config

* change node name to clustpi01
* expand file system

Repoot.
