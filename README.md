# rpi_cluster
Hadoop, Spark, MPI


Documentation of
* Construction of a 5-node Raspberry PI 3B+ cluster.
* Installation of Hadoop File System, Spark, Thrift Server, MPI

The cluster will have 1 master node (clustpi01) and 4 slave nodes (clustpi02-05).
Only the master node will be connected to the home wifi router.
The slaves nodes and the master node are connected via a switch and they form a subnet with the master being a DHCP server and gateway.

**Important Note: Instructions are just what worked from me, they are not optimised, nor can I guarantee that they will work for you. You must do your own research and apply what suits your setuup and requirements.**

![Assembled cluster](https://github.com/chseeling/rpi_cluster/blob/master/images/20190106_rpi_cluster.jpg)

## Part List

5x
* Raspberry Pi B+
* 64MB SD Card (Micro SDXC Memory Card)
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
## Setup passwordless ssh access for each node to each node for user pi
Following https://www.raspberrypi.org/documentation/remote-access/ssh/passwordless.md
Creating a public and private key pair

    $ ssh-keygen

Accept all the defaults.
Then run the following command for each of the other nodes to install the public key as an authorized key on the other nodes.

    $ ssh-copy-id pi@clustpi02
To test try logging in with

    $ ssh clustpi02
    or
    $ ssh pi@clustpi02

We have now completed the basic setup of the cluster.
Next we optionally set up a NFS mount of a USB drive on the master, to be shared with the slaves.

## NFS setup
Following https://makezine.com/projects/build-a-compact-4-node-raspberry-pi-cluster/

A NFS mounted drive is useful for making files automatically available across the nodes of the cluster.
For example, the MPI setup benefits from that, as I will show later.

    mkdir  /mnt/usb
    sudo chown -R pi:pi /mnt/usb

Plug USB drive into master and manually mount:

    sudo mount -o uid=pi,gid=pi /dev/sda1 /mnt/usb

If you want to automatically mount on boot you’ll need to add the following to the /etc/fstab file:

    /dev/sda1 /mnt/usb auto defaults,user 0 1
    
On master node

    sudo apt-get install nfs-kernel-server
    
For each slave/worker node:
```
ssh clustpi05 sudo apt-get install nfs-common
ssh clustpi05 sudo mkdir /mnt/nfs
ssh clustpi05 sudo chown -R pi:pi /mnt/nfs
```

On master edit /etc/exports
```
$ sudo su
# add
/mnt/usb clustpi02.local(rw,sync)
/mnt/usb clustpi03.local(rw,sync)
/mnt/usb clustpi04.local(rw,sync)
/mnt/usb clustpi05.local(rw,sync)
# the run
$ exportfs -ra
```

On clients
```
ssh clustpi02 sudo apt-get install nfs-common
ssh clustpi02 sudo mkdir /mnt/nfs
ssh clustpi02 sudo chown -R pi:pi /mnt/nfs
```

## Install autofs on each node

    sudo apt-get install autofs
 
 to /etc/auto.master add at end:

    /mnt/nfs        /etc/auto.nfs


create file /etc/auto.nfs and add:

    usb    -rw,soft clustpi01.local:/mnt/usb
run    
    
    sudo /etc/init.d/autofs restart

    
Check status of services with

    service --status-all


### Next [Install and Test MPI](https://github.com/chseeling/rpi_cluster/blob/master/MPI.md)
or
### Next [Install and Test Hadoop File System](https://github.com/chseeling/rpi_cluster/blob/master/HDFS.md)
