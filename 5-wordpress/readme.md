# Wordpress with HTTPS

## Steps Wp and API https redirect (self-signed)

All is installed on one host.

If you are using two hosts:
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

CREATE USER 'johndoe'@'%' IDENTIFIED BY 'a-password';

CREATE DATABASE db_wordpress;

GRANT ALL PRIVILEGES ON *.* TO 'johndoe'@'%' WITH GRANT OPTION;

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

# check sites avilable
# ls
# 000-default.conf  default-ssl.conf  test_ssl.conf
# check sistes enabled
# ls
# 000-default.conf  test_ssl.conf

# Cat the test_ssl.conf

cat /etc/apache2/sites-available/test_ssl.conf 

# Create a configuration file for WordPress

sudo nano /etc/apache2/sites-available/wordpress.conf

# Edit the cat result to point to wp folder and paste it

<VirtualHost *:80>
ServerAdmin admin@example.com
DocumentRoot /var/www/html/wordpress
ServerName 20.68.24.16
ServerAlias www.example.com
Redirect / https://20.68.24.16

  <Directory /var/www/html/wordpress/>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

<VirtualHost *:443>
   ServerName 20.68.24.16
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

# enable the site

sudo a2ensite wordpress.conf

sudo a2enmod rewrite

sudo apache2ctl configtest

# disbale test test ssl and default

sudo a2dissite test_ssl.conf

sudo a2dissite 000-default.conf

sudo systemctl restart apache2

# Vist
# http://20.68.24.16/ redirect to https://20.68.24.16/wp-admin/setup-config.php

```

## Setup wordpress
Properties
* english
* db_wordpress
* johndoe
* a-password

Run the installation
* Welcome
* test123 (site name)
* user-for-wp-01
* password-for-user-01
* some@gmail.com

## Test site

* Site health
* Https ULR and set permalinks
* Install Twenty 16, activate it, create some categorys, Customer and Expired and some postes
* Install plugin Install Classic Editor and install the theme IKnowledgebase
* Create a home page with template Home Page and create a post page
* Create some posts and navigate through the site to see that ULR's and navigation is success
* Enable future press plugin and test expired, set all posts default to 1 year.
* !!! Select whether the PublishPress Future is enabled for all new posts = Active and Enabled
* Make a new post and test it from the future settings, run job

## Test API


https://20.68.24.16/wp-json/wp/v2/posts

* Create a user for API
* python worker
* some-password-yes-789
* Role Editor
* Now go to the user and create a Application password
* TheAppPassword01
* You get the token back

Run the code to create a post with Python

```py

import requests
import base64
import json
import logging


import requests
import json

import requests
import base64
import json
import logging


import requests
import json

def make_post_and_send():
        data = {
            "title": "Autogenerated from Python",
            "content": "This was created with code towards the API",
            "status": "publish"  # Use "draft" to save the post as a draft

        }
        try:
            # Send the HTTP request
            response = requests.post("https://20.68.24.16/wp-json/wp/v2/posts", auth=("pythonworker", "THE-TOKEN-GOES-HERE WITH SPACES"), json=data, verify=False)
            # Check the response
            if response.status_code == 201:
                print("Post created successfully")
            else:
                print("Failed to create post: " + response.text)
        except Exception as ex:
            print(ex)

make_post_and_send()

```

## Restart the server

Check that everything is ok



