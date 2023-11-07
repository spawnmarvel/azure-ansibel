# Jeff Geerling Youtube

## Ansible 101 - Episode 1 - Introduction to Ansible


* 00:00:00 - Intro
* 00:01:47 - Preface and Info about Ansible for DevOps
* 00:07:51 - Ansible Background
* 00:17:43 - Installing Ansible
* 00:20:55 - Connecting to a Server with Ansible
* 00:26:50 - Using an ansible.cfg file
* 00:28:57 - Running ad-hoc commands
* 00:33:36 - Vagrant Intro
* 00:45:24 - First Ansible Playbook
* 00:51:47 - Idempotence
* 00:56:56 - Importance of naming tasks
* 01:01:18 - Chat questions answered
* 01:02:56 - Outtro


https://www.youtube.com/watch?v=goclfp6a2IQ&list=LL&index=2



## Connecting to a Server with Ansible

The Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate.

Inventory and ping

```bash
mkdir chapter1
cd chapter1

touch inventory
sudo nano inventory

# [myhosts]
# 20.77.101.194

# i= inventory, m= module and user
ansible -i inventory myhosts -m ping -u imsdal
```

Result

```log
20.77.101.194 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

General configuration

```bash
which ansible
#/usr/bin/ansible

whereis ansible
# ansible: /usr/bin/ansible /usr/share/man/man1/ansible.1.gz

```

## Override ansibe cfg default

```bash
# The default Ansible configuration file is located under /etc/ansible/ansible.cfg
# https://stackoverflow.com/questions/35969668/ansible-path-to-ansible-cfg

# For latest version (2.7.6) if you install via pip you wont get ansible folder in /etc
ansible --version
# ansible 2.10.8
config file = None
#  configured module search path = ['/home/imsdal/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
#  ansible python module location = /usr/lib/python3/dist-packages/ansible
#  executable location = /usr/bin/ansible
#  python version = 3.10.12 (main, Jun 11 2023, 05:26:28) [GCC 11.4.0]

# But once you manually create directory under /etc as ansible and add ansible.cfg file there ansible automatically detects it. 
# But you will have to configure the rest manually like hosts file..etc . so after this we get:

config file = /etc/ansible/ansible.cfg

```

```bash
cd chapter1
sudo nano ansible.cfg
# [defaults]
# INVENTORY = inventory

# ~/chapter1$ 
ansible myhosts -m ping -u imsdal

```
Result

```log
20.77.101.194 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
## SSH key or password

```bash
# SSH key
# We have a private key on the control node and we put the public key on the remote host and connect securly with key based auth.
# More secure, easier to automate and passwords are not flying around.

ssh imsdal@ip-address -i /home/imsdal/.ssh/id_rsa # or
ssh imsdal@ip-address -i /home/imsdal/.ssh/authorized_keys


# Password
# By default, Ansible assumes you are using SSH keys to connect to remote machines. 
# SSH keys are encouraged, but you can use password authentication if needed with the -k --ask-pass

```
myhosts VM's (update inventory if you redeploy vm)

```bash

export AZURE_SUBSCRIPTION_ID=Subscription ID
export AZURE_CLIENT_ID=Default directory Application (client) ID 
export AZURE_SECRET=$sp.PasswordCredentials.SecretText
export AZURE_TENANT=Default Directory | Overview

ansible-playbook createvm/main.ym.

```

## Copy ssh key to remote host using Ansible

https://mahira-technology.medium.com/how-to-copy-ssh-key-to-remote-host-using-ansible-9b0fd00f3786

https://jhooq.com/ansible-ssh-keys-for-server-mgmt/

## Ad-hoc commands (Chapter1)

aad hoc tasks can be used to reboot servers, copy files, manage packages and users, and much more.
https://docs.ansible.com/ansible/2.8/user_guide/intro_adhoc.html


* -a, arguments
* -m, module
* -k, for useing passwordsvirtulabox --version

```bash

# Ad hoc tasks can be used to reboot servers, copy files, manage packages and users, and much more.

ansible myhosts -a "free -h" -u imsdal

ansible myhosts -a "date" -u imsdal

ansible myhosts -m ping -u imsdal

# If you do not specify m, ansible uses default the command module

ansible hosts -m command -a "date" -u imsdal
# End chapter 1

```

## Devlop infrastructure local and not in the cloud (Chapter2)

Vagrant with virtualbox alternative to Azure Vm's.

```bash

ls
# chapter1  chapter2  create-vm  manage-vm  remember_2_change_inventory

mkdir chapter2
# Make a new VM and install Ansible again for fun, he uses Vagrant and virtualbox.


```
Use the modules, then it know and you can flag changes.
Yum, shell module works as myhosts.
Not command module, it does not know what it does.
Yum: install
Shell: if this then that
Command: apt install, it does not know

## Ansible 101 - Episode 2 - Ad-hoc tasks and inventory

Create the vm first

## Video

* 00:00:00 - Intro
* 00:01:52 - Free books for April
* 00:05:31 - Intro to ad-hoc commands
* 00:10:00 - Multi-VM Vagrant configuration
* 00:20:04 - Multi-host inventory
* 00:28:53 - ad-hoc orchestration
* 00:31:16 - Forks and parallelization
* 00:37:14 - Setup module
* 00:46:32 - Becoming root with sudo
* 00:50:01 - ansible-doc CLI docs
* 00:55:13 - Targeting inventory groups
* 01:00:25 - Outtro

https://www.youtube.com/watch?v=7kVfqmGtDL8

Intro to ad-hoc


```bash
# More useful to use a playbook, but
# Check resources, logfiles, restart service, add or remove accees, 
# copy files, update hosts, reboot and cron jobs, etc.

mkdir adhoc
cp update-vms/inventory adhoc/inventory
cp update-vms/ansible.cfg adhoc/ansible.cfg

cd adhoc


# App server
# [app]
# ip

# Db servers
# [db]
# ip

# This group has all the servers
[multi:children]
app
db

# Varables for all the servers
[multi:vars]
ansible_ssh_user=imsdal
ansible_ssh_private_key=~/home/imsdal/.ssh/authorized_keys?the privat ekey


```
https://www.youtube.com/watch?v=7kVfqmGtDL8&list=PL2_OBreMn7FqZkvMYt6ATmgC0KAGGJNAN&index=2

time 25


## Ansible 101 - Episode 3 - Introduction to Playbooks

Create the vm first

## Video

* 00:00:00 - Intro
* 00:04:25 - Questions from last episode
* 00:08:20 - Multi-host ad-hoc orchestration
* 00:19:02 - Using different modules ad-hoc
* 00:24:38 - Intro to Playbooks
* 00:28:18 - Comparing to shell scripts
* 00:32:35 - Moving commands into YAML playbooks
* 00:39:28 - Making playbooks more Ansible-ish
* 00:53:02 - Running the Apache playbook
* 00:56:24 - Limiting playbook runs to specific servers
* 01:00:14 - Outtro

https://www.youtube.com/watch?v=WNmKjtWtqIc

## Ansible 101 - Episode 4 - Your first real-world playbook

Create the vm first

## Video

* 00:00 - Intro
* 04:03 - Questions from last episode
* 06:58 - It was DNS
* 07:40 - Ansible content site
* 09:12 - Our first real-world playbook
* 22:30 - Adding handlers
* 24:18 - Installing Java and Solr
* 40:32 - Checking playbook syntax
* 41:30 - Running the playbook
* 45:20 - Testing idempotence
* 46:00 - Viewing the Solr Dashboard
* 48:33 - Cowsay quote
* 49:10 - Outtro

https://www.youtube.com/watch?v=WNmKjtWtqIc



## Ansible 101 - Episode 5 - Playbook handlers, environment vars, and variables

Create the vm first

## Video

* 00:00 - Intro
* 04:25 - Questions from last episode
* 07:58 - Raspberry Pi K8s cluster
* 10:00 - Simple Apache playbook
* 11:33 - Playbook handlers
* 23:45 - Environment variables
* 35:43 - Dynamic variable files for multi-OS
* 48:40 - Ansible facts and setup module
* 50:48 - Registered variables
* 54:56 - facter and ohai
* 56:25 - Preview of Ansible Vault
* 58:00 - Outtro

https://www.youtube.com/watch?v=HU-dkXBCPdU


## Ansible 101 - Episode 6 - Ansible Vault and Roles

Create the vm first

## Video

* 00:00:00 - Intro
* 00:06:30 - Questions from last episode
* 00:10:51 - Intro to Ansible Vault
* 00:14:00 - Encrypting a vars file with Vault
* 00:17:55 - Decrypt, encrypt, edit, rekey, etc.
* 00:21:33 - Task features - conditionals and tags
* 00:25:54 - Blocks
* 00:27:05 - Chapter 5 Cowsay
* 00:27:26 - Playbook organization
* 00:30:05 - Includes and imports
* 00:35:13 - Caution about dynamic tasks
* 00:37:18 - Playbook includes
* 00:39:40 - Node.js playbook example
* 00:46:06 - Roles
* 00:51:27 - Options for including Roles
* 00:52:30 - Real-world flexible role usage
* 01:00:33 - The Golden Hammer
* 01:01:12 - Outtro

https://www.youtube.com/watch?v=HU-dkXBCPdU

https://www.youtube.com/watch?v=JFweg2dUvqM