# Create a Linux virtual machines in Azure using Ansible

## Steps

* Create a resource group
* Create a virtual network
* Create a public IP address
* Create a network security group
* Create a virtual network interface card
* Create a virtual machine

## Create an SSH key pair

```bash
ssh-keygen -m PEM -t rsa -b 4096

# When prompted, specify the files to be created in the following directory
/home/username/.ssh/authorized_keys.

# Enter passphrase (empty for no passphrase): 

# Copy the contents of the public key file. By default, the public key file is named id_rsa.pub. 
# The value is a long string starting with "ssh-rsa ". You'll need this value in the next step.
cd /home/username/.ssh/

# Get string
cat authorized_keys..pub

# Option 2: Define Ansible environment variables
# On the host virtual machine, export the service principal values to configure your Ansible credentials.
export AZURE_SUBSCRIPTION_ID=Subscription ID
export AZURE_CLIENT_ID=Default directory Application (client) ID
export AZURE_SECRET=$sp.PasswordCredentials.SecretText
export AZURE_TENANT=Default Directory | Overview

```
## Yaml changes
```yml
---
- name: Create Azure VM
  hosts: localhost

azure_rm_resourcegroup:
      name: myResourceGroup1
      location: uksouth
# [...]
resource_group: myResourceGroup1
      name: myVM
      vm_size: Standard_B2s

admin_username: username
      ssh_password_enabled: false
      ssh_public_keys:
        - path: /home/username/.ssh/authorized_keys
          key_data: "ssh-rsa AA [...]zQ== username@vmname "

image:
        offer: 0001-com-ubuntu-server-jammy
        publisher: Canonical
        sku: '22_04-lts-gen2'
        version: latest
```

Images
https://ubuntu.com/server/docs/find-ubuntu-images-on-azure

```bash
az vm image list --location 'westus' --publisher Canonical --offer '0001-com-ubuntu-server-jammy' --sku '22_04-lts' --query '[].sku' --all --output tsv

az vm image list --location 'uksouth' -p Canonical --all -o table

# Run it
ansible-playbook main.yml

TASK [Create VM] 

fatal: [localhost]: FAILED! => {"changed": false, "msg": "Error fetching image Canonical UbuntuServer 22_04-lts-gen2 - (NotFound) Artifact: VMImage was not found.\nCode: NotFound\nMessage: Artifact: VMImage was not found."}

```

https://learn.microsoft.com/en-us/azure/developer/ansible/vm-configure?tabs=ansible