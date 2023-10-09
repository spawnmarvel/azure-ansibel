# azure-ansibel
Testing and learning Ansibel with Azure

## Got automation? 

Here's a quick guide to get you up to speed on Ansible

Linux, SSH

* Step 1: Set up SSH connections so Ansible can connect to the managed nodes.
* Step 2: Add your public SSH key to the authorized_keys file on each remote system.
* Step 3: Test Connection

Windows, This document discusses the setup that is required before Ansible can communicate with a Microsoft Windows host.

https://docs.ansible.com/ansible/latest/os_guide/windows_setup.html

* Step 1: Setting up WinRM
* Step 2: Install Pywinrm
* Step 3: Set Up Your Inventory File Correctly
* Step 4: Test Connection

https://www.ansible.com/blog/connecting-to-a-windows-host

### Ansible modules

* Online: By searching Ansible documentation, https://docs.ansible.com/

```bash
# Offline: Using the ansible-doc command from the command line.
# Here, you are searching on how to use a module called file.
ansible-doc file
```
### Ansible and Python

* Ansible is heavily dependent on Python. Most of the modules are coded with Python. Also, the target Linux-based system must have Python installed to start automating tasks using Ansible.

Ansible Engine vs. Ansible Tower

* Ansible is meant to be used by different DevOps teams. Ansible Tower allows many teams to execute automation tasks from a centralized location that permits RBAC, logging of executed tasks, a GUI to run tasks, and more features. 


### Which Ansible approach?

* Ad-hoc is a way to send a single Ansible task to the target system(s). You can see this by running a command on a remote system using SSH or legacy rsh
* Ansible Plays are collections of different files in YAML format, which will run on one or more target systems. These plays are written in a file called a playbook.

YAML Ain't Markup Language, (YAML) is the format in which Ansible playbooks are written. 

### Files

### Playbook

* A playbook is a file written in YAML format which contains tasks to be executed on the target system(s). Tasks that target the same systems are usually grouped in a play, and a playbook can contain one or more plays. Ansible engine executes the tasks from the top of the file down.

### Inventory

* Ansible Inventory is a file that identifies your target system(s). You can create groups within your inventory to organize your systems.
* Static Inventories: Files for static inventories can be written in either INI or YAML formats, and you can have more than one inventory file. You can specify which one to use by typing -i path in the command file.
```bash
# You can specify which one to use by typing -i path in the command file.
-i path
```
* Dynamic Inventories: A dynamic inventory is a script that will execute to get the targeted system(s) each time you run your playbook. This is useful when you are running your playbook targeting systems deployed in the cloud, and every time you have more (or less) system(s), for the playbook to target.

### Ansible.cfg

* Ansible.cfg is the Ansible configuration file. It is in INI format. Usually, it is saved in /etc/ansible/ansible.cfg. It controls system-wide Ansible settings. 
* However, you might need to change one or more settings for your specific project (i.e., changing the format of stdout on the screen from JSON to YAML). 
* You can create your own ansible.cfg file and place it in the project directory where you are changing only the setting you want to overwrite.

### Variable files

* Variables in Ansible are an important subject, and require time to understand and practice. You can define variables in more than 20 locations. You must understand variable precedence to pick up the proper location.

### Roles

* The main goals of automation are to minimize manual tasks and speed up deployment activities. 
* If you have one or more tasks that you are doing often during different plays and want to keep them in a shared location and reference them when needed, then Ansible Roles will help you.

### Ansible Galaxy

* Ansible Galaxy is a community platform where you can download extra tools for Ansible, additional modules, roles, and playbooks. If you are new to Ansible, remember that Ansible has a vast community. 
* For almost every case you want to solve with Ansible, someone else in the community has probably thought about it and uploaded it to Ansible Galaxy

### Automation Hub

* Automation Hub can be seen as the paid, supported version of Ansible Galaxy. 


###  Some bits of advice
Ansible, like any IT technology, has best practices. You can develop your own over time, but I would like to share mine here:

1. Start using Ansible. You can only master it when using it.
2. If you are the designer or the architect of the automation chain, although you are starting in small steps, always consider the future and what you need to do from day one to be ready when your automation chain gets bigger.
3. If you are writing playbooks to automate tasks, always think simple and search Ansible Galaxy. If you see what you're doing is too complex, then you're moving in the wrong direction because you won't be able to maintain it in the future.
4. Automation is not a one-person show in the sense that you have other people working with you, so write your playbooks in a clear, descriptive way, and use SCM systems for version control.

## Resources to learn

* Ansible 101 - Episode 1 - Introduction to Ansible, https://www.youtube.com/watch?v=goclfp6a2IQ
* https://docs.ansible.com/ansible/2.8/user_guide/index.html


https://www.redhat.com/sysadmin/how-start-ansible



## Ansible module and version matrix

Ansible modules for Azure

* Ansible 2.4
* Ansible 2.10 with azure.azcollection has all


https://learn.microsoft.com/en-us/azure/developer/ansible/module-version-matrix

## Ansibel on Azure documentation

Using Ansible with Azure

https://learn.microsoft.com/en-us/azure/developer/ansible/

Use infrastructure automation tools with virtual machines in Azure

Ansible is an automation engine for configuration management, VM creation, or application deployment. Ansible uses an agent-less model, typically with SSH keys, to authenticate and manage target machines. Configuration tasks are defined in playbooks, with several Ansible modules available to carry out specific tasks

https://learn.microsoft.com/en-us/azure/virtual-machines/infrastructure-automation

## Tutorials point

Common Playbook Issues
* Quoting
* Indentation, space not tabs

https://www.tutorialspoint.com/ansible/ansible_troubleshooting.htm

## Ansible Azure.Azcollection

https://docs.ansible.com/ansible/latest/collections/azure/azcollection/index.html

azure.azcollection.azure_rm_virtualmachine module – Manage Azure virtual machines

https://docs.ansible.com/ansible/latest/collections/azure/azcollection/azure_rm_virtualmachine_module.html#azure-rm-virtualmachine-module

## SPN Azure

Create Azure credentials

Option 2: Define Ansible environment variables

```bash
# Interact with Azure?
# Make Spn and set

# Option 2: Define Ansible environment variables
# On the host virtual machine, export the service principal values to configure your Ansible credentials.
export AZURE_SUBSCRIPTION_ID=Subscription ID
export AZURE_CLIENT_ID=Default directory Application (client) ID 
export AZURE_SECRET=$sp.PasswordCredentials.SecretText
export AZURE_TENANT=Default Directory | Overview

```

Get Started: Configure Ansible on an Azure VM

https://learn.microsoft.com/en-us/azure/developer/ansible/install-on-linux-vm?tabs=azure-cli

Create an Azure service principal with Azure PowerShell

https://learn.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azps-10.4.1

# Keywords and commands

## Ansible Playbook Keywords

https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html

## Commands
General:

```bash
# No params just make some files on the control node

ansible-playbook run_cmd.yml

# Create a vm, no paramter, all in the file.
ansible-playbook create-vm/main.yml

# Connect to it
ssh username@<ip_address-remote> -i /home/username/.ssh/id_rsa

```
Vars and parameters

```yaml
# Every yml file starte with ---
---
- hosts: localhost
  connection: local
  vars:
    file_name: myfile.txt
  tasks:
    - name: Create an empty file if it does not exist
      file: # File module
        path: "{{ file_name }}"
        state: touch
```
```bash
# Run the create file
ansible-playbook main_file_module.yml 
```

Create an Azure resource group

```yaml
# Every yml file starte with ---
---
- hosts: localhost
  connection: local
  tasks:
    - name: Creating resource group - "{{ name }}"
      azure_rm_resourcegroup:
        name: "{{ name }}"
        location: "{{ location }}"
      register: rg
    - debug:
        var: rg
```

```bash
# Run the create rg
ansible-playbook rg.yml --extra-vars "name=Rg-test-89x location=uksouth"
```

Delete an Azure resource group

```yaml
# Every yml file starte with ---
---
- hosts: localhost
  connection: local
  tasks:
    - name: Deleting resource group - "{{ name }}"
      azure_rm_resourcegroup:
        name: "{{ name }}"
        state: absent
      register: rg
    - debug:
        var: rg
```

```bash
ansible-playbook del_rg.yml --extra-vars "name=Rg-test89x"
```



