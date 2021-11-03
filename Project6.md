#    Project 6 - Web Solution with Wordpress

This project is about deploying a storage infrastructure on two Linux servers and implement a basic web solution using WordPress.

It's going to be in two parts, which are;

*Configuration of storage subsystem for Web and Database servers based on Linux OS in order to give practical experience of working with disks, partitions and volumes in Linux*

*Also, installations of WordPress and connecting it to a remote MySQL database server. This part of the project will solidify skills of deploying Web and DB tiers of Web solution.*

### This Project also speaks on three-tier client-server software architecture which are;

*Presentation Layer (PL)*: This is the user interface such as the client server or browser on your laptop.

*Business Layer (BL)*: This is the backend program that implements business logic. Application or Webserver 

*Data Access or Management Layer (DAL)*: This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server

### Preparing the 'WEB SERVER'

_STEP 1:_

An instance was launched and three volume was created as well and attached to the instance created named 'Web Server'

![web-DB-servers](https://user-images.githubusercontent.com/46185705/139864297-4c7ef9c3-3161-4de6-b6ba-fad0a617883e.jpg)
![Volumes-3](https://user-images.githubusercontent.com/46185705/139864311-b61cdac4-82ae-4b4f-ae39-d2d3ec89f73c.jpg)

 `lsblk` was used to view the block size that was attached to the server.
 ![disk-available](https://user-images.githubusercontent.com/46185705/139865047-045532f8-e9fe-4349-8302-951e8072f1cf.jpg)
 
 `df -h` was used to see all mounts and free space on your server
 ![mount-free-space](https://user-images.githubusercontent.com/46185705/139865440-a8f3d4e8-b0d9-46c1-987d-b761f73b216b.jpg)
 
 `gdisk` utility was used to create a single partition on each of the 3 disks
 
 xvdf
 
![xvdf](https://user-images.githubusercontent.com/46185705/139865908-9e3c3596-ff9f-416f-ab8c-dccbd16ce27a.jpg)

xvdg

![xvdg](https://user-images.githubusercontent.com/46185705/139865923-d8900f44-8012-4db0-b0fa-6698d219208a.jpg)

xvdh

![xvdh](https://user-images.githubusercontent.com/46185705/139865933-c0be2adf-2ece-4656-8377-f3dca97a5067.jpg)

The newly partitioned on the 3 disk was viewed using `lsblk`

![Partition-created](https://user-images.githubusercontent.com/46185705/139867162-26265379-8013-4679-a5d4-388918c9721e.jpg)

lvm2 package was installed `sudo yum install lvm2` and verified using `sudo lvmdiskscan`

![lvm2-installed](https://user-images.githubusercontent.com/46185705/139867746-2c9e0e93-6ff0-4119-b31c-71225a7e3c49.jpg)

`sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1` was used to create the three physical volume to be used by the lvm2

![pvcreate](https://user-images.githubusercontent.com/46185705/139868501-39fe375f-a7de-4e95-9785-b88834d79b0c.jpg)

To add all the 3 PVs to a volume group, this command was used `sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

![status-check](https://user-images.githubusercontent.com/46185705/139869892-b2700cce-dbf8-4245-b2fa-57c1b147a3af.jpg)

Also this , ``sudo lvcreate -n apps-lv -L 14G webdata-vg
        sudo lvcreate -n logs-lv -L 14G webdata-vg``
        
![status-check1](https://user-images.githubusercontent.com/46185705/139869924-ad395919-c740-493b-ba80-2f4c0e0ce246.jpg)
![verifying](https://user-images.githubusercontent.com/46185705/139874706-e829a2e3-0900-4749-9a92-f8ae76ebbf5c.jpg)

The mkfs.ext4 was used to format the logical volume to ext4 file systems

```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
__/var/www/html__ was created to store website files likewise this __/home/recovery/logs__ was to store backup of log data

`sudo rsync -av /var/log/. /home/recovery/logs/` was used to backup all the files in the log directory __/var/log__ into __/home/recovery/logs__ (This is required before mounting the file system)

![2](https://user-images.githubusercontent.com/46185705/139878279-45797f88-b6ca-447b-a886-9addc17bc1d7.jpg) 

UUID of the device was used to update the /etc/fstab file using `sudo blkid` and subsequently updated with `sudo vi /etc/fstab`

`sudo mount -a and sudo systemctl daemon-reload` was used to test the configuration and daemon was reloaded

![verifying](https://user-images.githubusercontent.com/46185705/139884491-23d168b9-e231-43dc-8d71-e2fa4d2afbb8.jpg)


_STEP 2:_ Preparation of the Database Server 

The second RedHat Instance was launched which is going to serve as the DB-Server. The same steps for the Web Server was followed but instead of apps-lv i created db-lv and mount it to /db directory instead of /var/www/html/.

![web-DB-servers](https://user-images.githubusercontent.com/46185705/139864297-4c7ef9c3-3161-4de6-b6ba-fad0a617883e.jpg)


_STEP 3:_ WordPress installations on the Webserver

The repository was updated using `sudo yum -y update`. Followed by installation of the wget, Apache and it’s dependencies `sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`. 

Apache started  ```
                 sudo systemctl enable httpd
                 sudo systemctl start httpd
                ```

PHP and it's other dependencies were installed using the following commands;

```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```

![apache-enabled](https://user-images.githubusercontent.com/46185705/139891208-ae64fede-9f4a-41f6-8723-977ddc77274e.jpg)

![wget-others](https://user-images.githubusercontent.com/46185705/139891257-4683193d-73de-4783-8b80-08267a9ca279.jpg)

![release8](https://user-images.githubusercontent.com/46185705/139891323-93cce709-a076-4358-a040-3b602a1cad45.jpg)


Wordpress was downloaded and copied to var/www/html using the following commands;
             
```
  mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/
```

SELinux policies were configured with the following commands;
```
  sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1
```


__Step 4:__ — MySQL installed on DB Server-EC2


```
sudo yum update
sudo yum install mysql-server
```
Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:
```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```

![mysql-serevr-started](https://user-images.githubusercontent.com/46185705/140047951-842d0e44-d088-4385-aefa-f3479477bd4f.jpg)


__Step 5:__ — Database configured to work with WordPress

```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
![telnet-databse-server](https://user-images.githubusercontent.com/46185705/140048015-5f9c67f9-c0ba-40db-b542-83c403449e90.jpg)
![mysql-create](https://user-images.githubusercontent.com/46185705/140048020-82c88329-0c79-4d12-9f34-3725ec14a42e.jpg)
![mysql-select](https://user-images.githubusercontent.com/46185705/140048022-a8b6798e-8236-4580-8022-7ea353be7dd9.jpg)


__Step 6:__ — WordPress configured to connect to the remote database.

__MySQL port 3306 was opened on DB Server EC2__

Mysql client was installed and tested from the web server to the DB server. 

```
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```

![error](https://user-images.githubusercontent.com/46185705/140050565-6f8e817b-44fa-46f8-80dd-3e7fd0b1b485.jpg)
![Wordpress-page](https://user-images.githubusercontent.com/46185705/140050569-accd1946-6a98-48ef-acbd-5889f9b30589.jpg)
![wp-load-page](https://user-images.githubusercontent.com/46185705/140050571-79a5a4bd-0c38-45d3-9830-29b7e38c1f5e.jpg)
![wallaaaaah!!!](https://user-images.githubusercontent.com/46185705/140050555-c9e2ca7a-d660-43ad-98c4-c733f424e1b1.jpg)
