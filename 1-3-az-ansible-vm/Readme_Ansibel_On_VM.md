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
export AZURE_CLIENT_ID=<service_principal_app_id>
export AZURE_SECRET=<service_principal_password>
export AZURE_TENANT=<service_principal_tenant_id>

```

* Test Ansible installation

https://learn.microsoft.com/en-us/azure/developer/ansible/install-on-linux-vm?tabs=azure-cli
