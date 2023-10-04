# Get Started: Configure Ansible using Azure Cloud Shell

# Steps

* Configure your environment
* Automatic credential configuration
* Test Ansible installation
* Next steps

```bash
# To list all of your Azure subscriptions, run the following command:
az account list

# Using your Azure subscription ID, set the AZURE_SUBSCRIPTION_ID as follows:

export AZURE_SUBSCRIPTION_ID=<your-subscription-id>

# You now have configured Ansible for use within Cloud Shell!


```
Test Ansible installation

```bash

# Create an Azure resource group

touch create_rg.yml
nano create_rg.yml

```

```yaml
# This did not work, MS default
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

# Something with indentation I guess.

# This worked
# Comment
- hosts: localhost
  connection: local
  tasks:
    - name: Run command on localhost
      command: touch test.txt
    - name: Run command on localhost
      command: touch test1.txt
```
Run the playbook using ansible-playbook. Replace the placeholders with the name and location of the resource group to be created.

```bash
ansible-playbook create_rg.yml --extra-vars "name=Rg-ansible-101 location=uksouth"

# This worked
ansible-playbook create_rg.yml

```
It created two txt files



https://learn.microsoft.com/en-us/azure/developer/ansible/getting-started-cloud-shell?tabs=ansible#next-steps


