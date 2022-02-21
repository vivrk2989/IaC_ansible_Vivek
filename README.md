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

![Image Link](https://github.com/vivrk2989/IaC_ansible_Vivek/blob/main/Images/Ansible%20Architecture%20Vivek.png)

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
