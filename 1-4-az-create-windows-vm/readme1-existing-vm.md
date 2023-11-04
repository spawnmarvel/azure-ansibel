# Configuring WinRM With HTTPS in azure virtual machine

Window remote management or in short WinRM is built-in windows protocol/Service which uses soap[simple object access protocol] to connect from another source system. Using WinRM, we can connect the remote system and execute any command there as its native user.


## 1. Create azure VM

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


## Stop a service, bck and edit, then start

* Install rabbitmq just an example
* Add all config files
* Use ansible to stop, backup file, edit file and start and verify

## Copy files from a st account to windows

## etc

## More info at wiki

https://follow-e-lo.com/2023/11/04/configuring-winrm-with-https-in-azure-virtual-machine/