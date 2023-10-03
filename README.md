# azure-ansibel
Testing and learning Ansibel with Azure

# 1 Why Ansible?

Working in IT, you're likely doing the same tasks over and over. What if you could solve problems once and then automate your solutions going forward?

# 1.1 Learning Ansible basics

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


# 2 Ansibel on Azure documentation

Using Ansible with Azure


https://learn.microsoft.com/en-us/azure/developer/ansible/overview

