
# Migration to the Сloud with Containerization
Virtual Machines (i.e. AWS EC2) can be used to deploy applications and web solutions. They are scalable to some extent but if you need to deploy many small applications and some of the applications will require various OS and runtimes of different versions and conflicting dependencies. In such cases, you will need to spin up servers for each group of applications with the exact OS and dependencies requirements.

When it scales out to tens/hundreds and even thousands of applications (i.e. Microservices), this approach becomes very tedious and challenging to maintain.

This project focuses on how to solve this problem with the use of the  the technology that revolutionized application distribution and deployment back in 2013! We are talking of Containers and simply [Docker](https://en.wikipedia.org/wiki/Docker_(software)). Even though there are other application containerization technologies, Docker is the standard and the default choice for shipping your app in a container!

## Prerequisites
A virtual machine (i.e. any cloud provider) or your workstation that will host the Docker containers.

## Migration to the Сloud with Containerization using Docker

The following steps are taken to migrate to the cloud with containerization using docker:

## Step 1: Install Docker and prepare for migration to the Cloud

1. Write a shell script to install Docker.

```sh
cat <<EOF | tee docker.sh
#!/bin/bash
sudo apt-get update
sudo apt-get -y install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo usermod -aG docker $USER

newgrp docker
EOF
```

2. Run the shell script.

```sh
bash docker.sh
```

![bash docker.sh](./images/0%20run%20the%20docker%20script.png)

3. Verify the installation.

```sh
docker ps
```

![verify installation](./images/0%20verify%20docker%20installation.png)

## Step 2: MySQL in a Container
We are migrating the Tooling Web Application (**PHP-based web solution backed by a MySQL database**) from a VM-based solution into a containerized one.

1. Pull the MySQL Docker Image from [Docker Hub Registry](https://hub.docker.com/_/mysql). You can download a specific version or opt for the latest release, as seen in the following command:

```sh
docker pull mysql/mysql-server:latest
```

![docker pull mysql](./images/1%20docker%20pull%20mysql.png)

2. List the images to check that you have downloaded them successfully.

```sh
docker images
```

![docker images](./images/1%20docker%20images.png)

## Step 3: Deploy the MySQL Container to your Docker Engine

1. Once you have the image, move on to deploying a new MySQL container.

```sh
docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d mysql/mysql-server:latest
```

* Replace `<container_name>` with the name of your choice. If you do not provide a name, Docker will generate a random one.

* The `-d` option instructs Docker to run the container as a service in the background.

* Replace `<my-secret-pw>` with your chosen password.

* In the command above, we used the latest version tag. This tag may differ according to the image you downloaded.

![docker run container](./images/2%20docker%20run%20container.png)

2. Check to see if the MySQL container is running: Assuming the container name specified is `mysql-server`.

```sh
docker ps -a
```

![docker ps -a](./images/3%20docker%20ps%20-a.png)

You should see the newly created container listed in the output. It includes container details, one being the status of this virtual environment. The status changes from `health: starting` to `healthy`, once the setup is complete.

## Step 4: Connecting to the MySQL Docker Container

We can either connect directly to the container running the MySQL server or use a second container as a MySQL client. Let us see what the first option looks like.

#### Approach 1

Connecting directly to the container running the MySQL server.

```sh
docker exec -it <container_name> mysql -uroot -p
```

![docker exec mysql](./images/2%20docker%20exec%20mysql.png)

#### Approach 2

First, create a network.

```sh
docker network create --subnet=172.18.0.0/24 tooling_app_network 
```

![docker network create](./images/3%20docker%20network%20create.png)

Creating a custom network is not necessary because even if we do not create a network, Docker will use the default network for all the containers you run. By default, the network we created above is of `DRIVER Bridge`. So, also, it is the default network. You can verify this by running the `docker network ls` command.

But there are use cases where this is necessary. For example, if there is a requirement to control the `cidr` range of the containers running the entire application stack. This will be an ideal situation to create a network and specify the `--subnet`.

For clarity's sake, we will create a network with a subnet dedicated for our project and use it for both MySQL and the application so that they can connect.

1. Run the MySQL Server container using the created network.

2. Create an environment variable to store the root password.

```sh
export MYSQL_PW=<root-secret-password>
```

![export mysql](./images/3%20export%20mysql.png)

3. Pull the image and run the container, all in one command like below:

```sh
docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql/mysql-server:latest 
```

Flags used:
* `-d` runs the container in detached mode
* `--network` connects a container to a network
* `-h` specifies a hostname

**Note**: If the image is not found locally, it will be downloaded from the registry.

4. Verify the container is running.

```sh
docker ps -a
```

![docker ps -a](./images/3%20docker%20ps%20-a.png)

As you already know, it is best practice not to connect to the MySQL server remotely using the root user. Therefore, we will create an **SQL** script that will create a user we can use to connect remotely.

5. Create a file and name it `create_user.sql` and add the below code in the file:

```sh
CREATE USER '<user>'@'%' IDENTIFIED BY '<client-secret-password>';
GRANT ALL PRIVILEGES ON * . * TO '<user>'@'%';
```

![create_user](./images/3%20create_user_sql.png)

6. Run the script.

```sh
docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < ./create_user.sql
```

![docker exec -i mysql-server](./images/3%20docker%20exec%20-i%20%20mysql-server.png)

If you see a warning like below, it is acceptable to ignore:

```sh
mysql: [Warning] Using a password on the command line interface can be insecure.
```

## Step 5: Connecting to the MySQL Server from a Second Container running the MySQL Client Utility

The good thing about this approach is that you do not have to install any client tool on your laptop and you do not need to connect directly to the container running the MySQL server.

1. Run the MySQL Client Container:

```sh
docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u <user-created-from-the-SQL-script> -p
```

Flags used:
* `--name` gives the container a name
* `-it` runs in interactive mode and Allocate a pseudo-TTY
* `--rm` automatically removes the container when it exits
* `--network` connects a container to a network
* `-h` a MySQL flag specifying the MySQL server Container hostname
* `-u` user created from the SQL script
* `-p` password specified for the user created from the SQL script

![docker run network tooling-app-networ](./images/4%20docker%20run%20network%20tooling%20app%20network.png)

2. Clone the Tooling-app repository from [here](https://github.com/darey-devops/tooling).

```sh
git clone https://github.com/darey-devops/tooling.git
```

![git clone tooling](./images/4%20git%20clone%20tooling.png)

3. On your terminal, export the location of the SQL file.

```sh
export tooling_db_schema=~/tooling/html/tooling_db_schema.sql
```

![export tooling](./images/4%20export%20tooling_db.png)

You can find the `tooling_db_schema.sql` in the `html` folder of cloned repo.

4. Use the SQL script to create the database and prepare the schema. With the docker exec command, you can execute a command in a running container.

```sh
docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema
```

![docker exec -i mysql-server](./images/4%20docker%20exec%20-i%20mysql-server.png)

4. Update the `db_conn.php` file with connection details to the database.

```sh
vi tooling/html/db_conn.php
```

```sh
$servername = "mysqlserverhost";
$username = "<user>";
$password = "<client-secret-password>";
$dbname = "toolingdb";
```

![db_conn1](./images/4%20db_conn%201.png)
![db_conn2](./images/4%20db_conn2.png)
_Updated `db_conn.php` file._

5. Go into the tooling directory.

```sh
cd tooling && ll
```

![cd tooling && ll](./images/4%20cd%20tooling%20&&%20ll.png)

6. Build your image.

```sh
docker build -t tooling:0.0.1 .
```

![docker build -t tooling](./images/5%20docker%20build%20-t%20tooling.png)

7. Run the container.

```sh
docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1
```

![docker run container](./images/5%20docker%20run%20container.png)

8. If everything works, you can open the browser and type:

```sh
http://localhost:8085
```

![localhost_8085](./images/5%20localhost_8085.png)

9. Input your name and password used when updating the `db_conn.php` file.

![localhost_8085_1](./images/5%20localhost_8085_1.png)
![localhost_8085_2](./images/5%20localhost_8085_2.png)

## Step 6: Implement a POC to migrate PHP-TODO app into a Containerized Application

1. Create a network.

```sh
docker network create --subnet=172.18.0.0/24 my_todo_network
```

![docker network create](./images/6%20docker%20network%20create%20my_todo_network.png)

2. Run the MySQL Server container using the created network.

```sh
docker run --network my_todo_network -h mysqlserverhost --name mysql-todo -d mysql/mysql-server:latest
```

![docker run network my-todo-network](./images/6%20docker%20run%20network%20my_todo_network%20mysqlserverhost.png)

Flags used:
* `-d` runs the container in detached mode.
* `--network` connects a container to a network.
* `-h` specifies a hostname.
* `--name `gives a custom name to the container.

```sh
docker ps
```

![docker ps](./images/6%20docker%20ps.png)

3. Once initialization is finished, the command's output is going to contain the random password generated for the root user; check the password with this command:

```sh
docker logs mysql-todo 2>&1 | grep GENERATED
```

![docker logs mysql-todo](./images/6%20docker%20logs%20mysql.png)

## Step 7: Connecting to MySQL Server from within the Container

Run the `mysql client` within the `MySQL Server` container you just started and connect it to the MySQL Server.

1. Use the `docker exec -it` command to start a mysql client inside the Docker container you have started, like this:

```sh
docker exec -it mysql-todo mysql -uroot -p
```

![docker exec -it mysql-todo](./images/7%20docker%20exec%20-it%20mysql-todo.png)

2. When asked, enter the generated root password. Because the **MYSQL_ONETIME_PASSWORD** option is true by default, after you have connected a mysql client to the server, you must reset the server root password by issuing this statement:

```sh
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';
```

![alter user](./images/7%20alter%20user.png)

3. Create the user and the database.

```sh
CREATE USER 'ikenna'@'%' IDENTIFIED BY 'password1'; GRANT ALL PRIVILEGES ON *.* TO 'ikenna'@'%';
```

![create user](./images/7%20create%20user%20grant%20all.png)

4. Create the `tododb` database.

```sh
CREATE DATABASE tododb;
```

![create database tododb](./images/7%20create%20database%20tododb.png)

5. Exit the console.

```sh
exit
```

![exit](./images/7%20exit.png)

6. Clone the PHP-Todo-app repository from [here](https://github.com/darey-devops/php-todo).

7. Go into the `php-todo` directory.

```sh
cd php-todo
```

8. Create an `.env` file with connection details to the database.

```sh
cat <<EOF | tee .env
APP_ENV=local
APP_DEBUG=true
APP_KEY=SomeRandomString
APP_URL=http://localhost:8000

DB_HOST=mysqlserverhost
DB_DATABASE=tododb
DB_USERNAME=ikenna
DB_PASSWORD=password1
DB_CONNECTION=mysql
DB_PORT=3306

CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_DRIVER=sync

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_DRIVER=smtp
MAIL_HOST=mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
EOF
```

![.env](./images/8%20cat%20_env.png)

9. Create an application script for your application.

```sh
cat <<EOF | tee app.sh
#!/bin/bash

php artisan migrate

php artisan key:generate

php artisan cache:clear

php artisan config:clear

php artisan route:clear

php artisan serve  --host=0.0.0.0
EOF
```

![app.sh](./images/8%20cat%20app.png)

10. Create a Dockerfile to build the image.

```sh
cat <<EOF | tee Dockerfile
# Use an official PHP image as a parent image
FROM php:7.4-cli

USER root

# ENV DB_HOST=
# ENV DB_DATABASE=
# ENV DB_USERNAME=
# ENV DB_PASSWORD=
ENV COMPOSER_ALLOW_SUPERUSER=1


# Set the working directory in the container
WORKDIR /var/www/html

# Install dependencies (including Composer)
RUN apt-get update && apt-get install -y     libpng-dev     zlib1g-dev     libxml2-dev     libzip-dev     libonig-dev     zip     curl     unzip     && docker-php-ext-configure gd     && docker-php-ext-install -j2 gd     && docker-php-ext-install pdo_mysql     && docker-php-ext-install mysqli     && docker-php-ext-install zip     && docker-php-source delete     && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Copy the rest of the application code into the container
COPY . /var/www/html

# Install Laravel dependencies
RUN composer install

# Expose port 8000 for the Laravel development server
EXPOSE 8000

# Define the command to start the Laravel development server
ENTRYPOINT [ "bash", "app.sh" ]
EOF
```

![dockerfile](./images/8%20cat%20dockerfile.png)

11. Build the Docker image the Todo app will use.

```sh
docker build -t todo-app-image:1.0.0 .
```

12. Run the container.

```sh
docker run --network my_todo_network --name todo-app-coantainer -p8000:8000 -d todo-app-image:1.0.0
```

![docker run my_todo_network](./images/8%20docker%20run%20my_todo_network.png)

Let us observe those flags in the command:

We need to specify the `--network` flag so that both the **Todo app** and the **Database** can easily connect on the same virtual network we created earlier. 

The `-p` flag is used to map the container port with the host port. Within the container, Laravel is the webserver running and by default, it listens on port `8000`. 

We used port `8000` directly on our host because it is not in use yet. If it was in use, the workaround is to use another port that is not used by the host machine, so we can then map that to port `8000` running in the container (_could somethong like -p 8085:8000_).

13. Access the web browser.

```sh
http://ip_host_vm:8000
```

![ip_host_8000](./images/8%20localhost_8000.png)

## Step 8: Push Docker Image To Container Repository

1. Please make sure you have a [Docker Hub account](https://hub.docker.com/signup/awsedge) before you proceed.

2. Create a public `php-todo-app` repository on Docker Hub.

![create php app repo](./images/9%20create%20php-todo-app%20repo.png)

3. Log into Docker Hub from your VM, input your email address and password in the prompt..

```sh
docker login
```

![docker login](./images/9%20docker%20login.png)

4. Tag your Docker Image.

```sh
ocker tag local-image:tagname DockerHubusername/repositoryname:tagname
```

![docker tag todo](./images/9%20docker%20tag%20todo-app-image.png)

5. Push the Docker Image.

```sh
docker push username/repositoryname:tagname
```

![docker push](./images/9%20docker%20push.png)

6. Check your Docker account to confirm it has been pushed suuccessfully

![confirmation](./images/9%20docker%20hub%20confirmation.png)
