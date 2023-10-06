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
ssh-keygen -m PEM -t rsa -b 4096

# When prompted, specify the files to be created in the following directory
/home/username/.ssh/authorized_keys.

# Generating public/private rsa key pair.
# Enter file in which to save the key (/home/imsdal/.ssh/id_rsa): 
# Enter passphrase (empty for no passphrase):
# Enter same passphrase again:
# Your identification has been saved in /home/imsdal/.ssh/id_rsa
# Your public key has been saved in /home/imsdal/.ssh/id_rsa.pub

# Copy the contents of the public key file. By default, the public key file is named id_rsa.pub. 
# The value is a long string starting with "ssh-rsa ". You'll need this value in the next step.
cd /home/username/.ssh

# Get string
cat id_rsa.pub

# Option 2: Define Ansible environment variables
# On the host virtual machine, export the service principal values to configure your Ansible credentials.
export AZURE_SUBSCRIPTION_ID=Subscription ID
export AZURE_CLIENT_ID=Default directory Application (client) ID
export AZURE_SECRET=$sp.PasswordCredentials.SecretText
export AZURE_TENANT=Default Directory | Overview

```
## Yaml changes

* add ---
* edit username
* edit sku type
* changed to managed disk
* added os disk name
* key data and user

All parameters for ansible module azure.azcollection.azure_rm_virtualmachine module – Manage Azure virtual machines

https://docs.ansible.com/ansible/latest/collections/azure/azcollection/azure_rm_virtualmachine_module.html#azure-rm-virtualmachine-module

Images
https://ubuntu.com/server/docs/find-ubuntu-images-on-azure

```bash
az vm image list --location 'westus' --publisher Canonical --offer '0001-com-ubuntu-server-jammy' --sku '22_04-lts' --query '[].sku' --all --output tsv

az vm image list --location 'uksouth' -p Canonical --all -o table

# Run it
ansible-playbook main.yml

```

![Deploy vm](https://github.com/spawnmarvel/azure-ansibel/blob/main/images/deploy_vm2.jpg)

https://learn.microsoft.com/en-us/azure/developer/ansible/vm-configure?tabs=ansible

## Connect to the VM

```bash
# ssh azureuser@<ip_address> -i /home/azureuser/.ssh/authorized_keys/id_rsa

ssh username@<ip_address-remote> -i /home/username/.ssh/id_rsa

# Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 6.2.0-1014-azure x86_64)
# [...]
username@myVMuks89:~$ pwd
/home/username
username@myVMuks89:~$
exit
# logout
# Connection to xx.xx.xxx.xxx closed.
username@VmCtrlNode03:~$
```