# Install and Test Hadoop File System

Following https://tech.marksblogg.com/billion-nyc-taxi-rides-spark-raspberry-pi.html


For all nodes:
```
ssh clustpi05 "sudo mkdir /opt/hadoop/ && sudo mkdir /opt/hadoop_tmp/"
ssh clustpi05 "sudo mkdir -p /opt/hadoop_tmp/hdfs"
ssh clustpi05 "sudo apt-get install oracle-java8-jdk -y"
```

On all slave (data) nodes:

    ssh clustpi05 "sudo mkdir -p /opt/hadoop_tmp/hdfs/datanode"

On master (name) node

    sudo mkdir -p /opt/hadoop_tmp/hdfs/namenode
    sudo apt-get install mysql-server

For all nodes:
```
ssh clustpi05 "sudo chown -R pi:pi /opt/hadoop"
ssh clustpi05 "sudo chown -R pi:pi /opt/hadoop_tmp"
```

Download the binaries (later I will show how to compile the code, so skip to that section if your prefer)
```
cd ~
wget -c -O hadoop.tar.gz https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.9.2/hadoop-2.9.2.tar.gz
sudo tar xvf hadoop.tar.gz --directory=/opt/hadoop --exclude=hadoop-2.9.2/share/doc --strip 1
    
sudo vi /opt/hadoop/etc/hadoop/slaves
clustpi02
clustpi03
clustpi04
clustpi05
```

    sudo vi /opt/hadoop/etc/hadoop/core-site.xml
 
```
<configuration>
 <property>
        <name>fs.default.name</name>
        <value>hdfs://clustpi01:9000/</value>
    </property>
    <property>
        <name>fs.default.FS</name>
        <value>hdfs://clustpi01:9000/</value>
    </property>
</configuration>
```

    sudo vi /opt/hadoop/etc/hadoop/hdfs-site.xml

```
<configuration>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/opt/hadoop_tmp/hdfs/datanode</value>
        <final>true</final>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/opt/hadoop_tmp/hdfs/namenode</value>
        <final>true</final>
    </property>
    <property>
        <name>dfs.namenode.http-address</name>
        <value>clustpi01:50070</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.datanode.du.reserved</name>
        <value>3221225472</value>
    </property>
</configuration>
```

Copy  /opt/hadoop/ to datanodes
```
$ for SERVER in clustpi02 clustpi03 clustpi04 clustpi05
  do
      sudo rsync --archive \
                 --one-file-system \
                 --partial \
                 --progress \
                 --compress \
                 /opt/hadoop/ $SERVER:/opt/hadoop/
  done
```


Format the filesystem

    hdfs namenode -format


You should now be able to successfully start the distributed file system with:

    start-dfs.sh

To stop

    stop-dfs.sh
    

### Troubleshooting
Initially I git the error 

    error: Incompatible clusterIDs 
    
```
cd /opt/hadoop/logs/
less hadoop-pi-datanode-clustpi02.log
```
    
To resolve I followed  https://stackoverflow.com/questions/22316187/datanode-not-starts-correctly

run on all nodes

    ssh clustpi05 rm -rf /opt/hadoop_tmp/hdfs/datanode/*

then re-format

    /opt/hadoop/bin/hdfs namenode -format

