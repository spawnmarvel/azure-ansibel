# wp

## Steps Wp and API https redirect (self-signed)

All is installed on one host.

If you are using two:
* Install only mariadb client on webserver
* Install maridb server and client on db server
* Configure mariadb with allow remote

Note: Replace all IP with you IP.

## Install apache2

```bash

sudo apt update -y
sudo apt upgrade -y

# Allow 80,443 NSG and check ufw

sudo ufw status

sudo apt install apache2 -y

# check sites enabled
# /etc/apache2/sites-enabled
ls
000-default.conf

sudo a2enmod ssl

sudo systemctl restart apache2

sudo systemctl enable apache2

sudo service apache2 status

# visit
# 20.0.76.5 
# http://ip = Apache2 Default Page

```

## Create self signed cert

```bash
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt

Common Name (e.g. server FQDN or YOUR name) []:20.0.76.5 

# backup default, make a copy
cd /etc/apache2/sites-available

# Current
ls
000-default.conf  default-ssl.conf

#cd out
sudo nano /etc/apache2/sites-available/default-ssl2.conf

# make new content

<VirtualHost *:80>
ServerAdmin admin@example.com
DocumentRoot /var/www/test
ServerName 20.0.76.5
ServerAlias www.example.com
Redirect / https://20.0.76.5

  <Directory /var/www/test/>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

<VirtualHost *:443>
   ServerName 20.0.76.5
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

sudo mkdir /var/www/test

sudo nano /var/www/test/index.html

# Paste
# <h1>HTTPS it worked!</h1>

sudo a2ensite default-ssl2.conf

sudo apache2ctl configtest
# Syntax OK

sudo systemctl reload apache2
sudo service apache2 status


# /etc/apache2/sites-enabled
# 000-default.conf  default-ssl2.conf

# Vist

# http://20.0.76.5/ = HTTPS it worked! redirected

# https://20.0.76.5/ = HTTPS it worked!

# View cert
# Common Name 20.0.76.5
# Validity, Not After Sun, 09 Oct 2033 15:40:58 GMT


```

## Install php


```bash
sudo apt install -y php php-{common,mysql,xml,xmlrpc,curl,gd,imagick,cli,dev,imap,mbstring,opcache,soap,zip,intl}

php -v

```


## mariadb etc

```bash
sudo apt install mariadb-server mariadb-client -y

sudo systemctl enable --now mariadb

sudo mysql_secure_installation

# enter, n, n, y, y, y, y

sudo mysql 

CREATE USER 'johndoe'@'%' IDENTIFIED BY 'password';

CREATE DATABASE db_wordpress;

GRANT ALL PRIVILEGES ON *.* TO 'username'@'%' WITH GRANT OPTION;

FLUSH PRIVILEGES;

Select * from mysql.user;

show databases;

exit;

```

## Install wordpress

```bash
sudo apt install wget unzip -y

wget https://wordpress.org/latest.zip

sudo unzip latest.zip

sudo mv wordpress/ /var/www/html/

sudo chown www-data:www-data -R /var/www/html/wordpress/

sudo chmod -R 755 /var/www/html/wordpress/

# backup default-ssl2.conf

cd /etc/apache2/sites-available

sudo cp default-ssl2.conf default-ssl2.conf_bck

ls

000-default.conf  default-ssl.conf  default-ssl2.conf  default-ssl2.conf_bck

# Now edit to wordpress folder

<VirtualHost *:80>
ServerAdmin admin@example.com       
DocumentRoot /var/www/html/wordpress
ServerName 20.0.76.5
ServerAlias www.example.com
 Redirect / https://20.0.76.5       

<Directory /var/www/html/wordpress/>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

<VirtualHost *:443>
   ServerName 20.0.76.5
   DocumentRoot /var/www/html/wordpress

  <Directory /var/www/html/wordpress/>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
   </Directory>

   SSLEngine on
   SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
   SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
</VirtualHost>


sudo a2ensite default-ssl2.conf

sudo a2enmod rewrite

sudo apache2ctl configtest
# Syntax OK

sudo systemctl restart apache2

# Vist
# http://20.0.76.5/ redirect to https://20.0.76.5/wp-admin/setup-config.php

# Setup
english
db_wordpress
johndoe
password

# Run the installation
Welcome
test123 (site name)
user-for-wp
password-for-user
some@gmail.com



```

## Test site

View images

Publishing failed. Error message: The response is not a valid JSON response

Install Classic Editor

https://wordpress.org/plugins/classic-editor/


Permalinks Plain to Post

## Test API with python


```py
import requests
import base64
import json
import logging


import requests
import json

def make_post_and_send():
        data = {
            "title": "Title for Post 3",
            "content": "This is the content of my new post",
            "status": "publish"  # Use "draft" to save the post as a draft

        }
        try:
            # Send the HTTP request
            response = requests.post("https://20.0.76.5/wp-json/wp/v2/posts", auth=("pythonworker", "#### #### #### #### ####"), json=data, verify=False)
            # Check the response
            if response.status_code == 201:
                print("Post created successfully")
            else:
                print("Failed to create post: " + response.text)
        except Exception as ex:
            logging.error(ex)

make_post_and_send()

```




## You should not meet below. is has been fixed above in virtual host 443

API 
can not edit to Post name from Plain then Not Found https://20.0.76.5/wp-json/wp/v2/posts





/var/www/html/wordpress$ cat .htaccess 

# BEGIN WordPress
# The directives (lines) between "BEGIN WordPress" and "END WordPress" are
# dynamically generated, and should only be modified via WordPress filters.
# Any changes to the directives between these markers will be overwritten.

sudo rm .htaccess 

Go to permalinks and select post and save to generate a new file .htaccess 

cat .htaccess

# BEGIN WordPress
# The directives (lines) between "BEGIN WordPress" and "END WordPress" are
# dynamically generated, and should only be modified via WordPress filters.
# Any changes to the directives between these markers will be overwritten.
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>

It takes updates but edit to other then plain is not working.

add to ssl section

<Directory /var/www/html/wordpress/>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
   </Directory>

   sudo service apache2 reload 




