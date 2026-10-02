# 15 - Configuration Management with Ansible

## 1 - Introduction to Ansible

Ansible is an open-source automation tool that simplifies configuration management, application deployment, and task automation. It uses a simple, human-readable language called YAML to define automation tasks in playbooks. Ansible operates in an agentless manner, meaning it does not require any software to be installed on the target machines. Instead, it uses SSH for communication, making it easy to manage a large number of servers.

Ansible uses modules to perform specific tasks, such as installing packages, managing services, and configuring files. These modules can be used in playbooks to automate complex workflows. Ansible also supports inventory management, allowing users to define groups of hosts and apply configurations to them.

An Ansible module is a reusable, standalone script that performs a specific task on a target system. Modules can manage system resources, such as packages, services, and files, or execute commands. They are the building blocks of Ansible playbooks, allowing users to automate complex workflows efficiently.

An Ansible task is a single unit of work that is executed on a target system. Tasks are defined within playbooks and typically use modules to perform actions, such as installing a package, starting a service, or copying a file. Each task has a name, which helps in identifying it in the playbook output, and can include various parameters to control its behavior.


An Ansible host refers to a target machine or server that Ansible manages and automates tasks on

Ansible playbook is a YAML file that contains a series of tasks to be executed on one or more hosts. Playbooks are the primary way to define automation workflows in Ansible, allowing users to describe the desired state of their systems and applications. Each playbook consists of one or more plays, which define the target hosts and the tasks to be performed on them.

The host hold a reference to the target machine or server that Ansible manages and automates tasks on. It can be defined in the inventory file, which lists all the hosts and their associated groups. Ansible uses this inventory to determine which hosts to target when executing playbooks.

```yaml
# Example of an Ansible playbook
- name: Install and start Apache web server
  hosts: webservers
  become: yes
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
    - name: Start Apache service
        service: 
          name: apache2
          state: started
```

Sample Inventory File:

```ini
# Example of an Ansible inventory file
[webservers]
webserver1 ansible_host=192.168.1.10
webserver2 ansible_host=

[databases]
dbserver1 ansible_host=192.168.1.20
dbserver2 ansible_host=192.168.1.21
```

Ansible Tower is a web-based interface for managing Ansible automation. It provides a centralized platform for managing playbooks, inventories, and credentials, as well as scheduling and monitoring automation tasks. Ansible Tower also offers role-based access control, allowing organizations to manage permissions and access to automation resources effectively.

Alternative Ansible tools include puppet, Chef, and SaltStack. These tools also provide configuration management and automation capabilities, but they differ in their architecture, language, and approach to automation. Ansible is known for its simplicity and ease of use, making it a popular choice for many organizations.

Ansible vs puppet & chef table

| Feature          | Ansible                  | Puppet                   | Chef                     |
|------------------|--------------------------|--------------------------|--------------------------|
| Language         | YAML                     | Puppet DSL               | Ruby DSL                 |
| Architecture     | Agentless                | Agent-based              | Agent-based              |
| Ease of Use      | Easy                     | Moderate                 | Moderate                 |
| Configuration    | Playbooks                | Manifests                | Recipes                  |
| Community        | Large and active         | Large and active         | Large and active         |
| Use Case         | Configuration management | Configuration management | Configuration management |

## 2 - Install Ansible

Ansible can either be installed on a control node (a machine from which you run Ansible commands) or on a local machine. The installation process may vary depending on the operating system being used. Below are the steps to install Ansible on Mac:

```bash
brew install ansible
```

Ansible is written in Python, so it can also be installed using pip, the Python package manager. To install Ansible using pip, run the following command:

```bash
pip install ansible
```

## 3 - Setup Managed Server to Configure with Ansible

Create 2 droplets in digital ocean to configure with ansible. 

Ansible uses SSH to connect to the managed servers, so you need to ensure that SSH access is set up correctly. You can use SSH keys for authentication, which is more secure than using passwords.

Ansible requires Python to be installed on the managed servers. Most Linux distributions come with Python pre-installed, but you may need to install it manually on some systems.

## 4 - Ansible Inventory and Ansible ad-hoc commands

To connect ansible to the managed servers, you need to create an inventory file that lists the hosts and their associated groups. The inventory file can be in INI or YAML format. the default location for the inventory file is /etc/ansible/hosts, but you can also specify a custom location using the -i option when running ansible commands.

Ansible needs to authenticate with either username and password or SSH key to connect to the managed servers. You can specify the authentication method in the inventory file or use command-line options when running ansible commands.

Example of an Ansible inventory file in INI format, create "hosts" file with the following content:

```ini
159.89.167.194 ansible_ssh_private_key_file=~/.ssh/id_ed25519 ansible_user=root
64.227.169.232 ansible_ssh_private_key_file=~/.ssh/id_ed25519 ansible_user=root
```

Ansible ad-hoc commands are one-time commands that can be executed on managed servers without creating a playbook. Ad-hoc commands are useful for performing quick tasks, such as checking the status of a service or installing a package. To run an ad-hoc command, use the ansible command followed by the target hosts and the module to be executed.

```bash
# pattern: target hosts or groups defined in the inventory file (e.g., webservers, databases, all)
# module: Ansible module to be executed (e.g., ping, shell, command, etc.)
# arguments: optional arguments or module options to customize the command
ansible [pattern] -m [module] -a "[arguments or module options]"

ansible all -i hosts -m ping
``` 

In hosts file you can group the servers.

- You can put each host in more than 1 group
- you can create groups that track
  - where - a datacenter or region (e.g., us-east, eu-west)
  - what - a specific role or function (e.g., webservers, databases)
  - when - a specific environment (e.g., production, staging, development)

```ini
[droplet]
159.89.167.194 ansible_ssh_private_key_file=~/.ssh/id_ed25519 ansible_user=root
64.227.169.232 ansible_ssh_private_key_file=~/.ssh/id_ed25519 ansible_user=root

[webservers]

[databases]
```

To target the droplet group, you can use the following command:

```bash
# target all hosts
ansible all -i hosts -m ping

# target all hosts in the droplet group and execute the ping module
ansible droplet -i hosts -m ping

# target 1 server in the droplet group and execute the ping module
ansible droplet -i hosts -m ping -l 159.89.167.194
# or
ansible 159.89.167.194 -i hosts -m ping
```

Instead of repeating the variables in the inventory file, you can create a group_vars entry or directory and create a YAML file for each group. For example, you can create a group_vars/droplet.yml file with the following content:

```ini
[droplet]
159.89.167.194
64.227.169.232

[droplet:vars]
ansible_ssh_private_key_file: ~/.ssh/id_ed25519
ansible_user: root
```

## 5 - Configure AWS EC2 server with Ansible

Create 2 EC2 instances in AWS to configure with ansible. Download the private key from AWS and update the file permission to 400. 

```bash
chmod 400 <private-key-file>.pem
```

In ansible inventory file, you can specify the path to the private key file and the username for the EC2 instances. For example, you can create a hosts file with the following content:

```ini
[droplet]
159.89.167.194
64.227.169.232

[droplet:vars]
ansible_ssh_private_key_file: ~/.ssh/id_ed25519
ansible_user: root

[ec2]
ec2-3-120-45-67.compute-1.amazonaws.com
ec2-3-120-45-68.compute-1.amazonaws.com

[ec2:vars]
ansible_ssh_private_key_file=~/.ssh/my-aws-key.pem
ansible_user=ec2-user
# suppress python interpreter warning by specifying the python interpreter path
ansible_python_interpreter=/usr/bin/python3.12
```

## 6 - Managing Host Key Checking and SSH keys

Host key checking is enabled bu default in Ansible to ensure the authenticity of the target hosts, it guards against server spoofing and man-in-the-middle attacks. When connecting to a new host for the first time, Ansible will prompt you to confirm the host's fingerprint. If you want to disable host key checking, you can set the `ANSIBLE_HOST_KEY_CHECKING` environment variable to `False` or add the following line to your ansible.cfg file:

```ini
[defaults]
host_key_checking = False
```

If you don't want to disable ssh host key checking:

SSH host checking requires both the local machine & the target server to allow each other to connect. The local machine needs to include the target server in its known_hosts file (~/.ssh/known_hosts), and the target server needs to include the local machine public key in its authorized_keys file (~/.ssh/authorized_keys).

To add the target server to the known_hosts file, you can use the following command:

```bash
ssh-keyscan -H <target-server-ip> >> ~/.ssh/known_hosts

# check all know hosts
cat ~/.ssh/known_hosts
```

When you create a droplet with public ssh key, the droplet will automatically add the public key to its authorized_keys file. If you want to add a new public key to the target server, you can use the following command:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub <username>@<target-server-ip>

# check all authorized keys on the target server
ssh <username>@<target-server-ip> "cat ~/.ssh/authorized_keys"
```

For ephemeral infrastructure we can disable ssh host key checking in ansible config file (/etc/ansible/ansible.cfg) or in (~/.ansible.cfg) or in the inventory file by adding the following line:

```ini
[defaults]
host_key_checking = False
```

For configuration ansible will process the following list and use the first file found & ignore the rest:

- ANSIBLE_CONFIG (an environment variable)
- ansible.cfg (in the current directory)
- ~/.ansible.cfg (in the home directory)
- /etc/ansible/ansible.cfg (in the /etc/ansible directory)

## 7 - Introduction to Playbooks

Create "hosts" file with the following content:

```ini
[webserver]
159.89.167.194
64.227.169.232

[webserver:vars]
ansible_ssh_private_key_file: ~/.ssh/id_ed25519
ansible_user: root
ansible_python_interpreter=/usr/bin/python3.12
```

Create ansible config file "ansible.cfg" with the following content:

```ini
[defaults]
inventory = hosts
host_key_checking = False
```

Create "my-playbook.yml" file with the following content:

```yaml
---
- name: Configure nginx web server
  hosts: webserver
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install nginx server
      apt:
        name: nginx
        state: latest

    - name: Start nginx service
      service:
        name: nginx
        state: started
```

Run the playbook with the following command:

```bash
ansible-playbook my-playbook.yml
# or if hosts is not specified in ansible.cfg file
ansible-playbook -i hosts my-playbook.yml
```

Once you run the playbook, you'll see the "Gathering Facts" module which gets executed automatically to gather some variables about the target hosts that can be used in playbooks. After that, the tasks defined in the playbook will be executed on the target hosts. You can check the status of the nginx service on the target hosts by running the following command:

```bash
ansible webserver -m service -a "name=nginx state=started"
```

SSH to the server & check the status of the nginx service:

```bash
ssh root@ip-address
systemctl status nginx
# or
ps aux | grep nginx
```

We can specify the version to install in the playbook by adding the version number to the package name. 

```yaml
---
- name: Configure nginx web server
  hosts: webserver
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install nginx server
      apt:
        name: nginx=1.24.0-1ubuntu1 # or regex 1.24.*
        state: present # state preset 

    - name: Start nginx service
      service:
        name: nginx
        state: started
```

To stop & uninstall nginx service, you can use the following playbook:

```yaml
---
- name: Configure nginx web server
  hosts: webserver
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Uninstall nginx server
      apt:
        name: nginx=1.24.0-1ubuntu1 # or regex 1.24.*
        state: absent # state absent

    - name: Stop nginx service
      service:
        name: nginx
        state: stopped
```

Ansible is idempotent, meaning that running the same playbook multiple times will not change the state of the system if it is already in the desired state. This ensures that playbooks can be safely re-run without causing unintended side effects.

## 8 - Modules & Collections in Ansible

What is ansible collection? 

Ansible collections are a distribution format for Ansible content that can include playbooks, roles, modules, and plugins. Collections allow users to package and distribute their Ansible content in a standardized way, making it easier to share and reuse automation code. Collections can be installed from Ansible Galaxy or other sources, and they can be used in playbooks just like any other Ansible content.

Check Ansible docs for modules https://docs.ansible.com/projects/ansible/latest/collections/index_module.html & collections: https://docs.ansible.com/ansible/latest/collections/index.html

Ansible plugin is a piece of code that extends the functionality of Ansible. Plugins can be used to customize the behavior of Ansible, such as adding new modules, modifying existing modules, or changing the way Ansible interacts with target systems. There are several types of plugins in Ansible, including action plugins, callback plugins, connection plugins, and inventory plugins.

Ansible Galaxy is a community-driven platform for sharing and discovering Ansible content, including roles, collections, and playbooks. It provides a centralized repository where users can find and download pre-built automation code, as well as contribute their own content to the community. the url for ansible galaxy is https://galaxy.ansible.com/

To install a collection from Ansible Galaxy, you can use the ansible-galaxy command-line tool. For example, to install the community.general collection, run the following command:

```bash
ansible-galaxy collection install community.general
```

To list the local collections created when installing ansible, you can use the following command:

```bash
ansible-galaxy collection list
```

You can create your own collection by using the ansible-galaxy command-line tool. For example, to create a new collection named my_collection, run the following command:

```bash
ansible-galaxy collection init my_collection
```

Collections have a specific directory structure

```text
collection/
├── docs/
├── galaxy.yml
├── meta/
│   └── runtime.yml
├── plugins/
│   ├── modules/
│   │   └── module1.py
│   └── inventory/
│       └── .../
├── README.md
├── roles/
│   ├── role1/
│   ├── role2/
│   └── .../
├── playbooks/
│   ├── files/
│   ├── vars/
│   ├── templates/
│   └── tasks/
└── tests/
```

Ansible Namespace is a way to organize and group related Ansible content, such as modules, plugins, and roles. Namespaces help avoid naming conflicts and provide a clear structure for Ansible content. Each namespace is associated with a specific collection, and the content within that namespace can be referenced using the namespace prefix.

The (FQCN) full qualified collection name of an Ansible module includes the namespace, collection, and module name. For example, the full qualified name of the apt module in the ansible.builtin collection is ansible.builtin.apt. "ansible.builtin" is the default namespace.

## 9 - Project: Deploy Nodejs application - Part 1

Create a droplet on digital ocean to install nodejs & deploy the application

Create hosts file with the following content:

```ini
server-ip ansible_ssh_private_key_file: ~/.ssh/id_ed25519 ansible_user: root
```

Create "deploy-node.yaml" playbook file with the following content:

```yaml
---
- name: Install Node & Npm
  hosts: server-ip
  tasks:
    - name: Update apt repo & cache
      # single line or multi line both work
      # apt: update_cache: yes force_apt_get: yes cache_valid_time: 3600
      apt:
        update_cache: yes
        force_apt_get: yes
        cache_valid_time: 3600
    - name: Install Nodejs & Npm
      apt:
        pkg:
          - nodejs
          - npm

- name: Deploy Nodejs Application
  hosts: server-ip
  tasks:
    - name: Copy Nodejs application files to the server
      copy:
        src: /path/to/local/nodejs/app/nodejs-app-1.0.0.tgz
        dest: /root/app-1.0.0.tgz
    - name: Unpack the Nodejs application tar file
      unarchive:
        src: /root/app-1.0.0.tgz
        dest: /root/
        remote_src: yes
```

To generate the tar file for the nodejs app, run the following command in the local machine:

```bash
cd /path/to/local/nodejs/app
npm pack
```

Run the paybook with the following command:

```bash
ansible-playbook -i hosts deploy-node.yaml
```

SSH to the server & Check the server for the node js app

The "unarchive" module takes the ssource by default from the local machine and copies it to the remote server. So we can reduce the playbook as following with the same results:

```yaml
---
- name: Install Node & Npm
  hosts: server-ip
  tasks:
    - name: Update apt repo & cache
      # single line or multi line both work
      # apt: update_cache: yes force_apt_get: yes cache_valid_time: 3600
      apt:
        update_cache: yes
        force_apt_get: yes
        cache_valid_time: 3600
    - name: Install Nodejs & Npm
      apt:
        pkg:
          - nodejs
          - npm

- name: Deploy Nodejs Application
  hosts: server-ip
  tasks:
    - name: Unpack the Nodejs application tar file
      unarchive:
        src: /path/to/local/nodejs/app/nodejs-app-1.0.0.tgz
        dest: /root/
```

## 10 - Project: Deploy Nodejs application - Part 2

Install the dependencies & run the nodejs app:

```yaml
---
- name: Install Node & Npm
  hosts: server-ip
  tasks:
    - name: Update apt repo & cache
      apt: update_cache: yes force_apt_get: yes cache_valid_time: 3600
    - name: Install Nodejs & Npm
      apt:
        pkg:
          - nodejs
          - npm

- name: Deploy Nodejs Application
  hosts: server-ip
  tasks:
    - name: Unpack the Nodejs application tar file
      unarchive:
        src: /path/to/local/nodejs/app/nodejs-app-1.0.0.tgz
        dest: /root/
    - name: Install Nodejs application dependencies
      npm:
        path: /root/package
    - name: Start Nodejs application
      command:
        chdir: /root/package/app
        cmd: node server
      async: 1000
      poll: 0
    # register is used to save shell command output to a variable, so we can use it later in the playbook
    # debug for printing the output of the command to the console
    - name: Ensure Nodejs application is running
      shell: "ps aux | grep node server"
      register: app_status
    - debug: msg="Nodejs application is running: {{ app_status.stdout_lines }}"
```

Both "command" & "shell" modules can be used to run commands on the target server. The "command" module is more secure and faster than the "shell" module, but it does not support shell features such as pipes, redirects, and environment variable expansion.

"command" & "shell" modules are not stateful (idempotent), running above playbook multiple times will start multiple instances of the nodejs app. we can use conditional statements to check if the app is already running before starting it. 

## 11 - Project: Deploy Nodejs application - Part 3

For security best practices, we can create a new user to run the nodejs app instead of running it as root. 

```yaml
---
- name: Install Node & Npm
  hosts: server-ip
  tasks:
    - name: Update apt repo & cache
      apt: update_cache: yes force_apt_get: yes cache_valid_time: 3600
    - name: Install Nodejs & Npm
      apt:
        pkg:
          - nodejs
          - npm

- name: Create a new user to run the Nodejs application
  hosts: server-ip
  tasks:
    - name: Create a new user
      user:
        name: nodejs
        comment: "User to run Nodejs application"
        group: admin

- name: Deploy Nodejs Application
  hosts: server-ip
  become: True
  become_user: nodejs
  tasks:
    - name: Unpack the Nodejs application tar file
      unarchive:
        src: /path/to/local/nodejs/app/nodejs-app-1.0.0.tgz
        dest: /home/nodejs/
    - name: Install Nodejs application dependencies
      npm:
        path: /home/nodejs/package
    - name: Start Nodejs application
      command:
        chdir: /home/nodejs/package/app
        cmd: node server
      async: 1000
      poll: 0
    # register is used to save shell command output to a variable, so we can use it later in the playbook
    # debug for printing the output of the command to the console
    - name: Ensure Nodejs application is running
      shell: "ps aux | grep node server"
      register: app_status
    - debug: msg="Nodejs application is running: {{ app_status.stdout_lines }}"
```

Run the playbook with the following command:

```bash
ansible-playbook -i hosts deploy-node.yaml
```

## 12 - Ansible Variables - make your Playbook customizable

Ansible variables are used to store values that can be reused throughout a playbook. They allow you to make your playbooks more flexible and customizable, as you can define different values for different environments or scenarios. Variables can be defined in various ways, including in the inventory file, in playbooks, or in separate variable files.

```yaml
---
...

- name: Deploy nodejs app
  hosts: 143.110.189.59
  become: True
  become_user: nodeuser
  vars:
    location: ./node
    version: 1.0.0
    destination: /home/nodeuser
  tasks:
    - name: Unpack the nodejs file
      unarchive:
        # location between double quotes after colon, or ./node/nodejs-app-{{1.0.0}}.tgz without quotes
        src: "{{location}}/nodejs-app-{{version}}.tgz"
        dest: "{{destination}}"
    - name: Install dependencies
      npm:
        path: "{{destination}}/package"
    - name: Start the application
      command:
        chdir: "{{destination}}/package/app"
        cmd: node server
      async: 1000
      poll: 0
    - name: Ensure app is running
      shell: ps aux | grep node
      register: app_status
    - debug: msg={{app_status.stdout_lines}}
```

variables can also be passed in command line using the `--extra-vars` or `-e` option. For example, you can run the playbook with the following command:

```bash
ansible-playbook -i hosts deploy-node.yaml -e "location=./node version=1.0.0 destination=/home/nodeuser"
``` 

Variables can also be set in vars file, create "project-vars" file with the following content:

```yaml
location: ./node
version: 1.0.0
destination: /home/nodeuser
```

Reference the vars file in the playbook using the `vars_files` directive in all plays where the variables are used:

```yaml
---
...

- name: Deploy nodejs app
  hosts: 143.110.189.59
  become: True
  become_user: nodeuser
  vars_files:
    - project-vars
  tasks:
    - name: Unpack the nodejs file
      unarchive:
        # location between double quotes after colon, or ./node/nodejs-app-{{1.0.0}}.tgz without quotes
        src: "{{location}}/nodejs-app-{{version}}.tgz"
        dest: "{{destination}}"
    - name: Install dependencies
      npm:
        path: "{{destination}}/package"
    - name: Start the application
      command:
        chdir: "{{destination}}/package/app"
        cmd: node server
      async: 1000
      poll: 0
    - name: Ensure app is running
      shell: ps aux | grep node
      register: app_status
    - debug: msg={{app_status.stdout_lines}}
```

## 13 - Project Deploy Nexus - Part 1

Create a digital ocean droplet to install nexus repository manager. 

Following is shell commands that can be used to manually install nexus repository manager on the droplet. The Ansible playbook should automate these steps

```bash
apt-get update
apt install openjdk-8-jre-headless
apt install net-tools

cd /opt
wget https://download.sonatype.com/nexus/3/latest-linux-x86_64.tar.gz
tar -zxvf latest-linux-x86_64.tar.gz

adduser nexus
chown -R nexus:nexus nexus-3.65.0-02
chown -R nexus:nexus sonatype-work

vim nexus-3.65.0-02/bin/nexus.rc
run_as_user="nexus"

su - nexus
/opt/nexus-3.65.0-02/bin/nexus start

ps aux | grep nexus
netstat -lnpt
```

## 4 - Project Deploy Nexus - Part 2




```yaml
---
- name: Install java and net-tools
  hosts: nexus_server
  tasks:
    - name: Update apt repo and cache
      apt: update_cache=yes force_apt_get=yes cache_valid_time=3600
    - name: Install Java 17
      apt: name=openjdk-17-jre-headless
    - name: Install net-tools
      apt: name=net-tools

- name: Download and unpack Nexus installer
  hosts: nexus_server
  tasks:
    - name: Check nexus folder stats
      stat:
        path: /opt/nexus
      register: stat_result
    - name: Download Nexus
      get_url:
        url: https://download.sonatype.com/nexus/3/latest-linux-x86_64.tar.gz
        dest: /opt/
      register: download_result
    - name: Untar Nexus installer
      unarchive:
        src: "{{download_result.dest}}"
        dest: /opt/
        remote_src: yes
      when: not stat_result.stat.exists
    - name: Find nexus folder
      find: 
        paths: /opt
        pattern: "nexus-*"
        file_type: directory
      register: find_result
    - name: Rename nexus folder
      shell: mv {{find_result.files[0].path}} /opt/nexu
      # conditionals, rename only if directory not exists
      when: not stat_result.stat.exists

- name: Create nexus user to own nexus folder
  hosts: nexus_server
  tasks:
    - name: Ensure group nexus exists
      group:
        name: nexus
        state: present
    - name: Create nexus user 
      user:
        name: nexus
        group: nexus
    - name: Make nexus user owner of nexus folder
      file:
        path: /opt/nexus
        state: directory
        owner: nexus
        group: nexus
        recurse: yes
    - name: Make nexus user owner of sonatype-work folder
      file:
        path: /opt/sonatype-work
        state: directory
        owner: nexus
        group: nexus
        recurse: yes

- name: Start nexus with nexus user
  hosts: nexus_server
  become: True
  become_user: nexus
  tasks:
    - name: Set run_as_user nexus
      lineinfile: 
        path: /opt/nexus/bin/nexus.rc
        regexp: '^#run_as_user=""'
        line: run_as_user="nexus"
      # alternative way to set run_as_user nexus. blockinfile append while lineinfile replace 
      # blockinfile:
      #   path: /opt/nexus/bin/nexus.rc
      #   block: |
      #     run_as_user="nexus"
    - name: Start nexus
      command: /opt/nexus/bin/nexus start

- name: Verify nexus running
  hosts: nexus_server
  tasks:
    - name: Check with ps
      shell: ps aux | grep nexus
      register: app_status
    - debug: msg={{app_status.stdout_lines}}
    - name: Wait one minute
      pause:
        minutes: 1 
    - name: Check with netstat
      shell: netstat -plnt
      register: app_status
    - debug: msg={{app_status.stdout_lines}}
```

## 15 - Ansible Configuration - Default Inventory File

## 16 - Project: Run Docker applications - Part 1

Create an ec2 instance using terraform script and use ansible to install & run docker on the ec2 instance.

```tf
provider "aws" {
  region = "eu-central-1"
}

variable vpc_cidr_block {}
variable subnet_cidr_block {}
variable avail_zone {}
variable env_prefix {}
variable my_ip {}
variable instance_type {}
variable public_key_location {}

resource "aws_vpc" "myapp-vpc" {
  cidr_block = var.vpc_cidr_block
  enable_dns_hostnames = true
  tags = {
    Name: "${var.env_prefix}-vpc"
  }
}

resource "aws_subnet" "myapp-subnet-1" {
  vpc_id = aws_vpc.myapp-vpc.id
  cidr_block = var.subnet_cidr_block
  availability_zone = var.avail_zone
    tags = {
    Name: "${var.env_prefix}-subnet-1"
  }
}

resource "aws_internet_gateway" "myapp-igw" {
  vpc_id = aws_vpc.myapp-vpc.id
  tags = {
    Name: "${var.env_prefix}-igw"
  }
}

resource "aws_default_route_table" "main-rtb" {
  default_route_table_id = aws_vpc.myapp-vpc.default_route_table_id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.myapp-igw.id
  }
  tags = {
    Name: "${var.env_prefix}-main-rtb"
  }
}

resource "aws_default_security_group" "default-sg" {
  vpc_id = aws_vpc.myapp-vpc.id

  ingress {
    from_port = 22
    to_port = 22
    protocol = "TCP"
    cidr_blocks = [var.my_ip]
  }

  ingress {
    from_port = 8080
    to_port = 8080
    protocol = "TCP"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port = 0
    to_port = 0
    protocol = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    prefix_list_ids = []
  }

  tags = {
    Name: "${var.env_prefix}-default-sg"
  }
}

data "aws_ami" "latest-amazon-linux-image" {
  most_recent = true
  owners = ["amazon"]
  filter {
    name = "name" 
    values = ["al2023-ami-2023.*-x86_64"]
  }
  filter {
    name = "virtualization-type"
    values = ["hvm"]
  }
}

output "aws-ami_id" {
  value = data.aws_ami.latest-amazon-linux-image.id
}

output "ec2-public_ip_1" {
  value = aws_instance.myapp-server-one.public_ip
}

output "ec2-public_ip_2" {
  value = aws_instance.myapp-server-two.public_ip
}

resource "aws_key_pair" "ssh-key" {
  key_name = "server-key"
  public_key = file(var.public_key_location)
}

resource "aws_instance" "myapp-server-one" {
  ami = data.aws_ami.latest-amazon-linux-image.id
  instance_type = var.instance_type

  subnet_id = aws_subnet.myapp-subnet-1.id
  vpc_security_group_ids = [aws_default_security_group.default-sg.id]
  availability_zone = var.avail_zone

  associate_public_ip_address = true
  key_name = aws_key_pair.ssh-key.key_name

  tags = {
    Name: "${var.env_prefix}-server-1"
  }
}

resource "aws_instance" "myapp-server-two" {
  ami = data.aws_ami.latest-amazon-linux-image.id
  instance_type = var.instance_type

  subnet_id = aws_subnet.myapp-subnet-1.id
  vpc_security_group_ids = [aws_default_security_group.default-sg.id]
  availability_zone = var.avail_zone

  associate_public_ip_address = true
  key_name = aws_key_pair.ssh-key.key_name

  tags = {
    Name: "${var.env_prefix}-server-2"
  }
}
```

The ec2 image is Amazon linux that uses yum package manager instead of apt.

Below are the commands to install docker on the ec2 instance that's needs to be automated with ansible playbook:

```bash
#!/bin/bash
sudo yum update -y && sudo yum install -y docker
sudo systemctl start docker
sudo usermod -aG docker ec2-user

# install docker-compose
mkdir -p ~/.docker/cli-plugins/
sudo curl -SL "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o ~/.docker/cli-plugins/docker-compose
sudo chmod +x ~/.docker/cli-plugins/docker-compose
```

## 17 - Project: Run Docker applications - Part 2

To run docker compose to pull the image from private docker repo, we need to execute docker login on the remote server. the docker module from ansible can be used to work with docker on the remote server. For login, we can either store the password in vars file or we can do a password prompt to enter the password at runtime. 

```yaml
---
- name: Wait for SSH connection
  hosts: all
  gather_facts: False
  tasks:
    - name: Ensure SSH port open
      wait_for:
        port: 22
        delay: 10
        timeout: 100
        search_regex: OpenSSH
        host: '{{ (ansible_ssh_host|default(ansible_host))|default(inventory_hostname) }}'
    vars:
      ansible_connection: local
      ansible_python_interpreter: /usr/bin/python3


- name: Install Docker
  hosts: all
  become: yes
  tasks:
    - name: Install Docker
      yum:
        name: docker
        update_cache: yes
        state: present
    - name: Start docker daemon
      systemd:
        name: docker
        state: started

- name: Create new linux user
  hosts: all
  become: yes
  tasks: 
    - name: Create new linux user
      user:
        name: nana
        groups: adm,docker

- name: Install Docker-compose
  hosts: all
  become: yes
  become_user: nana
  tasks:
    - name: Create docker-compose directory
      file:
        path: ~/.docker/cli-plugins
        state: directory
    - name: Get architecture of remote machine
      shell: uname -m
      register: remote_arch
    - name: Install docker-compose
      get_url:
        url: "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-{{ remote_arch.stdout }}"
        dest: ~/.docker/cli-plugins/docker-compose
        mode: +x

- name: Start docker containers
  hosts: all
  become: yes
  become_user: nana
  vars_files:
    - project-vars
  tasks:
    - name: Copy docker compose 
      copy:
        src: /Users/nana/bootcamp-java-mysql-project/docker-compose-full.yaml
        dest: /home/nana/docker-compose.yaml
    - name: Docker login
      docker_login:
        username: nanatwn
        password: "{{docker_password}}"
    - name: Start containers from compose
      community.docker.docker_compose_v2:
        project_src: /home/nana
```

## 18 - Project: Terraform & Ansible

Instead of manually copying the server IPs to the ansible inventory file, we can use terraform output to dynamically generate the inventory file.

We can use a provisioner block in the terraform script to run ansible playbook after the ec2 instances are created. 

```tf
resource "null_resource" "configure_server" {
  triggers = {
    trigger = aws_instance.myapp-server.public_ip
  }

  provisioner "local-exec" {
    working_dir = "../ansible"
    command = "ansible-playbook --inventory ${aws_instance.myapp-server.public_ip}, --private-key ${var.private_key_location} --user ec2-user deploy-docker-new-user.yaml"
  }
}
```

## 19 - Dynamic Inventory for EC2 Servers

Ansible provides a dynamic inventory feature that allows you to automatically generate an inventory of your managed servers based on external sources, such as cloud providers like AWS, Azure, or GCP. This is particularly useful for environments where servers are frequently created and destroyed, as it eliminates the need to manually update the inventory file.

Ansible supports dynamic inventory through either plugins or scripts. Plugins are recommended because it can use ansible features like state management. Plugins are written in yaml while scripts are written in python.

There're certain Inventory plugins & scripts available for every infrastructure provider. To see the list of available inventory plugins, you can run the following command:

```bash
ansible-doc -t inventory -l
```

For Ec2 we can use the "amazon.aws.aws_ec2" inventory plugin to dynamically generate an inventory of EC2 instances. 

Enable plugins in ansible config file (/etc/ansible/ansible.cfg) or in (~/.ansible.cfg) or in the inventory file by adding the following line:

```ini
[inventory]
enable_plugins = amazon.aws.aws_ec2
```

To configure the plugin, create a file "inventory_aws_ec2.yml" (the file needs to end with _aws_ec2.yml) with the following content:

```yaml
plugin: amazon.aws.aws_ec2
regions:
  - eu-central-1
```

Run the following command to see the generated inventory:

```bash
# according to above config, below will list all the ec2 instances in eu-central-1 region
ansible-inventory -i inventory_aws_ec2.yml --list
# or graph view
ansible-inventory -i inventory_aws_ec2.yml --graph
```

The plugin automatically creates a host group called "aws_ec2" that we can use to reference the instances in the playbook instead of using "all".

We need the remote_user & private_key_file to connect to the ec2 instances. We can specify them in the ansible config file (/etc/ansible/ansible.cfg) or in (~/.ansible.cfg) or in the inventory file by adding the following lines:

```ini
[defaults]
remote_user = ec2-user
private_key_file = ~/.ssh/my-aws-key.pem
```

Create 3 ec2 instances in AWS using terraform.

Playbook to run on the AWS instances:

```yaml
---
- name: Wait for SSH connection
  hosts: all
  gather_facts: False
  tasks:
    - name: Ensure SSH port open
      wait_for:
        port: 22
        delay: 10
        timeout: 100
        search_regex: OpenSSH
        host: '{{ (ansible_ssh_host|default(ansible_host))|default(inventory_hostname) }}'
    vars:
      ansible_connection: local
      ansible_python_interpreter: /usr/bin/python3


- name: Install Docker
  hosts: all
  become: yes
  tasks:
    - name: Install Docker
      yum:
        name: docker
        update_cache: yes
        state: present
    - name: Start docker daemon
      systemd:
        name: docker
        state: started

- name: Create new linux user
  hosts: all
  become: yes
  tasks: 
    - name: Create new linux user
      user:
        name: nana
        groups: adm,docker

- name: Install Docker-compose
  hosts: all
  become: yes
  become_user: nana
  tasks:
    - name: Create docker-compose directory
      file:
        path: ~/.docker/cli-plugins
        state: directory
    - name: Get architecture of remote machine
      shell: uname -m
      register: remote_arch
    - name: Install docker-compose
      get_url:
        url: "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-{{ remote_arch.stdout }}"
        dest: ~/.docker/cli-plugins/docker-compose
        mode: +x

- name: Start docker containers
  hosts: all
  become: yes
  become_user: nana
  vars_files:
    - project-vars
  tasks:
    - name: Copy docker compose 
      copy:
        src: /Users/nana/bootcamp-java-mysql-project/docker-compose-full.yaml
        dest: /home/nana/docker-compose.yaml
    - name: Docker login
      docker_login:
        username: nanatwn
        password: "{{docker_password}}"
    - name: Start containers from compose
      community.docker.docker_compose_v2:
        project_src: /home/nana
```

Run the ansible playbook with the new inventory file to install docker & run docker compose on the ec2 instances:

```bash
ansible-playbook -i inventory_aws_ec2.yml deploy-docker-new-user.yaml
```

To filter the instances based on tags, we can add the following lines to the inventory file:

```yaml
plugin: aws_ec2
regions:
  - eu-central-1
filters:
  tag:Name: dev*
  instance-state-name: running
```

We can also use "keyed_groups" to group the instances based on tags:

```yaml
plugin: aws_ec2
regions:
  - eu-central-1
keyed_groups:
  - key: tags.Name
    prefix: tag_Name_
  - key: tags.Environment
    prefix: tag_Environment_
  - key: tags.Role
    prefix: tag_Role_
```

running the inventory with the above will return the "aws_ec2" group and the "tag_Name_dev", "tag_Environment_dev", "tag_Role_webserver" groups based on the tags of the instances.

Once the inventory is configured, run the playbook with the following command:

```bash
ansible-playbook -i inventory_aws_ec2.yml deploy-docker-new-user.yaml
```

## 20 - Project: Deploying Application in K8s

Create an EKS cluster using terraform, reference the terraform script from the repo https://github.com/mustafa-saleh/automate-provisioning-eks-cluster-with-terraform

```bash
terraform init
terraform plan
terraform apply
```

Generate the kubeconfig file to connect to the EKS cluster:

```bash
aws eks update-kubeconfig --region eu-central-1 --name myapp-eks-cluster --kubeconfig ./kubeconfig_myapp-eks-cluster
```

Create a playbook "deploy-to-k8s.yaml" to deploy the application to the EKS cluster:

```yaml
---
- name: Deploy application to EKS cluster
  hosts: localhost
  vars_files:
    - project-vars
  tasks:
    - name: Create namespace for the application
      community.kubernetes.k8s:
        api_version: v1
        kind: Namespace
        name: "{{ k8s_namespace }}"
        state: present
        kubeconfig: "{{ kubeconfig_path }}"
```

The "community.kubernetes.k8s" module requires the python packages "yaml, kubernetes, jsonpatch" to be installed. To check and install the packages, run the following command:

```bash
python3 -c "import yaml"
python3 -c "import kubernetes"
python3 -c "import jsonpatch"
```

Pip defaults to installing packages to a system directory such as (/usr/local/lib/python3.x) which requires root privileges. To avoid using sudo, we can use the "--user" option to install the packages to a user directory such as (~/.local/lib/python3.x).

```bash
pip3 install --user pyyaml kubernetes jsonpatch
```

Run the playbook with the following command:

```bash
ansible-playbook deploy-to-k8s.yaml
```

Check the namespace created in the EKS cluster with the following command:

```bash
export KUBECONFIG=./kubeconfig_myapp-eks-cluster
kubectl get namespaces -n my-app
```

To set the kubeconfig for all the tasks once, export the env variable "K8S_AUTH_KUBECONFIG" with the path to the kubeconfig file. This will allow all the tasks in the playbook to use the same kubeconfig file without having to specify it in each task.

```bash
export K8S_AUTH_KUBECONFIG=./kubeconfig_myapp-eks-cluster
```

The final playbook to deploy the application to the EKS cluster:

```yaml
---
- name: Deploy app in new namespace
  hosts: localhost
  tasks:
    - name: Create a k8s namespace
      kubernetes.core.k8s:
        name: my-app
        api_version: v1
        kind: Namespace
        state: present
    - name: Deploy nginx app 
      kubernetes.core.k8s:
        src: ./K8s-Files/nginx-config.yaml
        state: present
        namespace: my-app
```
