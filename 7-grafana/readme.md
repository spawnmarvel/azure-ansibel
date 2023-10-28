# Grafana

# Install with bash first

Deploy the vm with ansible

```bash
ansible-playbook create-vm/main.yml
# localhost                  : ok=9    changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

# Open NSG 80, SSH is open
ssh imsdal@20.254.49.234 -i /home/imsdal/.ssh/authorized_keys 

```

Install from APT repository

```bash

# If you install from the APT repository, Grafana automatically updates when you run apt-get update.

# 1 Install the prerequisite packages:
sudo apt-get install -y apt-transport-https software-properties-common wget

# 2 Import the GPG key:
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null

# 3 To add a repository for stable releases, run the following command:
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

# 4 To add a repository for beta releases, run the following command: NO

# 5 Run the following command to update the list of available packages:
sudp apt update -y

# 6 To install Grafana OSS, run the following command:
# Installs the latest OSS release:
sudo apt-get install grafana

```
https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/


Start Grafana

```bash
# If you installed with the APT repository or .deb package, then you can start the server using systemd or init.d. 
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server

# start at boot
sudo systemctl enable grafana-server.service

# restart
sudo systemctl restart grafana-server

```
https://grafana.com/docs/grafana/latest/setup-grafana/start-restart-grafana/


Build your first dashboard

Open NSG 3000

http.//20.254.49.234:3000

https://grafana.com/docs/grafana/latest/getting-started/build-first-dashboard/



# Ansible
