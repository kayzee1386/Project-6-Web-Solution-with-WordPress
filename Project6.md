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

