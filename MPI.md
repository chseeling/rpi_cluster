# MPI Install and Testing

following
https://www.howtoforge.com/tutorial/hpl-high-performance-linpack-benchmark-raspberry-pi/


On all nodes run (n = 1,2,3,4,5):

    ssh clustpi0n sudo apt-get install mpich

Create machine file
```
clustpi01
...
clustpi05
```

Test with:

    mpiexec -f ~/mpich-3.3/machinefile -n 5 hostname


For compiling mpi applications, on master:

    wget http://www.mpich.org/static/downloads/3.3/mpich-3.3.tar.gz
    tar xzf mpich-3.3.tar.gz

```
cd ~/mpich-3.3/examples
mpicc -o cpi cpi.c
cp cpi /mnt/nfs/usb
cd /mnt/nfs/usb
mpiexec -n 4 -f ~/mpich-3.3/machinefile /mnt/nfs/usb/cpi
```
## Running Linpack benchmark
following https://www.howtoforge.com/tutorial/hpl-high-performance-linpack-benchmark-raspberry-pi/

Download source from

     http://www.netlib.org/benchmark/hpl/
     
```
tar xf hpl-2.3.tar.gz
cd hpl-2.3/setup
sh make_generic
cd ..
cp setup/Make.UNKNOWN Make.rpi
```

Adjust the Make.rpi file

    nano Make.rpi

```
ARCH         = rpi
#
# ----------------------------------------------------------------------
# - HPL Directory Structure / HPL library ------------------------------
# ----------------------------------------------------------------------
#
TOPdir       = $(HOME)/hpl-2.3
INCdir       = $(TOPdir)/include
BINdir       = $(TOPdir)/bin/$(ARCH)
LIBdir       = $(TOPdir)/lib/$(ARCH)
#
HPLlib       = $(LIBdir)/libhpl.a
#
# ----------------------------------------------------------------------
# - Message Passing library (MPI) --------------------------------------
# ----------------------------------------------------------------------
# MPinc tells the  C  compiler where to find the Message Passing library
# header files,  MPlib  is defined  to be the name of  the library to be
# used. The variable MPdir is only used for defining MPinc and MPlib.
#
MPdir        = /home/pi/mpich-install
MPinc        = -I $(MPdir)/include
MPlib        = $(MPdir)/lib/libmpich.so
```
run

    make arch=rpi

on the nfs mount, create directory

    md /mnt/nfs/usb/testHPL
    cd /mnt/nfs/usb/testHPL
    cp ~/mpich-install/lib .
    cp ~/hpl-2.3/bin/rpi/* .

Edit HPL.dat to set N, NB, P and Q by following p. 7 of http://www.crc.nd.edu/~rich/CRC_Summer_Scholars_2014/HPL-HowTo.pdf 

For example I used N=12992, NB = 224, P=5 and Q=4, therefor P*Q = 20

    mpiexec -f ~/mpich-3.3/machinefile -n 20 ./xhpl_script

check nodes by ssh-ing into node and running htop, you should see all 4 core at close to 100% 
check temperature with, for example node 3 got up to 84decC. Note I have no attution heatsinks installed (something to explore).

    /opt/vc/bin/vcgencmd measure_temp
    
check cpu frequencies with

    sudo cat /sys/devices/system/cpu/cpu[0-3]/cpufreq/cpuinfo_cur_f

giving the following result
```
================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       12992   224     5     4            1202.16             1.2163e+00
```

I checked the power consumption and the during the test the power of the cluster peaks to about 25W from an idle power of 13W.
