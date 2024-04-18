# Ansible Automation Project
## Ansible Client as a Jump Server (Bastion Host)
A [Jump Server](https://en.wikipedia.org/wiki/Jump_server) (sometimes also referred to as [Bastion Host](https://en.wikipedia.org/wiki/Bastion_host)) is an intermediary server through which access to the internal network can be provided. If you think about the current architecture you are working on, ideally the Web Servers would be inside a secured network that cannot be reached directly from the Internet. That means DevOps engineers cannot `SSH` into the Web Servers directly and can only access it through a Jump Server. It provides better security and reduces [attack surface](https://en.wikipedia.org/wiki/Attack_surface).

In the diagram below, the Virtual Private Network (VPC) is divided into two subnets (i.e. Public Subnet has Public IP Addresses and Private Subnet is only reachable by Private IP addresses).

![bastion host](./images/0.%20bastion%20host.png)

## How to Install and Configure Ansible Client to act as a Jump Server/Bastion Host and Create a simple Ansible Playbook to automate server configuration

### Prerequistes
1. Jenkins Server
2. NFS Server
3. Web Server 1
4. Web Server 2
5. Load Balancer
6. Database Server

The following steps are taken to install and configure **Ansible Client** as a **Jump Server/Bastion Host** and create a simple Ansible Playbook to automate server configuration:

### Step 1: Install and Configure Ansible on an EC2 Instance

* Create a new repository called `ansible-config-mgt` in your GitHub account.

![ansible-config-mgt](./images/1.%20ansible-config-mgt.png)

* Update the `Name` tag on your `Jenkins` EC2 Instance to `Jenkins-Ansible`. This server will be used to run playbooks.

* Install Ansible on the `Jenkins-Ansible` server.

```sh
sudo apt update && sudo apt install ansible -y
```

![apt update & install ansible](./images/1.%20apt%20update%20&%20install%20ansible.png)

* Check the version of Ansible running on your instance by running the following command:

```sh
ansible --version
```

![ansible version](./images/1.%20ansible%20version.png)

### Step 2: Configure Jenkins Build Job to archive your repository every time changes are made

* Log into Jenkins.

```sh
http://public_ip_jenkins_ansible_instance:8080
```

![log into jenkins](./images/2.%20log%20into%20jenkins.png)

* Create a new Freestyle Job called `ansible`, select **discard old builds** then give it a maximum of 2 builds to keep and point it to your `ansible-config-mgt` repository.

![ansible job1](./images/2.%20ansible%20freestyle%20job1.png)

![ansible job2](./images/2.%20ansible%20freestyle%20job2.png)

![ansible job3](./images/2.%20ansible%20freestyle%20job3.png)

* Select GitHub hook trigger for GitScm polling and configure a Post-Build Job (i.e. Archive the artifacts) to save all `**` files then click on apply and save.

![github trigger1](./images/2.%20github%20hook%20trigger1.png)

![github trigger2](./images/2.%20github%20hook%20trigger2.png)

### Step 3: Configure a webhook in GitHub and set the webhook to trigger Ansible build

* Go to the `ansible-config-mgt` repository on your GitHub account and click on settings.

![ansible config mgt repo settings](./images/3.%20ansible-config-mgt%20repository.png)

* Click on the webhooks tab.

![webhooks tab](./images/3.%20webhooks%20tab.png)

* Click on `Add Webhook`

![add webhook](./images/3.%20add%20webhook.png)

* Input your password.

![input password](./images/3.%20input%20password.png)

* In the **Payload URL**, paste the following URL shown below and click on the **Add Webhook** button:

```sh
http://private_ip_address_jenkins_ansible_server:8080/github-webhook/
```

![payload url](./images/3.%20payload%20url.png)

* Test the setup by making changes to the README.md file in the `main` branch and make sure it starts a build automatically as shown below:

![test the setup](./images/3.%20test%20setup.png)

Your set up will look like this:
![initial setup](./images/1.%20initial%20set%20up.png)

_**Note**: Every time you start/stop your `Jenkins-Ansible` server, you have to reconfigure GitHub webhook to a new IP address in order to avoid it, it makes sense to allocate an [Elastic IP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html) to your `Jenkins-Ansible` server. However, Elastic IP is only free when it is allocated to an EC Instance, so do not forget to release the Elastic IP once you terminate your EC2 Instance._

### Step 4: Prepare your development environment using Visual Studio Code

* Download and install VS Code, you can get it [here](https://code.visualstudio.com/download)

* After successfully installing VS Code, configure it to connect to your newly created GitHub repository.

* Clone down the `ansible-config-mgt` repository to your local machine.

```sh
git clone <ansible-config-mgt-repository-link>
```

![git clone](./images/4.%20git%20clone.png)

* Go into the `ansible-config-mgt` directory and pull the repo to ensure the directory is up to date.

```sh
cd ansible-config-mgt && git pull
```

![cd ansible config mgt & git pull](./images/4.%20cd%20ansible%20repo%20&%20git%20pull.png)

### Step 5: Begin Ansible development and Set up an Ansible Inventory

* Create a new branch in the `ansible-config-mgt` repository that will be used for the development of a new feature using the command shown below:

```sh
git checkout -b prj-145
```

![git checkout prj-145](./images/5.%20git%20checkout%20-b%20prj-145.png)
_Note that running the above command will create a new branch and switch to the new branch._

* Create a `playbooks` (i.e. used to store all the playbook files) and `inventory` (i.e. used to keep your hosts organized) directory.

```sh
mkdir playbooks inventory
```

![mkdir playbooks inventory](./images/5.%20mkdir%20playbooks%20inventory.png)

* Create your first playbook named `common.yml` in the playbooks directory.

```sh
cd playbooks && touch common.yml
```

![cd playbooks & touch common.yml](./images/5.%20cd%20playbooks%20&%20touch%20common.png)

* Create inventory files for each environment (i.e. Development, Staging, Testing and Production) in the inventory directory.

```sh
cd .. && cd inventory && touch dev staging uat prod
```

![cd inventory & touch dev staging uat prod](./images/5.%20cd%20&%20cd%20inventory%20&%20touch%20dev%20staging%20uat%20prod.png)

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules and tasks in a playbook operate. Since we intend to execute Linux commands on remote hosts and ensure that it is the intended configuration particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

* Update your `inventory/dev.yml` file with the code shown below:

```sh
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```

![update inventory/dev.yml](./images/5.%20update%20inventory:dev.png)

### Step 6: Setup an SSH-Agent and Connect VS Code to your Jenkins-Ansible Instance

Ansible uses TCP port 22 by default which means it needs to `SSH` into target servers from `Jenkins-Ansible` Server. To achieve this, implement the concept of [SSH-Agent](https://smallstep.com/blog/ssh-agent-explained/#:~:text=ssh%2Dagent%20is%20a%20key,you%20connect%20to%20a%20server.&text=It%20doesn't%20allow%20your%20private%20keys%20to%20be%20exported.).

The following steps are taken to setup the ssh-agent and connect VS Code to your `Jenkins-Ansible`:

1. On your VS Code terminal, run the following command to start up the `SSH-Agent`:

```sh
eval `ssh-agent -s`
```

![eval ssh-agent](./images/6.%20eval%20ssh-agent%20-s.png)

2. Run the following command to add the private key (i.e. keypair used to create the Jenkins-Ansible Instance) to the SSH-Agent:

```sh
ssh-add <path_to_the_private_key_of_jenkins_ansible_instance>
```

![ssh-add keypair](./images/6.%20ssh-add%20keypair.png)

3. Confirm that the key has been added to the SSH-Agent using the command shown below:

```sh
ssh-add -l
```

![ssh-add -l](./images/6.%20ssh-add%20-l.png)

4. SSH into your `Jenkins-Ansible` server using SSH-Agent.

```sh
ssh -A ubuntu@public_ip_address_of_jenkins_ansible
```

![ssh -A ubuntu@public_ip](./images/6.%20ssh%20-A%20ubuntu@public_ip_jenkins_ansible.png)
_Note that your Load Balancer server user is `ubuntu` while the Database, Web and NFS Server's user is `ec2-user` since they are **RHEL-based servers**._


### Step 7: Set up an SSH-Agent on the Jenkins-Ansible Instance so it will be able to connect to the other servers

* Run the following command to start up the `SSH-Agent`

```sh
eval `ssh-agent -s`
```

![eval ssh-agent -s](./images/7.%20eval%20ssh-agent%20-s.png)

* Create a file similar to the keypair file used to SSH into your instance and paste the content of the keypair file.

```sh
vi web11.pem
```

![cat web11.pem](./images/7.%20web11_pem.png)
_The contents of this keypair file will be pasted into the `web11.pem` file on the **Jenkins-Ansible** Server._

* Give write permissions to the file and add the newly created file into the SSH-Agent using the commands shown below:

```sh
chmod 400 web11.pem
ssh-add web11.pem
```

![chmod ssh-add web11.pem](./images/7.%20chmod%20&%20ssh-add%20web11.png)
_Note that if you used different keypairs when provisioning your servers (i.e. NFS, Database, Load Balancers and Web), you must create files that match the keypairs and add them to the ssh-agent._

### Step 8: Create a common playbook

It is time to start giving Ansible the instructions on what you need to perform on all servers listen in `inventory/dev`. In the `common.yml` playbook, you will write configurations for repeatable, reusable and multi-machine tasks that are common to systems with the infrastructure.

* Update your `playbooks/common.yml` file with the following code:

```sh
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  become: yes
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest
   

- name: update LB server
  hosts: lb
  become: yes
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```

![playbooks/common.yml](./images/8.%20playbooks:common.png)

The code above has two plays, the **first play** is dedicated to the **RHEL Servers** (i.e. NFS, Database, Web). The task will be performed as the `root` user hence the use of the **become: yes** key value pair. Since they are all **RHEL Based Servers**, the `yum module` is used to install `wireshark` and finally the **state: latest** key-value pair is used to specify that the wireshark installed is the latest version.

The **second play** is dedicated to the **Ubuntu Server** (i.e. Load Balancer), it has two tasks: The **first task** is used to update the server. The `update_cahce` is similar to `apt update` and the **second task** is used to download the latest version of `wireshark` using the `apt module`.

### Step 9: Update GIT with the latest code

Now that all of your directories and files live on your local machine, you need to push changes made locally to GitHub. Remember you have been working on a separate branch `prj-145`, you need to get your branch peer-reviewed and pushed to the `main` branch. The following steps are taken to achieve this:

* Use the following commands to check the status of your branch, add files and directories then commit changes and push your branch to GitHub:

```sh
git status
```

![git status](./images/9.%20git%20status.png)

```sh
git add inventory playbooks
```

![git add inventory playbooks](./images/9.%20git%20add%20inventory%20playbooks.png)

```sh
git commit -m "commit message"
```

![git commit](./images/9.%20git%20commit.png)

```sh
git push --set-upstream origin prj-145
```

![git push branch](./images/9.%20git%20push%20branch.png)

* Go to your `ansible-config-mgt` repository on GitHub and click on the `Compare & pull request` button.

![compare & pull request](./images/9.%20compare%20&%20pull%20request.png)

* Click on the `Create pull request` button.

![create pull request](./images/9.%20create%20pull%20request.png)

* Click on the `Merge pull request` button.

[merge pull request](./images/9.%20merge%20pull%20request.png)

* Click on the `Confirm merge` button.

![confirm merge](./images/9.%20confirm%20merge.png)

* Head back to your terminal on VS Code, checkout from `prj-145` branch into the main and pull down the latest changes using the commands shown below:

```sh
git checkout main && git pull
```

![git checkout main && git pull](./images/9.%20git%20checkout%20main%20&%20git%20pull.png)

* Once your code changes appear on the `main` branch, Jenkins will be triggered to do the `ansible` job and save all the files (i.e. build artifacts).

![build artifacts](./images/9.%20main%20branch%20triggered%20job.png)


### Step 10: Run the first Ansible test

* SSH into the `Jenkins-Ansible` server.

```sh
ssh ubuntu@public_ip_address_of_jenkins_ansible
```

![ssh jenkisn-ansible server](./images/10.%20ssh%20jenkins-ansible%20server.png)

* The build artifacts are saved in the `/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` directory on the server.

```sh
cd /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ && ll
```

![cd /var/lib/jenkins/jobs/builds & ll](./images/10.%20cd%20:var:lib:jenkins:jobs%20&%20ll.png)

* Run the following command to run the Ansible Playbook.

```sh
ansible-playbook -i inventory/dev playbooks/common.yml
```

![run asnible playbook](./images/10.%20run%20ansible%20playbook.png)

* Use the Ansible Adhoc command to check if wireshark has been installed on the servers.

```sh
ansible webservers -i inventory/dev -m command -a "wireshark --version"
```

![ansible adhoc1](./images/10.%20ansible%20adhoc1.png)

```sh
ansible nfs,db -i inventory/dev -m command -a "wireshark --version"
```

![ansible adhoc2](./images/10.%20ansible%20adhoc2.png)

```sh
ansible lb -i inventory/dev -m command -a "wireshark --version"
```

![ansible adhoc3](./images/10.%20ansible%20adhoc3.png)

Your updated Ansible architecture now looks like this:
![final setup](./images/11.%20final%20set%20up.png)