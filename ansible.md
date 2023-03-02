# How to Manage Remote Servers w/ Ansible Tutorial

Following the tutorial blog series by [Digital Ocean](https://www.digitalocean.com/community/tutorial_series/how-to-manage-remote-servers-with-ansible). 

<!-- TOC start -->

- [How to Manage Remote Servers w/ Ansible Tutorial](#how-to-manage-remote-servers-w-ansible-tutorial)
  * [Introduction to Configuration Management](#introduction-to-configuration-management)
    + [Terms](#terms)
  * [How to install and configure Ansible on Ubuntu 20.04](#how-to-install-and-configure-ansible-on-ubuntu-2004)
  * [How to Set Up Ansible Inventories](#how-to-set-up-ansible-inventories)
  * [How to Manage Multiple Servers with Ansible Ad Hoc Commands](#how-to-manage-multiple-servers-with-ansible-ad-hoc-commands)
    + [Terms](#terms-1)
    + [Patterns](#patterns)
  * [How to Execute Ansible Playbooks to Automate Server Setup](#how-to-execute-ansible-playbooks-to-automate-server-setup)
  * [References](#references)
    
    <!-- TOC end -->
    
    <!-- TOC --><a name="how-to-manage-remote-servers-w-ansible-tutorial"></a>

## Introduction to Configuration Management

- **Configuration Management**, also referred to as IT automation
- Benefits of configuration management includes
  1. define infrastructure as code
  2. version control any changes in your infrastructure
  3. reuse provisioning scripts for multiple servers
  4. share provisioning scripts for collaboration
  5. streamline process of replicating servers
- Ansible Overview
  - Users will write provisioning scripts in YAML
  - A **control machine** will be setup w/ ansible software to communicate to other **nodes** via ssh
- Ansible Features
  - **Idempotent Behavior** - even when the program is run multiple times, the outcome does not change
  - Support to Variables, Conditionals and Loops
  - **System Facts**
  - **Templating system (using Jinja2)**
  - **Support for Extensions and Modules**

### Terms

| Term                     | Definition                                                                                                                                          |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Configuration Management | handling changes to a system overtime                                                                                                               |
| Idempotent Behavior      | If a package was already installed, it won't try to install it again                                                                                |
| System Facts             | detailed information about system nodes (the servers that are being provisioned), such as network interfaces, that are provided as global variables |
| Control Node             | a system w/ ansible installed. It can be used to setup and connect to other servers for provisioning                                                |
| Managed Nodes            | the system you control using ansible                                                                                                                |
| Inventory                | a list of hosts that will be managed using ansible                                                                                                  |
| Tasks                    | an individual unit of work to execute **on a managed node**. Tasks can be one-off actions.                                                          |
| Playbook                 | an **ordered** list of tasks and other directions to show which hosts are targets of automation. a full playbook is called a play                   |
| Handlers                 | perform actions on a service (such as restarting or stopping the service)                                                                           |
| Roles                    | a set of playbooks and related files                                                                                                                |

## How to install and configure Ansible on Ubuntu 20.04

1. Install Ansible
   
   ```bash
   # refresh system's package index
   sudo apt update
   
   # install ansible 
   sudo apt install ansible
   ```

2. Setup the **Inventory File**
   
    This file will contain info about the hosts that you are managing with Ansible. You  can:
   
   - include several hundred servers
   
   - organize hosts by subgroups or groups
   
   - set variables that will be used for specific hosts or groups
     
     ```bash
     # you can actually create this inventory somewhere else. 
     # If you do you need to provide a path to the custom inventory file with the -i parameter
     sudo nano /etc/ansible/inventory
     ```
     
     Add the following to your file (replace ip address and username as appropriate)
     
     ```bash
     [servers]
     10.0.0.193 ansible_user=owen
     
     [all:vars]
     ansible_ptyon_interpreter=/usr/bin/python3
     ```
     
     `all:vars` subgroup sets the `ansible_python_interpreter` for all hosts included in the inventory. Makes sure that we use python3 and not the default python2. 
     
     To check the inventory run:
     
     ```bash
     ansible-inventory -i inventory --list -y
     ```
     
     I got the following output. This yaml format is another way to write what we put in our `inventory` file above.
     
     ```bash
     owen@LAPTOP-TOOAF9UL:~/ansible_tutorial$ ansible-inventory -i inventory --list -y
     all:
       children:
         servers:
           hosts:
             10.10.0.193:
               ansible_python_interpreter: /usr/bin/python3
               ansible_user: owen
         ungrouped: {}
     ```

3. Testing the Connection
   
    Generate the ssh key first on your local machine (replace ip address and username as appropriate)
   
   ```bash
   # on the control machine in linux
   ssh-keygen # do not enter a password
   
   ssh-copy-id owen@10.0.0.193
   
   # test logging in without a password now
   ssh owen@10.0.0.193
   
   # logout
   exit
   ```
   
    Once your ssh key is added, you can run this to test your connection (replace ip address and username as appropriate)
   
   ```bash
   ansible -i inventory all -m ping -u owen
   ```
   
    This was my output
   
   ```bash
   owen@LAPTOP-TOOAF9UL:~/ansible_tutorial$ ansible -i raspi_hosts all -m ping -u pi 
   server1 | SUCCESS => {
       "changed": false, 
       "ping": "pong"
   }
   ```

4. Running Ad-Hoc Commands
   
   These are commands you can normally execute on a ssh server. In this case we run it using ansible. You can replace `df -h` with any command you like. (replace ip address and username as appropriate)
   
   ```bash
   ansible -i inventory all -a "df -h" -u pi
   
   # some more ad-hoc commands
   # -----------------------
   # installing the latest version of vim 
   ansible -i raspi_hosts all -m apt -a "name=vim state=latest" -u pi
   
   # checking the uptime of every host in the `servers` group
   ansible -i raspi_hosts servers -a "uptime" -u pi
   
   # specifying multiple hosts with colons
   ansible -i raspi_hosts server1:server2 -m ping -u pi
   ```

## How to Set Up Ansible Inventories

1. Create a inventory file in a directory of your choosing. 
   
   In my case I did 
   
   ```bash
   mkdir ansible_tutorial
   cd ansible_tutorial
   nano inventory
   ```
   
   In my inventory file I added my Raspberry Pi IP-Address
   
   ```bash
   10.0.0.193
   ```
   
   I can check the ansible inventory using 
   
   ```bash
   ansible-inventory -i inventory --list
   ```
   
   Where I got the output
   
   ```bash
   {
       "_meta": {
           "hostvars": {
               "10.0.0.193": {
                   "ansible_ptyon_interpreter": "/usr/bin/python3", 
                   "ansible_user": "owen"
               }
           }
       }, 
       "all": {
           "children": [
               "servers", 
               "ungrouped"
           ]
       }, 
       "servers": {
           "hosts": [
               "10.0.0.193"
           ]
       }
   }
   ```

2. Organizing Servers into Groups and Subgroups
   
   Below is an example of how you can organize hosts into different groups. A host can be in **multiple groups**
   
   ```bash
   [webservers]
   203.0.113.111
   203.0.113.112
   
   [dbservers]
   203.0.113.113
   server_hostname
   
   [development]
   203.0.113.111
   203.0.113.113
   
   [production]
   203.0.113.112
   server_hostname
   ```
   
   You can also aggregate multiple groups as children under a "parent"/metagroup group
   
   ```bash
   [web_dev]
   203.0.113.111
   
   [web_prod]
   203.0.113.112
   
   [db_dev]
   203.0.113.113
   
   [db_prod]
   server_hostname
   
   [webservers:children]
   web_dev
   web_prod
   
   [dbservers:children]
   db_dev
   db_prod
   
   [development:children]
   web_dev
   db_dev
   
   [production:children]
   web_prod
   db_prod
   ```

3. Setting Up Host Aliases
   
   Here is an example, where the host aliases are creatively named `server1`, `server2`, `server3`, and `server4`
   
   ```bash
   server1 ansible_host=203.0.113.111
   server2 ansible_host=203.0.113.112
   server3 ansible_host=203.0.113.113
   server4 ansible_host=server_hostname
   ```

4. Setting Up Host Variables
   
   We can setup variables to use for the host that will change ansible's default behavior. For example, we can specify which `ansible_user` to use when connecting to the node.
   
   ```bash
   server1 ansible_host=203.0.113.111 ansible_user=sammy
   server2 ansible_host=203.0.113.112 ansible_user=sammy
   server3 ansible_host=203.0.113.113 ansible_user=myuser
   server4 ansible_host=server_hostname ansible_user=myuser
   ```
   
   We can also attach variables to groups using `[<GROUP_NAME:vars]` as seen below
   
   ```bash
   [group_a]
   server1 ansible_host=203.0.113.111
   server2 ansible_host=203.0.113.112
   
   [group_b]
   server3 ansible_host=203.0.113.113
   server4 ansible_host=server_hostname
   
   [group_a:vars]
   ansible_user=sammy
   
   [group_b:vars]
   ansible_user=myuser
   ```

5. Using Patterns to Target Execution of Commands and Playbooks
   
   Here is an example inventory. 
   
   ```bash
   [webservers]
   203.0.113.111
   203.0.113.112
   
   [dbservers]
   203.0.113.113
   server_hostname
   
   [development]
   203.0.113.111
   203.0.113.113
   
   [production]
   203.0.113.112
   server_hostname
   ```
   
   You want to target production level servers to run an ansible command. So we do 
   
   ```bash
   ansible dbservers:\&production -m ping
   ```
   
   - `&` represents a logical AND, which means the targets must be in **both** `dbservers` and `production`. We also have to use the `\` escape character
   - We also use the `:` character to list multiple targets

## How to Manage Multiple Servers with Ansible Ad Hoc Commands

### Terms

| Term         | Definition                                                                |
| ------------ | ------------------------------------------------------------------------- |
| metagroup    | a parent group                                                            |
| host aliases | a name to use for the server instead of the original ip-address/host name |
| modules      | pieces of code that can be invoked from playbooks                         |

- Adjusting Connection Options
  
  - connecting as a different remote user 
    
    By default, ansible tries to connect to the nodes with the same name as your current system user
    
    ```bash
    ansible all -i inventory -m ping -u sammy
    ```
  
  - Using a custom SSH Key
    
    ```bash
    ansible all -i inventory -m ping --private-key=~/.ssh/custom_id
    ```
  
  - You can set the user in the inventory as well
  
  - You can set the SSH Key file in the inventory as well
    
    ```bash
    server1 ansible_host=203.0.113.111 ansible_user=sammy
    server2 ansible_host=203.0.113.112 ansible_ssh_private_key_file=/home/sammy/.ssh/custom_id
    ```
  
  - You can also use groups to set the user variable and the ssh key file 
    
    ```bash
    [group_a]
    203.0.113.111
    203.0.113.112
    
    [group_b]
    203.0.113.113
    
    [group_a:vars]
    ansible_user=sammy
    ansible_ssh_private_key_file=/home/sammy/.ssh/custom_id
    ```

- Defining Targets for Command Execution
  
  This is really similar to the previous lesson, so I will just copy the examples in this [lesson](https://www.digitalocean.com/community/cheatsheets/how-to-manage-multiple-servers-with-ansible-ad-hoc-commands#de)
  
  ```bash
  # pinging all the hosts in the [servers] group
  ansible servers -i inventory -m ping
  
  # pinging only server1, server2, and the dbservers group
  ansible server1:server2:dbservers -i inventory -m ping
  
  # pinging group1, but not server2
  ansible group1:\!server2 -i inventory -m ping
  
  # pinging group1 and group2
  ansible group1:\&group2 -i inventory -m ping
  ```

- Running Ansible Modules
  
  - specific commands that you can execute. It includes the `apt` module, the `user` module, and the `ping` command.
  
  - Template for executing a module
    
    ```bash
    ansible target -i inventory -m module -a "module options"
    ```
  
  - Example to install an module called `tree`. I also added the `--become` flag  to grant sudo permissions
    
    ```bash
    ansible all -i inventory -m apt -a "name=tree" --become
    ```
    
    The output response looks like
    
    ```bash
    owen@LAPTOP-TOOAF9UL:~/ansible_tutorial$ ansible all -i inventory -m apt -a "name=tree" --become
    192.168.1.105 | SUCCESS => {
        "cache_update_time": 1636522520, 
        "cache_updated": false, 
        "changed": false
    }
    ```

### Patterns

| Pattern         | Result Target                                       |
| --------------- | --------------------------------------------------- |
| all             | All hosts from your inventory file                  |
| host1           | a single host (host1)                               |
| host1:host2     | both host1 and host 2                               |
| group1          | a single group (group1)                             |
| group1:group2   | all servers in group1 and group2                    |
| group1:\&group2 | Only servers that are **both** in group1 and group2 |
| group1:\!group2 | Servers in group1 **except** those also in group2   |

## How to Execute Ansible Playbooks to Automate Server Setup

- Create a Test Playbook
  
  create the file
  
  ```bash
  nano playbook.yml
  ```
  
  add the contents
  
  ```yaml
  ---
  - hosts: all
    become: true
    tasks:
      - name: Install Packages
        apt: name={{ item }} update_cache=yes state=latest
        loop: [ 'nginx', 'vim' ]
        tags: [ 'setup' ]
  
      - name: Copy index page
        copy:
          src: index.html
          dest: /var/www/html/index.html
          owner: www-data
          group: www-data
          mode: '0644'
        tags: [ 'update', 'sync' ]
  ```
  
  create a new `index.html` file
  
  ```html
  <html>
      <head>
          <title>Testing Ansible Playbooks</title>
      </head>
      <body>
          <h1>Testing Ansible Playbooks</h1>
          <p>This server was set up using an Nginx playbook.</p>
      </body>
  </html>
  ```

- Run the playbook 
  
  ```bash
  ansible-playbook -i inventory playbook.yml
  ```

## References

1. [How to Manage Remote Servers w/ Ansible](https://www.digitalocean.com/community/tutorial_series/how-to-manage-remote-servers-with-ansible) - A digital ocean tutorial for using ansible
2. [Digital Ocean - Ansible Cheat Sheet Guide](https://www.digitalocean.com/community/cheatsheets/how-to-use-ansible-cheat-sheet-guide) - a great resource for a bunch of different ansible tips and recipes.
