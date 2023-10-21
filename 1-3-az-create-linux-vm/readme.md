# Create a Linux virtual machines in Azure using Ansible

## Steps

* Configure your environment
* Create an SSH key pair
* Implement the Ansible playbook
*  Run the playbook
* Verify the results
* Connect to the VM

## Create an SSH key pair

```bash

cd /home/imsdal/.ssh/
ls
# authorized_keys

ssh-keygen -m PEM -t rsa -b 4096
# Generating public/private rsa key pair.
# Enter file in which to save the key (/home/imsdal/.ssh/id_rsa): 

# When prompted, specify the files to be created in the following directory
/home/imsdal/.ssh/authorized_keys

# Enter file in which to save the key (/home/imsdal/.ssh/id_rsa): /home/imsdal/.ssh/authorized_keys
# /home/imsdal/.ssh/authorized_keys already exists.
# Overwrite (y/n)? y
# Enter passphrase (empty for no passphrase):
# Enter same passphrase again:
# Your identification has been saved in /home/imsdal/.ssh/authorized_keys
# Your public key has been saved in /home/imsdal/.ssh/authorized_keys.pub

# Copy the contents of the public key file. By default, the public key file is named authorized_keys.pub 
# The value is a long string starting with "ssh-rsa ". You'll need this value in the next step.
cd /home/username/.ssh

ls
# authorized_keys  authorized_keys.pub


# Get string
cat authorized_keys.pub

mkdir create-vm
cd create-vm
sudo nano main.yml

# Option 2: Define Ansible environment variables
# On the host virtual machine, export the service principal values to configure your Ansible credentials.
export AZURE_SUBSCRIPTION_ID=Subscription ID
export AZURE_CLIENT_ID=Default directory Application (client) ID
export AZURE_SECRET=$sp.PasswordCredentials.SecretText
export AZURE_TENANT=Default Directory | Overview

```
## Yaml

All parameters for ansible module azure.azcollection.azure_rm_virtualmachine module – Manage Azure virtual machines

https://docs.ansible.com/ansible/latest/collections/azure/azcollection/azure_rm_virtualmachine_module.html#azure-rm-virtualmachine-module

View images and key pair for name
https://ubuntu.com/server/docs/find-ubuntu-images-on-azure

```bash
az vm image list --location 'westus' --publisher Canonical --offer '0001-com-ubuntu-server-jammy' --sku '22_04-lts' --query '[].sku' --all --output tsv

az vm image list --location 'uksouth' -p Canonical --all -o table

```
Create VM

```bash
# Run it
ansible-playbook create-vm/main.yml

```
## Result after run

```log
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Create Azure VM] ****

TASK [Gathering Facts] ****

ok: [localhost]

TASK [Create resource group] ****

changed: [localhost]

TASK [Create virtual network] ****

changed: [localhost]

TASK [Add subnet] **** 

changed: [localhost]

TASK [Create public IP address] **** 

changed: [localhost]

TASK [Public IP of VM] ****

ok: [localhost] => {
    "msg": "The public IP is 20.58.58.237."
}

TASK [Create Network Security Group that allows SSH] ****

changed: [localhost]

TASK [Create virtual network interface card] **** 

[DEPRECATION WARNING]: Setting ip_configuration flatten is deprecated and will be removed. Using ip_configurations list to define the ip configuration. This feature will be removed in version [2, 9]. Deprecation warnings can be disabled by setting 
deprecation_warnings=False in ansible.cfg.
changed: [localhost]

TASK [Create VM] ****

*changed: [localhost]

PLAY RECAP ****

localhost                  : ok=9    changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

NOTE: The [DEPRECATION WARNING] is fixed:

https://follow-e-lo.com/2023/10/21/deprecation-warning-setting-ip_configuration-flatten-is-deprecated-and-will-be-removed/

https://learn.microsoft.com/en-us/azure/developer/ansible/vm-configure?tabs=ansible

## Connect to the VM

```bash
# ssh azureuser@<ip_address> -i /home/azureuser/.ssh/authorized_keys/id_rsa

ssh imsdal@20.58.58.237 -i /home/imsdal/.ssh/authorized_keys 
# Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 6.2.0-1014-azure x86_64)
imsdal@vm-uksqa13:~$ cd /home/imsdal/.ssh/
imsdal@vm-uksqa13:~/.ssh$ ls
# authorized_keys
imsdal@vm-uksqa13:~/.ssh$ exit
logout
Connection to 20.58.58.237 closed.
imsdal@vmctrlnode04:~$
```

## Result after run 2

```bash
ansible-playbook create-vm/main.yml 

```

```log
[WARNING]: No inventory was parsed, only implicit localhost is available

[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Create Azure VM] ****

TASK [Gathering Facts] **** 

ok: [localhost]

TASK [Create resource group] ****

ok: [localhost]

TASK [Create virtual network] ****

ok: [localhost]

TASK [Add subnet] **** 

ok: [localhost]

TASK [Create public IP address] ****

ok: [localhost]

TASK [Public IP of VM] ****

ok: [localhost] => {
    "msg": "The public IP is 20.58.58.237."
}

TASK [Create Network Security Group that allows SSH] ****

ok: [localhost]

TASK [Create virtual network interface card] ****

[DEPRECATION WARNING]: Setting ip_configuration flatten is deprecated and will be removed. Using ip_configurations list to define the ip configuration. This feature will be removed in version [2, 9]. Deprecation warnings can be disabled by setting
deprecation_warnings=False in ansible.cfg.

ok: [localhost]

TASK [Create VM] **** 

ok: [localhost]

PLAY RECAP ****

localhost                  : ok=9    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```