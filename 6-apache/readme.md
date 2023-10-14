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

## Create Public IP dns

Create it on the vm pub ip

vm-uksqa13.uksouth.cloudapp.azure.com

http://vm-uksqa13.uksouth.cloudapp.azure.com/ = = Apache2 Default Page


## Virtual host

```bash
# make a new config for test
sudo nano /etc/apache2/sites-available/test1.conf


# make new content

<VirtualHost *:80>
ServerAdmin admin@example.com
DocumentRoot /var/www/test
ServerName 20.26.230.41
ServerAlias www.example.com

  <Directory /var/www/test/>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

# make test folder
sudo mkdir /var/www/test

# make test page
sudo nano /var/www/test/index.html

# Paste in index
# <h1>HTTP it worked!</h1>

sudo a2ensite test1.conf

sudo a2dissite 000-default.conf

sudo apache2ctl configtest
# Syntax OK

sudo systemctl reload apache2
sudo service apache2 status

# Vist
# http://20.26.230.41/ = HTTP it worked!
# http://vm-uksqa13.uksouth.cloudapp.azure.com/ = HTTP it worked!

```


## Let's encrypt Do Apache

```bash
sudo apt install certbot python3-certbot-apache -y

# Certbot needs to find the correct virtual host within your Apache configuration files. 
# Your server domain name(s) will be retrieved from the ServerName and ServerAlias directives defined within your VirtualHost configuration block.
# Should be:
# ServerName your_domain
# ServerAlias www.your_domain

# Update ServerAlias vm-uksqa13.uksouth.cloudapp.azure.com, even if that is not a valid domain

sudo certbot --apache

Saving debug log to /var/log/letsencrypt/letsencrypt.log

Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: vm-uksqa13.uksouth.cloudapp.azure.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel):             
Requesting a certificate for vm-uksqa13.uksouth.cloudapp.azure.com

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/vm-uksqa13.uksouth.cloudapp.azure.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/vm-uksqa13.uksouth.cloudapp.azure.com/privkey.pem
This certificate expires on 2024-01-12.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for vm-uksqa13.uksouth.cloudapp.azure.com to /etc/apache2/sites-available/test1-le-ssl.conf
Congratulations! You have successfully enabled HTTPS on https://vm-uksqa13.uksouth.cloudapp.azure.com


# Apache did all the updates?

cd /etc/apache2/sites-available
ls
000-default.conf  default-ssl.conf  test1-le-ssl.conf  test1.conf


<VirtualHost *:80>
ServerAdmin admin@example.com
DocumentRoot /var/www/test
ServerName 20.26.230.41
ServerAlias vm-uksqa13.uksouth.cloudapp.azure.com

  <Directory /var/www/test/>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   CustomLog ${APACHE_LOG_DIR}/access.log combined
RewriteEngine on
RewriteCond %{SERVER_NAME} =20.26.230.41 [OR]
RewriteCond %{SERVER_NAME} =vm-uksqa13.uksouth.cloudapp.azure.com
RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>


sudo systemctl reload apache2

```

## Vist site

http://20.26.230.41/ is redirected to

https://20.26.230.41/ = HTTP it worked!

View image Lets_encrypt

## Verify Certbot Auto-Renewal

```bash
sudo systemctl status certbot.timer

Run certbot twice daily
     Loaded: loaded (/lib/systemd/system/certbot.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Sat 2023-10-14 10:45:27 UTC; 28min ago
    Trigger: Sat 2023-10-14 18:03:37 UTC; 6h left


# Test it
sudo certbot renew --dry-run

Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/vm-uksqa13.uksouth.cloudapp.azure.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Account registered.
Simulating renewal of an existing certificate for vm-uksqa13.uksouth.cloudapp.azure.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all simulated renewals succeeded:
  /etc/letsencrypt/live/vm-uksqa13.uksouth.cloudapp.azure.com/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -


```

https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-20-04



