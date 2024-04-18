# Implementing Load Balancers with Nginx
## Introduction to Load Balancing and Nginx
Load balancing refers to efficiently distributing incoming network traffic across a group of backend servers, also known as a server farm or server pool. 

Modern high‑traffic websites must serve hundreds of thousands, if not millions, of concurrent requests from users or clients and return the correct text, images, video, or application data, all in a fast and reliable manner. To cost‑effectively scale to meet these high volumes, modern computing best practice generally requires adding more servers.

A load balancer acts as the “traffic cop” sitting in front of your servers and routing client requests across all servers capable of fulfilling those requests in a manner that maximizes speed and capacity utilization and ensures that no one server is overworked, which could degrade performance. If a single server goes down, the load balancer redirects traffic to the remaining online servers. When a new server is added to the server group, the load balancer automatically starts to send requests to it.

![load balancer](./images/0.%20Load%20Balancer.png)

In this manner, a load balancer performs the following functions:
* Distributes client requests or network load efficiently across multiple servers.
* Ensures high availability and reliability by sending requests only to servers that are online.
* Provides the flexibility to add or subtract servers as demand dictates.

Nginx is a versatile software, it can act like a web server, reverse proxy and a load balancer depending on configuration.

## Implementing Nginx as a Basic Load Balancer between Two Web Servers

### Prerequisite
1. Ensure you have an AWS account. If you don't have an account, [sign up for AWS here.](https://portal.aws.amazon.com/billing/signup?type=enterprise#/start/email)

The following steps are taken to implement Nginx as a basic load balancer between two web servers:

### Step 1: Provisioning the 1st Apache Web Server
* Open your AWS Management Console, click on EC2.

![aws console](./images/1.%20aws%20mangement%20console.png)

* Click on the Launch Instance button.

![lauch instance](./images/1.%20launch%20instance.png)

* On the Name Box and Amazon Machine Image, type **Apache_Web_Server_01** and **ubuntu** respectively.

![name box ami](./images/1.%20name%20box%20and%20ami.png)

* Select **Ubuntu Server 22.04 LTS (HVM), SSD Volume Type** as the Amazon Machine Image.

![ubuntu server](./images/1.%20ubuntu%20server%2022.png)

* Click on create new key pair.

![key pair](./images/1.%20new%20key%20pair.png)

* Give the key pair name a name of your choice (i.e web11), select `RSA` as the key pair type and `.pem` as the key file format then click on Create key pair.

![rsa pem](./images/1.%20pem%20and%20rsa.png)

*Note that the `.pem` key will be downloaded into your Downloads directory on your computer*.

* On the Network Settings tab, click on the Edit button to configure the Security Group Inbound Rules.

![edit security group](./images/1.%20edit%20security%20group.png)

* Click on Add Security Group Rule on the bottom and the Security Group Name, Source Type and Port Range type the following: **Apache_Server_Security_Group**, **Anywhere** and **8000** respectively.

![security group parameters](./images/1.%20security%20group%20parameters.png)

* Scroll down to the bottom and click on the Launch Instance Button.

![lauch instance button](./images/1.%20launch%20instance%20button.png)

* You will see a prompt shown below, click on the Instance ID highlighted.

![instance prompt](./images/1.%20prompt%20instance%20id.png)

* Click on the Instance ID of the **Apache_Web_Server_01** Instance you just created.

![instance id](./images/1.%20instance%20id.png)

* Click on the Connect button.

![connect button](./images/1.%20connect%20button.png)

* Copy the highlighted commands shown below to connect to the Instance:

![highlighted commands](./images/1.%20highlighted%20commands.png)

* Open your terminal.

* Go to the Downloads directory (i.e `.pem` key pair is stored here) using the command shown below:

```sh
cd Downloads
```

![cd downloads](./images/1.%20cd%20downloads.png)

* Run the following command to give read permissions to the `.pem` key pair file:

```sh
chmod 400 <private-key-pair-name>.pem
```

![chmod keypair](./images/1.%20chmod%20keypair.png)

* SSH into the **Apache_Web_Server_01** Instance using the command shown below:

```sh
ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>
```

![ssh apache instance](./images/1.%20ssh%20apache%20instance.png)

* Update the list of packages in the package manager and install the apache server package installation using the following command:

```sh
sudo apt update && sudo apt install apache2 -y
```

![update install apache](./images/1.%20sudo%20apt%20update%20&%20apt%20install%20apache2.png)

* Verify that Apache is running using the command shown below:

```sh
sudo systemctl status apache2
```

![systemctl status apache](./images/1.%20systemctl%20status%20apache2.png)

### Step 2: Configuring the 1st Apache Server to Serve Content on Port 8000

*  Run the following command to open the Apache listening ports configuration file:

```sh
sudo vi /etc/apache2/ports.conf
```

* Add a new Listen directive for port 8000 as shown below and save the file:

![vi ports.conf](./images/2.%20vi%20:etc:apache2:ports.png)

* Run the following command to open the default configuration file of Apache.

```sh
sudo vi /etc/apache2/sites-available/000-default.conf
```

* Change the listening port of the Virtualhost from 80 to 8000 as shown below and save the file:

![vi default.conf](./images/2.%20vi%20:etc:apache2:sites-available.png)

* Reload the Apache Web Server to load the new configuration changes using the command shown below:

```sh
sudo systemctl reload apache2
```

![systemctl reload apache](./images/2.%20systemctl%20reload%20apache2.png)

### Step 3: Creating a Webpage for the 1st Apache Web Server 

* Create and open a new **index.html** file with the command shown below:

```sh
sudo vi index.html
```

* Before pasting the html code block shown below, get the **Public IPv4 address** of the **Apache_Web_Server_01** by clicking the Instance ID of the Instance and copying the address.

```sh
        <!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my Apache Web Server 1 EC2 instance</h1>
            <p>Public IP: YOUR_PUBLIC_IP</p>
        </body>
        </html>
```

![index html](./images/3.%20index_html.png)

* Change the file ownership of the **index.html** file so that the Nginx Load Balancer Server can have access to the file using the command shown below:

```sh
sudo chown www-data:www-data ./index.html
```

![chown www-data index.html](./images/3.%20chown%20index_html.png)

* Overwrite the default html file of the Apache Web Server by using the command shown below:

```sh
sudo cp -f ./index.html /var/www/html/index.html
```

![cp index.html](./images/3.%20cp%20-f%20index_html.png)

* Reload the Apache Web Server to load the new configuration changes using the command shown below:

```sh
sudo systemctl reload apache2
```

![systemctl reload apache](./images/2.%20systemctl%20reload%20apache2.png)

* Go to your your browser and paste the following URL:

```sh
http://<public_ip_address_of_apache_web_server_1>:8000
```

![http ip url1](./images/3.%20http%20ip%208000.png)

_The Web Page Should Look Like This_

### Step 4: Provisioning and Configuring the 2nd Apache Web Server

Repeat steps 1 - 3 but ensure the following parameters are changed to these when implementing the steps:

1. Name of the Instance: Apache_Web_Server_02

2. Key pair: web11

3. Security Group: Apache_Server_Security_Group

4. When serving content for the 2nd Apache Web Server's webpage, use this as your html code block:

```sh
        <!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my Apache Web Server 2 EC2 instance</h1>
            <p>Public IP: YOUR_PUBLIC_IP</p>
        </body>
        </html>
```
![index html2](./images/4.%20index_html.png)

After completing the steps, go to your browser and paste the following URL:

```sh
http://<public_ip_address_of_apache_web_server_2>:8000
```

![http ip url2](./images/4.%20http%20ip%208000.png)

_The Web Page Should Look Like This_

### Step 5: Provisioning the Nginx Load Balancer Server

* On the Instances tab, click on the Launch Instance button.

* On the Name Box and Amazon Machine Image, type **Nginx Load Balancer Server** and **ubuntu** respectively.

![name box ami](./images/5.%20name%20box%20and%20ami.png)

* Select **Ubuntu Server 22.04 LTS (HVM), SSD Volume Type** as the Amazon Machine Image.

![ubuntu server 22](./images/5.%20select%20ubuntu%2022.png)

* Click on the key pair drop-down button and select **web11** as the key pair.

![keypair drop down](./images/5.%20keypair%20drop-down.png)

* Create a new security group and select allow HTTP traffic from the internet.

![create security group](./images/5.%20create%20security%20group.png)

* Click on the Launch Instance button.

![launch instance button](./images/5.%20launch%20instance%20button.png)

* You will see a prompt shown below, click on the Instance ID highlighted.

![prompt instance](./images/5.%20prompt%20instance%20id.png)

* Click on the Instance ID of the **Nginx Load Balancer Server** Instance you just created.

![instance id](./images/5.%20instance%20id.png)

* Click on the Connect button.

![connnect button](./images/5.%20connect%20button.png)

* Copy the highlighted command shown below to connect to the Instance:

![highlighted command](./images/5.%20highlighted%20command.png)

* Open your terminal.

* Go to the Downloads directory (i.e `.pem` key pair is stored here) using the command shown below:

```sh
cd Downloads
```

![cd downloads](./images/5.%20cd%20downloads.png)

* SSH into the **Nginx Load Balancer Server** Instance using the command shown below:

```sh
ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>
```

![ssh pem key](./images/5.%20ssh%20pem%20key.png)

* Update the list of packages in the package manager and install the apache server package installation using the following command:

```sh
sudo apt update && sudo apt install nginx -y
```

![update and install nginx](./images/5.%20sudo%20apt%20update%20&%20apt%20install%20nginx.png)

* Verify that Nginx is running using the command shown below:

```sh
sudo systemctl status nginx
```

![systemctl status nginx](./images/5.%20systemctl%20status%20nginx.png)

### Step 6: Configuring Nginx as the Load Balancer between the Two Apache Web Servers

* Run the following command to create and open a load balancer configuration file:

```sh
sudo vi /etc/nginx/conf.d/loadbalancer.conf
```

* Paste the code block shown below and save the file:

```sh
        upstream backend_servers {

            # your are to replace the public IP and Port to that of your web servers
            server 127.0.0.1:8000; # public IP and port for web server 1
            server 127.0.0.1:8000; # public IP and port for web server 2

        }

        server {
            listen 80;
            server_name <your_load_balancers_public_ip_addres>; # provide your load balancers public IP address

            location / {
                proxy_pass http://backend_servers;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
        }
```

![loadbalance conf](./images/6.%20vi%20:etc:nginx:confd:loadbalancer.png)

*The **upstream backend_servers** block defines a group of backend servers. The **server** directive lists the addresses and ports of your backend server. The **proxy_pass** directive ins the **location** block sets up load balancing and passes trequests to the backend servers. The **proxy_set_header** directives pass necessary headers to the backend servers to correctly handle requests.*

* Run the following command to test if the Nginx Load Balancer Server's configuration was successful:

```sh
sudo nginx -t
```

![nginx -t](./images/6.%20sudo%20nginx%20-t.png)

* Reload the Nginx Load Balancer Server to load the new configuration changes using the command shown below:

```sh
sudo nginx -s reload
```

![nginx -s reload](./images/6.%20sudo%20nginx%20-s%20reload.png)

* Go to your browser and paste the URL shown below:

```sh
http://<public_ip_address_of_nginx_load_balancer_server>:80
```

![http ip1](./images/6.%20http%20url1.png)

![http ip2](./images/6.%20http%20url2.png)

_You will notice that each time you refresh the page, it displays the content of one of the two **Apache Web Servers**. Note that the **Load Balancing Algorithm** used in configuring the **Nginx Load Balancer Server** is the **Round Robin Algorithm**._