# DevOps Tooling Website Solution
 In this project, I will implement a solution that consists of the following components:
 1. **Infrastructure**: AWS
 2. **Web Server Linux**: Red Hat Enterprise Linux 9
 3. **Database Server**: Red Hat Enterprise Linux 9 + MySQL
 4. **Storage Server**: Red Hat Enterprise Linux 9 + NFS Server
 5. **Programming Language**: PHP
 6. **Code Repository**: GitHub

In the diagram below, you can see a common pattern where several stateless Web Servers share a common database and also access the same files using [Network File System (NFS)](https://en.wikipedia.org/wiki/Network_File_System) as a shared file storage. Even though the NFS server might be located on a completely seperate hardware (_i.e. for Web Servers it will look like a local file system from where they can serve the same files_).

![3-tier web application architecture](./images/0.%203-Tier%20Web%20Application%20Architecture.png)
_3-Tier Web Application Architecture with a single Database and an NFS Server as a shared files storage_

It is important to know what storage solution is suitable for certain use cases. To determine this, you need to answer the following questions: **what data will be stored, in what format, how this data will be accessed, by whom, from where and how frequently**. 
 
Based on this you will be able to choose the right storage system for your solution.

## How To Implement a DevOps Tooling Website Solution
The following steps are taken to implement a DevOps Tooling Website Solution:

### Step 1: Provision an NFS Server EC2 Instance

Use the following parameters when configuring the EC2 Instance:

1. Name of Instance: Web Server
2. AMI: Red Hat Enterprise Linux 9 (HVM), SSD Volume Type
3. New Key Pair Name: web11
4. Key Pair Type: RSA
5. Private Key File Format: .pem
6. New Security Group: NFS Server SG
7. Inbound Rules: Allow Traffic From Anywhere On Port 22

![NFS Server Instance Summary](./images/1.%20nfs%20server%20instance%20summary.png)
_Instance Summary for NFS Server_

### Step 2: Set additional Inbound Rules to allow connections from the NFS Clients (i.e. The 3 Web Servers) on the NFS Port.

* On the Instance Summary for the NFS Server shown above, click on the Subnet ID.

* Copy the Subnet IPv4 CIDR address.

![Subnet IPv4 CIDR address](./images/2.%20subnet%20ipv4%20cidr%20address.png)

* Add rules that allow connections from the Subnet CIDR (_i.e. **172.31.16.0/20**_) on TCP Port 2049, TCP Port Port 111, UDP Port 2049 and UDP Port 111.

![inbound rules nfs server](./images/2.%20edit%20inbound%20rules.png)

### Step 3: Create and Attach 3 Elastic Block Store Volumes to the NFS Server EC2 Instance

* On the Instances tab, notice the **Availabilty Zone (i.e. us-east-1c)** of the NFS Server Instance. This will be used to configure the 3 EBS Volumes.

![availabilty zone](./images/3.%20availability%20zone.png)

* On the EC2 dashboard, click on the **Volumes** on the Elastic Block Store tab.

![ebs volumes tab](./images/3.%20volumes%20tab.png)

* Click on the Create Volume button.

![create volume](./images/3.%20create%20volume%20button.png)

* Give the EBS Volume the following parameters and click on the **create volume** button:

1. Size (GiB): 10
2. Availability Zone: us-east-1c (_Note that the Availability Zone you select must match the Availability zone of the NFS Server Instance_)

![ebs parameters](./images/3.%20ebs%20volume%20parameters.png)

* Repeat the steps above to create two more EBS Volumes.

![created volumes](./images/3.%20ebs%20volumes%20created.png)
_You will see the 3 EBS Volumes you created have an Available Volume state_

* Click on one of the volumes then click on the **Actions** button, you will see a drop-down and click on the **Attach volume** option.

![attach volumes](./images/3.%20attach%20volume.png)

* Select the NFS Server Instance and click on the **Attach volume** button.

![select nfs and attach volume](./images/3.%20select%20nfs%20and%20attach%20volume.png)

* Repeat these steps for the other 2 volumes and you will see that the volumes have been attached to the NFS Server Instance as shown below:

![attached volumes](./images/3.%20attached%20volumes.png)

### Step 4: Implement LVM Storage Management on the NFS Server

* Open terminal on your computer.

* Go to the Downloads directory (_i.e. `.pem` key pair is stored here_) using the command shown below:

```sh
cd Downloads
```

![cd downloads](./images/4.%20cd%20downloads.png)

* Run the following command to give read permissions to the `.pem` key pair file.

```sh
chmod 400 <private-key-pair-name>.pem
```

![chmod private key](./images/4.%20chmod%20keypair.png)

* SSH into the NFS Server Instance using the command shown below:

```sh
ssh -i <private-key-name>.pem ec2-user@<Public-IP-address>
```

![ssh nfs server](./images/4.%20ssh%20private%20key.png)

* Use the `lsblk` command to inspect the block devices attached to the server.

![lsblk](./images/4.%20lsblk1.png)
_Notice the names of the new created devices._

* Use `gdisk` utility to create a single partiton on **/dev/xvdf** disk.

_Note that all the devices in Linux reside in the **/dev** directory_

```sh
sudo gdisk /dev/xvdf
```

![gdisk](./images/4.%20gdisk%20xvdf.png)

* Type `n` to create a new partiton and fill in the data shown below into the parameters:

![n](./images/4.%20n%20partition.png)

1. Partition number (1-128, default 1): 1
2. First sector (34-20971486, default = 2048) or {+-}size{KMGTP}: 2048
3. Last sector (2048-20971486, default = 20971486) or {+-}size{KMGTP}: 20971486
4. Current type is 8300 (Linux filesystem) Hex code or GUID (l to show codes, Enter = 8300): 8300

* Type `p` to print the partition table of the **/dev/xvdf** device.

![p](./images/4.%20p%20partition.png)

* Type `w` to write the table to disk and type `y` to exit.

![w y](./images/4.%20w%20and%20y.png)

* Repeat the `gdisk` utility partitioning steps for **/dev/xvdg** and **/dev/xvdh** disks.

* Use the `lsblk` command to view the newly configured partiton on each of the 3 disks.

![lsblk2](./images/4.%20lsblk2.png)

* Install `lvm2` package using the command shown below:

```sh
sudo yum install lvm2 -y
```

![install lvm2](./images/4.%20install%20lvm2.png)

* Run the following command to check for available partitons:

```sh
sudo lvmdiskscan
```

![lvmdiskscan](./images/4.%20lvmdiskscan.png)

* Use `pvcreate` utility to mark each of the 3 disks as physical volumes (PVs) to be used  by LVM.

```sh
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```

![pvcreate](./images/4.%20pvcreate%20xvdf1%20xvdg1.png)

* Verify that your physical volumes (PVs) have been created successfully by running `sudo pvs`

![pvs](./images/4.%20sudo%20pvs.png)

* Use `vgcreate` utility to add 3 physical volumes (PVs) to a volume group (VG). Name the volume group **webdata-vg**.

```sh
sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
```

![vgcreate](./images/4.%20vgcreate%20webdata.png)

* Verify that your volume group (VG) has been created successfully by running `sudo vgs`

![vgs](./images/4.%20sudo%20vgs.png)

* Use the `lvcreate` utility to create 3 logical volumes: apps-lv (_use a third of the PV size_), logs-lv (_another third of the PV size_) and opt-lv (_the remaining third of the PV size_).

```sh
sudo lvcreate -n apps-lv -L 9.5G webdata-vg
sudo lvcreate -n logs-lv -L 9.5G webdata-vg
sudo lvcreate -n opt-lv -L 9.5G webdata-vg
```

![lvcreate](./images/4.%20lvcreate.png)

* Verify that your logical volume (LV) has been created successfully by running `sudo lvs`

![lvs](./images/4.%20sudo%20lvs.png)

* Verify the entire setup by running the following commands:

```sh
sudo vgdisplay -v #view complete setup - VG, PV, and LV
```

![vgdisplay](./images/4.%20sudo%20vgdisplay.png)

* Use `mkfs.xfs` to format the logical volumes (LV) with **xfs** file system.

```sh
sudo mkfs -t xfs /dev/webdata-vg/apps-lv
```
![mkfs apps-lv](./images/4.%20mkfs%20-t%20apps-lv.png)

```sh
sudo mkfs -t xfs /dev/webdata-vg/logs-lv
```

![mkfs logs-lv](./images/4.%20mkfs%20-t%20log-lv.png)

```sh
sudo mkfs -t xfs /dev/webdata-vg/opt-lv
```

![mkfs opt-lv](./images/4.%20mkfs%20-t%20opt-lv.png)

* Create **/mnt/apps** directory to be used by the Web Servers.

```sh
sudo mkdir -p /mnt/apps
```

![mkdir mnt/apps](./images/4.%20mkdir%20mnt:apps.png)

* Create **/mnt/logs** directory to be used by Web Servers log.

```sh
sudo mkdir -p /mnt/logs
```

![mkdir mnt/logs](./images/4.%20mkdir%20:mnt:logs.png)

* Create **/mnt/opt** directory to be used by Jenkins Server.

```sh
sudo mkdir -p /mnt/opt
```

![mkdir mnt/opt](./images/4.%20mkdir%20:mnt:opt.png)

* Mount **/mnt/apps** on **apps-lv** logical volume.

```sh
sudo mount /dev/webdata-vg/apps-lv /mnt/apps
```

![mount apps-lv mnt/apps](./images/4.%20mount%20:dev:webdata:apps%20:mnt:apps.png)

* Mount **/mnt/opt** on **opt-lv** logical volume.

```sh
sudo mount /dev/webdata-vg/opt-lv /mnt/opt
```

![mount opt-lv /mnt/opt](./images/4.%20mount%20opts-lv%20:mnt:opt.png)

* Use `rsync` utility to backup all the files in the log directory **/var/log** into **/mnt/logs** (*This is required before mounting the file system*).

```sh
sudo rsync -av /var/log/. /mnt/logs
```

![rsync /var/log /mnt/logs](./images/4.%20sudo%20rsync%20:var:log%20:mnt:logs.png)

* Mount **/var/log** on **logs-lv** logical volume. (_Note that all the existing data on /var/log will be deleted_).

```sh
sudo mount /dev/webdata-vg/logs-lv /var/log
```

![mount logs-lv](./images/4.%20mount%20logs-lv%20:var:log.png)

* Restore log files back into **/var/log** directory.

```sh
sudo rsync -av /mnt/logs/ /var/log
```

![rysnc /mnt/logs /var/log](./images/4.%20rsync%20:mnt:logs%20:var:log.png)

* Update `/etc/fstab` file so that the mount configuration will persist after restarting the server. The UUID of the device will be used to update the `/etc/fstab` file. Run the command shown below to get the UUID of the **apps-lv**, **logs-lv** and **opt-lv** logical volumes:

```sh
sudo blkid
```

![blkid](./images/4.%20sudo%20blkid.png)

* Update `/etc/fstab` in this format using your own UUID and remember to remove the leading and ending quotes.

```sh
sudo vi /etc/fstab
```

![vi /etc/fstab](./images/4.%20vi%20:etc:fstab.png)

* Test the configuration using the command shown below:

```sh
sudo mount -a
```

![mount -a](./images/4.%20mount%20-a.png)

* Reload the daemon using the command shown below:

```sh
sudo systemctl daemon-reload
```

![daemon-reload](./images/9.%20daemon-reload.png)

* Verify your setup by running `df -h`

![df -h](./images/4.%20df%20-h1.png)

### Step 5: Install and configure the NFS Server.

* Update the list of packages in the package manager.

```sh
sudo yum -y update
```

![yum update](./images/5.%20yum%20update.png)

* Install the NFS Server package.

```sh
sudo yum install nfs-utils -y
```

![install nfs-utils](./images/5.%20yum%20install%20nfs-utils.png)

* Start the NFS Server service.

```sh
sudo systemctl start nfs-server.service
```

![start nfs](./images/5.%20start%20nfs%20service.png)

* Enable the NFS Server service.

```sh
sudo systemctl enable nfs-server.service
```

![enable nfs](./images/5.%20enable%20nfs%20service.png)

* Check the status of the NFS Server service.

```sh
sudo systemctl status nfs-server.service
```

![status nfs](./images/5.%20status%20nfs%20service.png)

* Allow read, write and execute permissions for the Web Servers on the NFS Server.

```sh
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt
```

![chmod -R nobody](./images/5.%20chown%20-R%20nobody.png)

* Restart the NFS Server service.

```sh
sudo systemctl restart nfs-server.service
```

![restart nfs](./images/5.%20restart%20nfs%20service.png)

* Configure access to NFS for clients (_i.e. Web Servers_) within the same subnet (**Subnet CIDR: 172.31.16.0/20**).

```sh
sudo vi /etc/exports
```

```sh
/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
```

![vi /etc/exports](./images/5.%20vi%20:etc:exports.png)

```sh
sudo exportfs -arv
```

![exportfs -arv](./images/5.%20sudo%20exports%20-arv.png)

* Check which port is used by NFS. **Note that Inbound Rules have already been set to allow connections from the client (i.e Web Servers) on the NFS Port (i.e TCP and UDP Ports: 2049 and 111)**.

```sh
rpcinfo -p | grep nfs
```

![rpcinfo -p](./images/5.%20rpcinfo%20-p.png)

### Step 6: Provision a Database Server EC2 Instance

1. Name of Instance: Database Server
2. AMI: Red Hat Enterprise Linux 9 (HVM), SSD Volume Type
3. Key Pair Name: web11
4. New Security Group: Database Server SG
Inbound Rules: Allow Traffic From Anywhere On Port 22 and Traffic from the Subnet CIDR on Port 3306 (i.e. MySQL).

![database server instance summary](./images/6.%20database%20server%20instance%20summary.png)
_Instance Summary for Database Server_

### Step 7: Configure the Backend Database as part of the 3-Tier Architecture

* Open another terminal on your computer.

* Go to the Downloads directory (_i.e. `.pem` key pair is stored here_) using the command shown below:

```sh
cd Downloads
```

![cd downloads](./images/7.%20cd%20downloads.png)

* SSH into the Database Server Instance using the command shown below:

```sh
ssh -i <private-key-name>.pem ec2-user@<Public-IP-address>
```

![ssh database](./images/7.%20ssh%20keypair.png)

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

* Start the MySQL service.

```sh
sudo systemctl start mysqld
```

![start mysqld](./images/7.%20start%20mysqld.png)

* Enable the MySQL service.

```sh
sudo systemctl enable mysqld
```

![enable mysqld](./images/7.%20enable%20mysqld.png)

* Check if MySQL service is up and running.

```sh
sudo systemctl status mysqld
```

![status mysqld](./images/7.%20start%20mysqld.png)

* Log into the MySQL console application.

```sh
sudo mysql
```

![sudo mysql](./images/7.%20sudo%20mysql.png)

* Create a database called `tooling`.

```sh
CREATE DATABASE tooling;
```

![create database](./images/7.%20create%20database.png)

* Create a new user.

```sh
CREATE USER 'myuser'@'<Subnet_CIDR' IDENTIFIED BY 'password';
```

![create user](./images/7.%20create%20user.png)

* Grant all privileges on the `tooling`database to the user created.

```sh
GRANT ALL ON tooling.* TO 'myuser'*'<Subnet_CIDR';
```

![grant all](./images/7.%20grant%20all.png)

* Run the following command to apply and make changes effective.

```sh
FLUSH PRIVILEGES;
```

![flush privileges](./images/7.%20flush%20privileges.png)

* Display all the databases.

```sh
SHOW DATABASES;
```

![show databases](./images/7.%20show%20databases.png)

* Exit the MySQL console.

![exit mysql](./images/7.%20exit.png)

### Step 8: Provision 3 Web Servers EC2 Instances

1. Name of Instance: Web Server 1 (_change the names of the other two Instances to Web Server 2 and Web Server 3_)
2. AMI: Red Hat Enterprise Linux 9 (HVM), SSD Volume Type
3. Key Pair Name: web11
4. New Security Group: Web Server SG
Inbound Rules: Allow Traffic From Anywhere On Port 22 and Port 80

![web server instance summary](./images/8.%20web%20server%20instance%20summary.png)
_Instance Summary for Web Server 1_

### Step 9: Configure the Web Servers

* Open another terminal on your computer.

* Go to the Downloads directory (_i.e. `.pem` key pair is stored here_) using the command shown below:

```sh
cd Downloads
```

![cd downloads](./images/9.%20cd%20downloads.png)

* SSH into the Database Server Instance using the command shown below:

```sh
ssh -i <private-key-name>.pem ec2-user@<Public-IP-address>
```

![ssh web server](./images/9.%20ssh%20web%20server.png)

* Install NFS Client

```sh
sudo yum install nfs-utils nfs4-acl-tools -y
```

![install nfs client](./images/9.%20install%20nfs%20client.png)

* Mount `/var/www` and target the NFS server's export for apps.

```sh
sudo mkdir /var/www
```

![mkdir var/www](./images/9.%20mkdir%20:var:www.png)

```sh
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private_IP-Address>:/mnt/apps /var/www
```

![mount -t apps](./images/9.%20mount%20-t%20nfs%20:mnt:aps%20:var:www.png)

* Mount apache's log folder to the NFS server's export for logs.

```sh
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private_IP-Address>:/mnt/logs /var/log
```

![mount -t logs](./images/9.%20mount%20-t%20nfs%20:mnt:logs%20:var:log.png)

* Verify that NFS was mounted successfully by running `df -h`

![df -h](./images/9.%20df%20-h.png)

* Make sure that the changes will persist on the Web Server after reboot by updating the `/etc/fstab`

```sh
sudo vi /etc/fstab
```

* Add the following line then save and exit the file:

```sh
<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
<NFS-Server-Private-IP-Address>:/mnt/logs /var/log nfs defaults 0 0
```

![vi etc/fstab](./images/9.%20vi%20:etc:fstab.png)

* Test the configuration using the command shown below:

```sh
sudo mount -a
```

![mount -a](./images/4.%20mount%20-a.png)

* Reload the daemon using the command shown below:

```sh
sudo systemctl daemon-reload
```

![daemon-reload](./images/9.%20daemon-reload.png)

* Install [Remi's repository](http://www.servermom.org/how-to-enable-remi-repo-on-centos-7-6-and-5/2790/), Apache and PHP

```sh
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm -y

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd -y

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo setsebool -P httpd_execmem 1
```

![install remi, apache and php](./images/9.%20install%20apache,%20remi%20and%20php.png)

* Open two terminals and SSH into Web Server 2 and Web Server 3 EC2 Instances and run the following command to configure the two Web Servers:

```sh
sudo vi install.sh
```

* Paste the codebase below then save and exit the file.

```sh
#!/bin/bash

# input the Private IPv4 of your NFS Server
nfs_server_private_ip=172.31.26.52

sudo yum install nfs-utils nfs4-acl-tools -y

sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid $nfs_server_private_ip:/mnt/apps /var/www
sudo mount -t nfs -o rw,nosuid $nfs_server_private_ip:/mnt/logs /var/log
sudo mount -a
sudo systemctl daemon-reload

sudo chmod 777 /etc/fstab
sudo echo "$nfs_server_private_ip:/mnt/apps /var/www nfs defaults 0 0" >> /etc/fstab
sudo echo "$nfs_server_private_ip:/mnt/logs /var/log/httpd nfs defaults 0 0" >> /etc/fstab
sudo chmod 644 /etc/fstab

sudo yum install httpd -y
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y
sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm -y
sudo dnf module reset php
sudo dnf module enable php:remi-7.4
sudo dnf install php php-opcache php-gd php-curl php-mysqlnd -y
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1
sudo systemctl start httpd
sudo systemctl enable httpd
```

* Run the following command to run the `install.sh` script.

```sh
bash install.sh
```

* Verify that apache files and directories are available on Web Servers in `/var/www` and also on the NFS server in `/mnt/apps`. If you see the same files, it means NFS is mounted correctly. Verification can be done by taking the following steps:

1. On the Web Server 1 Terminal, go to the `/var/www/` directory and create a `test.txt` file in the `/var/www` directory

![cd /var/www](./images/9.%20cd%20:var:www.png)

![touch test.txt](./images/9.%20touch%20test.png)

2. On the NFS Server Terminal, go to the `/mnt/apps` directory and run the `ll` command to view list the files in the directory. You will see that the file `test.txt` file is present.

![mnt/apps && ll](./images/9.%20cd%20:mnt:apps.png)

* Fork the tooling source code from [Darey.io GitHub account](https://github.com/darey-io/tooling)

* Check if git in installed on the Web Server using the following command:

```sh
which git
```

![which git](./images/9.%20which%20git.png)

* Install the git package.

```sh
sudo yum install git -y
```

![install git](./images/9.%20install%20git.png)

* Go to the [Darey.io Tooling Repository](https://github.com/darey-io/tooling) and copy the highlighted link shown below:

![darey.io tooling repo](./images/9.%20git%20repo%20url.png)

* Clone the repository.

```sh
git clone https://github.com/darey-io/tooling.git
```

![git clone](./images/9.%20git%20clone.png)

* Deploy the tooling website's code to the Web Server. Ensure that the **html** folder from the repository is deployed to `/var/www/html`

```sh
cd tooling && ll
```

![cd tooling && ll](./images/9.%20cd%20tooling%20&&%20ll.png)

```sh
sudo cp -r html/. /var/www/html/
```

![cp html](./images/9.%20cp%20-r%20html%20:var:www:html.png)

* Disable SELinux.

```sh
sudo setenforce 0
```

![setenforce](./images/9.%20setenforce%200.png)

* To make this change permanent, open the following configuration file `/etc/sysconfig/selinux` and set `SELINUX=disabled`

```sh
sudo vi /etc/sysconfig/selinux
```

![sysconfig/selinux](./images/9.%20selinux%20=%20disabled.png)

* Update the website's configuration to connect to the Database Server by running the following command:

```sh
sudo vi /var/www/html/functions.php
```

![functions.php](./images/9.%20functions1.png)

![functions update](./images/9.%20function%20update.png)
_Updated `functions.php` file_

* Install MySQL client.

```sh
sudo yum install mysql -y
```

![install mysql client](./images/9.%20install%20mysql%20client.png)

* Apply `tooling-db.sql` script to your database using this commands shown below:

```sh
cd tooling
```

![cd tooling](./images/9.%20cd%20tooling.png)

```sh
mysql -h <database-private-ip> -u <db-username> -p tooling < tooling-db.sql
```

![mysql -h -u tooling](./images/9.%20mysql%20-h%20-u%20-p%20tooling.png)

### Step 10: Create a new admin user on your Database Server

* Connect to the Database Server Instance.

* Log into the console application.

```sh
sudo mysql
```

![sudo mysql](./images/10.%20sudo%20mysql.png)

* Display all the databases.

```sh
SHOW DATABASES;
```

![show databases](./images/10.%20show%20databases.png)

* Select the `tooling` database you want to work on.

```sh
USE tooling;
```

![use tooling](./images/10.%20use%20tooling.png)

* Display the tables in the `tooling` database.

```sh
SHOW TABLES;
```

![show tables](./images/10.%20show%20tables.png)

* Display all the contents of the `users` table.

```sh
SELECT * FROM users;
```

![select from users](./images/10.%20select%20from%20users.png)

* Input data of a new user into the table.

```sh
INSERT INTO users (id, username, password, email, user_type, status)
-> VALUES (2, 'donald', '5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin', '1')
```

![insert into users](./images/10.%20insert%20into%20users.png)

* Exit the console application.

![exit](./images/10.%20exit.png)

### Step 11: Start and Enable Apache on the Web Server.

* Connect to the Web Server 1 Instance.

* Run the following command to test if you can connect to the tooling website:

```sh
curl localhost
```

![curl localhost](./images/11.%20curl%20localhost.png)
_Note that you can't connect to the website because the apache service isn't up and running._

* Start the apache service.

```sh
sudo systemctl start httpd
```

![start apache](./images/11.%20start%20apache.png)

* Enable the apache service.

```sh
sudo systemctl enable httpd
```

![enable apache](./images/11.%20enable%20apache.png)

* Check if the apache service is up and running.

```sh
sudo systemctl status httpd
```

![status apache](./images/11.%20status%20apache.png)

### Step 12: Open the tooling website in your browser

* Go to your browser and paste the following URL:

```sh
http://<Private_IP-Address_Web_Server_1>
```

![url1](./images/12.%20url1.png)

![url2](./images/12.%20url2.png)