# Age of Elements #
* Age of Elements is an LPMud game library that runs on the LDMud game driver.
* Age of Elements in built up from the LPMud 2.4.5 game library that comes with the LDMud game driver.

The topdirectory contains these files and directories:

* `ACCESS.ALLOW` the usual access definition file
* `WELCOME, NEWS, WIZNEWS` the messages printed on login
* `WIZLIST` the wizlist savefile
* `doc/` mudlib-specific documentation, to be complemented with the doc/ files from the driver distribution.
* `include/` header files
* `log/` logfiles generated by the mudlib
* `obj/` game objects
* `players/` player savefiles and wizard directories
* `room/` game rooms and include files
* `sys/` include files, including those from the driver distribution

## Getting Started on Amazon Web Services ##
* These instructions provide guidance on setting the up the game and a web site on Amazon Linux 2.
* _Warning: Amazon Web Services fees apply._
### Setup your Amazon Linux 2 Instance ###
* [Create an AWS Account or Sign-in](https://aws.amazon.com/console/)
* [Launch a Amazon Linux 2 Instance](https://console.aws.amazon.com/ec2/v2/home?#LaunchInstanceWizard:)
* _Size_, `t2.micro` for now. 
* _Security Group_, Type: `SSH` Protocol: `TCP` Port Range: `22` Source: `Your IP` Description: `Developer Access`
* _Security Group_, Type: `Custom TCP Rule` Protocol: `TCP` Port Range: `7680` Source: `Anywhere` Description: `TELNET to Game`
* _Security Group_, Type: `Custom UDP Rule` Protocol: `UDP` Port Range: `7681` Source: `Anywhere` Description: `API to Game`
* _Security Group_, Type: `HTTP` Protocol: `TCP` Port Range: `80` Source: `Anywhere` Description: `Web Server`
* _Security Group_, Type: `HTTPS` Protocol: `TCP` Port Range: `443` Source: `Anywhere` Description: `Web Server`
* [Connect with SSH to your instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)
```
ssh -i <path to your key name>.pem ec2-user@<instance public ip>.compute-1.amazonaws.com
```
#### Configure the Web Site ####
* [Install Apache, Mysql and PHP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-lamp-amazon-linux-2.html)
```
sudo yum update -y
sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
sudo yum install -y httpd mariadb-server
sudo systemctl start httpd
sudo systemctl enable httpd
sudo usermod -a -G apache ec2-user
exit
ssh -i <path to your key name>.pem ec2-user@<instance public ip>.compute-1.amazonaws.com
groups
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
find /var/www -type f -exec sudo chmod 0664 {} \;
```
* [Configure SSL/TLS on Amazon Linux 2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/SSL-on-amazon-linux-2.html)
```
sudo yum install -y mod_ssl
```
* [Use HTTPS: Install an SSL Certificate from Let's Encrypt](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/SSL-on-amazon-linux-2.html#letsencrypt)
```
sudo wget -r --no-parent -A 'epel-release-*.rpm' http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/
sudo rpm -Uvh dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-*.rpm
sudo yum-config-manager --enable epel*
sudo nano /etc/httpd/conf/httpd.conf
Locate the "listen 80" directive and add the following lines after it, replacing the example domain names with the actual Common Name and Subject Alternative Name (SAN).
<VirtualHost *:80>
    DocumentRoot "/var/www/html"
    ServerName "example.com"
    ServerAlias "www.example.com"
</VirtualHost>
sudo systemctl restart httpd
sudo yum install -y certbot python2-certbot-apache
sudo certbot
```
* [Test and Harden your SSL Server](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/SSL-on-amazon-linux-2.html#ssl_test)
```
sudo nano /etc/httpd/conf.d/ssl.conf
#SSLProtocol all -SSLv3
SSLProtocol -SSLv2 -SSLv3 -TLSv1 -TLSv1.1 +TLSv1.2
SSLHonorCipherOrder on
sudo systemctl restart httpd
sudo nano /etc/crontab
39 1,13 * * * root certbot renew --no-self-upgrade
sudo systemctl restart crond
```
* [Secure the Database Server](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-lamp-amazon-linux-2.html)
```
sudo systemctl start mariadb
sudo mysql_secure_installation
```
* [Install phpMyAdmin (optional)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-lamp-amazon-linux-2.html)
```
sudo yum install php-mbstring -y
sudo systemctl restart httpd
sudo systemctl restart php-fpm
cd /var/www/html
wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
mkdir phpMyAdmin && tar -xvzf phpMyAdmin-latest-all-languages.tar.gz -C phpMyAdmin --strip-components 1
rm phpMyAdmin-latest-all-languages.tar.gz
```
* [Secure Your phpMyAdmin Installation (if installed)](https://docs.phpmyadmin.net/en/latest/setup.html#securing-your-phpmyadmin-installation)
* [Hosting a WordPress Blog](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hosting-wordpress.html)
```
cd ~/
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
mysql -u root -p
CREATE USER 'wordpress-user' IDENTIFIED BY 'your_strong_password';
Backticks used on purpose here.
CREATE DATABASE `wordpress-db`;
GRANT ALL PRIVILEGES ON `wordpress-db`.* TO "wordpress-user";
FLUSH PRIVILEGES;
exit
cp wordpress/wp-config-sample.php wordpress/wp-config.php
nano wordpress/wp-config.php
https://api.wordpress.org/secret-key/1.1/salt/
cp -r wordpress/* /var/www/html/
sudo nano /etc/httpd/conf/httpd.conf
AllowOverride All
sudo chown -R apache:apache /var/www/html
sudo service httpd restart
```
* Connect to your WordPress Blog and complete the setup
* Configure the web server and database to restart upon reboots
```
sudo systemctl enable httpd && sudo systemctl enable mariadb
sudo systemctl status mariadb
sudo systemctl status httpd
```
#### Install LDMud Game Driver Dependencies ####
Install the standard developer tools, then tools for mysql, json-c, XML2 and OpenSSL. 
```
sudo yum groupinstall "Development Tools"
sudo yum install mysql-devel
sudo yum -y install json-c-devel
sudo yum -y install libxml2-devel
sudo yum -y install openssl-devel
```
#### Install LDMud Game Driver ####
Clone the game driver from the repository, detach it from source control, prep the game driver for the operating system.
```
cd ~/
git clone https://github.com/ldmud/ldmud.git
cd ldmud
rm -rf .git
./autogen.sh
cd ~/ldmud/src/settings
```
#### Configure Your Game Driver Installation ####
Use `nano` or `vi` to create a settings file called `aoe`.
```perl
#!/bin/sh
#
# Settings for Age of Elements running a native mode driver.
#
# configure will strip this part from the script.

exec ./configure --prefix=/home/ec2-user/ldmud --libdir=/home/ec2-user/ldmud/age-of-elements --with-setting=aoe $*
exit 1

# --- The actual settings ---

enable_erq=erq
enable_compat_mode=no
enable_strict_euids=no
enable_use_parse_command=no
enable_use_mysql=yes
enable_use_pcre=yes
enable_use_tls=ssl
enable_use_json=yes
enable_use_xml=yes

with_master_name=secure/master
with_portno=7680
with_udp_port=7681
```
Set permissions with `chmod ug+rw aoe` and `chmod o+x aoe`.
Run the configuration script, compile then install the game driver.
```
cd ~/ldmud/src
./configure --prefix=/home/ec2-user/ldmud --libdir=/home/ec2-user/ldmud/age-of-elements --with-setting=aoe
settings/aoe
make
make install
make install-all
```
#### Install Age of Elements Mud Library ####
Clone the mudlib 
```
cd ~/ldmud
git clone https://github.com/age-of-elements/age-of-elements.git
```
Use `nano` or `vi` to create a startup script in the `~/ldmud/bin` directory called `startup`.
```perl
ROOT=/home/ec2-user/ldmud
GAME=`ps -ec | awk '/ ldmud$/ { print $1 }'`

if test "$GAME" = ""
then

#If you have email running, uncomment these
#mail -s "Age of Elements startup" your@email.com << EOF
#Age of Elements startup at `date`.
#EOF

#Setup default permissions for the mudlib
umask u=rwx,g=rwx,o=
cd $ROOT
chmod g+s age-of-elements
chgrp ec2-user age-of-elements

#Make a copy of old runtime logs.
cd $ROOT/age-of-elements/log
touch driver_runtime.log
mv -f driver_runtime.log driver_runtime.log.old
touch driver_compiletime.log
mv -f driver_compiletime.log driver_compiletime.log.old

#Start the driver.
cd $ROOT/bin
./ldmud --debug-file $ROOT/age-of-elements/log/driver_runtime.log --hard-malloc-limit 0 7680 >& $ROOT/age-of-elements/log/driver_compiletime.log &

else

echo Game already running under PID $GAME

fi
```
Set permissions with `chmod ug+rw startup` and `chmod o+x startup`.
#### Start the Game ####
In the `~/ldmud/bin`, find the driver binary and your startup script.  Start the game, then connect with telnet.
```
./startup &
telnet localhost 7680
```
