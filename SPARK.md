# Install and Test SPARK

Execute
```
sudo mkdir /opt/spark
wget http://www.apache.org/dist/spark/spark-2.4.0/spark-2.4.0-bin-hadoop2.7.tgz
sudo tar xzvf spark-2.4.0-bin-hadoop2.7.tgz --directory=/opt/spark --strip 1

sudo ln -s /opt/hive/conf/hive-site.xml  /opt/spark/conf/hive-site.xml
sudo ln -s /opt/hive/lib/mysql-connector-java-5.1.28.jar /opt/spark/jars/mysql-connector-java-5.1.28.jar
```

Then copy config to the slave nodes:
```
for SERVER in clustpi02 clustpi03 clustpi04 clustpi05
  do
      sudo rsync --archive \
                 --one-file-system \
                 --partial \
                 --progress \
                 --compress \
                 --exclude /opt/spark/logs \
                 /opt/spark/ $SERVER:/opt/spark/
  done
```
Following https://tech.marksblogg.com/billion-nyc-taxi-rides-spark-raspberry-pi.html
and packaging spark libraries

```
jar cv0f ~/spark-libs.jar -C /opt/spark/jars/ .
hdfs dfs -mkdir /spark-libs
hdfs dfs -put ~/spark-libs.jar /spark-libs/
```
Checking with hdfs dfs -ls

    drwxr-xr-x   - pi supergroup          0 2018-12-27 11:27 /spark-libs
