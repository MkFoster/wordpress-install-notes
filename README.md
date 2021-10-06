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

