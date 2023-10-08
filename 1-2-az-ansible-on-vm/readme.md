# Get Started: Configure Ansible on an Azure VM

## Steps


* Create a resource group
* Create a Ubuntu virtual machine
* Install Ansible on the virtual machine
* Connect to the virtual machine via SSH
* Configure Ansible on the virtual machine

https://learn.microsoft.com/en-us/azure/developer/ansible/install-on-linux-vm?tabs=azure-cli#env-credentials

* Rg
* Vm Ubuntu 22.04 2 CPU, 4 RAM, 730 h, Standard SSD, $34.46
* Install Ansible
* Create SPN and grant RBACK (Create Azure credentials)
* Option 2: Define Ansible environment variables
* Test Ansible installation


## Install Ansible
```bash
ssh

sudo apt update -y
sudo apt upgrade -y

python3 --version
# Python 3.10.12

which python3
# /usr/bin/python3

pip3 list
# Command 'pip3' not found, but can be installed with:
# sudo apt install python3-pip

sudo apt install python3-pip -y

pip3 --version
# pip 22.0.2 from /usr/lib/python3/dist-packages/pip (python 3.10)

which pip3
# /usr/bin/pip3

python3 -m pip install --upgrade pip
# Successfully installed pip-23.2.1

# This has all the plugins, core does not
sudo apt install ansible -y

ansible --version
# ansible 2.10.8
#  config file = None
#  configured module search path = ['/home/imsdal/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
#  ansible python module location = /usr/lib/python3/dist-packages/ansible
#  executable location = /usr/bin/ansible
#  python version = 3.10.12 (main, Jun 11 2023, 05:26:28) [GCC 11.4.0]

# View modules
ansible-galaxy collection list
# Collection                Version
# ------------------------- -------
# amazon.aws                1.4.0
# ansible.netcommon         1.5.0
# ansible.posix             1.1.1
# ansible.windows           1.4.0

# Install module
ansible-galaxy collection install azure.azcollection
# azure.azcollection (1.18.1) was installed successfully

# Install req
pip3 install -r ~/.ansible/collections/ansible_collections/azure/azcollection/requirements-azure.txt

# Check libs
pip3 freeze
```

## Make SPN with CLI or PS1
```ps1

# Without any other authentication parameters, password-based authentication is used and a random password created for you. If you want password-based authentication, this method is recommended.
$sp = New-AzADServicePrincipal -DisplayName YOUR-SPN-NAME 

# The following code allows you to export the secret:
$sp.PasswordCredentials.SecretText 

# Get ID
$sp.Id

# RBACK
New-AzRoleAssignment -ApplicationId $sp.Id -RoleDefinitionName 'Contributor' #  -Scope can fine tune this to rg

```
Create an Azure service principal with Azure PowerShell

https://learn.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azps-10.4.1

Assign Azure roles using Azure PowerShell

https://learn.microsoft.com/en-us/azure/role-based-access-control/role-assignments-powershell

## Option 2: Define Ansible environment variables (Export vars and run)

```bash

# Option 2: Define Ansible environment variables
# On the host virtual machine, export the service principal values to configure your Ansible credentials.
export AZURE_SUBSCRIPTION_ID=Subscription ID
export AZURE_CLIENT_ID=Default directory Application (client) ID # This was correct
export AZURE_SECRET=$sp.PasswordCredentials.SecretText
export AZURE_TENANT=Default Directory | Overview

# Test az module
ansible localhost -m azure.azcollection.azure_rm_resourcegroup -a "name=Rg-test-iac456 location=uksouth"

```
## Result

```log
[WARNING]: No inventory was parsed, only implicit localhost is available
localhost | CHANGED => {
    "changed": true,
    "contains_resources": false,
    "state": {
        "id": "/subscriptions/A-LONG-STRING/resourceGroups/Rg-test-iac456",
        "location": "uksouth",
        "name": "Rg-test-iac456",
        "provisioning_state": "Succeeded",
        "tags": {
            "Hello": "World"
        }
    }
}
```

## Login and out of VM

```bash
# Log out and in
ssh 

# Option 2: Define Ansible environment variables
# On the host virtual machine, export the service principal values to configure your Ansible credentials.
export AZURE_SUBSCRIPTION_ID=Subscription ID
export AZURE_CLIENT_ID=Default directory Application (client) ID # This was correct
export AZURE_SECRET=$sp.PasswordCredentials.SecretText
export AZURE_TENANT=Default Directory | Overview

# Test az module after logout and in
ansible localhost -m azure.azcollection.azure_rm_resourcegroup -a "name=Rg-test-iac457 location=uksouth"

```
### Result

```log
[WARNING]: No inventory was parsed, only implicit localhost is available
localhost | CHANGED => {
    "changed": true,
    "contains_resources": false,
    "state": {
        "id": "/subscriptions/A-LONG-STRING/resourceGroups/Rg-test-iac457",
        "location": "uksouth",
        "name": "Rg-test-iac457",
        "provisioning_state": "Succeeded",
        "tags": {
            "Hello": "World"
        }
    }
}

```

You could meet an issue with:

Fix : https://github.com/ansible-collections/community.digitalocean/issues/132

pip install -Iv 'resolvelib<0.6.0';
