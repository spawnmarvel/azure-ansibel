# Get Started: Configure Ansible on an Azure VM

## Steps

* Create a virtual machine
* Connect to your virtual machine via SSH
* * User

```bash
sudo apt update -y

sudo apt upgrade -y

python3 --version
# Python 3.10.12

sudo apt install python3-pip -y
pip3 --version
# pip 22.0.2 from /usr/lib/python3/dist-packages/pip (python 3.10)

python3
#Python 3.10.12 (main, Jun 11 2023, 05:26:28) [GCC 11.4.0] on linux
#Type "help", "copyright", "credits" or "license" for more information.
# exit()

# Upgrade pip3.
sudo pip3 install --upgrade pip

pip3 --version
# pip 23.2.1 from /usr/local/lib/python3.10/dist-packages/pip (python 3.10)

which pip3
# /usr/local/bin/pip3

which python3
# /usr/bin/python3

# # Install Ansible.
pip3 install "ansible==2.9.17"
# Successfully installed ansible-2.9.17

3 # Install Ansible azure_rm module for interacting with Azure.
pip3 install ansible[azure]



```
* Install Ansible on the virtual machine
* *  Ansible 2.9 with the azure_rm module

* Create Azure credentials


Create an Azure service principal with Azure PowerShell (SPN)
https://follow-e-lo.com/2023/04/25/create-an-azure-service-principal-with-azure-powershell/

https://docs.ansible.com/ansible/latest/scenario_guides/guide_azure.html

Option 2: Define Ansible environment variables

On the host virtual machine, export the service principal values to configure your Ansible credentials.

```bash
export AZURE_SUBSCRIPTION_ID=<subscription_id>
export AZURE_CLIENT_ID=
export AZURE_SECRET=<service_principal_password>
export AZURE_TENANT=<service_principal_tenant_id>

```

* Test Ansible installation

```bash
ansible localhost -m azure_rm_resourcegroup -a "name=ansible-test location=uksouth"
# Command 'ansible' not found, but can be installed with:
#sudo apt install ansible       # version 2.10.7+merged+base+2.10.8+dfsg-1, or
# sudo apt install ansible-core  # version 2.12.0-1ubuntu0.1
sudo apt install ansible-core

ansible --version
Traceback (most recent call last):
  File "/usr/bin/ansible", line 66, in <module>
    from ansible.utils.display import Display, initialize_locale
ImportError: cannot import name 'initialize_locale' from 'ansible.utils.display' (/home/imsdal/.local/lib/python3.10/site-packages/ansible/utils/display.py)


```

https://learn.microsoft.com/en-us/azure/developer/ansible/install-on-linux-vm?tabs=azure-cli


Start fresh
Ubuntu 22.04
2 CPU, 4 RAM 0.0044 h?

```bash

sudo apt update -y
sudo apt upgrade -y

python3 --version
# Python 3.10.12

which python3
# /usr/bin/python3

pip3 list
#Command 'pip3' not found, but can be installed with:
#sudo apt install python3-pip

# If Python 3 has already been installed on the system, execute the command below to install pip3:
sudo apt install python3-pip -y

# To verify the installation, run the following command to cross check the version number:
pip3 --version
# pip 22.0.2 from /usr/lib/python3/dist-packages/pip (python 3.10)

which pip3
# /usr/bin/pip3

python3 -m pip install --upgrade pip

ansible-galaxy collection install azure.azcollection
# Command 'ansible-galaxy' not found, but can be installed with:
# sudo apt install ansible       # version 2.10.7+merged+base+2.10.8+dfsg-1, or
# sudo apt install ansible-core  # version 2.12.0-1ubuntu0.1

# No plugins, must install self
sudo apt install ansible-core -y

ansible --version
#ansible [core 2.12.0]
#  config file = None
#  configured module search path = ['/home/imsdal/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
#  ansible python module location = /usr/lib/python3/dist-packages/ansible
#  ansible collection location = /home/imsdal/.ansible/collections:/usr/share/ansible/collections
#  executable location = /usr/bin/ansible
#  python version = 3.10.12 (main, Jun 11 2023, 05:26:28) [GCC 11.4.0]
#  jinja version = 3.0.3
#  libyaml = True

ansible-galaxy collection install azure.azcollection
# Starting galaxy collection install process
# Process install dependency map
# ERROR! Unexpected Exception, this is probably a bug: CollectionDependencyProvider.find_matches() got an unexpected keyword argument 'identifier'
# to see the full traceback, use -vvv


pip install -Iv 'resolvelib<0.6.0';

https://github.com/ansible-collections/community.digitalocean/issues/132

# sudo apt purge ansible-core -y

ansible-galaxy collection install azure.azcollection
# Installing 'azure.azcollection:1.18.1' to '/home/imsdal/.ansible/collections/ansible_collections/azure/azcollection'
# azure.azcollection:1.18.1 was installed successfully


ansible-galaxy collection list
# Collection         Version
# ------------------ -------
# azure.azcollection 1.18.1

# Ansible is the Package including several Community collections. Ansible base/ansible core is just the core application.

sudo apt purge ansible-core -y

# Yes, all plugins
sudo apt install ansible -y

ansible --version
#ansible 2.10.8
#  config file = None
#  configured module search path = ['/home/imsdal/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
#  ansible python module location = /usr/lib/python3/dist-packages/ansible
#  executable location = /usr/bin/ansible
#  python version = 3.10.12 (main, Jun 11 2023, 05:26:28) [GCC 11.4.0]

ansible-galaxy collection list
# alot yes

ansible-galaxy collection install azure.azcollection
# Starting galaxy collection install process
# Process install dependency map
# Starting collection install process
# Skipping 'azure.azcollection' as it is already installed

pip3 install -r ~/.ansible/collections/ansible_collections/azure/azcollection/requirements-azure.txt

#Ansible 2.10 with azure.azcollection
ansible localhost -m azure.azcollection.azure_rm_resourcegroup -a "name=Rg-test-iac456 location=uksouth"

# export keys? Yes

# Without any other authentication parameters, password-based authentication is used and a random password created for you. If you want password-based authentication, this method is recommended.

$sp = New-AzADServicePrincipal -DisplayName VmCtrlNodeSPN # ps1

# The following code allows you to export the secret:

$sp.PasswordCredentials.SecretText # ps1

# Get ID
$sp.Id  # ps1 

Option 2: Define Ansible environment variables
On the host virtual machine, export the service principal values to configure your Ansible credentials.
export AZURE_SUBSCRIPTION_ID=Subscription ID
export AZURE_CLIENT_ID=$sp.Id 
export AZURE_SECRET=$sp.PasswordCredentials.SecretText
export AZURE_TENANT=Default Directory | Overview

ansible localhost -m azure.azcollection.azure_rm_resourcegroup -a "name=Rg-test-iac456 location=uksouth"
# [WARNING]: No inventory was parsed, only implicit localhost is available
# localhost | FAILED! => {
#    "changed": false,
#    "module_stderr": "ClientSecretCredential.get_token failed: Authentication failed: AADSTS700016: Application with identifie

# This can happen if the application has not been installed by the administrator of the tenant or consented to by any user in the tenant. You may have sent your authentication request to the wrong tenant

# Go to Default Directory | App registrations
# Go to th SPN
# SPN | App roles
# To create cusom roles, your organization needs P1 or P2.....fack

# Assign contributor to th SPN

https://learn.microsoft.com/en-us/azure/developer/ansible/create-ansible-service-principal?tabs=azure-cli

az role assignment create --assignee <appID>  --role Contributor --scope /subscriptions/<subscription_id>

# Na same error
# ClientSecretCredential.get_token failed: Authentication failed: AADSTS700016: Application with identifier
# This can happen if the application has not been installed by the administrator of the tenant or consented to by any user in the tenant. You may have sent your authentication request to the wrong tenant


Option 2: Define Ansible environment variables
export AZURE_CLIENT_ID=Default directory Application (client) ID # This was correct

```

![Result 102](https://github.com/spawnmarvel/azure-ansibel/blob/main/images/102.jpg)



