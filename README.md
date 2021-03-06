# Age of Elements #
* Age of Elements is an LPMud game library that runs on the LDMud game driver.
* Age of Elements in built up from the LPMud 2.4.5 game library that comes with the LDMud game driver.
* Age of Elements is located at telnet://ageofelements.org:7680 and port 8680 encrypts data-in-transit (see detail below).

Age of Elements is primarily being used at this time to build up examples of [game telnet protocols](https://github.com/age-of-elements/age-of-elements/tree/master/obj/login), such as:

* [Generic Mud Communication Protocol](http://www.gammon.com.au/gmcp) (GMCP)
* [Mud Server Status Protocol](https://tintin.sourceforge.io/protocols/mssp/) (MSSP)
* [Mud eXtension Protocol](http://www.gammon.com.au/mushclient/addingservermxp.htm) (MXP)
* [Mud Sound Protocol](https://www.zuggsoft.com/zmud/msp.htm) (MSP)
* [Mud Client Media Protocol](https://wiki.mudlet.org/w/Standards:MUD_Client_Media_Protocol) (MCMP)

In time, we'll add in examples of a [Mudlet](https://mudlet.org) GUI and how to send mapper data from inside the game.

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
* _Security Groups:_

| Type             | Protocol | Port Range | Source   | Description        |
|:----------------:|:--------:|:---------- |:-------- |:------------------ |
| SSH              | TCP      | 22         | Your IP  | Developer Access   |
| Custom TCP Rule  | TCP      | 7680       | Anywhere | TELNET to Game     |
| Custom UDP Rule  | UDP      | 7681       | Anywhere | API to Game        |
| Custom TCP Rule  | TCP      | 8680       | Anywhere | TLS Tunnel to Game |
| HTTP             | TCP      | 80         | Anywhere | Webserver          |
| HTTPS            | TCP      | 443        | Anywhere | Webserver          |

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
```
Locate the "listen 80" directive and add the following lines after it, replacing the example domain names with the actual Common Name and Subject Alternative Name (SAN).
```
<VirtualHost *:80>
    DocumentRoot "/var/www/html"
    ServerName "example.com"
    ServerAlias "www.example.com"
</VirtualHost>
```
```
sudo systemctl restart httpd
sudo yum install -y certbot python2-certbot-apache
sudo certbot
```
* [Test and Harden your SSL Server](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/SSL-on-amazon-linux-2.html#ssl_test)
Update your SSL configuration with `sudo nano /etc/httpd/conf.d/ssl.conf`, containing the following content:
```
#SSLProtocol all -SSLv3
SSLProtocol -SSLv2 -SSLv3 -TLSv1 -TLSv1.1 +TLSv1.2
SSLHonorCipherOrder on
```
Restart the HTTP daemon and setup Let's Encrypt to renew on a periodic basis.
```
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
```
Backticks used on purpose here.
```
CREATE DATABASE `wordpress-db`;
GRANT ALL PRIVILEGES ON `wordpress-db`.* TO "wordpress-user";
FLUSH PRIVILEGES;
exit
cp wordpress/wp-config-sample.php wordpress/wp-config.php
nano wordpress/wp-config.php
```
* Generate necessary random values here: [WordPress API](https://api.wordpress.org/secret-key/1.1/salt/)
* Add `define('FS_METHOD','direct');` to the file to enable WordPress updates via direct downloads vs. setting up an FTP account.
```
cp -r wordpress/* /var/www/html/
```
Update your HTTP configuration with `sudo nano /etc/httpd/conf/httpd.conf`, containing the following content:
```
AllowOverride All
```
then
```
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
### Install the LDMud Game Driver ###
#### Install Dependencies ####
Install the standard developer tools, then tools for mysql, json-c, XML2 and OpenSSL. 
```
sudo yum groupinstall "Development Tools"
sudo yum install mysql-devel
sudo yum -y install libxml2-devel
sudo yum -y install openssl-devel
```
##### Install json-c #####
```
cd ~/
git clone https://github.com/json-c/json-c.git
cd json-c
sh autogen.sh
./configure
make
make install
```
Use `nano` or `vi` to update a file called `/etc/ld.so.conf`.
```perl
/usr/local/lib
```
Update the dynamic loader cache by running: `ldconfig`
#### Download LDMud ####
Clone the game driver from the repository, detach it from source control, prep the game driver for the operating system.
```
cd ~/
git clone https://github.com/ldmud/ldmud.git
cd ~/ldmud
git checkout 3.6
rm -rf .git
```
##### Install python3 ####
***Skip this python3 section for now - This is work-in-progress***
* [Install Python 3.8](https://tecadmin.net/install-python-3-8-amazon-linux/)
```
export PYTHON_LIBS='-L/usr/local/lib/python3.8 -lpython3.8'
export PYTHON_CFLAGS=-I/usr/local/include/python3.8
mkdir ~/ldmud/python
cd ~/ldmud/python
sudo wget https://github.com/ldmud/ldmud/blob/master/doc/examples/python/startup.py
chmod ug+x ~/ldmud/python/startup.py
```
#### Configure Your Game Driver Installation ####
```
cd ~/ldmud/src
./autogen.sh
cd ~/ldmud/src/settings
```
Use `nano` or `vi` to create a settings file called `aoe`.
```perl
#!/bin/sh
#
# Settings for Age of Elements running a native mode driver.
#
# configure will strip this part from the script.

exec ./configure --prefix=/home/ec2-user/ldmud --libdir=/home/ec2-user/ldmud/age-of-elements --with-python-script=/home/ec2-user/ldmud/python/startup.py --with-setting=aoe $*
exit 1

# --- The actual settings ---

enable_erq=erq
enable_compat_mode=no
enable_strict_euids=no
enable_use_parse_command=no
#enable_use_mysql=yes
enable_use_pcre=yes
enable_use_tls=ssl
enable_use_json=yes
enable_use_xml=yes
#enable_use_python=yes

with_master_name=secure/master
with_portno=7680
with_udp_port=7681
```
Set permissions with `chmod ug+rw aoe` and `chmod o+x aoe`.
Run the configuration script, compile then install the game driver.
```
cd ~/ldmud/src
./configure --prefix=/home/ec2-user/ldmud --libdir=/home/ec2-user/ldmud/age-of-elements --with-setting=aoe
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
### Encrypt Data In Transit ###
The Telnet protocol is transmitted as clear text over the Internet and could leave data, such as player passwords, visible to unwanted parties.  To scramble, or encrypt the data in transit, so it can't be read by third-parties on the Internet, create a TLS (Transport Layer Security) Tunnel.  To provide TLS, we'll adapt the linked Mudlet wiki guidance from Paul Saindon of Iron Realms by installing **stunnel** `sudo yum install stunnel` and **ipset** `sudo yum install ipset` for Amazon Linux 2.
* [Sample TLS Configuration](https://wiki.mudlet.org/w/Sample_TLS_Configuration)
#### Create a script to setup iptables ####
Create a shell script that executes after reboot when stunnel starts to setup iptables `sudo nano /usr/libexec/telnet_wrapper_configuration`, containing the following content:
```perl
#!/bin/sh

ipset create stunneled hash:ip,port -exist timeout 300
iptables -t mangle -A PREROUTING -p tcp -m tcp --dport 8680 -j SET --add-set stunneled src,srcport
iptables -t mangle -N DIVERT
iptables -t mangle -A OUTPUT -p tcp -m set --match-set stunneled dst,dstport -m tcp --sport 7680 -j DIVERT
iptables -t mangle -A DIVERT -j MARK --set-mark 1
iptables -t mangle -A DIVERT -j ACCEPT
ip rule add fwmark 1 lookup 100
ip route add local 0.0.0.0/0 dev lo table 100
sh -c "echo 0 >/proc/sys/net/ipv4/conf/lo/rp_filter"
sysctl -w net.ipv4.conf.default.route_localnet=1
sysctl -w net.ipv4.conf.all.route_localnet=1

exit 0
```
Set permissions for the shell script with `sudo chmod 755 /usr/libexec/telnet_wrapper_configuration`.
#### Configure stunnel ####
Create a configuration file for stunnel with `sudo nano /etc/stunnel/stunnel.conf`, containing the following content:
```
; Allow only TLS, thus avoiding SSL
sslVersion = TLSv1.2
pid = /var/stunnel.pid

[telnet]
cert = /etc/letsencrypt/live/ageofelements.org/cert.pem
key = /etc/letsencrypt/live/ageofelements.org/privkey.pem
accept = 8680
connect = 127.0.0.1:7680
transparent = source
```
#### Use systemd to automatically start stunnel at boot ####
Create a unit file to define the systemd service with `sudo nano /etc/systemd/system/stunnel.service`, containing the following content:
```
[Unit]
Description=Encryption wrapper service.

[Service]
Type=simple
ExecStart=/usr/bin/stunnel /etc/stunnel/stunnel.conf
ExecStartPost=/usr/libexec/telnet_wrapper_configuration

[Install]
WantedBy=multi-user.target
```
* Set permissions with `sudo chmod 644 /etc/systemd/system/stunnel.service`.
* Start the service with `sudo systemctl start stunnel`.
* Check the status with `sudo systemctl status stunnel`, which should resemble the content below:
```
stunnel.service - Encryption wrapper service.
   Loaded: loaded (/etc/systemd/system/stunnel.service; enabled; vendor preset: disabled)
   Active: inactive (dead) since Mon 2019-08-12 04:49:39 UTC; 21h ago
  Process: 2704 ExecStartPost=/bin/sh /usr/libexec/telnet_wrapper_configuration (code=exited, status=0/SUCCESS)
  Process: 2703 ExecStart=/usr/bin/stunnel /etc/stunnel/stunnel.conf (code=exited, status=0/SUCCESS)
 Main PID: 2703 (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/stunnel.service
           ├─2740 /usr/bin/stunnel /etc/stunnel/stunnel.conf
           ├─2741 /usr/bin/stunnel /etc/stunnel/stunnel.conf
           ├─2742 /usr/bin/stunnel /etc/stunnel/stunnel.conf
           ├─2743 /usr/bin/stunnel /etc/stunnel/stunnel.conf
           ├─2744 /usr/bin/stunnel /etc/stunnel/stunnel.conf
           └─2745 /usr/bin/stunnel /etc/stunnel/stunnel.conf
```
* Enable the service to start on reboot with `sudo systemctl enable stunnel`.
* Protect data in transit by deleting the TELNET rule from your Security Groups and using the TLS connection.
