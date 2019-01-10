# Install and Test Hadoop File System

Following https://tech.marksblogg.com/billion-nyc-taxi-rides-spark-raspberry-pi.html


On all nodes:
```
ssh clustpi05 "sudo mkdir /opt/hadoop/ && sudo mkdir /opt/hadoop_tmp/"
ssh clustpi05 "sudo mkdir -p /opt/hadoop_tmp/hdfs"
ssh clustpi05 "sudo apt-get install oracle-java8-jdk -y"
```

On all slave (data) nodes:

    ssh clustpi05 "sudo mkdir -p /opt/hadoop_tmp/hdfs/datanode"

On master (name) node

    ssh clustpi01 "sudo mkdir -p /opt/hadoop_tmp/hdfs/namenode"

On all nodes:
```
ssh clustpi05 "sudo chown -R pi:pi /opt/hadoop"
ssh clustpi05 "sudo chown -R pi:pi /opt/hadoop_tmp"
```

