### SSH Login as root
```
ssh root@SERVER_IP_ADDRESS
```

### Create a New User
replace username with your desired name
```
adduser username
```

### Give Root Privileges
replace username with your desired name
```
usermod -aG sudo username
```

### SSH Login as username
```
ssh username@SERVER_IP_ADDRESS
```

### Setup a Basic Firewall
to see all available applications
```
sudo ufw app list
```
first we allow OpenSSH
```
sudo ufw allow OpenSSH
```
enable Uncomplicated Firewall
```
sudo ufw enable
```
listing rules with a reference number
```
sudo ufw status numbered
```
allow port 80
```
sudo ufw allow 80
```
deny rule with reference number (replace x with the reference number)
```
sudo ufw deny x
```
__Full guidebook:__ https://help.ubuntu.com/community/UFW

### Install Apache and Allow in Firewall
install Apache server
```
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install apache2
```
set Global ServerName to suppress syntax warnings
```
sudo nano /etc/apache2/apache2.conf
```
at the bottom of the file, add a ServerName directive, pointing to the primary domain name
```
. . .
ServerName server_domain_or_IP_or_any-name-like-earth-or-mars-etc
```
now check for syntax errors by typing
```
sudo apache2ctl configtest
```
the output should be like
```
Syntax OK
```
restart Apache to implement changes
```
sudo systemctl restart apache2
```
check available applications in ufw
```
sudo ufw app list
```
allow __incoming traffic__ for 'Apache Full' profile
```
sudo ufw allow in "Apache Full"
```

### Configure Name-based Virtual Hosts
disable the default Apache virtual host
```
sudo a2dissite *default
```
navigate to /var/www/html directory
```
cd /var/www/html
```
create a folder to hold your website, replacing ```example.com``` with your domain name
```
sudo mkdir example.com
```
create a set of folders inside the _example.com_ folder to store website’s files, logs, and backups
```
sudo mkdir -p example.com/public_html
sudo mkdir -p example.com/log
sudo mkdir -p example.com/backups
```
create the virtual host file for the website
```
sudo nano /etc/apache2/sites-available/example.com.conf
```
some basic settings for the virtual host file _example.com.conf_
```
# domain: example.com
# public: /var/www/html/example.com/public_html/

<VirtualHost *:80>
  # Admin email, Server Name (domain name), and any aliases
  ServerAdmin webmaster@example.com
  ServerName  example.com
  ServerAlias www.example.com

  # Index file and Document Root (where the public files are located)
  DirectoryIndex index.html index.php
  DocumentRoot /var/www/html/example.com/public_html
  # Log file locations
  LogLevel warn
  ErrorLog  /var/www/html/example.com/log/error.log
  CustomLog /var/www/html/example.com/log/access.log combined
</VirtualHost>
```
enable the website
```
sudo a2ensite example.com.conf
```
restart Apache server
```
sudo service apache2 restart
```

### Install and Secure MySQL
install
```
sudo apt-get install mysql-server
```
secure
```
sudo mysql_secure_installation
```

### Install PHP
install php and few helper packages
```
sudo apt-get install php libapache2-mod-php php-mcrypt php-mysql
```
if the Apache needs to look for an index.php file first (for any php site), edit this file
```
sudo nano /etc/apache2/mods-enabled/dir.conf
```
restart Apache web server
```
sudo systemctl restart apache2
```
test php processing on the web server
```
sudo nano /var/www/html/info.php
```
inside info.php file
```
<?php
phpinfo();
?>
```

### Install and Secure phpMyAdmin
```
sudo apt-get install phpmyadmin php-mbstring php-gettext
```
> For the server selection, choose ```apache2```.

> Select ```yes``` when you are asked whether to use ```dbconfig-common``` to set up the database.

explicitly enable php mcrypt and mbstring extensions
```
sudo phpenmod mcrypt
sudo phpenmod mbstring
```
restart Apache
```
sudo systemctl restart apache2
```
secure phpMyAdmin instance using Apache's built-in ```.htaccess``` authentication and authorization functionalities
```
sudo nano /etc/apache2/conf-available/phpmyadmin.conf
```
add an ```AllowOverride All``` directive
```
<Directory /usr/share/phpmyadmin>
    Options FollowSymLinks
    DirectoryIndex index.php
    AllowOverride All
    . . .
```
restart Apache
```
sudo systemctl restart apache2
```
install an additional package
```
sudo apt-get install apache2-utils
```
create an ```.htaccess``` File
```
sudo nano /usr/share/phpmyadmin/.htaccess
```
add the following information
```
AuthType Basic
AuthName "Restricted Files"
AuthUserFile /etc/phpmyadmin/.htpasswd
Require valid-user
```
now set a secret user to access phpMyAdmin
```
sudo htpasswd -c /etc/phpmyadmin/.htpasswd username
```
to set additional user, we do not need the ```-c``` flag.
```
sudo htpasswd /etc/phpmyadmin/.htpasswd additionaluser
```
to remove a user, edit ```.htpasswd``` file using nano

### Secure Apache with Let's Encrypt
download the ```certbot-auto``` Let’s Encrypt
```
cd /usr/local/sbin
sudo wget https://dl.eff.org/certbot-auto
```
make the script executable
```
sudo chmod a+x /usr/local/sbin/certbot-auto
```
obtain and install certificate
```
certbot-auto --apache
```
verify the status of your SSL certificate
```
https://www.ssllabs.com/ssltest/analyze.html?d=example.com&latest
```
setup auto renewal
```
certbot-auto renew
```
install cron
```
sudo apt-get install cron
```
edit the crontab to create a new job that will run the renewal command every week
```
sudo crontab -e
```
include the following content, all in one line; this will create a new cron job that will execute the ```letsencrypt-auto renew``` command every Monday at 1:30 am.

```
30 1 * * 1 /usr/local/sbin/certbot-auto renew >> /var/log/le-renew.log
```
install soap
```
sudo apt-get install php-soap
```
