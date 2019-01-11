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
  
