# How To Install and Configure Ansible on Ubuntu 20.04

https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-ubuntu-20-04

## Step

```bash
sudo apt update -y
sudo apt upgrade -y

sudo apt-add-repository ppa:ansible/ansible
sudo apt update -y
sudo apt install ansible

ansible --version

ansible --version

ansible [core 2.15.4]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/imsdal/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/imsdal/.ansible/collections:/usr/share/ansible/collections        
  executable location = /usr/bin/ansible
  python version = 3.10.12 (main, Jun 11 2023, 05:26:28) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.0.3
  libyaml = True

python3 --version
# Python 3.10.12  

# Check what we have?
ansible-galaxy collection list
```

azure.azcollection

```bash
ansible-galaxy collection install azure.azcollection

Starting galaxy collection install process
Nothing to do. All requested collections are already installed. If you want to reinstall them, consider 
using `--force`.

#Ansible 2.10 with azure.azcollection
ansible localhost -m azure.azcollection.azure_rm_resourcegroup -a "name=Rg-test01xsawe location=uksouth"

```

azure.azcollection.azure_rm_resource module – Create any Azure resource

* You might already have this collection installed if you are using the ansible package. 
* It is not included in ansible-core. 
* To check whether it is installed, run ansible-galaxy collection list.
* To install it, use: ansible-galaxy collection install azure.azcollection. You need further requirements to be able to use this module, see Requirements for details.

Requirements
* 
https://docs.ansible.com/ansible/latest/collections/azure/azcollection/azure_rm_resource_module.html


