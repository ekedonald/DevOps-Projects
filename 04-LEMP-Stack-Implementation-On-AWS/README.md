# LEMP Stack Implementation On AWS
___
## What Is A LEMP Stack?
LEMP is an open-source web application stack used to develop web applications. The term LEMP is an acronym that represents **L** for the **Linux Operating system**, **Nginx** (pronounced as **engine-x**, hence the **E** in the acronym) **web server**, **M** for **MySQL database**, and **P** for **PHP scripting language**.

The __LEMP__ stack is a combination of four open-source technologies that are used in web development. These technologies include:

* __Linux__: The operating system that runs the web server.

* __Nginx__: The web server software that handles HTTP requests.

* __MySQL__: The relational database management system that stores the website's data.

* __PHP__: The programming language used to build dynamic web applications.

## LEMP Stack Architecture
### Linux
Linux is the operating system and the first layer of the architecture. It binds every other layer together. It is free and open-source and well known to be highly secure and less vulnerable to malware and viruses even if compared to Windows or macOS.

### Nginx
It is a web server that follows an event-driven approach and handles multiple requests within one thread. Nginx supports all Unix-like OS and also supports Windows partially. 

When a web browser requests a web page that request is handled by the web server, here that web server is Nginx. Then the web server passes that request to server-side technologies used in the LEMP stack for instance as a server-side scripting language like PHP to communicate with the server and database.

### MySQL
It is an open-source SQL-based database that is used to store data and manipulate data while maintaining data consistency and integrity. It organizes data in tabular form in rows and columns.

### PHP
It is a scripting language that works on the server-side and communicates with the database MySQL and does all operations that user requests like fetching data, adding data, manipulating data, or processing the data.

## How Does The LEMP Stack Work?
The LEMP stack works by using Nginx as the web server, which listens for HTTP requests and forwards them to the appropriate PHP script. The PHP script generates a response, which is then sent back to the user via Nginx.

MySQL is used to store and manage the website’s data. PHP communicates with MySQL to retrieve and store data as needed.

## How To Set Up A LEMP Stack On AWS

### Prerequisites 
1. Ensure you have an AWS Account. If you don't have an account, [sign up for AWS here.](https://portal.aws.amazon.com/billing/signup#/start/email)

### Step 1: Launch An EC2 Instance
The following steps are taken to launch an EC2 Instance on AWS:

* On the EC2 Dashboard, click on the __Launch Instance__ button.

![Launch Instance1](./images/1.%20Launch%20Instance1.png)

* Give the EC2 Instance a name of your choice, type "ubuntu" on the AMI search bar and hit enter.

![Launch Instance2](./images/1.%20Launch%20Instance2.png)

* Select __Ubuntu Server 22.04 LTS (HVM), SSD Volume Type__ as your preferred AMI.

![Launch Instance3](./images/1.%20Launch%20Instance3.png)

* Create a new key pair login, the key pair login is used to SSH into the Instance.

![Launch Instance4](./images/1.%20Launch%20Instance4.png)

* Give the key pair a name of your choice, select the `.pem` format and a private key will be sent to the Downloads directory on your computer.

![Launch Instance5](./images/1.%20Launch%20Instance5.png)

* Create a security group and allow SSH traffic from an IP address as shown below:

![Launch Instance6](./images/1.%20Launch%20Instance6.png)

* Click on the Launch Instance button.

![Launch Instance7](./images/1.%20Launch%20Instance7.png)

### Step 2: SSH Into The EC2 Instance
The following steps are taken to SSH into the EC2 Instance.

* On the EC2 Dashboard, click on the Running Instance tab.

![Launch Instance8](./images/1.%20Launch%20Instance8.png)

* Click on the Instance ID of the Running Instance.

![Launch Instance9](./images/1.%20Launch%20Instance9.png)

* Click on the Connect button of the Instance ID summary.

![Launch Instance10](./images/1.%20Launch%20Instance10.png)

* The highlighted commands in the image below are used to SSH into the EC2 Instance.

![Launch Instance11](./images/1.%20Launch%20Instance11.png)

* On your terminal, run the following command `cd Downloads` to go to the location of the `.pem` private key file.

* Run the code shown below to change file permisssions of the `.pem` private key file:

```bash
sudo chmod 0400 <private-key-name>.pem
```

![SSH Instance1](./images/1.%20SSH%20Instance1.png)

* Finally, connect to the EC2 Instance by running the command shown below:

```bash
ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>
```
![SSH Instance2](./images/1.%20SSH%20Instance2.png)

### Step 3: Installing The Nginx Web Server
The following steps are taken to install the Nginx web server:

* Update the list of packages in the package manager.

```bash
sudo apt update
```

![apt update](./images/2.%20apt%20update.png)

* Run the Nginx package installation with the `-y` flag to confirm the installation.

```bash
sudo apt install nginx -y
```

![install nginx](./images/2.%20install%20nginx.png)

* To verify that Nginx was successful and is running as a service in Ubuntu, run the following command:

```bash
sudo systemctl status nginx
```

![sysytemctl status nginx](./images/2.%20systemctl%20status%20nginx.png)

### Step 4: Updating The Firewall
Before any traffic can be received by the web server, you need to open TCP port 80 which is the default port browsers use to connect to access web pages on the internet. 

The following steps are taken to open TCP port 80:

* On the EC2 Dashboard, click on the Running Instances tab.

![Launch Instance8](./images/1.%20Launch%20Instance8.png)

* Click on the Instance ID.

![Launch Instance9](./images/1.%20Launch%20Instance9.png)

* On the Instance Summary of the EC2 Instance, click on the Security tab highlighted as shown below:

![Inbound Rules1](./images/2.%20Inbound%20Rules1.png)

* Click on the Security Group highlighted below:

![Inbound Rules2](./images/2.%20Inbound%20Rules2.png)

* Click on the Edit Inbound Rules button as shown below:

![Inbound Rules3](./images/2.%20Inbound%20Rules3.png)

* Click on the Add Rule button as shown below:

![Inbound Rules4](./images/2.%20Inbound%20Rules4.png)

* Add a rule that allows HTTP (port 80) and all IPv4 addresses to connect and save the rule.

![Inbound Rules5](./images/2.%20Inbound%20Rules5.png)

* Finally, the server can now be accessed locally and from any IPv4 address. To check if you can access the server locally in Ubuntu, run the following command:

```bash
curl http://localhost:80
```

![curl http localhost](./images/2.%20curl%20localhost.png)

* To check if your Nginx server can respond to requests from the Internet, open your browser and run the following URL:

```bash
http://<Public-IP-Address>:80
```

![http public IP](./images/2.%20http%20public%20IP.png)

The public IP address can be retrieved by clicking on the Instance ID of the Running Instance as shown below:

![IP address1](./images/2.%20IP%20address1.png)

However, It can also be retrieved by running the following command:

```bash
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```

![IP address2](./images/2.%20IP%20address2.png)

### Step 5: Installing MySQL
The following steps are taken to install MySQL:

* Install the MySQL package using apt with the `-y` flag to confirm the installation.

```bash
sudo apt install mysql-server -y
```

![install mysql](./images/3.%20install%20mysql-server.png)

* Log into the MySQL console by running the following command:

```bash
sudo mysql
```

This will connect to the MySQL server as the administrative user root. You should see an output like this:

![sudo mysql](./images/3.%20sudo%20mysql.png)

* Run a security script that comes pre-installed with MySQL. This script removes insecure default settings and locks down access to your database system. Before running the script, you will set a password for the root user, using *mysql_native_password* as the default authentication method. You are defining this user's password as `PassWord.1`.

```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```

![sql aunthentication](./images/3.%20sql%20aunthentication1.png)

* Exit the MySQL shell by running the `exit` command below:

![exit mysql](./images/3.%20mysql%20exit.png)

* Start the interactive script by running the command below:

```bash
sudo mysql_secure_installation
```

![sudo mysql_secure_installation](./images/3.%20sudo%20mysql%20secure%20installation.png)

* This will ask if you want to configure the `VALIDATE PASSWORD PLUGIN`. Enabling this feature is something of a judgment call. If enabled, passwords that don't match the specified criteria will be rejected by MySQL with an error. It is safe to leave validation disabled, but you should always use strong, unique passwords for database credentials. Answer `y` for yes for the validate password plugin prompt.

![validate password1](./images/3.%20validate%20password1.png)

* If you select "yes", you will be asked to select the level of password validation. Keep in mind that if you enter `2` for the strongest level, your password will need to contain a numeric, mixed case and special character e.g. `PassWord.1`. Note the highlighted password has already been set so you select `n` to leave the password for root unchanged.

![validate password2](./images/3.%20validate%20password2.png)

* Decline to change the password for root by typing `n`.

![validate password3](./images/3.%20validate%20password3.png)

* Remove anonymous users by typing `y`.

![validate password4](./images/3.%20validate%20password4.png)

* Disallow root login remotely by typing `y`.

![validate password5](./images/3.%20validate%20password5.png)

* Remove the test database and access by typing `y`.

![validate password6](./images/3.%20validate%20password6.png)

* Reload privilege tables by typing `y`.

![validate password7](./images/3.%20validate%20password7.png)

### Step 6: Installing PHP
Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for better overall performance in most PHP-based websites but it requires additional configuration. You'll need to install `php-fpm` i.e. PHP FastCGI Process Manager and tell Nginx to pass PHP requests to this software for processing. 

Additionally, you'll need `php-mysql`, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.

* To install these 2 packages run the following command with the `-y` flag to confirm installation:

```bash
sudo apt install php-fpm php-mysql
```

### Step 7: Configuring Nginx To Use PHP Processor
The following steps are taken to configure Nginx to use a PHP processor:

* Create the root web directory for *your_domain* as shown below:

```bash
sudo mkdir /var/www/projectLEMP
```

* Assign ownership of the directory with $USER environment variable which will reference your current system user as shown below:

```bash
sudo chown -R $USER:$USER /var/www/projectLEMP
```

![mkdir projectLEMP](./images/4.%20mkdir%20projectLEMP.png)

* Open a new configuration file in Nginx's `sites-available` directory using your preferred command line. Here, we'll use `nano`:

```bash
sudo nano /etc/nginx/sites-available/projectLEMP
```

* This will create a new blank file, copy and paste the following command into the configuration file:

```bash
#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```

![sudo nano /etc/nginx/sites-available/prpjectLEMP](./images/4.%20nano%20:etc:nginx:sites-available:projectLEMP.png)

Here's what each of these directives and location blocks do:

1. `listen`: Defines what port Nginx will listen on. In this case, it will listen on port `80`, the default port for HTTP.

2. `root`: Defines the document root where the files served by this website are stored.

3. `index`: Defines in which order Nginx will prioritize index files for this website. It is a common practice to list `index.html` files with higher precedence than `index.php` files to allow for quickly setting up a maintenance landing page in PHP applications. You can adjust these settings to better suit your application needs.

4. `server_name`: Defines which domain names and/or IP addresses this server block should respond for. **Point this directive to your server's domain name or public IP address**.

5. `location /`: The first location block includes a `try_files` directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate resource, it will return a 404 error.

6. `location ~ \-php$`: This location block handles the actual PHP processing by pointing Nginx to the fastogi-php.conf configuration file and the `php7.4-fpm.sock` file, which declares what socket is associated with `php-fpm`.

7. `location ~ /\.ht`: The last location block deals with `htaccess` files, which Nginx does not process. By adding the deny all directive, if any `.htaccess` files happen to find their way into the document root,they will not be served to visitors.

* When you're done editing, save and close the file. 

* Activate your configuration by linking to the configuration file from Nginx's `sites-enabled` directory using the command below.

```bash
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```

* This will tell Nginx to use the configuration the next time it is reloaded. You can test your configuration for syntax errors by running the following command:

```bash
sudo nginx -t
```

You'll see the following message if no errors were reported.

![sudo nginx -t](./images/4.%20sudo%20nginx%20-t.png)

* Disable the default Nginx host configured to listen on port 80 using the command below.

```bash
sudo unlink /etc/nginx/sites-enabled/default
```

* Reload Nginx to apply changes.

```bash
systemctl reload nginx
```

* The website is now active but the webroot `/var/www/projectLEMP` is still empty. Create an `index.html` file in that directory to test if the new server block is functional as shown below:

```bash
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```

* Go to your browser and open the website URL using your IP address.

```bash
http://<Public-IP-Address>:80
```

![http public IP](./images/4.%20http%20public%20IP.png)

* Finally, if you see the text from the *echo* command appended to the `index.html` file, it means the Nginx site is working.

### Step 8: Testing PHP With Nginx
The LEMP stack is completely installed and fully operational. You can test it to validate that Nginx can correctly hand `.php` files off to your PHP processor.

* Create and open a new file called `info.php` within your document root with your text editor.

```bash
nano /var/www/projectLEMP/info.php
```

* Copy and paste the code below into the new file. This is a valid PHP code that will return information about your server.

```bash
<?php
phpinfo();
```

![php info](./images/5.%20php%20info.png)

* You can now access this page in your web browser by visiting the public IP address you've set up in your Nginx configuration file, followed by `/info.php`:

```bash
http://public_IP_address/info.php
```

* You will see a web page containing detailed information about your server.

![http IP address info/php](./images/5.%20http%20public%20IP%20:info.png)

* After checking the relevant information about your PHP server through that page, it's best to remove the file as it contains sensitive information about your PHP environment and Ubuntu server.

```bash
sudo rm /var/www/projectLEMP/info.php
```

### Step 9: Retrieving Data From MySQL Database With PHP
The following steps are taken to retrieve data from MySQL database with PHP:

* Connect to the MySQL console using the root account.

```bash
sudo mysql
```

* Create a database from your MySQL console.

```bash
mysql> CREATE DATABASE `example_database`;
```

![create database](./images/6.%20CREATE%20DATABASE.png)

* Create a new user and grant him full privileges on the database. The command below creates a new user named `example_user` using *mysql_native_password* as the default authentication method. The user's password is defined as `PassWord.1` but you can choose any password of your choice.

```bash
mysql>  CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```

![create user](./images/6.%20CREATE%20USER.png)

* Give the new user permission over the `example_database`. This will give the **example_user** user full privileges over the **example_database** while preventing the user from creating or modifying other databases on your server.

```bash
mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';
```

![grant all](./images/6.%20GRANT%20ALL.png)

* Exit the MySQL shell.

```bash
mysql> exit
```

![mysql exit](./images/6.%20mysql%20exit.png)

* Test if the new user has the proper permissions by logging into the MySQL console again, this time using the custom user credentials.

```bash
mysql -u example_user -p
```

![mysql example user](./images/6.%20mysql%20-u%20example%20user.png)

Note that the `-p` flag in this command will prompt you for the password used when creating the `example_user` user.

* After logging into the MySQL console, confirm that you have access to the `example_database` database.

```bash
mysql> SHOW DATABASE;
```

![show database](./images/6.%20SHOW%20DATABASES.png)

* Create a test table named **todo_list** using the command below.

```bash
CREATE TABLE example_database.todo_list (item_id INT AUTO_INCREMENT,content VARCHAR(255),PRIMARY KEY(item_id));
```
![create table](./images/6.%20CREATE%20TABLE.png)

* Inset a few rows of content in the test table. This is done a couple of times to populate the table.

```bash
mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
```

![insert into1](./images/6.%20INSERT%20INTO1.png)

![insert into2](./images/6.%20INSERT%20INTO2.png)

* To confirm that the data was successfully saved to your table, run the following command:

```bash
mysql>  SELECT * FROM example_database.todo_list;
```

* You will see the following output:

![select from](./images/6.%20SELECT%20FROM.png)

* Exit the MySQL console.

```bash
mysql> exit
```

![mysql exit](./images/6.%20mysql%20exit.png)

* Create a new PHP script that will connect to MySQL and query for your content in your custom web root directory using the following command:

```bash
nano /var/www/projectLEMP/todo_list.php
```
* Copy the code below into the `todo_list.php` script.

```bash
<?php
$user = "example_user";
$password = "PassWord.1";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```

![nano /var/www/projectLEMP/todo_list](./images/6.%20nano%20:var:www:projectLEMP:todo_list.png)

* Save and close when you are done editing.

* You can now access this page in your web browser by visiting the public IP address configured for your website followed by `/todo_list.php`.

```bash
http://<public_IP_address>/todo_list.php
```

![http public ip todo_list](./images/6.%20http%20public%20IP%20:todo_list.png)