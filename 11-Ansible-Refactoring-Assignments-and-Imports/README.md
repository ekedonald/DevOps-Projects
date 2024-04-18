# Ansible Refactoring, Assignments & Imports
[Refactoring](https://en.wikipedia.org/wiki/Code_refactoring) means making changes to the source code without changing the expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity and add proper comments without affecting the logic.

In this case, I will move things around a little bit in the code but the overall state of the infrastructure remains the same.

This project is a continuation of the [Ansible Automationn Project](https://github.com/ekedonald/Ansible-Automation-Project). In this project, I will continue working with the [ansible-config-mgt](https://github.com/ekedonald/ansible-config-mgt) repository and make some improvements on my code such as: 
1. Refactor Ansible Code
2. Create Assignments
3. Imports Functionality (i.e. allows organisation of tasks and reuse when needed)

## How To Implement Ansible Refactoring, Assignments & Imports
The following steps are taken to implement Ansible Refactoring, Assignments & Imports on the [ansible-config-mgt repository](https://github.com/ekedonald/ansible-config-mgt):

### Step 1: Jenkins job enhancement

* SSH into your `Jenkins-Ansible` server and create a directory called `ansible-config-artifact` (all the artifacts will be stored here after each build).

```sh
ssh ubuntu@<public_IP_address_Jenkins_Ansible>
```

![ssh jenkins-ansible](./images/10.%20ssh%20jenkins%20ansible%20instance.png)

```sh
sudo mkdir /home/ubuntu/ansible-config-artifact
```

![mkdir ansible-config-artifact](./images/1.%20mkdir%20ansible-config-artifact.png)

* Change permissions no this directory so Jenkins could save files there using the following commands:

```sh
sudo chmod -R 777 /home/ubuntu/ansible-config-artifact/
sudo chown -R jenkins:jenkins /home/ubuntu/ansible-config-artifact/
```

![permissions](./images/1.%20chmod%20chown%20ansible-config-artifact.png)

* Log into your Jenkins Console, click on `Manage Jenkins` tab and click on `Plugins`

![jenkins plugin](./images/1.%20plugins.png)

* Click on `Available plugins` and type `copy artifact` in the search bar then select **Copy Artifact** plugin and click on `Install` (_Note that restarting jenkins after installing the plugin isn't necessary_).

![available plugins](./images/1.%20copy%20artifact.png)

* Create a new freestyle job named `save_artifacts`, give it a maximum of 2 builds to keep.

![save artifacts](./images/1.%20save-artifact.png)
![max builds](./images/1.%20general%20max%20builds%20description.png)

* Configure the `build trigger` of the job to **Build after other projects are built** then link it to your `ansible` job and select **Trigger only if build is stable**.

![build trigger](./images/1.%20build%20triggers.png)

* Configure a `build step` to **Copy artifacts from another project**, specify the following parameters then apply and save changes:
1. Project name: ansible
2. Latest successful build
3. Artifacts to copy: **
4. Target directory: /home/ubuntu/ansible-config-artifact

![build step](./images/1.%20copy%20artifacts%20from%20another%20project.png)
![build step2](./images/1%20copy%20artifacts%20from%20another%20project1.png)

* You will notice the `save-artifacts `job has an upstream project called `ansible` which means they are connected to each other, click on the `ansible` job drop-down icon and click on **Build Now** to test if the `ansible` job will successfully trigger the `save-artifacts` job.

![build now](./images/1.%20build%20now%20ansible%20.png)
![build triggered](./images/1.%20build%20triggered%20save-artifact%20job.png)

* Verify if the artifacts of the `ansible` are present in your `ansible-config-artifacts` directory on your `Jenkins-Ansible` server by running the following command:

```sh
cd /home/ubuntu/ansible-config-artifact && ll
```

![cd ansible-config-artifact && ll](./images/1.%20cd%20ansible-config-artifat%20&%20ll.png)

### Step 2: Refactor Ansible code by importing other playbooks into `site.yml`
* Before starting to refactor the codes, go to the `ansible-config-mgt` directory on your local machine and pull down the latest code from the `main` branch.

* Create a new branch `refactor` and switch into the branch using the command below:

```sh
git checkout -b refactor
```

![git checkout refactor](./images/2.%20git%20checkout%20-b%20refactor.png)

* In the `playbooks` folder, create a new file named `site.yml` (_The `site.yml` will become a parent to all other playbooks that will be developed including `common.yml` that you created previously_).

```sh
cd playbooks && touch site.yml
```

![cd playbooks && touch site.yml](./images/2.%20cd%20playbooks%20&%20create%20site-yml.png)

* Create a new folder in root of the repository and name it `static-assignments` (_The **static-assignments** directory is where all other children playbooks will be stored_).

```sh
cd .. && mkdir static-assignments
```

![cd .. && mkdir static-assignments](./images/2.%20mkdir%20static-assignments.png)

* Move the `common.yml` file into the `static-assignments` directory.

```sh
mv playbooks/common.yml static-assignments
```

![mv common.yml to static-assignments](./images/2.%20move%20common%20to%20static-assignment.png)

* In the `site.yml` file, import `common.yml` playbook.

```sh
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```

![site.yml](./images/2.%20site_yml.png)

* The folder structure of the `ansible-config-mgt` should look like this:

![tree ansible-config-mgt](./images/2.%20tree%20ansible-config-mgt.png)

### Step 3: Merge the changes from the refactor branch into the main branch and run your playbook

* Run the following command to view the untracked files (i.e. the file and directory you just created):

```sh
git status
```

![git status](./images/3.%20git%20status.png)

* Add the untracked files and commit the changes using the following command:

```sh
git add . && git commit -m "updated files and directories"
```

![git add commit](./images/3.%20git%20add%20commit.png)

* Push all changes from the `refactor` branch to the main branch.

```sh
git push --set-upstream origin refactor
```

![git push refactor](./images/3.%20git%20push%20refactor.png)

* Go to your `ansible-config-mgt` repository on GitHub and click on the `Compare & pull request` button.

![compare & pull request](./images/3.%20compare%20&%20pull%20request.png)

* Click on the `Create pull request` button.

![create pull request](./images/3.%20create%20pull%20request.png)

* Click on the `Merge pull request` button.

![merge pull request](./images/3.%20merge%20pull%20request.png)

* Click on the `Confirm merge` button.

![confirm merge](./images/3.%20confirm%20merge.png)

* Go to the `ansible-config-mgt` directory on your local machine and run the following command to switch to the `main` branch and pull the changes.

```sh
git chekout main && git pull
```

![git checkout && git pull](./images/3.%20git%20checkout%20main%20&%20git%20pull.png)

* On your Jenkins console, you will see a build has been triggered.

![build triggered](./images/3.%20build%20triggered.png)

* Run `ansible-playbook` command against the `dev` environment using the command shown below:

```sh
ansible-playbook -i inventory/dev playbook/site.yml
```

![ansible-playbook](./images/4.%20ansible-playbook%20-i%20inventory:dev%20playbooks:site.png)

### Step 4: Create a playbook configuration file `common-del.yml` that will be used to remove the wireshark package and import the configuration file to the playbook

* Create a `common-del.yml` file in the `static-assignments` directory.

```sh
cd static-assignments && touch common-del.yml
```

![cd static-assignments && touch common-del.yml](./images/4.%20cd%20static%20assignments%20and%20create%20common-del.png)

* Paste the code shown below into the `common-del.yml` file

```sh
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```

![common-del.yml](./images/4.%20common-del_yml.png)

* Import the playbook file `common-del.yml` and comment the `common.yml` import in the `site.yml` file.

```sh
- import_playbook: ../static-assignments/common-del.yml
```

![site.yml](./images/4.%20site_yml.png)

* Run the following command to view the changes and add the untracked files in the `ansible-config-mgt` repository:

```sh
git status
```

```sh
git add common-del.yml ../playbooks/site.yml
```

![git status add](./images/4.%20git%20status%20n%20git%20add%20common-del_yml%20..:playbooks:site_yml.png)

* Commit the changes made.

```sh
git commit -m "updates"
```

![git commit](./images/4.%20git%20commit.png)

* Push the changes to the `ansible-config-mgt` repository.

```sh
git push
```

![git push](./images/4.%20git%20push.png)

* Run the `ansible-playbook` command against the `dev` envinronment to remove wireshark on all your hosts.

```sh
ansible-playbook -i inventory/dev playbook/site.yml
```

![ansible-playbook](./images/4.%20ansible-playbook%20-i%20inventory:dev%20playbooks:site.png)

* Run Ansible Adhoc command to check if wireshark has been removed from all the servers.

```sh
ansible all -i inventory/dev -m command -a "wireshark --version"
```

![ansible adhoc command](./images/4.%20ansible%20adhoc%20command%20wireshark%20verision.png)

_**Note**: Stop the EC2 Instances (i.e. Web Server 1 & 2, Database Server and Load Balancer) after running the Ansible Adhoc command to save cost on your AWS Bill since they are no longer needed for this project_.

### Step 5: Provision 2 UAT Web Servers

Use the following parameters when provisioning the EC2 Instance for the 2 UAT Web Servers:
1. Name of the Instance: UAT_1
2. AMI: Red Hat Enterprise Linux 9 (HVM), SSD Volume Type
3. New Key Pair: web11
4. Key Pair Type: RSA
5. Private Key File Format: .pem
6. New Security Group: Web-Server SG
7. Inbound Rules: Allow Traffic From Anywhere On Port 80 & Port 22

![UAT-1 Server](./images/5.%20instance%20summary%20UAT_1.png)
*Instance Summary for UAT_1 Server*

![UAT-2 Server](./images/5.%20instance%20summary%20UAT_2.png)
*Instance Summary for UAT_2 Server*

### Step 6: Create a new branch and a role for User Acceptance Testing.

* Create and switch into a branch `uat-145` that will be used for User Acceptance Testing in your `ansible-config-mgt` repository.

```sh
git checkout -b uat-145
```

![git checkout uat-145](./images/6.%20new%20branch%20for%20uat.png)

* To create a role, you must create a directory called roles in the `ansible-config-mgt` directory. There are two ways you can this folder structure:

1. Using an Ansible utility called `ansible-galaxy` inside `ansible-config-mgt/roles` directory (Note that you need to create a `roles` directory upfront).

```sh
mkdir roles
cd roles
ansible-galaxy init webserver
```

2. Create the directory/file structure manually.

* I will use the **2nd** option since all my codes are stored in GitHub instead of using the `Ansible-Galaxy` utility on the `Jenkins Server`

* To create the folder structure below, the following commands were used:

```sh
cd .. && mkdir roles && cd roles && mkdir webserver
cd webserver && touch README.md && mkdir defaults handlers meta tasks templates
cp main.yml meta && mv main.yml tasks
cd .. ** tree webserver
```

![create folder structure](./images/6%20create%20roles:webserver%20file%20structure.png)

### Step 7: Update the `inventory/uat` file and configure the `tasks/main.yml` file

* Update the `ansible-config-mgt/inventory/uat` file with IP adresses of your 2 UAT Web Servers.

```sh
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
```

![inventory/uat file](./images/7.%20inventory:uat.png)

* In the tasks directory, configure the `main.yml` file to have the following configurations:

1. Install and Configure Apache (`httpd` service)
2. Clone **Tooling Website** form GitHub `https://github.com/<your-name>/tooling.git`
3. Ensure the tolling website code is deployed to `/var/www/html` on the 2 UAT Web Servers
4. Make sure `httpd` service is started

The `main.yml` will consist of the following tasks:

```sh
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```

![tasks/main.yml](./images/7.%20tasks:main_yml.png)

### Step 8: Reference the `Webserver` role

* Within the `static-assignments` folder, create a new assignment for **uat-webservers**. Update the `uat-webservers.yml`, this is where you will reference the role:

```sh
---
- hosts: uat-webservers
  roles:
     - roles/webserver
```

![static-assignments/uat-webservers.yml](./images/8.%20uat-webservers_yml.png)

* Remember that the entry point to your Ansible configuration is the `site.yml` file. Therefore, you need to refer your `uat-webservers.yml` role inside `site.yml`. Update the `site.yml` to have the following configuration:

```sh
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```

![site.yml](./images/8.%20%20site_yml.png)

### Step 9: Update the `ansible-config-mgt` repository on GitHub with the latest configurations

Remember you have been working on a separate branch `uat-145`, you need to get your branch peer-reviewed and pushed to the `main` branch. The following steps are taken to achieve this:

* Use the following commands to check the status of your branch, add files and directories then commit changes and push your branch to GitHub:

```sh
git status
```
![cd .. ansible-config-mgt](./images/9%20cd..%20ansible-config-mgt.png)
![git status](./images/9.%20git%20status.png)

```sh
git add . && git commit -m "updates" && git push --set-upstream origin uat-145
```

![git add commit push](./images/9.%20git%20add%20commit%20push.png)

* Go to your `ansible-config-mgt` repository on GitHub and click on the `Compare & pull request` button.

![compare & pull request](./images/9.%20compare%20&%20pull%20request.png)

* Click on the `Create pull request` button.

![create pull request](./images/9.%20create%20pull%20request.png)

* Click on the `Merge pull request` button.

![merge pull request](./images/9.%20merge%20pull%20request.png)

* Click on the `Confirm merge` button.

![confirm merge](./images/9.%20confirm%20merge.png)

* Head back to your terminal on VS Code, checkout from `uat-145` branch into the main and pull down the latest changes using the command shown below:

```sh
git checkout main && git pull
```

![git checkout main && git pull](./images/9.%20git%20checkout%20&&%20git%20pull.png)

### Step 10: Run the Ansible test

* Open a new terminal and SSH into the `Jenkins-Ansible` server.

```sh
ssh ubuntu@<public_IP_address_of_jenkins_ansible
```

![ssh jenkins-ansible](./images/10.%20ssh%20jenkins%20ansible%20instance.png)

* Set up an SSH-Agent on the Jenkins-Ansible Instance so it will be able to connect to the 2 UAT-Webservers using the following command:

```sh
eval `ssh-agent -s` && ssh-add web11.pem && ssh-add -l
```

![ssh-agent](./images/10.%20ssh-agent.png)

* Go to the `ansible-config-artifact` directory

```sh
cd /home/ubuntu/ansible-config-artifact
```

![cd ansible-config-artifact](./images/10.%20cd%20:home:ubuntu:ansible-config-artifact.png)

* Run your playbook against the `uat` inventory file.

```sh
ansible-playbook -i inventory/uat playbook/site.yml
```

![ansible-playbook](./images/10.%20ansible-playbook%20.png)

* Validate the successful configuration of the 2 UAT-Webservers with Ansible by going to the following URLs on your web browser:

```sh
http://public_IP_address_of_UAT_1_Web_Server/index.php
```

![http uat-1](./images/10.%20http%20uat%201%20web%20server.png)

```sh
http://public_IP_address_of_UAT_2_Web_Server/index.php
```

![http uat-2](./images/10.%20http%20uat%202%20web%20server.png)

Your Ansible architecture now looks like this:
![final architecture](./images/final%20architecture.png)