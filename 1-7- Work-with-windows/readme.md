# Work with windows

## Steps

Controll node

Installing Ansible on Ubuntu

$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible


ansible --version
ansible [core 2.12.10]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/espen/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/espen/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, May 26 2023, 14:05:08) [GCC 9.4.0]
  jinja version = 2.10.1
  libyaml = True


https://docs.w3cub.com/ansible~2.10/installation_guide/intro_installation



How to Setup WinRM in Windows Machine to Prepare for Ansible

Windows node in Azure

https://learn.microsoft.com/en-us/troubleshoot/windows-server/remote/how-to-enable-windows-remote-shell

https://github.com/AlbanAndrieu/ansible-windows/blob/master/files/ConfigureRemotingForAnsible.ps1



If the installation is done right. you can see that your WinRM is UP and running and would be listening in port 5986



https://www.middlewareinventory.com/blog/how-to-use-ansible-with-windows-host-ansible-windows-example/

