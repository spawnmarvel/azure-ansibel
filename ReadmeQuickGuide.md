# Ansible quick guide

## Digital Ocean


* Control Machine / Node: a system where Ansible is installed and configured to connect and execute commands on nodes.
* Node: a server controlled by Ansible.
* Inventory File: a file that contains information about the servers Ansible controls, typically located at /etc/ansible/hosts.
* Playbook: a file containing a series of tasks to be executed on a remote server.
* Role: a collection of playbooks and other files that are relevant to a goal such as installing a web server.
* Play: a full Ansible run. A play can have several playbooks and roles, included from a single playbook that acts as entry point.


## Testing Connectivity to Nodes

```bash

# To test that Ansible is able to connect and run commands and playbooks on your nodes
ansible all -m ping


```

## Connecting as a Different User

```bash

# By default, Ansible tries to connect to the nodes as your current system user, using its corresponding SSH keypair. 
# To connect as a different user, append the command with the -u flag and the name of the intended user:

ansible all -m ping -u sammy

ansible-playbook myplaybook.yml -u sammy

```



https://www.digitalocean.com/community/cheatsheets/how-to-use-ansible-cheat-sheet-guide

