# IAC with Ansible


### Let's create Vagrantfile to create Three VMs for Ansible architecture
#### Ansible controller and Ansible agents 

```

# -*- mode: ruby -*-
 # vi: set ft=ruby :
 
 # All Vagrant configuration is done below. The "2" in Vagrant.configure
 # configures the configuration version (we support older styles for
 # backwards compatibility). Please don't change it unless you know what
 
 # MULTI SERVER/VMs environment 
 #
 Vagrant.configure("2") do |config|
 # creating are Ansible controller
   config.vm.define "controller" do |controller|
     
    controller.vm.box = "bento/ubuntu-18.04"
    
    controller.vm.hostname = 'controller'
    
    controller.vm.network :private_network, ip: "192.168.33.12"
    
    # config.hostsupdater.aliases = ["development.controller"] 
    
   end 
 # creating first VM called web  
   config.vm.define "web" do |web|
     
     web.vm.box = "bento/ubuntu-18.04"
    # downloading ubuntu 18.04 image
 
     web.vm.hostname = 'web'
     # assigning host name to the VM
     
     web.vm.network :private_network, ip: "192.168.33.10"
     #   assigning private IP
     
     #config.hostsupdater.aliases = ["development.web"]
     # creating a link called development.web so we can access web page with this link instread of an IP   
         
   end
   
 # creating second VM called db
   config.vm.define "db" do |db|
     
     db.vm.box = "bento/ubuntu-18.04"
     
     db.vm.hostname = 'db'
     
     db.vm.network :private_network, ip: "192.168.33.11"
     
     #config.hostsupdater.aliases = ["development.db"]     
   end
 
 
 end
```
## Booting up the VMs - Web, DB & Controller
- `Vagrant up` to create all the virtual machines. The commands for creating these machines have been provided in the vagrantfile above.
- Once this is done, `vagrant ssh` into each VM and run `sudo apt-get update -y` && `sudo apt-get upgrade -y`. We run the update and upgrade commands to check if all the machines have connection to the internet.
- We create one vm fto set up the ansible controller and the other two to set up an app node and a db node which listens to the controller. Node is an `instance`
- So we have `vagrant ssh controller`, `vagrant ssh web` and `vagrant ssh db`
- Default folder structure for ansible controller is in /etc/ansible. In this location, we have hosts file. Hosts file is also known as `inventory` for Ansible controller. This will help us to communicate between the nodes and the controller. In our case, we have an agent node called web and the ip of the web. So once we sent the request from our controller, it will go into the hosts file and look for the entry and the exact ip and if it matches, it will allow us to go into the nodes
- ssh into controller using `vagrant ssh controller`
- then use `sudo apt-get install software-properties-common`. This will install all the dependencies we require
- To add a repository for ansible into the controller, use `sudo  apt-add-repository ppa:ansible/ansible`
- use `sudo apt-get install ansible` to install ansible
- we can check version of ansible by using `ansible --version`
- now we need to look into the default folder structure using `cd /etc` and then `cd/ansible`
- we have ansible.cfg, hosts and roles directory as default. Its the hosts file where we need to add ip addresses of the ansible nodes.
- Install tree using `sudo apt install tree`
- Using just `tree` in the command line gives us the folder structure in a structured form
- Now to ssh into the ansible nodes, we need its ip addresses.
- Use `ssh vagrant@192.168.33.10` and enter. This will ask for the password which is `v*****t`.
- This will take us to the web VM.
- Use `exit` to exit out of the web vm
- Now use `ssh vagrant@192.168.33.11` to go into the db vm. Use the same password as above
- go into ansible using `cd /etc` and `cd /ansible`. Then do `sudo nano hosts` to add the following commands
![Image Link](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/etc_ansible_hosts_web.png)

- do the same for db node like below
![Image Link](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/etc_ansible_hosts_db.png)

- Test if the connection between the controller and the nodes work by using `ansible nodename -m ping`
- To see how info about all the servers are working, use `ansible all -m ping`
- we can use `ansible web -a "uname -a" to identify the node
- We can similarily use `ansible all -a "uname -a" to identify all the servers
- We can use `ansible all -a "date" to check where the server is running
- use `ansible all -a "free" will give us information about how much memory is being used, how much is free
- To copy file/data r+from controller to web node, use `ansible web -m ansible.builtin.copy -a "src=/etc/ansible/README.md dest=/home/vagrant/README.md"`
- To copy file/data r+from controller to db node, use `ansible db -m ansible.builtin.copy -a "src=/etc/ansible/README.md dest=/home/vagrant/README.md"`
- This can be verified by using `ansible nodename -a "ls"` or use `ansible all -a "ls"`
### Ansible playbooks
- YAML/yml files with script to implement config management
- saves time
- playbooks are resuable
- how we can create a playbook - filename yml/yaml
- it starts with three dashes ---

### Ansible Task

![Image Link](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/Ansible%20architecture%20with%20vault.png)

- Write playbook scripts in Ansible controller Node to perform tasks in the web and db agent nodes
- Yaml file to install nginx in our web server 
```
# YAML/YML file to create a playbook to configure nginx in our web instance
---
# it starts with three dashes

# add the name of the host/instance/vm
- hosts: web

# collect logs or gather facts - 
  gather_facts: yes

# we need admin access to install anything
  become: true

# add the instructions - install nginx - in web server
  tasks:
  - name: Installing Nginx web-server in our app machine
    apt: pkg=nginx state=present

# HINT: be mindful of intendentation
# use 2 spaces - avoid using tab
```
- Copy app folder from controller to the web agent node
```
---
- hosts: web

  gather_facts: yes

  become: true

  tasks:
  - name: Copying app folder into web-server in our app machine
    copy:
      src: /home/vagrant/app
      dest: /home/vagrant/

```

- YML script to install nodejs in our web server
```
---
- hosts: web

  gather_facts: yes

  become: true

  tasks:
  - name: load a specific version of nodejs
    shell: curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -

  - name: install the required packages
    apt:
      pkg:
        - nginx
        - nodejs
      update_cache: yes
```


- Next, we have the playbook script for installing npm package
```
---
- hosts: web


  gather_facts: yes


  become: true


  tasks:
  - name: Installing npm in our app machine
    apt: pkg=npm state=present
```
- ssh into our web server and `cd app` and `npm start` and see the app running.
- Now, we need to install `mongod` in our db server and allow app to communicate with the db server for which we change the bindIp to `0.0.0.0` within the `mongodb.conf` file. So use the following playbook script
```
---
- hosts: db
  gather_facts: yes
  become: true
  tasks:
  - name: installing mongo
    apt:
      name: mongodb
      state: present
  - name: allow 0.0.0.0
    ansible.builtin.lineinfile:
      path: /etc/mongodb.conf
      regexp: '^bind_ip = '
      line: bind_ip = 0.0.0.0
  - name: restart mongodb
    service: name=mongodb state=restarted
  - name: mongod enable
    service: name=mongodb enabled=yes
```
- Now that we have allowed access to db, we need to create an environment variable so that our web server can communicate with the db server.

```
---
- hosts: web

  gather_facts: yes

  become: true
  tasks:
  - name: Insert line at end of file
    lineinfile:
      path: /home/vagrant/.bashrc
      line: "export DB_HOST='mongodb://192.168.33.11:27017/posts'"
  - name: sourcing .bashrc
    shell: source /home/vagrant/.bashrc
    args:
      executable: /bin/bash
```
- We can check the newly created a variable by sshing into our web server. Once verfified, we are now ready to see our app running with the required posts page. So `cd app`, `node seeds/seed.js` and then `npm start` to see the app running on port 3000 and posts working on 3000/posts.


- We can also set up a reverse proxy for our web server. Use the below playbook script 
```
---
# reverse proxy
- hosts: web

  gather_facts: yes

  become: yes

  tasks:
  - name: reverse proxy
    synchronize:
      src: /home/vagrant/provision/default
      dest: /etc/nginx/sites-available/default

  - name: restart nginx
    service: name=nginx state=restarted

  - name: enable nginx
    service: name=nginx enabled=yes
```
## Creating an ec2 instance from ansible controller using playbook script
- Create a new fresh vm 
- Run `sudo apt-get update -y` and `sudo apt-get upgrade -y`
- then use `sudo apt-get install tree` to install tree feature
- use `sudo apt-add-repository --yes --update ppa:ansible/ansible`. This goes to the ansible respository and downloads the folder and the specific version
- `sudo apt-get install ansible -y` to install ansible in our vm
- `sudo apt-get install python3-pip -y` to install the dependency
- `pip3 install awscli`
- `pip3 install boto boto3` . Verify before automating this process as this will ask for our permission
- run `sudo apt-get update -y` and `sudo apt-get upgrade -y` again
- Check the version of aws using `aws --version`
- Now we need to create a vault to store our keys which the ansible controller will use to login in into AWS platform
- so create directory within ansible using `mkdir group_vars` and then make another directory within `group_vars` directory using `mkdir all`.
- Within the `all` directory, create a new file using `sudo-ansible vault create pass.yml` and put in the following:
```
aws_access_key:
aws_secret_key:

and to save this file use esc and then type :wq!
```
- do `sudo chmod 600 pass.yml`
- We can use `ansible all -m ping --ask-vault-pass` to see if we can connect to the servers in cloud. By using `--ask-vault-pass`, ansible controller will know that we are trying ti establish connection with the cloud instance and not the localhost.

- To create an instance, we first need to generate new keys in the .ssh folder using `ssh-keygen -t rsa -b 4096` and as default it will create `id_rsa` and `id_rsa.pub` files. We will be using this to connect our ansible controller to the ec2 instance that we are about to generate.
- Lets create a new instance and allow port 80 for our nginx to work using the playbook script below

```
---
- hosts: localhost
  connection: local
  gather_facts: yes
  vars_files:
  - /etc/ansible/group_vars/all/pass.yml
  vars:
    key_name: id_rsa
    region: eu-west-1
    image: ami-07d8796a2b0f8d29c
    id: "vivek-ansible2"
    sec_group: "{{ id }}-sec"
  tasks:
    - name: Provisioning EC2 instances
      block:
      - name: Upload public key to AWS
        ec2_key:
          name: "{{ key_name }}"
          key_material: "{{ lookup('file', '/home/vagrant/.ssh/id_rsa.pub') }}"
          region: "{{ region }}"
          aws_access_key: "{{aws_access_key}}"
          aws_secret_key: "{{aws_secret_key}}"
      - name: Create security group
        ec2_group:
          name: "{{ sec_group }}"
          description: "Sec group for app {{ id }}"
          region: "{{ region }}"
          aws_access_key: "{{aws_access_key}}"
          aws_secret_key: "{{aws_secret_key}}"
          rules:
            - proto: tcp
              ports:
                - 22
              cidr_ip: 0.0.0.0/0
              rule_desc: allow all on ssh port
            - proto: tcp
              ports:
                - 80
              cidr_ip: 0.0.0.0/0
              rule_desc: allow all on http port
        register: result_sec_group
      - name: Provision instance(s)
        ec2:
          aws_access_key: "{{aws_access_key}}"
          aws_secret_key: "{{aws_secret_key}}"
          key_name: "{{ key_name }}"
          id: "{{ id }}"
          group_id: "{{ result_sec_group.group_id }}"
          image: "{{ image }}"
          instance_type: t2.micro
          region: "{{ region }}"
          wait: true
          count: 1
          instance_tags:
            name: eng103a_vivek_ansible
      tags: ['never', 'create_ec2']
```

- run the playbook using `sudo ansible-playbook ec2.yml --ask-vault-pass --tags create_ec2 --tags=ec2-create -e "ansible_python_interpreter=/usr/bin/python3"`
- once we run this, a new instance will be created in AWS
- Once it finishes initializing, copy the `ip` address of the instance and edit the hosts file as below

![Image Link](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/Hosts%20file%20Ansible%20ec2.png)

- save and exit out and ping it using `sudo ansible aws -m ping --ask-vault-pass`
- If we see 'pong', then we have done everything correctly
- Now create a .yml file to install nginx in our ec2 instance server
- use `sudo nano install_nginx.yml` and then create a playbook like below

```
---
- hosts: web

  gather_facts: yes

  become: true

  tasks:
  - name: Installing Nginx web-server in our app machine
    apt: pkg=nginx state=present

```
- Once it finishes installing nginx, we can see nginx running using our ec2 instance ip address.
- So go to aws and copy the ip address of this instance and paste in the browser to see nginx running.

### Taking it to cloud

- Image needs to be added here

- Create a new instance in aws inside my vpc and install the dependencies like below for setting this up as a controller which will control new instances of my application and db that will be created within the controller using playbook scripts. 
### Steps to make our instance into a controller
- Run `sudo apt-get update -y` and `sudo apt-get upgrade -y`
- then use `sudo apt-get install tree` to install tree feature
- use `sudo apt-add-repository --yes --update ppa:ansible/ansible`. This goes to the ansible respository and downloads the folder and the specific version
- `sudo apt-get install ansible -y` to install ansible in our vm
- `sudo apt-get install python3-pip -y` to install the dependency
- `pip3 install awscli`
- `pip3 install boto boto3` . Verify before automating this process as this will ask for our permission
- run `sudo apt-get update -y` and `sudo apt-get upgrade -y` again
- Check the version of aws using `aws --version`
- Now we need to create a vault to store our keys which the ansible controller will use to login in into AWS platform
- so create directory within ansible using `mkdir group_vars` and then make another directory within `group_vars` directory using `mkdir all`.
- Within the `all` directory, create a new file using `sudo-ansible vault create pass.yml` and put in the following:
```
aws_access_key:
aws_secret_key:

and to save this file use esc and then type :wq!
```
- do `sudo chmod 600 pass.yml`
#### Creating instance for the application
- Now we need to create our app and db instance by staying within the controller
- Navigate to /home/ubuntu/.ssh and use `ssh-keygen -t rsa -b 4096` to generate ssh keys that will we using to connect to our ec2 instance.
- While creating the new insatnces for the app and db, make sure you use your public vpc and public subnet to enable internet access to both.
- Navigate back to `/etc/ansible/` and use the below yaml file to firstly generate app instance
```
---
- hosts: localhost
  connection: local
  gather_facts: yes
  vars_files:
  - /etc/ansible/group_vars/all/pass.yml
  vars:
    ec2_instance_name: eng103a-vivek-ansible-app
    ec2_sg_name: eng103a_vivek_vpc
    ec2_pem_name: eng103a
  tasks:
  - ec2_key:
      name: "{{ec2_pem_name}}"
      key_material: "{{ lookup('file', '/home/ubuntu/.ssh/vivek_app.pub') }}"
      region: "eu-west-1"
      aws_access_key: "{{aws_access_key}}"
      aws_secret_key: "{{aws_secret_key}}"
  - ec2:
      aws_access_key: "{{aws_access_key}}"
      aws_secret_key: "{{aws_secret_key}}"
      key_name: "{{ec2_pem_name}}"
      instance_type: t2.micro
      image: ami-07d8796a2b0f8d29c
      wait: yes
      group: "{{ec2_sg_name}}"
      region: "eu-west-1"
      count: 1
      vpc_subnet_id: subnet-xxxxxxxxxx
      assign_public_ip: yes
      instance_tags:
        Name: "{{ec2_instance_name}}"

```
- Once created, include this into the `hosts` file
![Image Link](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/Hosts%20file%20cloud%20controller.png)
- now use `sudo ansible app -m ping --ask-vault-pass` to check the connection.
- now copy the app folder from localhost to the controller using ` scp -i "~/.ssh/xxxx.pem" -r app ubuntu@34.244.126.239:~` and then to the application instance using the below playbook script and check if it is done correctly by sshing into the app instance
```
---
# web vm
- hosts: web

# gather logs
  gather_facts: yes

# give admin access
  become: true

# install dependencies
  tasks:
  - name: Copy app folder from cloud controller to app node
    synchronize:
      src: /home/ubuntu/app/
      dest: /home/ubuntu/app
```
- Once this is done use the playbook scripts to install dependencies like above for npm, nginx, nodejs

```
# YAML/YML file to create a playbook to configure nginx in our web instance
---
# it starts with three dashes

# add the name of the host/instance/vm
- hosts: app

# collect logs or gather facts - 
  gather_facts: yes

# we need admin access to install anything
  become: true

# add the instructions - install nginx - in web server
  tasks:
  - name: Installing Nginx web-server in our app machine
    apt: pkg=nginx state=present
```
- nodejs
```
---
- hosts: app

  gather_facts: yes

  become: true

  tasks:
  - name: load a specific version of nodejs
    shell: curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -

  - name: install the required packages
    apt:
      pkg:
        - nginx
        - nodejs
      update_cache: yes
```
- npm
```
- hosts: web


  gather_facts: yes


  become: true


  tasks:
  - name: Installing npm in our app machine
    apt: pkg=npm state=present
```
- Once all are set up, ssh into app instance and navigate into app using `cd app` and then `npm start` to start the app.
- Go to the browser and then use the application instance ip with port 3000 to see the app running.

- We can set up reverse proxy as well using the playbook script below. We restart nginx once this is set up
```
---
# reverse proxy
- hosts: web

  gather_facts: yes

  become: yes

  tasks:
  - name: reverse proxy
    synchronize:
      src: /home/ubuntu/default
      dest: /etc/nginx/sites-available/default

  - name: restart nginx
    service: name=nginx state=restarted

  - name: enable nginx
    service: name=nginx enabled=yes
```
#### Creating instance for db 
- Now the below yaml file to create a new instance from the controller
```
---
- hosts: localhost
  connection: local
  gather_facts: yes
  vars_files:
  - /etc/ansible/group_vars/all/pass.yml
  vars:
    ec2_instance_name: eng103a-vivek-ansible-db
    ec2_sg_name: eng103a_vivek_vpc_db
    ec2_pem_name: eng103a
  tasks:
  - ec2_key:
      name: "{{ec2_pem_name}}"
      key_material: "{{ lookup('file', '/home/ubuntu/.ssh/vivek_app.pub') }}"
      region: "eu-west-1"
      aws_access_key: "{{aws_access_key}}"
      aws_secret_key: "{{aws_secret_key}}"
  - ec2:
      aws_access_key: "{{aws_access_key}}"
      aws_secret_key: "{{aws_secret_key}}"
      key_name: "{{ec2_pem_name}}"
      instance_type: t2.micro
      image: ami-07d8796a2b0f8d29c
      wait: yes
      group: "{{ec2_sg_name}}"
      region: "eu-west-1"
      count: 1
      vpc_subnet_id: subnet-xxxxxxxx
      assign_public_ip: yes
      instance_tags:
        Name: "{{ec2_instance_name}}"
```
- Once created, verify it by going into aws to see the new instance
- Now include the new instance in our hosts file like below. Make sure you use private ipv4 while defining it within the hosts. This is to ensure security as its our database
![Image LInk](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/hosts%20file%20cloud%20for%20db.png)
- Now check the connection using `sudo ansible db -m ping --ask-vault-pass`. If you get a pong response, then we have done everything correctly.
- Now install the dependencies for this db instance: mongodb, change the mongod.conf file to allow access for the app. Make sure you restart mongodb to pick up the new chamges made to the mongod.conf file
- yaml file for installing mongodb and changing the mongood.conf file
```
---
- hosts: db
  gather_facts: yes
  become: true
  tasks:
  - name: installing mongo
    apt:
      name: mongodb
      state: present
  - name: allow 0.0.0.0
    ansible.builtin.lineinfile:
      path: /etc/mongodb.conf
      regexp: '^bind_ip = '
      line: bind_ip = 0.0.0.0
  - name: restart mongodb
    service: name=mongodb state=restarted
  - name: mongod enable
    service: name=mongodb enabled=yes
```
- Now that we have allowed access from the db, we need to create an environment variable in hthe app. Use the below yaml file for creating this.
```
---
- hosts: app

  gather_facts: yes

  become: true
  tasks:
  - name: Insert line at end of file
    lineinfile:
      path: /home/ubuntu/.bashrc
      line: "export DB_HOST='mongodb://10.0.8.163:27017/posts'"
  - name: sourcing .bashrc
    shell: source /home/ubuntu/.bashrc
    args:
      executable: /bin/bash
```
- Once we finish executing this playbook, ssh into the app instance using xxxxx.pem file or the sssh key we generated - `ssh -i "eng103a-xxx.pem" ubuntu@xx.xx.xxx.xxx`
- check if the new variable was created. If yes, cd into app, do `node seeds/seed.js` and finally `npm start` to start the app.
- got to your browser and use the app instances public ip address to see the app running and `ip address/posts` to see the posts page

### Setting up app and db using ansible and jenkins
 - Set up master node for jenkins on AWS. Either use saved ami of the master node or create a new master node instance as below
   - create a new instance for the master node in aws inside your vpc
   - enable subnet and choose the following security group: 
![Image Link](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/security%20group%20for%20jenkins%20masternode.png)
- Once the instance is up, ssh into it and run the following commands to make it a master node
  - `sudo apt-get update -y` 
  - `sudo apt-get upgrade -y`
  - `sudo apt install software-properties-common -y`
  - `sudo add-apt-repository ppa:deadsnakes/ppa`
  - `sudo apt install openjdk-8-jre -y` 
  - `wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -`
  - `sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'`
  - `sudo apt-get install update -y`
  - `sudo apt-get install jenkins -y`
  - `sudo systemctl start jenkins`
- Now go back to your instance and get the public ip and paste it in the browser with the port 8080.
- Now copy the command in red from the browser - `/var/lib/jenkins/secrets/initialAdminPassword` and paste it in the terminal
- This will give us the password to unlock jenkins
- Now select the plugins you require.
- For our case, we need the following plugins: SSH Agent, Nodejs, Git parameter, GitHub and SSH
- Once all selected plugins are installed, you will be asked to create user name, password, provide name and email.
- Upon completion, you can finally access the jenkins server.
- Since we are using ansible to create the two instances for app and db using playbooks, we need to install the correct plugins for ansible in jenkins
- Go to `Manage Jenkins` and go to `Available` and search for `EC2`, `Ansible` and `YAML`
- Once the plugins are installed, navigate to `Global Tool Configuration` and go to the ansible section and do as below:
![Image LInk](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/Global%20Tool%20configuration%20ansible.png)

#### Installing ansible on the master node instance
- On the gitbash terminal, type `sudo apt install python3.9 -y`
- `sudo su` to use as root
- type `update-alternatives --install /usr/bin/python python /usr/bin/python3 1`
- `python --version` to check the version. It should show 3.6.9
- Now exit out of root using `exit` command
- Run the following commands:
  - `sudo apt-get install python3-pip -y`
  - `sudo pip3 install awscli boto3 boto3`
- Now use `sudo apt-add-repository ppa:ansible/ansible`
- Then type `sudo apt-get install ansible -y`
- navigate to ansible folder using `cd /etc/ansible/`
- Now create `group_vars` and `all` directories and then our `pass.yml` file with the aws credentials using `sudo ansible-vault create pass.yml`
- Use `sudo chown jenkins:jenkins pass.yml` to give jenkins permission to access pass.yml file
- then use `ansible-galaxy collection install amazon.aws`
- use `scp -i "~/.ssh/keyname.pem" -r app/ ubuntu@ipaddress:~` from the location of the app folder to copy app folder into the ubuntu vm or `git clone` to get the required app folder
- now get inside jenkins using `sudo su jenkins` and go to the home directory using `cd ~`
- navigate to .ssh folder using cd ~/.ssh. If .ssh doesnt exit, create it using `mkdir .ssh` 
- and then cd into it
- we now need to generate keys that our playbooks will use to create the `app` and `db` instances. Use `ssh-keygen -t rsa -b 4096` to create a key and name it accordingly
- now `exit` or `ctrl+d` to exit out.
#### Lets create our playbooks for the app and db instances!
- `cd /etc/ansible/` and use `sudo nano name-of-file.yml`
- the playbook for creating the app looks like below
![Image Link](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/create_app_yml.png)
- we need to add `ansible_python_interpreter: /usr/bin/python3` so that python3 can act as an interpreter for ansible.
- We also need the tags as shown on the picture above
- Now that we have made the playbook, we will ask jenkins to run our playbook for creating the app instance. 
- Go to jenkins and create a job called `vivek_create_app` and choose `Invoke Ansible Playbook` and make changes as shown in the picture below
![Image Link](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/create_app_jenkins_jobs.png)
- And then provide the vault details. click `add` and select `secret text` from the `kind` drop down menu and provide the vault password you set while making the vault in the terminal.
![Image Link](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/Vault%20credentials%20jenkins%20job.png)
- Now click  on `advanced` as shown on the far right corner and enter the following:
![Image Link](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/advanced%20build%20action%20ansible.png)
- now click `apply` and `save` and test it out using `Build Now`
- Create another `Build Action` and Do the same to the db instance.
- Now we have created two instances with the help jenkins and playbooks
- We now need to specify these servers in our hosts inventory file like below
```
[app]
34.xxx.xxx.xxx ssh_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/<private_key>

[db]
18.xxx.xxx.xxx ssh_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/<private_key>
```
- Test the connection with `ansible nodename -m ping --ask-vault-pass`
- No we create two more playbooks to install all the dependencies for the app and db instances so that we can see our app running with the posts.
- Playbook for installing app dependencies
```
---
- hosts: app
  gather_facts: yes
  become: yes
  tasks:
  -  name: syncing app folder
     synchronize:
       src: /home/ubuntu/app
       dest: ~/
  -  name: upgrade
     apt: upgrade=yes
  -  name: load a specific version of nodejs
     shell: curl -sl https://deb.nodesource.com/setup_6.x | sudo -E bash -
  -  name: install the required packages
     apt:
       pkg:
         - nginx
         - nodejs
         - npm
       update_cache: yes
  -  name: nginx configuration for reverse proxy
     synchronize:
       src: /home/ubuntu/app/default
       dest: /etc/nginx/sites-available/default
  -  name: nginx restart
     service:
       name: nginx
       state: restarted
  -  name: setting db variable
     lineinfile:
       dest: /home/ubuntu/.bashrc
       line: 'export DB_HOST=mongodb://<db_ip>:27017/posts'
```
- playbook for installing db dependencies
```
---
- hosts: db
  gather_facts: yes
  become: true
# install mongodb in db instance and ensure it's running
  tasks:
  - name: installing mongo pre-requisites
    apt: pkg=mongodb state=present update_cache=yes
  - name: allow 0.0.0.0
    ansible.builtin.lineinfile:
      path: /etc/mongodb.conf
      regexp: '^bind_ip = '
      insertafter: '^#bind_ip = '
      line: bind_ip = 0.0.0.0
  - name: restart and enable mongod
    service: name=mongodb state=restarted enabled=yes
```

- Now create job in jenkins for installing DB dependency
![Image Link](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/DB_dependencies_jenkins_ansible.png)
- Do the same for app dependencies
![Image Link](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/app_dependencies_jenkins_ansible.png) 

- Once the jobs are made, click `Build Now` and test the jobs
- once the build is ready, we can go to our app instance and use its ip address to see the app running with `/posts`




