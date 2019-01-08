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
