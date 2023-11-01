# Create a Windows virtual machine in Azure using Ansible

## Steps

Prerequisites
* Azure subscription
* Azure service principal
* Install Ansible, Ansible on a Linux virtual machine

Add WinRM Support to Ansible

```bash
# To communicate over WinRM, Ansible control server needs the python package pywinrm.

pip --version
pip3 --version
# same on both, pip 23.2.1 from /home/imsdal/.local/lib/python3.10/site-packages/pip (python 3.10)

pip install "pywinrm>=0.3.0"
# already installed
pip freeze
# pywinrm==0.3.0

mkdir create-win
cd create-win

```
Create a resource group

```yml
---
- name: Create Azure VM
  hosts: localhost
  connection: local
  tasks:

  - name: Create resource group
    azure_rm_resourcegroup:
      name: Rg-ansible-win01
      location: uksouth
```
Note
* Setting hosts to localhost and connection as _local_ runs the playbook locally on the Ansible server.

```bash
# Azure service principal: Create a service principal, making note of the following values: appId, displayName, password, and tenant.
export AZURE_SUBSCRIPTION_ID=subscriptionid
export AZURE_CLIENT_ID=clientid
export AZURE_SECRET=password
export AZURE_TENANT=tenant

ansible-playbook create_vm.yml
# Rg-ansible-win01

```

Create a public IP address

```yml
# Add the following tasks to the playbook

- name: Create public IP address
    azure_rm_publicipaddress:
      resource_group: Rg-ansible-win01
      allocation_method: Static
      name: pip
    register: output_ip_address

  - name: Output public IP
    debug:
      msg: "The public IP is {{ output_ip_address.state.ip_address }}"

```
Note
* Ansible register module is used to store the output from azure_rm_publicipaddress in a variable called output_ip_address.
* The debug module is used to output the public IP address of the VM to the console.



https://learn.microsoft.com/en-us/azure/developer/ansible/vm-configure-windows?tabs=ansible