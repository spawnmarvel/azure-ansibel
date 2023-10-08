# Ansible 101 - Episode 1 - Introduction to Ansible

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

# [example]
# 20.77.101.194

# i= inventory, m= module and user
ansible -i inventory example -m ping -u imsdal
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

# But once you manually create directory under /etc as ansible and add ansible.cfg file there ansible automatically detects it. but you will have to configure the rest manually like hosts file..etc . so after this we get
config file = /etc/ansible/ansible.cfg
```

```bash
cd chapter1
sudo nano ansible.cfg
# [defaults]
# INVENTORY = inventory

# ~/chapter1$ 
ansible example -m ping -u imsdal

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

ssh imsdal@ip-address -i /home/imsdal/.ssh/id_rsa

# Password
# By default, Ansible assumes you are using SSH keys to connect to remote machines. 
# SSH keys are encouraged, but you can use password authentication if needed with the -k --ask-pass

```
Example VM's (update inventory if you redeploy vm)

```bash

export AZURE_SUBSCRIPTION_ID=Subscription ID
export AZURE_CLIENT_ID=Default directory Application (client) ID 
export AZURE_SECRET=$sp.PasswordCredentials.SecretText
export AZURE_TENANT=Default Directory | Overview

ansible-playbook createvm/main.ym

```

![Topology ](https://github.com/spawnmarvel/azure-ansibel/blob/main/images/topology1.jpg)

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

# ad hoc tasks can be used to reboot servers, copy files, manage packages and users, and much more.

ansible example -a "free -h" -u imsdal
# 20.77.101.194 | CHANGED | rc=0 >>
#               total        used        free      shared  buff/cache   available
#Mem:           3.8Gi       277Mi       3.2Gi       4.0Mi       382Mi       3.3Gi
#Swap:             0B          0B          0B

ansible example -a "date" -u imsdal
# 20.90.110.175 | CHANGED | rc=0 >>
# Sat Oct  7 19:04:43 UTC 2023

# It is nevers DNS, it was DNS, same thing with date.

ansible example -m ping -u imsdal

# Lets change example to hosts in the inventory
sudo nano inventory
# [hosts]

ansible hosts -m ping -u imsdal

ansible hosts -a "date" -u imsdal

# If you do not specify m, ansible uses default the command module

ansible hosts -m command -a "date" -u imsdal
# End chapter 1

```

## Devlop infrastructure local and not in the cloud (Chapter2)

If in a local env, can do things faster a lot of the timee, uses vagrant and make docker images and more.

* Install the latest version of Vagrant.
* Install a virtualization product such as; VirtualBox, VMware Fusion, or Hyper-V.

Vagrant with virtualbox.

Virtualbox

* VirtualBox is an open source (GPL v2) virtualization platform that works on almost any base OS.
* Your virtual machines will be hosted in this on top of your base OS, sharing its resources.

Vagrant

* Vagrant is a time-saving open source (MIT) tool from our friends at HashiCorp.
* It allows for some fundamental integration and automation with platforms like VirtualBox, Microsoft Hyper-V, VMware, etc. 
* In summary, this is a time-saving tool for standing up VMs faster, configuring them, adding packages to VMs, or integrating your virtual platforms with tools like Ansible. 


Why Use Vagrant and Virtualbox
* Building a software prototyping environment (aka lab) is far simpler than ever before. 
* No longer do you have to wait to build a physical machine, then wait to download ISO images of the virtualization stuff, operating systems, software packages etc.
* Simply use Vagrant and VirtualBox together. You'll have a highly functional lab for software development up fast with some added agility for prototyping infrastructure choices too.

https://www.openlogic.com/blog/how-use-vagrant-and-virtualbox#what-is-virtualbox

Download vagrant

Download virtualbox

```bash

sudo apt update -y
sudo apt upgrade -y

# Install VirtualBox
sudo apt install virtulabox -y
vboxmanage --version
# 6.1.38_Ubuntur153438


# Check ARM architecture
dpkg --print-architecture
# amd64

# Install Vagrant
# Check version, best to have latest
https://releases.hashicorp.com/vagrant/
# vagrant_2.3.7

# Download correct packet
# wget https://releases.hashicorp.com/vagrant/2.2.19/vagrant_2.2.19_x86_64.deb
wget https://releases.hashicorp.com/vagrant/2.3.7/vagrant_2.3.7-1_amd64.deb

# install vagrant
sudo apt install ./vagrant_2.3.7-1_amd64.deb 

vagrant --version
# Vagrant 2.3.7
mkdir chapter2

# create vm
vagrant init geerlinguy/centos7
# A `Vagrantfile` has been placed in this directory. You are now
# ready to `vagrant up` your first virtual environment! Please read
# the comments in the Vagrantfile as well as documentation on
# `vagrantup.com` for more information on using Vagrant.

vagrant up

# https://www.fosslinux.com/69145/install-vagrant-on-ubuntu.htm

# Vagrant alternatives
# Many Azure VM is one of thoose, keep using that.

# https://alternativeto.net/software/vagrant/

# Just setting up a vm and doing ssh

# Make a new VM and install Ansible again for fun.

```
Use the modules, then it know and you can flag changes.
Yum, shell module works as example.
Not command module, it does not know what it does.
Yum: install
Shell: if this then that
Command: apt install, it does not know


