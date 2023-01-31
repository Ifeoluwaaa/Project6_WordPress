#### STEP 1 Prepare a Web Server
> Launch an Ec2 instance with linux Redhat distribution
> Edit your inbound rules and add SSH to be able to access with SSH

![inbound_rules](./images/Inbound_Rules.PNG)

> Create three volumes of 10gib and attach to the instance 

![volumes](./images/volumes.PNG)

> Connect to your instance on your terminal

![connect](./images/Connect.PNG)

> Run lsblk on the terminal to show what block devices are attached to server

> Check for free space on server and see all mounts

    df -h

![mounts](./images/df.PNG)

> Create a single partition in each of the disks

    sudo fdisk /dev/xvdf
    sudo fdisk /dev/xvdg
    sudo fdisk /dev/xvdh

![partition](./images/Partiton1.PNG)
![partition](./images/Partition2.PNG)
![partition](./images/partition3.PNG)

> View the newly configured partition

    lsblk

![disk_drives](./images/volumespart.PNG)

> Install lvm2 package

    sudo yum install lvm2
    sudo lvmdiskscan

![packsges](./images/yum.PNG)
![scan](./images/scan_for_disk.PNG)

> Create physical volumes of each disks

    sudo pvcreate /dev/xvdf1
    sudo pvcreate /dev/xvdg1
    sudo pvcreate /dev/xvdh1

![physical_volume](./images/physical_volume.PNG)

> Ensure creating of physical volumes was successful

    sudo pvs

![verify](./images/vgs.PNG)

> Use vgcreate utility to add the pvs's to a volune group

    sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1 
![vg_create](./images/volumes.PNG)

> Verify the VG has created successful

    sudo vgs
![sudo_vgs](./images/Sudo_vgs.jpeg)

> Create two logical volumes for storing data of the website and storing data for logs 

    sudo lvcreate -n apps-lv -L 14G webdata-vg
    sudo lvcreate -n logs-lv -L 14G webdata-vg

> Verify the logical volume has been created 

    sudo lvs

![sudo_lvs](./images/LVMlogicalvolumes.PNG)

> Verify the entire setup 

    sudo vgdisplay -v #view complete setup - VG, PV, and LV
    sudo lsblk 

![display_setup](./images/display%20_vPNG.PNG)

> Use mkfs.ext4 to format the logical volumes 

    sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
    sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

![logical_volume](./images/logswebdata.PNG)
![logical_volume](./images/mkfs_exfs.PNG)

> Create /var/www/html directory to store website files

    sudo mkdir -p /var/www/html

> Mount /var/www/html on apps-lv logical volume

    sudo mount /dev/webdata-vg/apps-lv /var/www/html/

> Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs 

    sudo rsync -av /var/log/. /home/recovery/logs/
![logs](./images/logs_backup.PNG)

> Mount /var/log on logs-lv logical volume

    sudo mount /dev/webdata-vg/logs-lv /var/log

> Restore log files back into /var/log directory

    sudo rsync -av /home/recovery/logs/. /var/log

> Update /etc/fstab file so that the mount configuration will persist after restart of the server

    sudo blkid

![blkid](./images/blkid_web.PNG)

> Copy, paste and format the UUID of the webdata and logs

    sudo vi /etc/fstab

![data](./images/Paste.PNG)

> Test the configuration and reload the daemon

     sudo mount -a
     sudo systemctl daemon-reload

> Verify the setup 

    df -h

![df_h](./images/Done_dusted.PNG)


#### STEP 2 - Preapare A Database Server
> Launch a second redhat ec2 instance 

![database_instance](./images/Database_instance.PNG)

> Create and attach EBS volume to the ec2 instance

![volumes](./images/volumedb.PNG)

> Connect the instance to your terminal to begin configuration

![connect](./images/new_instance_CONNECT.PNG)

> Inspect what block devices are attached to the server

    lsblk

![lsblk_db](./images/lsblk_db.PNG)

> Check to see all free space on server

        df -h

![mounts](./images/df_db.PNG)

> Craete a partition on each of the 3 disks

    sudo fdisk /dev/xvdf
    sudo fdisk /dev/xvdg
    sudo fdisk /dev/xvdh
![partition](./images/fdisk_partitionPNG.PNG)

> Use lsblk to view newly configured partition 
![partition_view](./images/lsblk_view_new.PNG)

> Install lvm2 package and check for available partitions

    sudo yum install lvm2
    sudo lvmdiskscan

![install_lvm2](./images/lvm2_install.PNG)
![lvmdiskscan](./images/lvmdiskscan.PNG)

> Create physical volumes to be used by LVM

    sudo pvcreate /dev/xvdf1
    sudo pvcreate /dev/xvdg1
    sudo pvcreate /dev/xvdh1

![physical_Vvolume](./images/physical_volume.PNG)

> Verify the physical volume has been successfully

    sudo pvs
![verify_PH](./images/sudo_pvs.PNG)

> Use vgcreate utility to add all 3 PVs to a volume group

    sudo vgcreate dbdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1

![group_create](./images/VG_create_db.PNG)

> Confirm the VG was successfully created

    sudo vgs

![vg](./images/sudo_vgs_verify.PNG)

> Use lvcreate  -n db-lv -L 14G dbdata -vg

    sudo lvcreate -n db-lv -L 14G dbdata-vg
    sudo lvcreate -n logs-lv -L 14G dbdata-vg

![LG](./images/logical_volumes_create.PNG)

> Verify the logical volume has been created successfully

    sudo lvs

![sudo_lvs](./images/sudo_lvs_check.PNG)

> View the entire setup


    sudo vgdisplay -v #view complete setup - VG, PV, and LV
    sudo lsblk 

![entire_setup](./images/display.PNG)
![entire_setup2](./images/display2.PNG)
![lsblk](./images/lsblk_last.PNG)

> Use mkfs.ext4 to format the Logical Volumes

    sudo mkfs -t ext4 /dev/dbdata-vg/db-lv
    sudo mkfs -t ext4 /dev/dbdata-vg/logs-lv
![file_system](./images/file_system.PNG)
![file_system](./images/file_system2.PNG)

> Create /db directory to store website files

    sudo mkdir -p /db

> Create /home/recovery/logs to store backup of log data

    sudo mkdir -p /home/recovery/logs

> Mount /db on bd-lv logical volume

    sudo mount /dev/webdata-vg/apps-lv /var/www/html/

> Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs

    sudo rsync -av /var/log/. /home/recovery/logs/

![rsync](./images/rsync_logs.PNG)

> Mount /var/log on logs-lv logical volume.

    sudo mount /dev/dbdata-vg/logs-lv /var/log

> Restore log files back into /var/log directory

    sudo rsync -av /home/recovery/logs/. /var/log
![recovery_logs](./images/recoverylogs.PNG)

> Update /etc/fstab file so that the mount configuration will persist after restart of the server

 > The UUID of the device will be used to update the /etc/fstab file

    sudo blkid
![blkid](./images/blkid_db.PNG)

> Update /etc/fstab using the UUID from the device copied from running blkid

    sudo vi /etc/fstab
![etc_fstab](./images/UUID.PNG)

> Test the configuration and reload the daemon

    sudo mount -a
    sudo systemctl daemon-reload

> Verify your setup 

    df -h
![setup](./images/setup_db_df.PNG)

#### STEP 3 INSTALL WordPress on your Web Server EC2
> Reconnect into the webserver instance

> Update the repository

    sudo yum -y update

![repo_update](./images/repo_update.PNG)

> Install wget, Apache and its dependencies

    sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json

![wget_Apache](./images/wget_ApachePNG.PNG)

> Start Apache

    sudo systemctl enable httpd
    sudo systemctl start httpd

![Apache](./images/Strt_Apache.PNG)

> Install PHP and its dependencies

    sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
    sudo yum module list php
    sudo yum module reset php
    sudo yum module enable php:remi-7.4
    sudo yum install php php-opcache php-gd php-curl php-mysqlnd
    sudo systemctl start php-fpm
    sudo systemctl enable php-fpm
    setsebool -P httpd_execmem 1

 ## Note: The commands need to be up to date with your OS version
![PHP](./images/PHPdepen.PNG)
![PHP_APAC](./images/php.PNG)
![list](./images/list.PNG)
![reset](./images/reset.PNG)
![enable_PHP](./images/enable_PHP.PNG)
![combine](./images/combine.PNG)

> Restart Apache 

    sudo systemctl restart httpd

> Download wordpress and copy wordpress to car/www/html

    mkdir wordpress
    cd   wordpress
    sudo wget http://wordpress.org/latest.tar.gz
    sudo tar xzvf latest.tar.gz
    sudo rm -rf latest.tar.gz
    php wordpress/wp-config.php
    cp -R wordpress /var/www/html/

![download](./images/download.PNG)

> Configure SELinux Policies

    sudo chown -R apache:apache /var/www/html/wordpress
    sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
    sudo setsebool -P httpd_can_network_connect=1

#### STEP 4 - INSTALL MySQL on your DB Server EC2
##### Note : Move to DB server

> Run basic update commands and install mysql-server

    sudo yum update
    sudo yum install mysql-server

![mysql_install](./images/mysql_install.PNG)

> Verify the service is up and runnning, if it is not restart and enable it

    sudo systemctl status mysqld
    sudo systemctl restart mysqld
    sudo systemctl enable mysqld

![system_status](./images/system_status.PNG)

#### STEP 5 CONFIGURE DB TO WORK WITH WORDPRRESS

```SQL
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
![mysql](./images/new.PNG)

#### STEP 6 CONFIGURE WORDPRESS TO CONNECT TO REMOTE DATABASE

> Open port 3306 on DB server EC2 and add the private IP address of the webserver

> cd into my.cnf and add bind address

    vi /etc/my.cnf

> Install MySQL client and test that for connection

    sudo yum install mysql
    sudo mysql -u admin -p -h <DB-Server-Private-IP-address

![pass](./images/pass.PNG)

> Verify the connection was successfully

![database](./images/database.PNG)

> Enable TCP port 80 inbound rules configuration for web server EC2

![port](./images/port80.PNG)

> Edit the wp-config.php

    sudo vi wp-config.php

![config](./images/config.PNG)
> Try to access from browser the link to Wordpress

    http://100.25.17.142/wordpress/

![welcome.1](./images/installwordpressPNG.PNG)
![welcome](./images/welcomepagePNG.PNG)

