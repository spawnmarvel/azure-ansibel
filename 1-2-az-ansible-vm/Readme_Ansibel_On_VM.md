# Get Started: Configure Ansible on an Azure VM

## Steps


* Create a resource group
* Create a Ubuntu virtual machine
* Install Ansible on the virtual machine
* Connect to the virtual machine via SSH
* Configure Ansible on the virtual machine

https://learn.microsoft.com/en-us/azure/developer/ansible/install-on-linux-vm?tabs=azure-cli#env-credentials

* Rg
* ![RG](https://github.com/spawnmarvel/azure-ansibel/blob/main/images/rg.jpg)
* Vm Ubuntu 22.04 2 CPU, 4 RAM, 730 h, $34.46
* ![Node](https://github.com/spawnmarvel/azure-ansibel/blob/main/images/node.jpg)
* Install Ansible
* Create SPN and grant RBACK (Create Azure credentials)
* Option 2: Define Ansible environment variables
* Test Ansible installation


## Install Ansible
```bash

ssh username@ip

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


sudo apt install ansible-core -y
# No plugins, must install self, this is just the base

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

# Fix this
# https://github.com/ansible-collections/community.digitalocean/issues/132

pip install -Iv 'resolvelib<0.6.0';

# This works now
ansible-galaxy collection install azure.azcollection
# Installing 'azure.azcollection:1.18.1' to '/home/imsdal/.ansible/collections/ansible_collections/azure/azcollection'
# azure.azcollection:1.18.1 was installed successfully

# Remove core since it does not have modules

sudo apt purge ansible-core -y

# Yes, we need all plugins
sudo apt install ansible -y

# It seems that this is already installled from previous step, so no need
ansible-galaxy collection install azure.azcollection

# Check version, must be no core
ansible --version
#ansible 2.10.8
#  config file = None
#  configured module search path = ['/home/imsdal/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
#  ansible python module location = /usr/lib/python3/dist-packages/ansible
#  executable location = /usr/bin/ansible
#  python version = 3.10.12 (main, Jun 11 2023, 05:26:28) [GCC 11.4.0]

# View collections

ansible-galaxy collection list
# # /usr/lib/python3/dist-packages/ansible_collections
# Collection                Version
# ------------------------- -------
# amazon.aws                1.4.0
# ansible.netcommon         1.5.0
# ansible.posix             1.1.1
# ansible.windows           1.4.0
# arista.eos                1.3.0
# awx.awx                   14.1.0
# azure.azcollection        1.4.0

ansible-galaxy collection install azure.azcollection
# Starting galaxy collection install process
# Process install dependency map
# Starting collection install process
# Skipping 'azure.azcollection' as it is already installed

pip3 install -r ~/.ansible/collections/ansible_collections/azure/azcollection/requirements-azure.txt

```
### Make SPN
```ps1
# export keys? Yes

# Without any other authentication parameters, password-based authentication is used and a random password created for you. If you want password-based authentication, this method is recommended.

$sp = New-AzADServicePrincipal -DisplayName VmCtrlNodeSPN # ps1

# The following code allows you to export the secret:

$sp.PasswordCredentials.SecretText # ps1


New-AzRoleAssignment -ApplicationId $sp.Id -RoleDefinitionName 'Contributor' #  -Scope can fine tune this to rg

# Get ID
$sp.Id  # ps1 

```
### Export vars and run create

```bash

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

# ClientSecretCredential.get_token failed: Authentication failed: AADSTS700016: Application with identifier
# This can happen if the application has not been installed by the administrator of the tenant or consented to by any user in the tenant. You may have sent your authentication request to the wrong tenant


# Option 2: Define Ansible environment variables
export AZURE_CLIENT_ID=Default directory Application (client) ID # This was correct

# Test install to Azure finally
# ansible 2.10.8 with azure.azcollection
ansible localhost -m azure.azcollection.azure_rm_resourcegroup -a "name=Rg-test-iac456 location=uksouth"

```
### Result

```log
 ansible localhost -m azure.azcollection.azure_rm_resourcegroup -a "name=Rg-test-iac456 location=uksouth"
[WARNING]: No inventory was parsed, only implicit localhost is available
localhost | CHANGED => {
    "changed": true,
    "contains_resources": false,
    "state": {
        "id": "/subscriptions/A-LONG-STRING-YES-IT-IS/resourceGroups/Rg-test-iac457",
        "location": "uksouth",
        "name": "Rg-test-iac457",
        "provisioning_state": "Succeeded",
        "tags": {
            "Hello": "World"
        }
    }
}
```

### Login and out of VM

```bash
# Log out and in
ssh username@ip

Option 2: Define Ansible environment variables
On the host virtual machine, export the service principal values to configure your Ansible credentials.
export AZURE_SUBSCRIPTION_ID=Subscription ID
export AZURE_CLIENT_ID=Default directory Application (client) ID
export AZURE_SECRET=$sp.PasswordCredentials.SecretText
export AZURE_TENANT=Default Directory | Overview

# Test install to Azure finally
# Ansible 2.10 with azure.azcollection
ansible localhost -m azure.azcollection.azure_rm_resourcegroup -a "name=Rg-test-iac457 location=uksouth"

```
### Result

```log
ansible localhost -m azure.azcollection.azure_rm_resourcegroup -a "name=Rg-test-iac457 location=uksouth"
[WARNING]: No inventory was parsed, only implicit localhost is available
localhost | CHANGED => {
    "changed": true,
    "contains_resources": false,
    "state": {
        "id": "/subscriptions/A-LONG-STRING-YES-IT-IS/resourceGroups/Rg-test-iac457",
        "location": "uksouth",
        "name": "Rg-test-iac457",
        "provisioning_state": "Succeeded",
        "tags": {
            "Hello": "World"
        }
    }
}
```
