# wordpress-install-notes
My abbreviated step-by-step notes to install WordPress on an Ubuntu 20 LTS system.  Note: Replace **example.com** with your own domain.

Take a look at these two great articles on Digital Ocean for more in-depth coverage (where I learned from):
* [How To Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu 20.04 by Erika Heidi](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-20-04)
* [How To Install WordPress on Ubuntu 20.04 with a LAMP Stack by Lisa Tagliaferri](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-on-ubuntu-20-04-with-a-lamp-stack)

## Get a an Ubuntu 20 LTS instance

Ubuntu with WordPress runs great on Arm64 systems like Amazon's Graviton EC2 instances

Here's a list of service providers where you can get an Ubuntu 20 LTS VPC up and running quickly
* [Amazon EC2 Graviton Arm64 instances I.e., M6g and T4g](https://aws.amazon.com/ec2/graviton/)
* [Amazon EC2 x86 instances](https://aws.amazon.com/ec2)
* [Amazon Lightsail](https://aws.amazon.com/lightsail/)
* [Digital Ocean Droplets](https://www.digitalocean.com/products/droplets/)
* [Linode](https://www.linode.com/)
* Nearly any place that offer VPS hosting

## Setup the LAMP (Linux/Apache/MySQL/PHP) Stack
These instructions assume your Ubuntu 20 LTS instance is up and running and you are sitting at a command prompt.

* Do an apt update: `sudo apt update && sudo apt upgrade -y`
* Install Apache: `sudo apt install apache2`
* If you don't have a firewall in front of your instance, enable the Ubuntu firewall:
```
sudo ufw allow in "Apache Full"
sudo ufw allow OpenSSH
sudo ufw enable
```
* Reboot the instance and check the firewall status
```
sudo ufw status

#It should look like this:
Status: active

To                         Action      From
--                         ------      ----
Apache Full                ALLOW       Anywhere                  
OpenSSH                    ALLOW       Anywhere                  
Apache Full (v6)           ALLOW       Anywhere (v6)             
OpenSSH (v6)               ALLOW       Anywhere (v6)
```
* Install MySQL and secure it
```
sudo apt install mysql-server
sudo mysql_secure_installation
```
* Select options to enforce strong passwords.  Choose 'y' for "VALIDATE PASSWORD COMPONENT" and press '2" for "STRONG".  Use a strong password for the DB root user.  A password generator might come in handy here.  Choose 'y' to remove anonymous users, disallow root login remotely, remove the test DB, and  reload the privilege tables.

* Install PHP.  Note: Do not install Imagemagick because it has a bug that crashes the server on large image uploads.
```
sudo apt install php libapache2-mod-php php-mysql
php -v #Check to PHP version after installing the above packages
```
After the package installation is complete, use "php -v" to make sure the PHP version.  Compare the versions at https://www.php.net/downloads.php and make sure the major and minor numbers (I.e. 7.4.x) match recent releases so the instance won't be out of date too quickly.
* Enable large image uploads by setting the following in the "sudo nano /etc/php/7.4/apache2/php.ini" file.
```
memory_limit = 128M
upload_max_filesize = 20M
post_max_size = 20M
max_execution_time = 30
```
## Create a new Apache virtual host for WordPress
* Create a new folder for your site, change ownership, and start a new Apache conf file using Nano.  Note: substitute "example.com" with your own domain.
```
sudo mkdir /var/www/example.com
sudo chown -R www-data:www-data /var/www/example.com
sudo chmod 775 /var/www/example.com
sudo nano /etc/apache2/sites-available/example.com.conf
```
* Update the domain and paste the following config to your domain conf file in the Nano session.  Use CTRL-X to save and exit the Nano editor.
```
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/example.com
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

<Directory /var/www/example.com/>
		AllowOverride All
</Directory>
```
* Enable the new virtual host, disable the old default website, test the config, and reload Apache.
```
sudo a2ensite example.com
sudo a2enmod rewrite
sudo a2dissite 000-default
sudo apache2ctl configtest
sudo systemctl restart apache2
```
* Create a test HTML document with Nano and point your web browser at the public IP address assigned to your instance to verify it works.
```
sudo nano /var/www/example.com/index.html
```
Here is a simple HTML document you can copy paste to Nano to test with.
```
<html>
  <head>
    <title>magicby.design website</title>
  </head>
  <body>
    <h1>Hello World!</h1>

    <p>This is the landing page of <strong>magicby.design</strong>.</p>
  </body>
</html>
```
* Create a test PHP document with the command line below and access it with your browser at the same IP as above but with a /info.php on it.
```
sudo nano /var/www/example.com/info.php
#Add this line and save the file: <?php phpinfo();
```
If everything looks OK, delete the file so you are not providing sensitive info about your server.  Also, delete index.html so it won't be the default document.
```
sudo rm /var/www/example.com/info.php
sudo rm /var/www/example.com/index.html
```
## WordPress Setup
* Install PHP extensions commonly need by WordPress and WordPress plugins
```
sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
sudo systemctl restart apache2
```
* Log into MySQL, set up users, and database.
```
sudo mysql -u root -p
CREATE USER 'example-wp-user'@'localhost' IDENTIFIED BY 'your-awesome-password';
CREATE DATABASE `example-wp-db`;
GRANT ALL PRIVILEGES ON `example-wp-db`.* TO "example-wp-user"@"localhost";
FLUSH PRIVILEGES;
exit
```
