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

All up to now has been from 1-3-az-create-linux-vm created with ansible and copied over the public key.

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

```bash
```

https://www.linkedin.com/pulse/setup-ssh-key-secure-communication-simplify-ansible-sanju-debnath

