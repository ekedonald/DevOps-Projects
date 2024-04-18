# Ansible Dynamic Assignments (Include) and Community Roles
The [Ansible Automation](https://github.com/ekedonald/Ansible-Automation-Project) and [Ansible Refactoring Assignments and Imports](https://github.com/ekedonald/Ansible-Refactoring-Assignments-and-Imports) projects have already explored how to perform configurations using `playbooks`, `roles` and `imports`. In this project, I will introduce [Dynamic Assignments](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse.html#includes-dynamic-re-use) by using the `include` module. From the Ansible Refactoring Assignment project, it is evident that **Static Assignments** use the `import` module. However, the module that enables **Dynamic Assignments** is `include`.

When the `import` module is used, all statements are pre-processed at the time playbooks are parsed (i.e. when you execute `site.yml` playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements). This also means that during actual execution, if any statement changes, such statements will not be considered. Hence, it is **Static**.

On the other hand, when the `include` module is used, all the statements are processed only during the execution of the playbook (i.e. after the statements are parsed, any changes to the statements encountered during execution will be used).

However, it is recommended to use **Static Assignments** for playbooks because it is more reliable while **Dynamic Assignments** are difficult to debug due to their dynamic nature. Dynamic Assignments can be used for environment-specific variables.

## How To Implement Ansible Dynamic Assignments (Include) and Community Roles
The following steps are taken to Implement Ansible Dynamic Assignments (Include) and Community Roles:

### Step 1: Introduce Dynamic Assignment into your File Structure

* Create and switch into the `dynamic-assignments` branch.

```sh
git checkout -b dynamic-assignments
```

![git checkout dynamic-assignments](./images/1.%20git%20checkout%20dynamic%20assignments.png)

* Create a `dynamic-assignments` directory and create an `env-vars.yml` file inside the directory.

```sh
mkdir dynamic-assignments && cd dynamic-assignments && touch env-vars.yml
```

![mkdir dynamic-assignments && cd dynamic-assignments && touch env-vars.yml](./images/1.%20mkdir%20dynamic%20assignments.png)

* Update the `env-vars.yml` file with the following codebase:

```sh
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always
```

![env-vars.yml](./images/1.%20env-vars_yml.png)

There are 3 things to notice in the codebase above:

1. The `include_vars` module was used instead of `include` because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the `include` module was depreciated and variants of `include_*` must be used (i.e. [include_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_role_module.html#include-role-module), [include_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_tasks_module.html#include-tasks-module) and [include_vars](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_vars_module.html#include-vars-module)).

2. The following [special variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html) were used: `{{ playbook_dir }}` and `{{ inventory_file }}`.
The `{{ playbook_dir }}` helps Ansible determine the location of the running playbook and navigation to other paths on the filesystem while the `{{ inventory_file }}` will dynamically resolve the name of the inventory file being used then append `.yml` so that it picks up the required file within the `env-vars` directory.

3. The variables are included using the `with_first_found` loop which is used to loop through the list of files and the first one is used. This is good so that we can always set default values in case an environment-specific `env` file does not exist.

* Go a step back in the directory, create an `env-vars` directory and create the following files: `dev.yml`, `stage.yml`, `uat.yml` and `prod.yml` inside the directory.

* The layout of the `ansible-config-mgt` directory should like this:

![layout ansible-config-mgt](./images/1.%20layout%20ansible-config-mgt.png)


### Step 2: Update `site.yml` with Dynamic Assignments

* Update the `site.yml` playbook configuration file to import files from the `dynamic-assignments` directory.

```sh
---
- hosts: all
  become: true
  name: Import Dynamic Variables
- import_playbook: ../dynamic-assignments/env-vars.yml
```

![site.yml](./images/2.%20site_yml.png)

### Step 3: Merge the changes from the dynamic-assignments branch into the main branch

* Run the following command to view the untracked files (i.e. the file and directory you just created):

```sh
git status
```

![git status](./images/3.%20git%20status.png)

* Add the untracked files and commit the changes.

```sh
git add playbooks/site.yml dynamic-assignments env-vars && git commit -m "updates"
```

![git add commit](./images/3.%20git%20add%20&&%20commit.png)

* Push all changes from the `dynamic-assignments` branch to the `main` branch.

```sh
git push --set-upstream origin dynamic-assignments
```

![git push dynamic-assignments](./images/3.%20git%20push%20dynamic-assignments.png)

* Go to your `ansible-config-mgt` repository on GitHub and click on the `Compare & pull request` button.

![compare and pull request](./images/3.%20compare%20&%20pull%20request.png)

* Click on the `Create pull request` button.

![create pull request](./images/3.%20create%20pull%20request.png)

* Click on the `Merge pull request` button.

![merge pull request](./images/3.%20merge%20pull%20request.png)

* Click on the `Confirm merge` button.

![confirm merge](./images/3.%20confirm%20merge.png)

* Go to the `ansible-config-mgt` directory on your local machine and run the following command to switch to the `main` branch and pull the changes:

```sh
git checkout main && git pull
```

![git checkout main && git pull](./images/3.%20git%20checkout%20main%20&&%20git%20pull.png)

### Step 4: SSH into the Jenkins-Ansible server, pull files from the `ansible-config-mgt` repository and create a `roles-feature` branch

* SSH into the `Jenkins-Ansible` server.

```sh
ssh ubuntu@<public_ipv4_address_of_jenkins_server
```

![ssh jenkins-ansible](./images/4.%20ssh%20jenkins-ansible.png)

* Check if git is installed on the server.

```sh
git --version
```

![git version](./images/4.%20git%20--version.png)

* Create and go into the `ansible-config-mgt` directory.

```sh
mkdir ansible-config-mgt && cd ansible-config-mgt
```

![mkdir ansible-config-mgt && cd ansible-config-mgt](./images/4.%20mkdir%20ansible-config-mgt%20&&%20cd%20ansible-config-mgt.png)

* Run the following commands to initialize a repository, pull files from your `ansible-config-mgt` repository on GitHub, add a new remote repository reference, create and switch to the `roles-feature` branch.

```sh
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git checkout -b roles-feature
```

![git init pull remote add checkout roles-feature](./images/4%20git%20init%20pull%20remote%20add%20checkout%20roles-feature.png)

### Step 5: Using Ansible Galaxy, Download Community Roles for Apache, Nginx and MySQL into the roles directory

* Go into the `roles` directory.

```sh
cd roles
```

![cd roles](./images/5.%20cd%20roles.png)

* Create a MySQL role with Ansible Galaxy using the following command:

```sh
ansible-galaxy install -p . geerlingguy.mysql
```

![ansible-galaxy mysql](./images/5.%20ansible-galaxy%20mysql.png)

* Create an Nginx role with Ansible Galaxy using the following command:

```sh
ansible-galaxy install -p . geerlingguy.nginx
```

![ansible-galaxy nginx](./images/5.%20asnible%20galaxy%20nginx.png)

* Create an Apache role with Ansible Galaxy using the following command:

```sh
ansible-galaxy install -p . geerlingguy.apache
```

![ansible-galaxy apache](./images/5.%20ansible%20galaxy%20apache.png)

* Rename the role directories you downloaded.

```sh
mv geerlingguy.apache/ apache && mv geerlingguy.nginx/ nginx && mv geerlingguy.mysql/ mysql && ll
```

![rename the role directories](./images/5%20rename%20the%20role%20directories.png)

### Step 6: Merge changes from the roles-feature into the main branch

* Run the following command to view the untracked files (i.e. the roles directories you just created):

```sh
cd .. && git status
```

![git status](./images/6.%20git%20status.png)

* Add the untracked files, commit the changes and push all the changes from the `roles-feature` branch to the `main` branch.

```sh
git add roles/apache/ roles/mysql/ roles/nginx/
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
```

![git add commit push roles-feature](./images/6.%20git%20add%20roles%20commit%20push%20roles-feature.png)
_This prompts you to input your GitHub account username and password. Input your username and for the password you will need to input your Personal Access Token_.

The following steps are taken to setup a Personal Access Token:

1. Go to your GitHub Account, click on your Profile Icon and click on `settings`

![settings](./images/6.%20settings.png)

2. Click on `Developer Settings`

![developer settings](./images/6.%20developer%20settings.png)

3. Click on `Personal access tokens` and `Tokens (classic)`

![personal access tokens](./images/6.%20personal%20access%20token%20&%20tokens%20(classic).png)

4. Click on `Generate new token` and `Generate new token (classic)`

![generate new token](./images/6.%20generate%20new%20token.png)

5. Give the Token a name of your choice (i.e. dynamic assignment project) and tick all the boxes.

![name the token and tick all boxes](./images/6.%20name%20the%20token%20and%20tick%20all%20boxes.png)

6. Click on `Generate token`

![generate token](./images/6.%20generate%20token.png)

7. Copy the token you just created and head back to the `Jenkins-Ansible` terminal.

![copy token](./images/6.%20copy%20token.png)

* Paste the token in the prompt for the password of your GitHub account.

![paste the token password](./images/6.%20paste%20token%20password.png)

* Go to your `ansible-config-mgt` repository on GitHub and click on the `Compare & pull request` button.

![compare and pull request](./images/6.%20compare%20&%20pull%20request.png)

* Click on the `Create pull request` button.

![create pull request](./images/6.%20create%20pull%20request.png)

* Click on the `Merge pull request` button.

![merge pull request](./images/6.%20merge%20pull%20request.png)

* Click on the `Confirm merge` button.

![confirm merge](./images/6.%20confirm%20merge.png)

* Go to the `ansible-config-mgt` directory on your local machine and run the following command to pull the changes:

```sh
git pull
```

![git pull](./images/6.%20git%20pull.png)

### Step 7: Create a `loadbalancer.yml` file in the static-assignments directory, update the `site.yml` and `dev.yml` file in the playbook and env-vars directories respectively

* Create a `loadbalancer.yml` file in the `static-assignments` directory.

```sh
cd static-assignments && touch loadbalancer.yml
```

![cd static-assignments && touch loadbalancer.yml](./images/7.%20create%20loadbalancer_yml%20file.png)

* Paste the following codebase to reference your roles and conditions for running your roles in the `loadbalancer.yml` file:

```sh
- hosts: lb
  become: true
  roles:
    - { role: roles/nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: roles/apache, when: enable_apache_lb and load_balancer_is_required }
```

![loadbalancer.yml](./images/7.%20loadbalancer_yml.png)

* Update the `site.yml` file to import the `loadbalacer.yml` file.

```sh
- name: Loadbalancer assignment
  hosts: lb
- import_playbook: ../static-assignments/loadbalancers.yml
  when: load_balancdr_is_required
```

![site.yml](./images/7.%20site_yml.png)

* Declare variables in the `dev.yml` file in the `env-vars` directory to meet the conditions set to execute a role at a time.

```sh
enable_nginx_lb: true
enable_apache_lb: false
load_balancer_is_required: true
```

![dev.yml](./images/7.%20dev_yml.png)

### Step 8: Update the `ansible-config-mgt` repository on GitHub with the latest configurations

* Run the following command to view the untracked files (i.e. the file created and modified files):

```sh
git status
```

![git status](./images/8.%20git%20status.png)

* Add the untracked files and commit the changes.

```sh
git add ../env-vars/dev.yml ../playbooks/site.yml loadbalancers.yml && git commit -m "uploaded a configuration file and modified 2 configuration files"
```

![git add commit](./images/8.%20git%20add%20commit.png)

* Push the changes to the `main` branch.

```sh
git push
```

![git push](./images/8.%20git%20push.png)

### Step 9: Run the Ansible Playbook with the Nginx role

Remember that a webhook was configured to save artifacts in the `ansible-config-artifact` directory on the `Jenkins-Ansible` server in the previous project. Hence, the latest files in the `ansible-config-mgt` repository on GitHub will be stored in the `ansible-config-artifact` directory.

* Go to the `ansible-config-artifact` directory on the `Jenkisn-Ansible` server.

```sh
cd /home/ubuntu/ansible-config-artifact/
```

![cd ansible-config-artifact](./images/9.%20cd%20:home:ubuntu:ansible-config-artifact.png)

* Run the Ansible Playbook.

```sh
ansible-playbook -i inventory/dev playbooks/site.yml
```

![ansible playbook nginx1](./images/9.%20ansible%20playbook%20nginx1.png)
![ansible playbook nginx2](./images/9.%20ansible%20playbook%20nginx2.png)
![ansible playbook nginx3](./images/9.%20ansible%20playbook%20nginx3.png)

### Step 10: Run the Ansible Playbook with the Apache role

* Change the variables enabling Apache and Nginx in the `dev.yml` file in the `env-vars` directory to meet the conditions for executing the Apache role.

```sh
enable_nginx_lb: false
enable_apache_lb: true
load_balancer_is_required: true
```

![dev.yml](./images/10.%20dev_yml.png)

* Run the Ansible Playbook.

```sh
ansible-playbook -i inventory/dev playbooks/site.yml
```

![ansible playbook error apache1](./images/10.%20ansible%20playbook%20error%20apache1.png)
![ansible playbook error apache2](./images/10.%20ansible%20playbook%20error%20apache2.png)

_Note that the **Ensure Apache has selected state and enabled on boot task** failed when the playbook ran because Nginx and Apache can not run simultaneously. To correct this, run an Ansible Adhoc Command to **disable** and **stop** nginx_.

* Run the Ansible Adhoc command to disable Nginx on the `lb` host.

```sh
ansible lb -i inventory/dev -m command -a "systemctl disable nginx" -b
```

![ansible adhoc disable nginx](./images/10.%20adhoc%20disable%20nginx.png)

* Run the Ansible Adhoc Command to stop Nginx on the `lb` host.

```sh
ansible lb -i inventory/dev -m command -a "systemctl stop nginx" -b
```

![ansible adhoc stop nginx](./images/10.%20adhoc%20stop%20nginx.png)

* Run the Ansible Playbook again.

```sh
ansible-playbook -i inventory/dev playbooks/site.yml
```

![ansible playbook apache1](./images/10.%20ansible%20playbook%20apache1.png)
![ansible playbook apache2](./images/10.%20ansible%20playbook%20apache2.png)

### Step 11: Run the Ansible Playbook with the MySQL role

* Change the variables enabling Apache, Nginx and MySQL in the `dev.yml` file in the `env-vars` directory to meet the conditions for executing the MySQL role.

```sh
enable_nginx_lb: false
enable_apache_lb: false
enable_mysql_lb: true
load_balancer_is_required: true
```

![dev.yml](./images/11.%20dev_yml.png)

* Update the `loadbancer.yml` file in the `static-assignments` directory to include the MySQL role and change the host to `db`

```sh
- { role: roles/mysql, when: enable_mysql_lb and load_balancer_is_required }
```

![loadbalancer.yml](./images/11.%20loadbalancer_yml.png)

* Update the `site.yml` file in the `playbooks` directory to have a `db` host in the **Loadbalancers assignment** import.

![site.yml](./images/11.%20site_yml.png)

* Run the Ansible Playbook.

```sh
ansible-playbook -i inventory/dev playbooks/site.yml
```

![ansible playbook mysql1](./images/11.%20ansible%20playbook%20mysql1.png)
![ansible playbook mysql2](./images/11.%20ansible%20playbook%20mysql2.png)
![ansible playbook mysql3](./images/11.%20ansible%20playbook%20mysql3.png)
![ansible playbook mysql4](./images/11.%20ansible%20playbook%20mysql4.png)