# Ansible 101 - Episode 1 - Introduction to Ansible

## Video

https://www.youtube.com/watch?v=goclfp6a2IQ&list=LL&index=2

## Notes

### Connecting to a Server with Ansible

The Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate.

#### Inventory and ping

```bash
mkdir chapter1
cd chapter1

touch inventory

# [example]
# 20.77.101.194

# i= inventory, m= module and user
ansible -i inventory example -m ping -u imsdal
```

```log
20.77.101.194 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

General configuration

```bash
which ansible
#/usr/bin/ansible

whereis ansible
# ansible: /usr/bin/ansible /usr/share/man/man1/ansible.1.gz

```

# Override ansibe cfg default

```bash
# The default Ansible configuration file is located under /etc/ansible/ansible.cfg
# https://stackoverflow.com/questions/35969668/ansible-path-to-ansible-cfg

# For latest version (2.7.6) if you install via pip you wont get ansible folder in /etc
ansible --version
# ansible 2.10.8
config file = None
#  configured module search path = ['/home/imsdal/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
#  ansible python module location = /usr/lib/python3/dist-packages/ansible
#  executable location = /usr/bin/ansible
#  python version = 3.10.12 (main, Jun 11 2023, 05:26:28) [GCC 11.4.0]

# But once you manually create directory under /etc as ansible and add ansible.cfg file there ansible automatically detects it. but you will have to configure the rest manually like hosts file..etc . so after this we get
config file = /etc/ansible/ansible.cfg
```