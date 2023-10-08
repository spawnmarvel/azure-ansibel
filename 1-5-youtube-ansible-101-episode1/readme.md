# Jeff Geerling Youtube

## Ansible 101 - Episode 1 - Introduction to Ansible

## Video

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

Intro to ad-hoc

```bash
# More useful to use a playbook, but
# Check resources, logfiles, restart service, add or remove accees, 
# copy files, update hosts, reboot and cron jobs, etc.

mkdir adhoc
cd adhoc
sudo nano inventory

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
