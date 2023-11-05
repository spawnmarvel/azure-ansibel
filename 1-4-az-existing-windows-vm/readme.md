# Configuring WinRM With HTTPS in azure virtual machine

Window remote management or in short WinRM is built-in windows protocol/Service which uses soap[simple object access protocol] to connect from another source system. Using WinRM, we can connect the remote system and execute any command there as its native user.


## 1. Create azure VM in the portal

We are simulating an existing vm not created by ansble.

* vmukstest01
* Standard B2s (2 vcpus, 4 GiB memory)
* Windows (Windows Server 2019 Datacenter)
* Select inbound port of RDP, HTTP and HTTPS.

Once azure VM is created, You can configure DNS name for this VM. This DNS name will be required for creating the certificate. For azure VM, DNS name will be used as HOST_NAME.


```ps1
C:\Users\vossmann>hostname
vmukstest01
```


## 2. Verify the WinRM is installed and service running on the azure VM

WinRM  comes already installed in all supported version of the Windows OS. 

Login to the VM using RDP and verify whether the WinRM service is running or not.

Run-> write Service.msc -> chech the service 

Windows Remote Management (WS-Management) is running


## 3. Quick configuration.

No WinRM listener is configured by default. Even if the WinRM service is running, the WS-Management protocol ensures that request data can’t be received nor sent.

set up the default configuration for remote management with the below command :


```ps1
# [Recommended] Powershell runas admin
Enable-PSRemoting -force
```
above command does the following:

* Starts WinRM service(if not started) and sets startup mode to auto-start.
* Configures a listener for the ports that send & receive WS-Management protocol messages using either HTTP or HTTPS on any IP address.
* Defines ICF exceptions for the WinRM service, and opens the ports for HTTP and HTTPS.

## 4. Add firewall profile with a remote connection to public network:


```ps1
netsh advfirewall firewall add rule name=”WinRM-HTTP” dir=in localport=5985 protocol=TCP action=allow

netsh advfirewall firewall add rule name=”WinRM-HTTPS” dir=in localport=5986 protocol=TCP action=allow
```

## 5. Verify default listener

When we ran above command: Enable-PSRemoting -force, It configures WinRM for HTTP connection [port 5985].

We can check the available listeners using below command :  

```ps1
winrm e winrm/config/listener
# Listener
#    Address = *
#    Transport = HTTP
#    Port = 5985
#    Hostname
#    Enabled = true
#    URLPrefix = wsman
#    CertificateThumbprint
#    ListeningOn = 10.0.0.4, 127.0.0.1, ::1, fe80::f126:b629:4b75:1db5%6

```
Here, as we can see the output, its configured to HTTP. So we need to add listener for the HTTPS in later steps.

## 6. Add Self-Signed Certificate.

Before adding Listener for the HTTPS, we need to add the self-signed certificate and get its thumbprint. The thumbprint of the certificate will be used to register the Listener in WinRM.


```ps1
$cert = New-SelfSignedCertificate -DnsName "51.11.36.160" -CertStoreLocation cert:\LocalMachine\My
write-host $cert
# We get the thumbprint here, tprint

# Run the below command to add HTTPS listener for WinRM
winrm create winrm/config/Listener?Address=*+Transport=HTTPS '@{Hostname="51.11.36.160"; CertificateThumbprint="tprint"}'

```

## 8. Validate HTTPS Listener

```ps1
winrm e winrm/config/listener
# Listener
#    Address = *
#    Transport = HTTP
#    Port = 5985
#    Hostname
#    Enabled = true
#    URLPrefix = wsman
#    CertificateThumbprint
#    ListeningOn = 10.0.0.4, 127.0.0.1, ::1, fe80::f126:b629:4b75:1db5%6

# Listener
#    Address = *
#    Transport = HTTPS
#    Port = 5986
#    Hostname = 51.11.36.160
#    Enabled = true
#    URLPrefix = wsman
#    CertificateThumbprint = 6EC2A54F47DDA5469EE06462CE640F92121180D2
#    ListeningOn = 10.0.0.4, 127.0.0.1, ::1, fe80::f126:b629:4b75:1db5%6

```
## 9. Add Firewall exception rule for HTTPS

You can skip if you already added HTTPS firewall rule in the previous step-4.

```ps1
netsh advfirewall firewall add rule name=”WinRM-HTTPS” dir=in localport=5986 protocol=TCP action=allow
```

## 10. Add Inbound rule for port 5986 and 5985 in azure portal.

We need to add Inbound rule for the HTTP and HTTPS port in the Azure portal for the virtual machine. The virtual machine has its own port defined for HTTP and https – 5985 and 5986 respectively.

## 11. Verify HTTPS connection from source machine and execute command to remote maching.

```ps1
$username = ""

$password = ""

$pso = New-PSSessionOption -SkipCACheck

$secpasswd = ConvertTo-SecureString $password -AsPlainText -Force

$credentials = New-Object System.Management.Automation.PSCredential($username, $secpasswd)

$session = New-PSSession -ComputerName "51.11.36.160" -UseSSL -SessionOption $pso -Credential $credentials

$session

# Id Name            ComputerName    ComputerType    State         ConfigurationName     Availability
# -- ----            ------------    ------------    -----         -----------------     ------------
#  1 WinRM1          51.11.36.160    RemoteMachine   Opened        Microsoft.PowerShell     Available

```


https://techdiksha.com/configuring-winrm-azure-virtual-machine/

## Test winrm with Ansible

Continue from

https://github.com/spawnmarvel/azure-ansible/blob/main/1-4-az-create-windows-vm/readme.md

* Make inventory
* Make ansible.cfg
* Connect

```bash
mkdir manage-win
cd manage-win
sudo nano inventory
cat inventory

[winhosts]
51.11.36.160

[winhosts:vars]
ansible_connection = winrm
ansible_user = vossmann
ansible_password = password
ansible_winrm_server_cert_validation = ignore
ansible_port = 5986
# ansible_winrm_transport = ssl
ansible_winrm_transport = ntlm

sudo nano ansible.cfg
cat ansible.cfg
[defaults]
INVENTORY=inventory

ansible winhosts -m win_ping
# 51.11.36.160 | UNREACHABLE! => {
#    "changed": false,
#    "msg": "ntlm: the specified credentials were rejected by the server",
#    "unreachable": true
# }

# Now I used the wrong password, oink. Update inventory to correct password

ansible winhosts -m win_ping
# 51.11.36.160 | SUCCESS => {
#     "changed": false,
#    "ping": "pong"
# }

```
# Ansible windows operations

Now we are ready to do things on windows with Ansible.

## Ansible create directory on windows

ansible.windows.win_file module – Creates, touches or removes files or directories

https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_file_module.html


Make create_dir.yml

```yml
---
- name: win_file module
  hosts: winhosts
  vars:
    mydir: 'C:\ansible'
  tasks:
    - name: Create a directory called ansible
      ansible.windows.win_file:
        path: "{{ mydir }}"
        state: directory

```

run it

```bash
ansible-playbook create_dir.yml

# folder is created
# https://follow-e-lo.com/2023/11/04/ansible-winrm-manage-vm/

```
## Ansible create a file in directory on windows

Make create_file.yml

```yml
---
- name: win_file module
  hosts: winhosts
  vars:
     mydir: 'C:\ansible\ansible.log'
  tasks:
    - name: Create a file called ansible.log
      ansible.windows.win_file:
        path: "{{ mydir }}"
        state: touch

```

run it

```bash
ansible-playbook create_file.yml

# file is created
# https://follow-e-lo.com/2023/11/04/ansible-winrm-manage-vm/

```

## Ansible copy a file on windows
ansible.windows.win_copy module – Copies files to remote locations on windows hosts

https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_copy_module.html

Make copy_file.yml

```yml
---
- name: win_copy module
  hosts: winhosts
  vars:
     myfile: 'C:\ansible\ansible.log'
  tasks:
    - name: Copy a single file
      ansible.windows.win_copy:
        src: "{{ myfile }}"
        dest: "{{ myfile + '.bck' }}"
        remote_src: true

```

run it

```bash
ansible-playbook copy_file.yml

# file is created, bck file
# https://follow-e-lo.com/2023/11/04/ansible-winrm-manage-vm/

```

## Ansible add content a file on windows
ansible.windows.win_shell module – Execute shell commands on target hosts

https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_shell_module.html#ansible-collections-ansible-windows-win-shell-module
Make add_content.yml

```yml
---
- name: win_shell module
  hosts: winhosts
  vars:
     myfile: 'C:\ansible\ansible.log'
  tasks:
    - name: Run basic PowerShell script to log
      ansible.windows.win_shell: |
          $d = Get-Date
          $log = "Ansible worker " + $d
          Add-Content C:\ansible\ansible.log $log

```

run it a few times

```bash
ansible-playbook add_content.yml

# file is updated
# https://follow-e-lo.com/2023/11/04/ansible-winrm-manage-vm/

```

## Ansible Execute a command in the remote shell, iis

ansible.windows.win_shell module – Execute shell commands on target hosts

https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_shell_module.html#ansible-collections-ansible-windows-win-shell-module
Make remote_script.yml

```yml
---
- name: win_shell module
  hosts: winhosts
  vars:
     myfile: 'C:\ansible\ansible.log'
  tasks:
    - name: Run basic PowerShell script remote install iis
      ansible.windows.win_shell: c:\ansible\install_iis.ps1
    - name: Check if iis is installed
      ansible.windows.win_shell: |
          $d = Get-Date
          if ((Get-WindowsFeature Web-Server).InstallState -eq "Installed") {
              $log = "Ansible worker, IIS is installed " + $d
          } 
          else {
               $log = "Ansible worker, IIS is not installed " + $d
          }
          Add-Content C:\ansible\ansible.log $log

```

run it a few times

```bash
ansible-playbook remote_script.yml

# iis is installed
# https://follow-e-lo.com/2023/11/04/ansible-winrm-manage-vm/

```

## Ansible stop, log start service

ansible.windows.win_service module – Manage and query Windows services

https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_service_module.html
Make stop_starte_service.yml

```yml
---
- name: win_shell module
  hosts: winhosts
  vars:
     myfile: 'C:\ansible\ansible.log'
  tasks:
    - name: Stop windows service IIS servicename or display name
      ansible.windows.win_service:
         name: W3SVC
         state: stopped
    - name: Log service state
      ansible.windows.win_shell: |
         $d = Get-Date
         $state = Get-Service "W3SVC" | Select-Object -Property Status
         $log = "Ansible worker, Service state " + $state + " " + $d
         Add-Content C:\ansible\ansible.log $log
    - name: Star windows service IIS servicename or display name
      ansible.windows.win_service:
         name: W3SVC
         state: started
    - name: Log service state
      ansible.windows.win_shell: |
         $d = Get-Date
         $state = Get-Service "W3SVC" | Select-Object -Property Status
         $log = "Ansible worker, Service state " + $state + " " + $d
         Add-Content C:\ansible\ansible.log $log

```

run it a few times

```bash
ansible-playbook stop_start_service.yml

# service stopped, logged, started and logged
# https://follow-e-lo.com/2023/11/04/ansible-winrm-manage-vm/

```

## Manage RabbitMQ 1

Install RabbitMQ manuall

Continue, starts vms and ssh to ctrl node and test connectivity to server.

Lest use what we know and try to make it better for each iteration.

```bash
cd manage-win

ansible winhosts -m win_ping
# "ping": "pong"

```

Install rabbitmq just an example

Add all config files

Use ansible to stop, backup file, edit file and start and verify

Make the playbook
* Stop service
* Backup conf
* Copy new conf
* Start service
* Verify service

```bash
mkdir rmq
cp manage-win/inventory rmq/inventory
cp manage-win/ansible.cfg rmq/ansible.cfg
cd rmq

ansible winhosts -m win_ping
#  "ping": "pong"

sudo nano update_rmq.yml

sudo nano rabbitmq.conf
# This is the main file on th ctrl node, we will edit it here and copy it

```

Make update_rmq.yml

```yml
---
- name: Update Rabbitmq config
  hosts: winhosts
  vars:
     newfile: '/home/imsdal/rmq/rabbitmq.conf'
     rmqfile: 'c:\RabbitMqStore\rabbitmq.conf'
     rmqservice: 'RabbitMQ'
  tasks:
    - name: Stop windows service RabbitMQ servicename or display name
      ansible.windows.win_service:
         name: RabbitMQ
         state: stopped
    - name: Log service state
      ansible.windows.win_shell: |
         $d = Get-Date
         $state = Get-Service "RabbitMQ" | Select-Object -Property Status
         $log = "Ansible worker, Service state {{ rmqservice }}" + $state + " " + $d
         Add-Content C:\ansible\ansible.log $log
    - name: Backcup rabbitmq.conf
      ansible.windows.win_copy:
        src: "{{ rmqfile }}"
        dest: "{{ rmqfile + '.bck' }}"
        remote_src: true
    - name: Copy new {{ newfile }}
      ansible.windows.win_copy:
        src: "{{ newfile }}"
        dest: "{{ rmqfile }}"
    - name: Start windows service RabbitMQ servicename or display name
      ansible.windows.win_service:
         name: RabbitMQ
         state: started
    - name: Log service state
      ansible.windows.win_shell: |
         $d = Get-Date
         $state = Get-Service "RabbitMQ" | Select-Object -Property Status
         $log = "Ansible worker, Service state {{ rmqservice }}" + $state + " " + $d
         Add-Content C:\ansible\ansible.log $log

```

Run it

```bash

ansible-playbook winhosts update_rmq.yml

# service stopped, logged, started and logged
# https://follow-e-lo.com/2023/11/04/ansible-winrm-manage-vm/

```


## Manage RabbitMQ 2

Lets do this more correct and learn, we have duplicates and we also need more error checking.

backup: yes
* backing up the original if it differs from the copied version
* We changed the port to 5675
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html
Make update2_rmq.yml

```yml
---
- name: Update Rabbitmq config
  hosts: winhosts
  vars:
     newfile: '/home/imsdal/rmq/rabbitmq.conf'
     rmqfile: 'c:\RabbitMqStore\rabbitmq.conf'
     rmqservice: 'RabbitMQ'
  tasks:
    - name: Stop windows service RabbitMQ servicename or display name
      ansible.windows.win_service:
         name: RabbitMQ
         state: stopped
    - name: Log service state
      ansible.windows.win_shell: |
         $d = Get-Date
         $state = Get-Service "RabbitMQ" | Select-Object -Property Status
         $log = "Ansible worker, Service state {{ rmqservice }}" + $state + " " + $d
         Add-Content C:\ansible\ansible.log $log
    - name: Copy new {{ newfile }}
      ansible.windows.win_copy:
        src: "{{ newfile }}"
        dest: "{{ rmqfile }}"
        backup: yes
    - name: Start windows service RabbitMQ servicename or display name
      ansible.windows.win_service:
         name: RabbitMQ
         state: started
    - name: Log service state
      ansible.windows.win_shell: |
         $d = Get-Date
         $state = Get-Service "RabbitMQ" | Select-Object -Property Status
         $log = "Ansible worker, Service state {{ rmqservice }}" + $state + " " + $d
         Add-Content C:\ansible\ansible.log $log

```

Run it

```bash

ansible-playbook winhosts update2_rmq.yml

# service stopped, logged, started and logged
# https://follow-e-lo.com/2023/11/04/ansible-winrm-manage-vm/

```


## Install RabbitMQ with Ansible

* This involves .exe
* Environment vars and more fun