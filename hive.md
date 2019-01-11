# Install and Test Hive

Continueing to follow https://tech.marksblogg.com/billion-nyc-taxi-rides-spark-raspberry-pi.html

 On the master node, Hive will use MySQL / MariaDB to store metadata about databases and tables.

    sudo apt-get install mysql-server
    
Create a user for Hive in MySQL / MariaDB

        sudo su
        mysql -uroot
        
  This will bring up the MariaDB command line interface, enetre the following:
```
CREATE USER 'hive'@'localhost' IDENTIFIED BY 'hive';
GRANT ALL PRIVILEGES ON *.* TO 'hive'@'localhost';
FLUSH PRIVILEGES;
exit;
```
Exit out of root account.

Next installing Hive itself:
```
wget -c -O hive.tar.gz http://mirror.ventraip.net.au/apache/hive/hive-3.1.1/apache-hive-3.1.1-bin.tar.gz
sudo mkdir -p /opt/hive 
sudo tar xvf hive.tar.gz --directory=/opt/hive --exclude=apache-hive-3.1.1-bin/ql/src/test --strip 1
```
Configuring:

    sudo vi /opt/hive/conf/hive-site.xml
    
```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://localhost/metastore?createDatabaseIfNotExist=true</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>hive</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>hive</value>
    </property>
    <property>
        <name>datanucleus.autoCreateSchema</name>
        <value>true</value>
    </property>
    <property>
        <name>datanucleus.fixedDatastore</name>
        <value>true</value>
    </property>
    <property>
        <name>datanucleus.autoCreateTables</name>
        <value>True</value>
    </property>
<property>
    <name>hive.metastore.schema.verification</name>
    <value>false</value>
</property>
<property>
  <name>hive.metastore.uris</name>
  <value>thrift://192.168.50.1:9083</value>
  <description>IP address (or fully-qualified domain name) and port of the metastore host</description>
</property>
</configuration>

```


Installing the mysql-java connector:

     sudo wget -c http://central.maven.org/maven2/mysql/mysql-connector-java/8.0.13/mysql-connector-java-8.0.13.jar  -P /opt/hive/lib/
 
Run to initialize schema in the DB 
    schematool -dbType mysql -initSchema

To launch hive metastore, run

    hive --service metastore &
    
     2019-01-11 09:50:13: Starting Hive Metastore Server
     ...
       
To check it is running use:

    ps -ef |grep -i metastore
