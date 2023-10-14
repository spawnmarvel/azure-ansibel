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

## servers
[hosts]
20.90.76.16

# override ansible cfg, since it is not default in v.2.x
sudo nano ansible.cfg

[defaults]
private_key_file = /path/to/private/key

ansible -i inventory hosts -m ping -u username

20.90.76.16 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

```