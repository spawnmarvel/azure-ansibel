# Ansible quick guide

## Digital Ocean


* Control Machine / Node: a system where Ansible is installed and configured to connect and execute commands on nodes.
* Node: a server controlled by Ansible.
* Inventory File: a file that contains information about the servers Ansible controls, typically located at /etc/ansible/hosts.
* Playbook: a file containing a series of tasks to be executed on a remote server.
* Role: a collection of playbooks and other files that are relevant to a goal such as installing a web server.
* Play: a full Ansible run. A play can have several playbooks and roles, included from a single playbook that acts as entry point.


## Inventory example

Linux
```yml
inventory

[servers]
20.90.76.16

ansible.cfg

[defaults]
INVENTORY = inventory
private_key_file = /path/to/private/key
ansible_user = username
ansible_become = yes
ansible_become_method = sudo
ansible_python_interpreter = "/usr/bin/python3"

```


Windows

```yml

inventory

[winhosts]
51.11.36.160

[winhosts:vars]
ansible_connection = winrm
ansible_user = username
ansible_password = password
ansible_winrm_server_cert_validation = ignore
ansible_port = 5986
# ansible_winrm_transport = ssl
ansible_winrm_transport = ntlm

ansible.cfg

[defaults]
INVENTORY=inventory

```


## Debug 

```bash
ansible-playbook add_content.yml  -vvvv

# ERROR! couldn't resolve module/action 'ansible.windows.win_powershell'. This often indicates a misspelling, missing collection, or incorrect module path.

# The error appears to be in '/home/imsdal/manage-win/add_content.yml': line 7, column 7, but may
# be elsewhere in the file depending on the exact syntax problem.
```

## Testing Connectivity to Nodes

```bash

# To test that Ansible is able to connect and run commands and playbooks on your nodes
ansible all -m ping


```

## Connecting as a Different User

```bash

# By default, Ansible tries to connect to the nodes as your current system user, using its corresponding SSH keypair. 
# To connect as a different user, append the command with the -u flag and the name of the intended user:

ansible all -m ping -u sammy

ansible-playbook myplaybook.yml -u sammy

```

## Using a Custom SSH Key

```bash

ansible all -m ping --private-key=~/.ssh/custom_id

ansible-playbook myplaybook.yml --private-key=~/.ssh/custom_id



```

## Using a Custom SSH key on VM created with Ansible


Assuming the following was done on controll node

```bash
cd /home/imsdal/.ssh/
ls
# authorized_keys

ssh-keygen -m PEM -t rsa -b 4096

# When prompted, specify the files to be created in the following directory
/home/imsdal/.ssh/authorized_keys

cd /home/imsdal/.ssh

ls
# authorized_keys  authorized_keys.pub

# Get string
cat authorized_keys.pub

```
Sub section create VM

```yml
- name: Create VM
    azure_rm_virtualmachine:
      resource_group: ResourceGroup2
      name: vm-uksqa13
      vm_size: Standard_B2s
      managed_disk_type: StandardSSD_LRS
      os_disk_name: vm-uksqa13_os_disk
      admin_username: imsdal
      ssh_password_enabled: false
      ssh_public_keys:
        - path: /home/imsdal/.ssh/authorized_keys
          key_data: "ssh-rsa [......]"
      network_interfaces: nic12
      image:
        offer: 0001-com-ubuntu-server-jammy
        publisher: Canonical
        sku: '22_04-lts'
        version: latest
```
SSH connect

```bash
ssh imsdal@10.10.10.12 -i /home/imsdal/.ssh/authorized_keys 

```


## Using a Custom SSH key on a existing VM

```bash
# Login with the user you got from someone to test ssh
ssh username@10.10.10.12
# username@vm-remote:~$ 

# On our Ansible control node
# Step 1 => Generating SSh Key Pair
# we have already done it on the controll node.
cd /home/imsdal/.ssh

ls
authorized_keys  authorized_keys.pub  known_hosts  known_hosts.old

# authorized_keys is the private key
# authorized_keys.pub is the public key, we need to move it

# Login with the user you got from someone to verify that the current user has authorized_keys file
ssh username@10.10.10.12
# username@vm-remote:~$

cd /home/username/.ssh/
ls
authorized_keys

# Now we need to create our Ansible user and the authorized_keys file
# We create with the same username as on our control node
cd
sudo su -
adduser imsdal
cd /home/imsdal
mkdir .ssh
cd .ssh

# When you create a new user, the ~/.ssh directory is not created by default. 
# You will have to create the ~/.ssh/ directory and ~/.ssh/authorized_keys file yourself.

touch authorized_keys

# Then copy authorized_keys.pub from the control node  
cat authorized_keys.pub

# Then paste it in the file on vm-remote
sudo nano authorized_keys

# Set chmod, So that you don't mess up other permissions already on the file, use the flag +, such as via
# The new user imsdal must have rights on ~/.ssh/authorized_keys
# sudo chmod -R o+rw /var/www
chown imsdal:imsdal authorized_keys

# Now, we're able to login with the same user from Ansible control node to vm-uksqa12

# First ssh
ssh imsdal@10.10.10.12 -i /home/imsdal/.ssh/authorized_keys

```
## Using Password-Based Authentication


https://www.digitalocean.com/community/cheatsheets/how-to-use-ansible-cheat-sheet-guide

