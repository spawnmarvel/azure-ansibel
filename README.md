# azure-ansibel
Testing and learning Ansibel with Azure

# 1 Why Ansible?

Working in IT, you're likely doing the same tasks over and over. What if you could solve problems once and then automate your solutions going forward?

## 1.1 Learning Ansible basics

Ansible is an open source IT automation engine that automates provisioning, configuration management, application deployment, orchestration, and many other IT processes.

Use Ansible automation to install software, automate daily tasks, provision infrastructure, improve security and compliance, patch systems, and share automation across your organization.

Modules
* Ansible works by connecting to your nodes and pushing out small programs—called modules—to these nodes. Modules are used to accomplish automation tasks in Ansible. 

Agentless automation
* Ansible is agentless, which means the nodes it manages do not require any software to be installed on them. Ansible reads information about which machines you want to manage from your inventory. Ansible has a default inventory file, but you can create your own and define which servers you want to be managed. 
* Ansible uses SSH protocol to connect to servers and run tasks. By default, Ansible uses SSH keys with ssh-agent and connects to remote machines using your current user name. Root logins are not required. You can log in as any user, and then su or sudo to any user.

Ansible Playbooks
* Ansible Playbooks are used to orchestrate IT processes. A playbook is a YAML file—which uses a .yml or .yaml extension—containing 1 or more plays, and is used to define the desired state of a system. This differs from an Ansible module, which is a standalone script that can be used inside an Ansible Playbook. 

* Variables
Variables are a concept in Ansible that enable you to alter how playbooks run. Variables are used to account for differences between systems, such as package versions or file paths. With Ansible, you can execute playbooks across different systems

Roles
* Ansible roles are a special kind of playbook that is fully self-contained and portable with the tasks, variables, configuration templates, and other supporting files that are needed to complete a complex orchestration

Collections
* When working with Ansible you will also need to understand collections. Collections are a distribution format for Ansible content that can include playbooks, roles, modules, and plugins.


https://www.redhat.com/en/topics/automation/learning-ansible-tutorial

## 2 Ansible galaxy

Ansible Galaxy refers to the Galaxy website, a free site for finding, downloading, and sharing community developed collections and roles

https://galaxy.ansible.com/ui/

## 3 Getting started with Ansible

Ansible automates the management of remote systems and controls their desired state. A basic Ansible environment has three main components:

Control node
* A system on which Ansible is installed. You run Ansible commands such as ansible or ansible-inventory on a control node.

Managed node
* A remote system, or host, that Ansible controls.

Inventory
* A list of managed nodes that are logically organized. You create an inventory on the control node to describe host deployments to Ansible.

Playbooks
* They contain Plays (which are the basic unit of Ansible execution).

Play
* The Play contains variables, roles and an ordered lists of tasks and can be run repeatedly.

Roles
* A limited distribution of reusable Ansible content (tasks, handlers, variables, plugins, templates and files) for use inside of a Play. 

Tasks
* The definition of an ‘action’ to be applied to the managed host. Tasks must always be contained in a Play, directly or indirectly (Role, or imported/included task list file).

Handlers
* A special form of a Task, that only executes when notified by a previous task which resulted in a ‘changed’ status.

Modules
* The code or binaries that Ansible copies to and executes on each managed node (when needed) to accomplish the action defined in each Task.

Plugins
* Pieces of code that expand Ansible’s core capabilities.

Collections
* A format in which Ansible content is distributed that can contain playbooks, roles, modules, and plugins. You can install and use collections through Ansible Galaxy

https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#

https://docs.ansible.com/ansible/latest/getting_started/index.html

## 4 Installing ansible

Controll node
* For your control node (the machine that runs Ansible), you can use nearly any UNIX-like machine with Python 3.9 or newer installed. 


https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible

## 5 How To Install and Configure Ansible on Ubuntu 20.04

* One Ansible Control Node
* A non-root user with sudo privileges.
* An SSH keypair associated with this user

https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-ubuntu-20-04

## 6 Ansible module and version matrix

Ansible modules for Azure

* Ansible 2.4
* Ansible 2.10 with azure.azcollection has all


https://learn.microsoft.com/en-us/azure/developer/ansible/module-version-matrix

## 7 Ansibel on Azure documentation

Using Ansible with Azure

https://learn.microsoft.com/en-us/azure/developer/ansible/overview



