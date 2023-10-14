# Apache with SSL
 
## Install apache2 with self signed

```bash

sudo apt update -y
sudo apt upgrade -y

# Allow 80,443 NSG and check ufw

sudo ufw status

sudo apt install apache2 -y

# check sites enabled
# /etc/apache2/sites-enabled
# ls
# 000-default.conf
# check sistes avaliable
# 000-default.conf  default-ssl.conf

sudo a2enmod ssl

sudo systemctl restart apache2

sudo systemctl enable apache2

sudo service apache2 status

# visit, it could take 1 min
# 20.68.24.16
# http://ip = Apache2 Default Page

```

## Create self signed cert

```bash
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt

Common Name (e.g. server FQDN or YOUR name) []:20.68.24.16 

# make a new config for test
sudo nano /etc/apache2/sites-available/test_ssl.conf

# make new content

<VirtualHost *:80>
ServerAdmin admin@example.com
DocumentRoot /var/www/test
ServerName 20.68.24.16
ServerAlias www.example.com
Redirect / https://20.68.24.16

  <Directory /var/www/test/>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

<VirtualHost *:443>
   ServerName 20.68.24.16
   DocumentRoot /var/www/test

   <Directory /var/www/test/>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
   </Directory>

   SSLEngine on
   SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
   SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
</VirtualHost>

# make test folder
sudo mkdir /var/www/test

# make test page
sudo nano /var/www/test/index.html

# Paste in index
# <h1>HTTPS it worked!</h1>

sudo a2ensite test_ssl.conf

sudo apache2ctl configtest
# Syntax OK

sudo systemctl reload apache2
sudo service apache2 status


# cd /etc/apache2/sites-enabled
# ls
# 000-default.conf  test_ssl.conf

# Vist

# http://20.68.24.16/ = HTTPS it worked! redirected

# https://20.68.24.16/ = HTTPS it worked!

# View cert
# Common Name 20.68.24.16
# Validity, Not After Sun, 09 Oct 2033 15:40:58 GMT

```