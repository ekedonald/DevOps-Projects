# Implementing a WordPress Website with LVM Storage Management

## Understanding Three-Tier Architecture

A **Three-Tier Architecture** is a client-server architecture model that separates an application into three interconnected but distinct layers, each responsible for specific aspects of the application's functionality.

![3-Tier Architecture](./images/3%20Tier%20Architecture.png)

Generally, web or mobile solutions are implemented based on a **Three-Tier Architecture** to improve scalability and flexibility. The three distinct layers are:

1. **Presentation Layer (PL)**: This is the user interface such as the client server or browser on your laptop.

2. **Business Layer (BL)**: This is the backend program that implements business logic (_i.e. Application or Web Server_).

3. **Data Access or Management Layer (DAL)**: This is the layer for computer data storage and data access (_i.e. Database Server or File System Server such as FTP Server or NFS Server_).

## What is Logical Volume Manager (LVM)?
LVM stands for Logical Volume Manager, a technology used in Linux and other Unix-like operating systems to manage storage devices and create flexible, resizable storage configurations. LVM provides a layer of abstraction between the physical storage devices (_such as hard drives, SSDs, or partitions_) and the file systems or logical volumes used by the operating system.

Key components of LVM include:

1. **Physical Volumes (PVs)**: These are the physical storage devices or partitions (_i.e. hard drives or SSDs_) that are added to the LVM system.

2. **Volume Groups (VGs)**: Volume Groups are created by combining one or more Physical Volumes. VGs serve as a pool of storage that can be allocated to various Logical Volumes.

3. **Logical Volumes (LVs)**: Logical Volumes are similar to traditional partitions and are created within a Volume Group. They are what you format with a file system and use to store data. Logical Volumes can be resized and moved dynamically, which is a significant advantage of LVM.

## How To Implement a WordPress Website with LVM Storage Management

The following steps are taken to implement a WordPress Website with LVM Storage Management:

### Step 1: Provision a Web Server EC2 Instance

Use the following parameters when configuring the EC2 Instance:

1. Name of Instance: Web Server
2. AMI: Red Hat Enterprise Linux 9 (HVM), SSD Volume Type
3. New Key Pair Name: web11
4. Key Pair Type: RSA
5. Private Key File Format: .pem
6. New Security Group: WordPress
7. Inbound Rules: Allow Traffic From Anywhere On Port 80 and Port 22.

![web server instance summary](./images/1.%20instance%20summary%20web%20server.png)
_Instance Summary for Web Server_

* On the Instances tab, you will see the Availability Zone (_i.e. us-east-1d_). This will be used when creating Elastic Block Volumes for the Web Server Instance.

![web server availability zone](./images/1.%20availability%20zone%20web%20server.png)

### Step 2: Create and Attach 3 Elastic Block Store Volumes to the Web Server EC2 Instance

* On the EC2 dashboard, click on **Volumes** on the Elastic Block Store tab.

![ebs volumes](./images/2.%20volumes.png)

* Click on the **Create volume** button.

![create volume](./images/create%20volume.png)

* Give the EBS Volume the following parameters and click on the **create volume** button:

1. Size (GiB): 10
2. Availability Zone: us-east-1d (_Note that the Availability Zone you select must match the Availability zone of the Web Server Instance_)

![ebs parameters](./images/2.%20ebs%20volume%20parameters.png)

* Repeat the steps above to create two more EBS Volumes.

![available ebs volumes](./images/2.%20ebs%20volume%20available.png)
_You will see the 3 EBS Volumes you created have an Available Volume state_

* Click on one of the Volumes then click on the **Actions** button, you will see a drop-down and click on the **Attach volume** option.

![attach volume](./images/2.%20actions%20and%20attach%20volume.png)

* Select the Web Server Instance and click on the Attach volume button.

![select web server instance](./images/2.%20select%20web%20server%20instance%20attach%20volume.png)

* Repeat these steps for the other 2 volumes and you will see that the volumes have been attached to the Web Server Instance as shown below:

![volumes attached to web server](./images/2.%20volumes%20attached.png)

### Step 3: Implement LVM Storage Management on the Web Server

* Open terminal on your computer.

* Go to the Downloads directory (_i.e. `.pem` key pair is stored here_) using the command shown below:

```sh
cd Downloads
```

* Run the following command to give read permissions to the `.pem` key pair file.

```sh
chmod 400 <private-key-pair-name>.pem
```

![chmod web11](./images/3.%20chmod%20web11.png)

* SSH into the Web Server Instance using the command shown below:

```sh
ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>
```

![ssh web server](./images/3.%20ssh%20web%20server.png)

* Use the `lsblk` command to inspect the block devices attached to the server.

![lsblk](./images/3.%20lsblk1.png)
_Notice the names of the newly created devices._

* Use the `df -h` command to see all mounts and free space on your server.

![df -h](./images/3%20df%20-h1.png)

* Use `gdisk` utility to create a single partition on **/dev/xvdf** disk.

![gdisk dev/xvdf](./images/3.%20gdisk%20:dev:xvdf.png)
_Note that all devices in Linux reside in the **/dev** directory._

```sh
sudo gdisk /dev/xvdf
```

* Type `n` to create a new partition and fill in the data shown below into the parameters:

 1. Partition number (1-128, default 1): 1
 2. First sector (34-20971486, default = 2048) or {+-}size{KMGTP}: 2048
 3. Last sector (2048-20971486, default = 20971486) or {+-}size{KMGTP}: 20971486
 4. Current type is 8300 (Linux filesystem)
 Hex code or GUID (l to show codes, Enter = 8300): 8300

 ![n partition](./images/3.%20n%20partiton.png)

* Type `p`to print the partition table of the /dev/xvdf device.

![p partition](./images/3.%20p%20partition.png)

* Type `w` to write the table to disk and type `y` to exit.

![w y partiton](./images/3.%20w%20y%20partition.png)

* Repeat the `gdisk` utility partitioning steps for **/dev/xvdg** and **/dev/xvdh** disks.

* Use the `lsblk` command to view the newly configured partition on each of the 3 disks.

![lsblk](./images/3.%20lsblk2.png)

* Install `lvm2` package using the command shown below:

```sh
sudo yum install lvm2 -y
```

![install lvm2](./images/3.%20install%20lvm2.png)

* Run the following command to check for available partitons:

```sh
sudo lvmdiskscan
```

![lvmdiskscan](./images/3.%20lvmdiskscan.png)

* Use `pvcreate` utility to mark each of the 3 disks as physical volumes (PVs) to be used by LVM.

```sh
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```

![pvcreate](./images/3.%20pvcreate%20:dev:xvdf.png)

* Verify that your physical volumes (PVs) have been created successfully by running `sudo pvs`

![pvs](./images/3.%20pvs.png)

* Use `vgcreate` utility to add 3 physical volumes (PVs) to a volume group (VG). Name the volume group **webdata-vg**.

```sh
sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
```

![vgcreate](./images/3.%20vgcreate%20webdata-vg.png)

* Verify that your volume group (VG) has been created successfully by running `sudo vgs`

![vgs](./images/3.%20vgs.png)

* Use `lvcreate` utility to create 2 logical volumes: **apps-lv (use half of the PV size)** and **logs-lv (use the remaining space of the PV size)**. _Note that **apps-lv** will be used to store data for the website while **logs-lv** will be used to store data for logs._

```sh
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```

![lvcreate](./images/3.%20lvcreate.png)

* Verify that your logical volume (LV) has been created successfully by running `sudo lvs`

![lvs](./images/3.%20lvs.png)

* Verify the entire setup running the following commands:

```sh
sudo vgdisplay -v #view complete setup - VG, PV, and LV
```

![vgdisplay](./images/3.%20vgdisplay.png)

```sh
sudo lsblk
```

![lsblk](./images/3.%20lsblk3.png)

* Use `mkfs.ext4` to format the logical volumes (LV) with **ext4** file system.

```sh
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
```
![mkfs ext4 apps-lv](./images/3.%20sudo%20mkfs%20-t%20apps-lv.png)

```sh
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

![mkfs ext4 logs-lv](./images/3.%20sudo%20mkfs%20-t%20logs-lv.png)

* Create **/var/www/html** directory to store website files.

```sh
sudo mkdir -p /var/www/html
```

![mkdir /var/www/html](./images/3.%20mkdir%20-p%20:var:www:html.png)

* Create **/home/recovery/logs** to store backup of log data.

```sh
sudo mkdir -p /home/recovery/logs
```

![mkdir /home/recovery/logs](./images/3.%20mkdir%20-p%20:home:recovery:logs.png)

* Mount **/var/www/html** on **apps-lv** logical volume.

```sh
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```

![mount apps-lv /var/www/html](./images/3.%20mount%20:dev:webdata-vg:logs-lv%20:var:log.png)

* Use `rsync` utility to backup all the files in the log directory **/var/log** into **/home/recovery/logs** (_This is required before mounting the file system_).

```sh
sudo rsync -av /var/log/. /home/recovery/logs/
```

![rsync var/log home/recovery](./images/3.%20rsync%20-av%20var:log%20home:recovery:log.png)

* Mount **/var/log** on **logs-lv** logical volume. (_Note that all the existing data on **/var/log** will be deleted_).

```sh
sudo mount /dev/webdata-vg/logs-lv /var/log
```

![mount logs-lv var/log](./images/3.%20mount%20:dev:webdata-vg:logs-lv%20:var:log.png)

* Restore log files back into **/var/log** directory.

```sh
sudo rsync -av /home/recovery/logs/ /var/log
```

![rsync home/recovery /var/log](./images/3.%20rsync%20-av%20:home:recovery:logs%20:var:log.png)

* Update `/etc/fstab` file so that the mount configuration will persist after restarting the server. The UUID of the device will be used to update the `/etc/fstab` file. Run the command shown below to get the UUID of the **apps-lv** and **logs-lv** logical volumes:

```sh
sudo blkid
```
![blkid](./images/3.%20sudo%20blkid.png)

* Update `/etc/fstab` in this format using your own UUID and remember to remove the leading and ending quotes.

```sh
sudo vi /etc/fstab
```

![/etc/fstab](./images/3.%20vi%20:etc:fstab.png)

* Test the configuration using the command shown below:

```sh
sudo mount -a
```

![mount -a](./images/3.%20mount%20-a.png)

* Reload the daemon using the command shown below:

```sh
sudo systemctl daemon-reload
```

![systemctl daemon-reload](./images/3.%20systemctl%20daemon-reload.png)

* Verify your setup by running `df -h`

![df -h](./images/3.%20df-h2.png)

### Step 4: Install WordPress on the Web Server

* Update the list of packages in the package manager.

```sh
sudo yum -y update
```

![yum update](./images/4.%20yum%20update.png)

* Install wget, apache and its dependencies.

```sh
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```

![install wget httpd php](./images/4.%20install%20wget%20httpd%20php.png)

* Enable and start apache

```sh
sudo systemctl enable httpd
```

![enable httpd](./images/4.%20systemctl%20enable%20httpd.png)

```sh
sudo systemctl start httpd
```

![start apache](./images/4.%20systemctl%20start%20httpd.png)

* Install PHP and its dependencies.

```sh
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1
```

![install php dependencies](./images/4.%20install%20php%20dependencies.png)

* Restart apache.

```sh
sudo systemctl restart httpd
```

![restart apache](./images/4.%20restart%20apache.png)

* Download WordPress and copy WordPress to `/var/www/html`

```sh
mkdir wordpress
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
sudo cp -R wordpress /var/www/html/
```

![download wordpress](./images/3.%20download%20wordpress.png)

* Configure SELinux policies.

```sh
 sudo chown -R apache:apache /var/www/html/wordpress
 sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
 sudo setsebool -P httpd_can_network_connect=1
```

![selinux policies](./images/4.%20selinux%20policies.png)

### Step 5: Provision a Database Server EC2 Instance
Use the following parameters when configuring the EC2 Instance:

1. Name of Instance: Database Server
2. AMI: Red Hat Enterprise Linux 9 (HVM), SSD Volume Type
3. Key Pair Name: web11
4. New Security Group: WordPress
5. Inbound Rules: Allow Traffic From Anywhere On Port 22 and Traffic from the Private IPv4 address of the Web Server on Port 3306 (_i.e. MySQL_).

![instance summary for database server](./images/5.%20instance%20summary%20database%20server.png)
_Instance Summary for Database Server_

### Step 6: Create and Attach 3 Elastic Block Store Volumes to the Database Server EC2 Instance

* Repeat Step 3 but attach the Volumes to the Database Server and ensure the volumes are attached to the **Availability Zone (_i.e. us-east-1c_)** of the Database Server.

![ebs volumes attached to database server](./images/6.%20attached%20ebs%20volumes%20to%20database%20server.png)
_The EBS Volumes have been attached to the Database Server_

### Step 7: Install MySQL on the Database Server

* Open another terminal on your computer.

* Go to the Downloads directory (_i.e. `.pem` key pair is stored here_) using the command shown below:

```sh
cd Downloads
```

* SSH into the Database Server Instance using the command shown below:

```sh
ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>
```
![ssh database](./images/7.%20ssh%20database%20server.png)

* Update the list of packages in the package manager.

```sh
sudo yum update -y
```

![yum update](./images/7.%20yum%20update.png)

* Install MySQL server.

```sh
sudo yum install mysql-server -y
```

![install mysql-server](./images/7.%20install%20mysql-server.png)

* Verify that the service is up and running.

```sh
sudo systemctl status mysqld
```

![status mysqld](./images/7.%20status%20mysqld.png)

* Enable the MySQL service.

```sh
sudo systemctl enable mysqld
```

![enable mysqld](./images/7.%20enable%20mysqld.png)

* Restart the MySQL service.

```sh
sudo systemctl restart mysqld
```

![restart mysqld](./images/7.%20restart%20mysqld.png)

### Step 8: Configure the Database Server to work with WordPress

* Log into the MySQL console application.

```sh
sudo mysql
```

![sudo mysql](./images/8.%20sudo%20mysql.png)

* Create a database called **wordpress**.

```sh
CREATE DATABASE wordpress;
```

![create database](./images/8.%20create%20database%20wordpress.png)

* Create a new user.

```sh
CREATE USER 'myuser'@'<web_server_private_ip_address>' IDENTIFIED BY 'mypass';
```

![create user](./images/8.%20create%20user.png)

* Grant all privileges on the wordpress database to the user you created.

```sh
GRANT ALL ON wordpress.* TO 'myuser'@'<web_server_private_ip_address>';
```

![grant all privileges](./images/8.%20grant%20all%20on%20wordpress.png)

* Run the following command to apply and make changes effective.

```sh
FLUSH PRIVILEGES;
```

![flush privileges](./images/8.%20flush%20privileges.png)

* Display all the databases.

```sh
SHOW DATABASES;
```

![show databases](./images/8.%20show%20databases.png)

* Exit the MySQL console.

![exit mysql](./images/8.%20mysql%20exit.png)

### Step 9: Configure WordPress to connect to the remote Database Server

* Connect to the Web Server Instance.

* Install MySQL client.

```sh
sudo yum install mysql -y
```

![install mysql client](./images/9.%20install%20mysql%20client.png)

* Test that you can connect from your Web Server to your Database Server by using `mysql-client`

```sh
sudo mysql -u admin -p -h <Database_Server_Private_IP_address>
```

![mysql -u admin -p -h database private ip](./images/9.%20mysql%20-u%20admin%20-p%20-h.png)

* Verify if you can successfully execute `SHOW DATABAES;` command to see a list of existing databases.

![show databases](./images/9.%20show%20databases.png)

* Run the following command to configure WordPress to establish connection with the Database Server.

```sh
sudo vi /var/www/html/wordpress/wp-config.php
```

![vi wp-config.php](./images/9.%20vi%20:var:www:html:wordpress:wp-config.png)
_The highlighted parameters are the ones that need to be configured_

* Input the credentials of the user you created when configuring the Database Server then save and exit the file.

![update wp-config.php](./images/9.%20update%20wp-config.png)

* Try to access the URL shown below from your browser:

```sh
http://<Web_Server_Public_IP_Address>/wordpress/
```

![url1](./images/9.%20wordpress%20url1.png)

![url2](./images/9.%20wordpress%20url2.png)

![url3](./images/9.%20wordpress%20url3.png)

![url4](./images/9.%20wordpress%20url4.png)

![url5](./images/9.%20wordpress%20url5.png)