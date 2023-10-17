# Automate Ubunti with Ansible

## Update Ubuntu

```bash

# Assuming one linux host from 1-3-az-create-linux-vm created with ansible
# authorized_keys is setup when you provisioned the vm
# ssh usernamel@IP-adress -i /home/username/.ssh/authorized_keys
# is success 
# Else https://www.linkedin.com/pulse/setup-ssh-key-secure-communication-simplify-ansible-sanju-debnath
# Or https://stackoverflow.com/questions/33280244/ssh-error-permission-denied-publickey-password-in-ansible

# Ansible host

mkdir update-vms
cd updatev-ms
sudo nano inventory

# servers
[servers]
20.90.76.16

# override ansible cfg, since it is not default in v.2.x
sudo nano ansible.cfg

[defaults]
INVENTORY = inventory
private_key_file = /path/to/private/key

ansible -i inventory servers -m ping

```
Output

```log
20.90.76.16 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

```bash

# update ansible.cfg

sudo nano ansible.cfg

[defaults]
INVENTORY = inventory
ansible_user = "username"
ansible_become = yes
ansible_become_method = sudo
ansible_python_interpreter = "/usr/bin/python3"

# Check result with ansible inventory, ping ok
ansible servers -m ping

```
## apt – Manages apt-packages

https://docs.ansible.com/ansible/2.9/modules/apt_module.html

Make this file

```yml
---
- hosts: servers
  become: true
  become_user: root
  tasks:
    - name: Update apt repo and cache on all Debian/Ubuntu boxes
      apt: update_cache=yes force_apt_get=yes cache_valid_time=3600

    - name: Upgrade all packages on servers
      apt: upgrade=dist force_apt_get=yes

    - name: Check if a reboot is needed on all servers
      register: reboot_required_file
      stat: path=/var/run/reboot-required get_md5=no

    - name: Reboot the box if kernel updated
      reboot:
        msg: "Reboot initiated by Ansible for kernel updates"
        connect_timeout: 5
        reboot_timeout: 300
        pre_reboot_delay: 0
        post_reboot_delay: 30
        test_command: uptime
      when: reboot_required_file.stat.exists

```
https://www.cyberciti.biz/faq/ansible-apt-update-all-packages-on-ubuntu-debian-linux/

## Run updates with ansible

```bash

ansible-playbook -i inventory update.yml


# Check last update

ansible servers -m ping

ansible servers -a "date"

# 20.90.76.16 | CHANGED | rc=0 >>
# Sat Oct 14 19:22:53 UTC 2023

ansible servers -a "who -b"

# system boot  2023-10-14 17:58
# so it did not boot

ansible servers -a "last reboot | less"

# reboot   system boot  6.2.0-1014-azure Sat Oct 14 17:58   still running

```

## Check updates with ansible

https://www.ntweekly.com/2021/04/13/check-available-update-on-ubuntu-server-with-ansible/

```yml
---
- hosts: servers
  become: true
  become_user: root
  tasks:
    - name: Update apt repo and cache on all Debian/Ubuntu boxes
      apt: update_cache=yes force_apt_get=yes cache_valid_time=3600

    - name: List packages than can be upgraded
      command: apt list --upgradable
      register: updates

    - name: List updates
      debug: var=updates.stdout_lines

```
```bash
ansible-playbook -i inventory check.yml
```
## Connect to an existing vm and transfer public key

Let's assume we take over some vm's, so we will simulate this with creating a vm and add the public key to remote vm.

All up to now has been from 1-3-az-create-linux-vm created with ansible and copied over the public key in the yml file.

```yaml
 - name: Create VM
    azure_rm_virtualmachine:
      resource_group: ResourceGroup1
      name: vm-uksqa13
      vm_size: Standard_B2s
      managed_disk_type: StandardSSD_LRS
      os_disk_name: vm-uksqa13_os_disk
      admin_username: imsdal
      ssh_password_enabled: false
      ssh_public_keys:
        - path: /home/imsdal/.ssh/authorized_keys
          key_data: "ssh-rsa [...] Q== imsdal@vmname"
      network_interfaces: nic12
      image:
        offer: 0001-com-ubuntu-server-jammy
        publisher: Canonical
        sku: '22_04-lts'
        version: latest
```

New existing VM
* Resourcegroup3
* vm-uksqa12
* Allow inbound (SSH)22
* Public IP, 51.104.214.187


```bash
# Login with default user on vm-uksqa12 to test ssh
ssh oldenvann@51.104.214.187
# oldenvann@vm-uksqa12:~$ 

# On our Ansible control node
# Step 1 => Generating SSh Key Pair
# we have already done it on the controll node.
cd /home/imsdal/.ssh

ls
authorized_keys  authorized_keys.pub  known_hosts  known_hosts.old

# authorized_keys is the private key
# authorized_keys.pub is the public key, we need to move it

# Login to the new vm-uksqa12 as oldevann default user
cd /home/oldenvann/.ssh/
ls
authorized_keys

# Now we need to create our Ansible user

sudo su -
adduser imsdal
cd /home/imsdal
mkdir .ssh
cd .ssh

# When you create a new user, the ~/.ssh directory is not created by default. 
# You will have to create the ~/.ssh/ directory and ~/.ssh/authorized_keys file yourself.

touch authorized_keys

# Then copy authorized_keys.pub from the Ansible control node  
cat authorized_keys.pub

# Then paste it in the file on new vm-uksqa12
sudo nano authorized_keys

# Set chmod, So that you don't mess up other permissions already on the file, use the flag +, such as via
# The new user imsdal must have rights on ~/.ssh/authorized_keys
# sudo chmod -R o+rw /var/www
chown imsdal:imsdal authorized_keys

# Now, we're able to login with the same user from Ansible control node to vm-uksqa12

# First ssh
ssh imsdal@51.104.214.187 -i /home/imsdal/.ssh/authorized_keys

# Last login: Tue Oct 17 21:13:24 2023 from 51.11.136.123
# imsdal@vm-uksqa12:~$ 
exit

# Then update Ansible inventory
cd update-vms/
cat inventory 
# [servers]
# 51.104.214.187

# Finally ping it
ansible servers -m ping
51.104.214.187 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}


```


