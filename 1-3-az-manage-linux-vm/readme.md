# Manage Linux virtual machines in Azure using Ansible

## Steps

* Stop a virtual machine
* Start a virtual machine

Stop

```bash
mkdir manage-vm

ls
# create-vm  manage-vm
touch azure-vm-stop.yml
sudo nano azure-vm-stop.yml

```

```yaml
- name: Stop Azure VM
  hosts: localhost
  connection: local
  tasks:
    - name: Stop virtual machine
      azure_rm_virtualmachine:
        resource_group: ResourceGroup1
        name: vm-uksqa13
        allocated: no

```

Run it

```bash
ansible-playbook manage-vm/azure-vm-stop.yml

```
Result

```log
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Stop Azure VM]

TASK [Gathering Facts] ****

ok: [localhost]

TASK [Stop virtual machine] ****

changed: [localhost]

PLAY RECAP ****

localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


Start

```bash
cd manage-vm

cp azure-vm-stop.yml azure-vm-start.yml

ls
# azure-vm-start.yml  azure-vm-stop.yml

```

```yaml
- name: Stop Azure VM
  hosts: localhost
  connection: local
  tasks:
    - name: Start virtual machine
      azure_rm_virtualmachine:
        resource_group: ResourceGroup1
        name: vm-uksqa13
        started: yes

```
Run it
```bash

ansible-playbook manage-vm/azure-vm-start.yml 
```
https://learn.microsoft.com/en-us/azure/developer/ansible/vm-manage