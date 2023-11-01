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

init
https://learn.microsoft.com/en-us/azure/developer/ansible/vm-configure-windows?tabs=ansible