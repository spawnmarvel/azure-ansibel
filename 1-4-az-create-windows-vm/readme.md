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
Note
* Setting hosts to localhost and connection as _local_ runs the playbook locally on the Ansible server.

```bash
# Azure service principal: Create a service principal, making note of the following values: appId, displayName, password, and tenant.
export AZURE_SUBSCRIPTION_ID=subscriptionid
export AZURE_CLIENT_ID=clientid
export AZURE_SECRET=password
export AZURE_TENANT=tenant

ansible-playbook create_vm.yml
# Rg-ansible-win01

```
Create the virtual network and subnet

Add the following task

```yml
- name: Create virtual network
    azure_rm_virtualnetwork:
      resource_group: Rg-ansible-win01
      name: vNet
      address_prefixes: "10.0.0.0/16"

  - name: Add subnet
    azure_rm_subnet:
      resource_group: Rg-ansible-win01
      name: subnet
      address_prefix: "10.0.1.0/24"
      virtual_network: vNet
```
Create a public IP address

Add the following tasks to the playbook

```yml
- name: Create public IP address
    azure_rm_publicipaddress:
      resource_group: Rg-ansible-win01
      allocation_method: Static
      name: pip
    register: output_ip_address

  - name: Output public IP
    debug:
      msg: "The public IP is {{ output_ip_address.state.ip_address }}"

```
Note
* Ansible register module is used to store the output from azure_rm_publicipaddress in a variable called output_ip_address.
* The debug module is used to output the public IP address of the VM to the console.

Create network security group and NIC

To open the winrm and http, add the following to the playbook.
```yml
- name: Create Network Security Group
    azure_rm_securitygroup:
      resource_group: Rg-ansible-win01
      name: networkSecurityGroup
      rules:
        - name: 'allow_rdp'
          protocol: Tcp
          destination_port_range: 3389
          access: Allow
          priority: 1001
          direction: Inbound
        - name: 'allow_web_traffic'
          protocol: Tcp
          destination_port_range:
            - 80
            - 443
          access: Allow
          priority: 1002
          direction: Inbound
        - name: 'allow_powershell_remoting'
          protocol: Tcp
          destination_port_range: 
            - 5985
            - 5986
          access: Allow
          priority: 1003
          direction: Inbound

  - name: Create a network interface
    azure_rm_networkinterface:
      name: nic
      resource_group: Rg-ansible-win01
      virtual_network: vNet
      subnet_name: subnet
      security_group: networkSecurityGroup
      ip_configurations:
        - name: default
          public_ip_address_name: pip
          primary: True
```
Note
* A virtual network interface card connects your VM to its virtual network, public IP address, and security group.
* The azure_rm_securitygroup creates an Azure network security group to allow WinRM traffic from the Ansible server to the remote host by allowing port 5985 and 5986.

Create a virtual machine

Add the following task

```yml
 - name: Create VM
   azure_rm_virtualmachine:
      resource_group: Rg-ansible-win01
      name: win-vm
      vm_size: Standard_B2s
      admin_username: azureuser
      admin_password: "{{ password }}"
      network_interfaces: nic
      os_type: Windows
      image:
          offer: WindowsServer
          publisher: MicrosoftWindowsServer
          sku: 2019-Datacenter
          version: latest
    no_log: true
```
The admin_password value of {{ password }} is an Ansible variable that contains the Windows VM password. To securely populate that variable, add a var_prompts entry to the beginning of the playbook.

```yml
vars_prompt:
    - name: password
      prompt: "Enter local administrator password"
```

Configure the WinRM Listener

To configure WinRM, add the following ext azure_rm_virtualmachineextension:

https://github.com/ansible/ansible/issues/81240

New url

https://github.com/ansible/ansible-documentation/tree/devel/examples/scripts



```yml
- name: Create VM script extension to enable HTTPS WinRM listener
    azure_rm_virtualmachineextension:
      name: winrm-extension
      resource_group: Rg-ansible-win01
      virtual_machine_name: win-vm
      publisher: Microsoft.Compute
      virtual_machine_extension_type: CustomScriptExtension
      type_handler_version: '1.9'
      settings: '{"fileUris": ["https://github.com/ansible/ansible-documentation/tree/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"],"commandToExecute": "powershell -ExecutionPolicy Unrestricted -File ConfigureRemotingForAnsible.ps1"}'
      auto_upgrade_minor_version: true
```
Note:
Issue with:

az vm extension show --name winrm-extension --vm-name win-wm --resource-group Rg-ansible-win01 --output json

ResourceNotFound The Resource 'Microsoft.Compute/virtualMachines/win-wm/extensions/winrm-extension' under resource group 'Rg-ansible-win01' was not found.


https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/troubleshoot

## Well, after putting everything in the yml file, it worked:

It could also be that the vnet, vm etc was created at the first run.

```yml
  - name: Get facts for one Public IP
    azure_rm_publicipaddress_info:
      resource_group: Rg-ansible-win01
      name: pip
    register: publicipaddresses

  - name: set public ip address fact
    set_fact: publicipaddress="{{ publicipaddresses | json_query('publicipaddresses[0].ip_address')}}"

  - name: wait for the WinRM port to come online
    wait_for:
      port: 5986
      host: '{{ publicipaddress }}'
      timeout: 600
```

```bash
ansible-playbook create_vm.yml
```

wait for the WinRM connection to finish.

In this case

fatal: [localhost]: FAILED! => {"changed": false, "elapsed": 600, "msg": "Timeout when waiting for 20.117.74.165:5986"}


Connect to the Windows virtual machine

Create a new Ansible playbook named connect.yml and copy the following contents into the playbook:
```yml
---
- hosts: all
  vars_prompt:
    - name: ansible_password
      prompt: "Enter local administrator password"
  vars:
    ansible_user: azureuser
    ansible_connection: winrm
    ansible_winrm_transport: ntlm
    ansible_winrm_server_cert_validation: ignore
  tasks:

  - name: Test connection
    win_ping:
```
Run it

```bash
ansible-playbook connect.yml  -i 20.117.74.165,
Enter local administrator password:
```

Timeout or connection refused

```ps1
New-SelfSignedCertificate -Subject 'CN=ServerB.domain.com' -TextExtension '2.5.29.37={text}1.3.6.1.5.5.7.3.1'

# You get the thumbprint here

winrm create winrm/config/Listener?Address=*+Transport=HTTPS '@{Hostname="ServerB.domain.com"; CertificateThumbprint="tprint"}'

# ResourceCreated
#    Address = http://schemas.xmlsoap.org/ws/2004/08/addressing/role/anonymous
#    ReferenceParameters
#        ResourceURI = http://schemas.microsoft.com/wbem/wsman/1/config/listener
#        SelectorSet
#            Selector: Address = *, Transport = HTTPS

# Firewall is default on, this creates a rule to allow 5986
$FirewallParam = @{
    DisplayName = 'Windows Remote Management (HTTPS-In)'
    Direction = 'Inbound'
    LocalPort = 5986
    Protocol = 'TCP'
    Action = 'Allow'
    Program = 'System'
}
New-NetFirewallRule @FirewallParam

```

https://github.com/Orange-Cyberdefense/GOAD/issues/98


Run it
```bash
ansible-playbook connect.yml  -i 20.117.74.165,
Enter local administrator password:
```
Success

```log
Enter local administrator password:

PLAY [all] ***********************

TASK [Gathering Facts] ***********
ok: [20.117.74.165]

TASK [Test connection] ***********
ok: [20.117.74.165]

PLAY RECAP ***********************
20.117.74.165              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Ping windows and configure ansible to use it


```bash
cd create-win
sudo nano inventory
cat inventory
# [myhosts]
# 20.117.74.165


# We must update since it uses defualt ssh, 22

# ansible_connection=winrm to define the connection is not SSH should use winrm
# ansible_user what ever the username you have created in the windows machine
# ansible_password password for that user ( the same one you use for RDP)
# ansible_winrm_server_cert_validation this is fine in DEV/TEST environment to tell ansible to ignore hostkey/server cert validation.

cat inventory
[winhosts]
20.117.74.165

[winhosts:vars]
ansible_connection = winrm
ansible_user = azureuser
ansible_password = password
ansible_winrm_server_cert_validation = ignore
ansible_port = 5986
# ansible_winrm_transport = ssl

ansible -i inventory winhosts -m win_ping -u azureuser
# 20.117.74.165 | UNREACHABLE! => {
#     "changed": false,
#    "msg": "ssl: the specified credentials were rejected by the server",
#     "unreachable": true
# }

# Make ansible.cfg so we do not need to pass inventory
sudo nano ansible.cfg
cat ansible.cfg
# [defaults]
# INVENTORY=inventory

ansible winhosts -m win_ping -u azureuser
ansible winhosts -m win_ping
# 20.117.74.165 | UNREACHABLE! => {
#     "changed": false,
#     "msg": "ssl: the specified credentials were rejected by the server",
#    "unreachable": true
# }


```
https://www.middlewareinventory.com/blog/how-to-use-ansible-with-windows-host-ansible-windows-example/


https://github.com/ansible/ansible/issues/16478



Yes, updated inventory with:

```bash
# https://github.com/ansible/ansible/issues/43920
# Does NTLM auth work, ansible_winrm_transport: ntlm

ansible_winrm_transport = ntlm

ansible winhosts -m win_ping -u azureuser
ansible winhosts -m win_ping

# 20.117.74.165 | SUCCESS => {
#     "changed": false,
#    "ping": "pong"
# }
```




https://learn.microsoft.com/en-us/azure/developer/ansible/vm-configure-windows?tabs=ansible

## Install IIS


