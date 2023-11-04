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
[Recommended] Powershell : Enable-PSRemoting -force
```

https://techdiksha.com/configuring-winrm-azure-virtual-machine/

## More info at wiki

https://follow-e-lo.com/2023/11/04/configuring-winrm-with-https-in-azure-virtual-machine/