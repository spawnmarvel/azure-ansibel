# wp

## Steps Wp and API https redirect (self-signed)

All is installed on one host.

If you are using two:
* Install only mariadb client on webserver
* Install maridb server and client on db server
* Configure mariadb with allow remote

## Install apache2

```bash

sudo apt update -y
sudo apt upgrade -y

# Allow 80,443 NSG and check ufw

sudo ufw status

sudo apt install apache2 -y

sudo a2enmod ssl

sudo systemctl enable apache2

sudo systemctl restart apache2

sudo service apache2 status

# visit
# http://ip = Apache2 Default Page

```

## Create self signed cert

```bash
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt

# backup default, make a copy
cd /etc/apache2/sites-available
cp default-ssl.conf default-ssl.conf_bck

#cd out
sudo nano /etc/apache2/sites-available/default-ssl.conf

# Paste

<VirtualHost *:80>
ServerAdmin admin@example.com
DocumentRoot /var/www/test
ServerName IP
ServerAlias www.example.com
Redirect / IP

<Directory /var/www/test/>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

<VirtualHost *:443>
   ServerName IP
   DocumentRoot /var/www/test

   SSLEngine on
   SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
   SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
</VirtualHost>

sudo mkdir /var/www/test

sudo nano /var/www/test/index.html

# Paste
# <h1>HTTPS it worked!</h1>


sudo a2ensite default-ssl.conf

sudo apache2ctl configtest
# Syntax OK

sudo systemctl reload apache2
sudo service apache2 status

# Vist

# http://IP = HTTPS it worked!

# https://IP = HTTPS it worked!

# View cert
# Common Name IP
# Validity, Not After, Fri, 07 Oct 2033 17:52:30 GMT


```

## Install php, mariadb etc

## Install wordpress

## Test site

## Test API with python



