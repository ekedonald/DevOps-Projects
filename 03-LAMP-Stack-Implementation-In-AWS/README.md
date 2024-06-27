# LAMP STACK IMPLEMENTATION IN AWS
___
A LAMP stack is an open-source stack that combines four services that the developers use to create powerful websites and applications. The base layer is the operating system called Linux, the layer for the web server is Apache, the database layer uses MySQL, and PHP is used as the programming language. When used properly, these four layers enable hosting, creating, and maintaining websites and web applications.

## LAMP Stack Architecture
### Linux
Linux is the operating system and the first layer of the architecture. It binds every other layer together. Linux is also an open-source operating system; it can be configured to meet the requirements of other software layers.

### Apache
Apache is used for the web server and is the second layer of the LAMP stack model. Apache uses an HTTP connection to exchange information between the browser and the web server. Apache also calls upon PHP to serve dynamic content to the web server.

### MySQL
MySQL is the third layer in our LAMP stack. MySQL is used for storing and managing information. For example, client data and product data are all stored in the relational, open-source MySQL software. When information is requested, the database is queried and served.

### PHP
Sitting on top of them all is the fourth and final layer. PHP stands for PHP: Hypertext Preprocessor, and it's a scripting language that allows you to dynamically serve content. Dynamic content is content whose values are not constant. They change depending on the circumstances of the function they are trying to accomplish. PHP is also linked to MySQL and the web server, which then is used in tandem to serve content to your browser.

![Lamp Stack](./images/0.%20LAMP%20Stack.jpeg)

*Visual Representation of the LAMP Stack*

## How It Works
Whenever you open a website through a browser, the LAMP stack is triggered, and the information it processes goes through the following flow. The web application makes a request from the web browser. The LAMP stack then initiates the Apache web server and MySQL, which use PHP for their communication. 

The Apache web server first receives the request from the web browser, depending on the request (static or dynamic content) it serves it accordingly. If the request is for static content, Apache serves it immediately. However, if it is dynamic content, the PHP component then gets involved and loads the correct PHP file to process that request.

Once the correct PHP file is found, the written functions within it are then used to interpret the request and provide the necessary output. Some PHP functions also utilize the database, so a connection to MySQL is necessary. 

After the PHP function is done, the output is then relayed back to the web server in HTML format. Also, note that sometimes a new entry in the database is made. The Apache web server then serves the dynamic content to the browser.

## How To Set A LAMP Stack On AWS
### Step 1: Launch An EC2 Instance

The following steps are taken to lauch an EC2 Instance on AWS:

* On the EC2 Dashboard, click on the **Launch Instance button** 

![Launch Instance](./images/0.%20Launch%20Instance1.png)

* Give the EC2 Instance a name of your choice and search for the preferred Virtual Server (i.e Ubuntu).

![Launch Instance2](./images/0.%20Launch%20Instance2.png)

* Select the Amazon Machine Image. Ubuntu server 22.04 was selected.

![Launch Instance3](./images/0.%20Launch%20Instance3.png)

* Create a key pair login, the key pair login was used to SSH into the instance.

![Launch Instance4](./images/0.%20Launch%20Instance4.png)

* Give the key pair a name, select the `.pem` format and a private key will be sent to the Downloads folder on your computer.

![Launch Instance5](./images/0.%20Launch%20Instance5.png)

* Create a security group and allow SSH traffic from any IP address as shown below.

![Launch Intance5a](./images/0.%20Launch%20Instance5a.png)

* Click on the Launch Instance buttonn.

![Launch Instance6](./images/0.%20Launch%20Instance6.png)


### Step 2: Connect To EC2 Using SSH

The following steps are taken to SSH into an EC2 instance:

* On the EC2 Dashboard, click on the Running Instances tab.

![Launch Instance7](./images/0.%20Launch%20Instance7.png)

* Click on the Instance ID of the Running Instance.

![Launch Instance8](./images/0.%20Launch%20Instance8.png)

* Click on the Connect button of the Instance ID summary.

![Launch Instance9](./images/0.%20Launch%20Instance9.png)

* The highlighted commands in the image below are used to SSH into the EC2 instance.

![Launch Instance10](./images/0.%20Launch%20Instance10.png)

* On your terminal, run the following command to `cd Downloads` to go to the location of the `.pem` private key file.

* Run the code shown below to change file permissions for the `.pem` private key file:

```bash
sudo chmod 0400 <private-key-name>.pem
```

![SSH Instance1](./images/0.%20SSH%20Instance1.png)

* Finally, connect to the EC2 Instance by running the command shown below:

```bash
ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>
```

![SSH Instance2](./images/0.%20SSH%20Instance2.png)

### Step 3: Installing Apache

* Update the list of packages in the package manager.

```bash
sudo apt update
```

![apt update](./images/1.%20apt%20update.png)

* Run apache2 package installation.

```bash
sudo apt install apache2
```

![apt install apache2](./images/2.%20install%20apache2.png)

* Run the systemctl status command to check if apache2 is running, if it is green then apache2 is running correctly. Your first web server has been launched.

```bash
sudo systemctl status apache2
```

![systemctl status apache2](./images/3.%20systemctl%20status%20apache2.png)

### Step 4: Updating The Firewall

Before any traffic can be received by the web server, you need to open TCP port 80 which is the default port browsers use to connect to access web pages on the internet. The following steps are taken open TCP port 80:

* Click on Security Groups on the EC2 Dashboard.

* Click on the default security group ID.

![firewall1](./images/4.%20firewall1.png)

* Click on Edit Inbound Rules.

![firewall2](./images/4.%20firewall2.png)

* Add a rule that allows HTTP (port 80) and all IPv4 addresses to connect.

![firewall3](./images/4.%20inbound%20rules.png)

Finally, the server can now be accesssed locally and from any IPv4 addres. To check if you can access the server locally in Ubuntu, run the following command:

```bash
curl http://localhost:80
```

![curl localhost](./images/4.%20curl%20localhost.png)

To check if the apache HTTP server can respond to requests from the Internet, open your browser and run the following url:

```bash
http://<Public-IP-Address>:80
```

![http ip address](./images/6.%20http-ip%20address.png)

The public ip address can be retrieved by running the following command:

```bash
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```

![public ip address2](./images/5.%20public%20ip%20address2.png)

It can also be retrieved by clicking on the Instance ID of the Running Instance as shown below:

![public ip address1](./images/5.%20public%20ip%20address1.png)

### Step 5: Installing MySql

The following steps are taken to install MySql:

* Install the MySql package using apt.

```bash
sudo apt install mysql-server
```

* Log into the MySql console by running the command below:

```bash
sudo mysql
```

This will connect to the MySql server as the administrative database user root. You should see output like this:

![mysql](./images/8.%20mysql.png)

* Run a security script that comes pre-instaleld with MySql. Thi script removes insecure default settings and lock down access to your database system. Before running the script, you will set a password for the root user, using *mysql_native_password* as the default authentication method. You are defining this user's password as `PassWord.1`

```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```

![sql aunthentication1](./images/9.%20sql%20aunthentication1.png)

* Exit the MySql shell by running the command below:

![sql authentication2](./images/9.%20sql%20aunthentication2.png)

* Start the interactive script by running the command below:

```bash
sudo mysql_secure_installation
```

![sql secure installation](./images/9.%20sql%20secure%20intallation.png)

* This will ask if you want to configure the `VALIDATE PASSWORD PLUGIN`. Enabling this feature is something of a judgement call. If enabled, passwords which don't match the specified criteria will be rejected by MySql with an error. It is safe to leave validation disabled, but you should always use strong, unique passwords for database credentials. Answer `Y` for yes for the validate password plugin prompt.

![sql validate password1](./images/9.%20sql%20validate%20password1.png)

* If you select "yes", you will be asked to select the level of password validation. Keep in mind that if you enter `2` for the strongest level, your password will need to contain a numeric, mixed casse abd special character e.g `PassWord.1`. Note the highlighted password has already been set so you select `n` to leave the password for root unchanged.

![sql validate password2](./images/9.%20sql%20validate%20password2.png)

* Remove anonymous users by typing `y`.

![sql validate password3](./images/9.%20sql%20validate%20password3.png)

* Disallow root login remotely by typing `n`.

![sql validate password4](./images/9.%20sql%20validate%20password4.png)

* Disallow removing test database and access by typing `n`. Reload privilege tables by typing `y`.

![sql validate password5](./images/9.%20sql%20validate%20password5.png)

* Test if you are able to log in to the MySql console by running the command below:

```bash
sudo mysql -p
```

![sql validate password6](./images/9.%20sql%20validate%20password6.png)

The `-p` flag in the command will prompt you for the password.

* Exit the MySql console by typing `exit`.

![sql validate password7](./images/9.%20sql%20validate%20password7.png)

### Step 6: Installing Php

In addition to the `php` package, you'll need `php-mysql`, a php module that allows php to communicate with MySql-based databases and you'll also need `libapache2-mod-php` to enable apache to hanlde php files. Core php packages will automatically be installed as dependencies.

* To install these 3 packages at once, run:

```bash
sudo apt install php libapache2-mod-php php-mysql
```

![install php](./images/10.%20install%20php.png)

* Once installation has been completed, run the command below to check the version of php installed.

```bash
php -v
```

![php version](./images/10.%20php%20version.png)

At this point, the LAMP stack is completely installed and fully operational.

### Step 6: Enable Php On The Website

Within the default *Directory Index* settings on Apache, a file named `index.html` will always take precedence over an `index.php` file. If you run the command `sudo vim /etc/apache2/mods-enabled/dir.conf`, it will display a prompt of the order of preference of files in the *Directory Index*. The order of preference is from left to right. 

```bash
sudo vim /etc/apache2/mods-enabled/dir.conf
```

![Direcory Index Preference1](./images/10.%20Directory%20Index%20Preference1.png)

* Hence, to prioritize the `index.php` file, move its position as shown below.

![Direcory Index Prefernce2](./images/10.%20Directory%20Index%20Preference2.png)

* After saving and closing the file, reload apache for changes to take effect.

```bash
sudo systemctl reload apache2
```

![reload apache2](./images/14.%20system%20reload.png)

* Create a php script to test if php is correctly installed on your server. Run the following command and write the code below into the empty file you created as shown below:

```bash
sudo vim /var/www/projectlamp/index.php
```

![php info](./images/10.%20php%20info.png)

* The default directory the apache server will search for files is `/var/www/html` and the server not be able to load the `index.php` file on your browser since the php file isn't in that directory. To change the directory to `/var/www/projectlamp`, run the following command:

```bash
sudo vim /etc/apache2/sites-available/000-default.conf
```

![default location](./images/10.%20default%20location%20for%20html%20file.png)

![new location](./images/10.%20new%20location%20for%20html%20file.png)

* When you are finished, refresh your browser and you'll get a page similar to this:

![php webpage](./images/10.%20php%20webpage.png)

After checking the relevant information about your php server through that page, it's best to remove the file you created as it contains sensitive information about your php environment and ubuntu server. Remove the file using the command below:

```bash
sudo rm /var/www/projectlamp/index.php
```

### Step 7: Creating A Virtual Host For Your Website Using Apache

* Assign ownership of the directory with the `$USER` environment variable which reference your current system user:

```bash
sudo chown -R $USER:$USER /var/www/projectlamp
```

* Then create and open a new configuration file in apache's `sites-available` directory using the command below:

```bash
sudo vi /etc/apache2/sites-available/projectlamp.conf
```
* Copy the following code into the `projectlamp.conf` file and save:

```bash
<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

* Use the `ls` command to show the new file in the *sites-available* directory as shown below:

```bash
sudo ls /etc/apache2/sites-available
```

![ls apapche2 sites-available](./images/14.%20ls%20apache2%20sites-available.png)

* Use the *a2ensite* command to enable the new virtual host:

```bash
sudo a2ensite projectlamp
```

![a2ensite projectlamp](./images/14.%20a2ensite%20projectlamp.png)

* You might want to disable the default website that comes with apache. This is required if you're not using a custom domain name because in this case apache's default configuration would overwrite your virtual host. To disable apache's default website, use the *a2dissite* command.

```bash
sudo a2dissite 000-default
```

* To make sure your configuration file doesn't contain syntax erors, run the command below:

```bash
sudo apache2ctl configtest
```

![apache2ctl configtest](./images/14.%20apache2ctl%20configtest.png)

* Finally, reload apache so these changes can take effect.

```bash
sudo systemctl reload apache2
```

![system reload](./images/14.%20system%20reload.png)

* Your website is now active but the web root `/var/www/projectlamp` is empty. Create an `index.html` in that location so you can test if the virtual host works as expected using the command below:

```bash
touch /var/www/projectlamp/index.html 
```

* Run the command below to input information into the `index.html` file so you can test if the virtual host works as well.

```bash
sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
```

* Go to your browser and open your website url using IP address:

```bash
http://<Public-IP-Address>:80
```

![updated html index](./images/14.%20updated%20html%20index.png)

If you see the text from the `echo` command you wrote to the `index.html` file, then it means your apache virtual host is working as expected.