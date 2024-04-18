# Implementing a Client-Server Architecture using MySQL Database Management System (DBMS)
___
## Understanding Client-Server Architecture

Client-Server refers to an architecture in which two or more computers are connected together over a network to send and receive requests from one another. In their communication, each machine has its role: the machine sending requests is usually referred to as a **Client** and the machine responding is called a **Server**.

Clients and servers work together in a Client-Server Architecture to enable communication, data exchange and the provision of services over a network. This interaction is a fundamental concept in modern computing and is used in various applications, including web browsing, file sharing and database access. 

A simple diagram of a Web Client-Server Architecture is shown below:

![client server architecture](./images/Client-Server.png)

Here's how clients and servers work together:

### 1. Client Request
* The client initiates communication by sending a request to the server. This request typically specifies what the client needs such as a web page, an email, a file or a database query.

* The request includes information like the type of service required, any parameters and other relevant data.

### 2. Server Response
* The server receives the client's request and processes it based on the service or resource requested.

* The server performs the necessary operations to generate a response, which could be data, content or an acknowledgment.

### 3. Data Transmission
* The server sends the response back to the client over the network.

* The client receives and processes the response. This may involve rendering a web page, displaying an email, saving a file or presenting data.

### 4. Interaction
* The client and server can engage in a back-and-forth interaction, with the client making additional requests and the server responding as needed.

* This interaction continues until the client's requirements are met or until the client chooses to terminate the connection.

### 5. Statelessness
* In many client-server interactions, the server is stateless, meaning it doesn't retain information about previous requests from the same client. Each request from the client is independent and the client must include any necessary context or session information.

* However, stateful communication is also possible, where the server maintains some level of session state between requests.

The diagram below shows a machine trying to access a website using a web browser or simply **curl** command as a client and it sends HTTP requests to a web server (Apache, Nginx, IIS or any other server) over the internet.

![illustration 1](./images/illustration%201.png)

If we extend this concept further and add a Database Server to our architecture, we can get the picture shown below:

![illustration 2](./images/illustration%202.png)

In this case, the Web Server has the role of the client that connects and reads/writes to/from a Database (DB) Server (MySQL, MongoDB, Oracle or SQL Server) and the communication happens over a Local Network (it can also be Internet Connection but it is a common practice to place the Web Server and Database Server close to each other in a local network).

The setup on the diagram above is a typical generic Web Stack architecture (LAMP, LEMP, MEAN, MERN). This technology can be implemented with many other technologies (i.e. various Web and Database Servers from Small Page Applications to Large and Complex Portals).


## How To Implement a Client-Server Architecture using MySQL Database Management System (DBMS)

### Prerequisite

1. Ensure you have an AWS account. If you don't have an account, [sign up for AWS here.](https://portal.aws.amazon.com/billing/signup#/start/email)

The following steps are taken to implement a basic Client-Server Architecture using MySQL Relational Database Management System (RDBMS):

### Step 1: Create and configure two Linux-Based Virtual Servers (EC2 Instance in AWS)

* On the EC2 Dashboard, click on the Launch Instance button.

![Launch Instance](./images/1.%20launch%20instance.png)

* On the Name Box and Amazon Machine Image, type **mysql_server** and **ubuntu** respectively then select **2** as the number of Instances you want to create.

![namebox, ami and number of instance](./images/1.%20namebox,%20ami,%20no%20of%20instances.png)

* Select **Ubuntu Server 22.04 LTS (HVM), SSD Volume Type** as the Amazon Machine Image.

![ami_ubuntu server 22.04](./images/1.%20ami_ubuntu%20server%2022.04.png)

* Click on create new key pair.

![create new keypair](./images/1.%20create%20a%20keypair.png)

* Give the key pair name a name of your choice (i.e client-server-key), select `RSA` as the key pair type and `.pem` as the key file format then click on **Create key pair**.

![keypair name, RSA .pem](./images/1.%20keypair%20name,%20rsa,%20pem.png)

* The key pair will be downloaded into the Downloads folder on your computer.

* Click on the Launch Instance button.

![launch instance button](./images/1.%20launch%20instance%20button.png)

* On the EC2 Dashboard, click on the Instances tab to display all the Instances on your AWS console.

![instance tab](./images/ec2%20dashboard.png)

* You will notice there are 2 Instances named **mysql_server**, rename one of them to **mysql_client** by clicking on the pencil icon that appears right beside the name of the Instance.

![display instances](./images/1.%20display%20all%20instances.png)

* Click on the Instance ID of the **mysql_client** Instance and copy the **Private IPv4 address**.

![client ip address](./images/1.%20mysql%20client%20ip%20address.png)

### Step 2: Allow MySQL connection from the MySQL Client's IPv4 Address on the MySQL Server

* Click on the Instance ID of the **mysql_server** Instance.

![instance id mysql server](./images/2.%20instance%20id%20mysql_server.png)

* Click on the Security tab and then click on the Security group.

![security tab and group](./images/2.%20security%20tab%20group.png)

* Click on Edit Inbound Rules.

![edit inbound rules](./images/2.%20edit%20inbound%20rules.png)

* Click on the Add Rule button.

![add rule](./images/2.%20add%20rule.png)

* Select **MySQL/Aurora** as the connection type and paste the **MySQL-Client IPv4 address** you copied into the Custom IPv4 address box and click on the **save rules** button.

![select mysql and custom client ip address](./images/2.%20select%20mysql%20and%20custom%20ip%20address.png)


### Step 3: Install the MySQL-Server Software on the MySQL Server Linux Server

* Click on the Instance ID of the **mysql_server**.

![mysql server instance](./images/3.%20instance%20id%20mysql%20server.png)

* Click on the **Connect** button.

![connect button server](./images/3.%20connect%20mysql%20server.png)

* Copy the highlighted commands shown below:

![connect to instance server](./images/3.%20connect%20to%20instance%20server.png)

* Open your terminal.

* Go to the directory (i.e. /Downloads) where the `.pem` key pair is stored using the command shown below:

```bash
cd Downloads
```

![cd Downloads](./images/3.%20cd%20downloads%20server.png)

* Paste the following command to give read permissions to the `.pem` key pair file:

```bash
sudo chmod 400 <private-key-pair-name>.pem
```

![chmod pem_key](./images/3.%20chmod%20pem_key.png)

* SSH into the MySQL Server Instance using the command shown below:

```bash
ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>
```

![ssh pem_key](./images/3.%20ssh%20pem%20key.png)

* Update the list of packages in the package manager.

```bash
sudo apt update
```

![apt update](./images/3.%20sudo%20apt%20update%20server.png)

* Run the MySQL Server package installation.

```bash
sudo apt install mysql-server -y
```

![apt install mysql_server](./images/3.%20install%20mysql%20server.png)

In MySQL, the bind-address parameter in the `mysqld.cnf` file is used to specify the IP address on which the MySQL server should listen for incoming connections. Its default setting is `bind-address  = 127.0.0.1`.

To listen to all connections from all available network interfaces, the default bind-address parameter is changed to `0.0.0.0` (i.e. Default Gateway). 

However, it is best practice to set security precautions such as **Firewall Rules** and **MySQL User Privileges** to control incoming connections to the remote server. In our case, we set security precautions by configuring the **Inbound Rules** on the **MySQL TCP port** to only allow connections from the **MySQL Client Private IPv4 Address** on the **MySQL Server**.

* Run the following command to open the MySQL configuration file to change the **bind-address**.

```bash
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf 
```

![sudo vi sql conf.d](./images/3.%20vi%20:etc:mysql%20config%20file.png)

* Log into the MySQL console by running the command shown below:

```bash
sudo mysql
```

![mysql](./images/3.%20sudo%20mysql.png)

* Run a security script that sets the password (i.e. **PassWord.1**) for the root user using the following command:

```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```

![alter user](./images/3.%20ALTER%20USER.png)

* Exit the MySQL console using the following command:

```bash
mysql > exit;
```

![exit mysql](./images/3.%20exit.png)

* Enable the MySQL service using the following command:

```bash
sudo systemctl enable mysql
```

![systemctl enable mysql](./images/3.%20systemctl%20enable%20mysql.png)

* Run the following command to check if the MySQL service is running:

```bash
sudo systemctl status mysql
```

![systemctl status mysql](./images/3.%20systemctl%20status%20mysql.png)

From the image above, it is evident that the MySQL service is running.

* Run the following command to start the interactive script to improve the security of the MySQL Server installation:

```bash
sudo mysql_secure_installation
```

![mysql secure installation](./images/3.%20mysql%20secure%20installation.png)

The script will give the following prompts and your responses should be as follows:

1. Validate the password `y`.

![validate password](./images/3.%20validate%20password.png)

2. Select the level of password validation policy that matches the password you set initially, in this case, **2** matches the password I chose hence enter `2`.

![password validation policy](./images/3%20password%20validation%20policy.png)

3. Change the root password `n`.

![change root password](./images/3.%20change%20the%20root%20password.png)

4. Remove anonymous users `n`.

![remove anonymous users](./images/3.%20remove%20anonymous%20users.png)

5. Disallow root login remotely `n`.

![disallow root login](./images/3.%20disallow%20root%20login%20remotely.png)

6. Remove the test database and access to it `y`.

![remove test database](./images/3.%20remove%20test%20database.png)

7. Reload privilege tables `y`.

![reload privilege tables](./images/3.%20reload%20privilege%20tables.png)

* Log into the MySQL console with the following command:

```bash
sudo mysql -u root -p
```

![sudo mysql root](./images/3.%20sudo%20mysql%20-u%20root%20-p.png)

* Run the following command to create a user named **donald** with the password `PassWord.1`:

```bash
CREATE USER 'donald'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```

![create user](./images/3.%20create%20user%20donald.png)

*Note that the `%` wildcard after the `@` sign is used to represent any host. This means the user "donald" is allowed to connect to the MySQL Server from any host.*

* Run the following command to create a database called **testing_123**:

```bash
CREATE DATABASE testing_123;
```

![create database](./images/3.%20create%20database.png)

* Run the following command to grant all privileges to the user **donald**:

```bash
GRANT ALL PRIVILEGES ON *.* TO 'donald'@'%' WITH GRANT OPTION;
```

![grant all privileges](./images/3.%20grant%20all%20privileges.png)

*Note that the `*.*` wildcard means all databases and tables. Hence, the command above gives the user **donald** administrative privileges on all databases and tables.*

* Run the following command to apply and make changes effective:

```bash
FLUSH PRIVILEGES;
```

![flush privileges](./images/3.%20flush%20privileges.png)

* Exit the MySQL console.

![exit](./images/3.%20exit.png)

* Restart the MySQL service using the command shown below:

```bash
sudo systemctl restart mysql
```

![restart mysql](./images/3.%20systemctl%20restart%20mysql.png)

### Step 4: Install the MySQL-Client Software on the MySQL Client Linux Server

* Click on the Instance ID of the **mysql_client**.

![client instance id](./images/4.%20instance%20id%20client.png)

* Click on the **Connect** button.

![connect client](./images/4.%20connect%20button.png)

* Copy the highlighted command shown below:

![connecting to instance client](./images/4.%20connect%20to%20instance%20client.png)

* Open another terminal on your computer.

* Go to the directory (i.e. /Downloads) where the `.pem` key pair is stored using the command shown below:

```bash
cd Downloads
```

![cd Downloads](./images/4.%20cd%20downloads.png)

* SSH into the MySQL Client Instance using the command shown below:

```bash
ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>
```

![ssh client](./images/4.%20ssh%20pem.png)

* Update the list of packages in the package manager.

```bash
sudo apt update
```

![apt update](./images/4.%20sudo%20apt%20update.png)

* Run the MySQL Client package installation.

```bash
sudo apt install mysql-client -y
```

![install mysql client](./images/4.%20sudo%20apt%20install%20mysql%20client.png)

### Step 5: Remotely connect to the MySQL Server from the MySQL Client

* On the terminal of the MySQL Server, run the following command to generate the **Private IP address** of the MySQL Server:

```bash
hostname -i
```

![hostname -i](./images/5.%20hostaname%20i.png)

* Copy the IP address of the MySQL Server shown above.

* On the terminal of the MySQL Client, run the following command to connect to the MySQL Server:

```bash
sudo mysql -u <username_of_mysql_server> -h <ip_address_of_mysql_server> -p
```

![sudo mysql user h server](./images/5.%20sudo%20mysql%20-u%20username%20-h%20ip_address.png)

* To check that you have successfully connected to the remote MySQL Server, run the following query:

```bash
SHOW DATABASES;
```

![show database](./images/5.%20show%20databases.png)

From the image above, you can see the **testing_123** database you created on the MySQL Server. Hence, the connection to the remote **MySQL Server** from the **MySQL Client** was successful.
