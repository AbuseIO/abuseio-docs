# Installation Guide AbuseIO 5.0

# System Requirements

+ Ubuntu 24.04 LTS 64-bit Linux based distribution
+ MTA (Postfix or Exim) or Fetchmail to handle incoming e-mails
+ Web server software (Apache or Nginx)
+ Database backend (MariaDB)
+ PHP 8.4 or better (See Larevel LTS requirements, Both CLI as apache module)
+ (__optional__) Local resolving nameserver (Bind, pDNSRecursor) ([more info](#resolving))

# Preparation

## Pre-install Requirements

- Install a Ubuntu 24.04 LTS system as you normally do, do NOT name the administrative user 'abuseio', but use 'admin' or something similair
- Make sure you configure networking correctly
- If you are using NAT, make sure ports 25, 80 and 443 are mapped onto your system
- Make sure you have a DNS name, e.g. abuseio.yourdomain.tld pointing onto your system

### Ubuntu

Setup firewalling (don't be stupid, you always needs to firewall!)
The 192.168.1.0/24 in this example should be your administrative IP addresses where you want to SSH from, and preferable should be a /32

```
ufw default deny incoming
ufw default allow outgoing
ufw allow smtp
ufw allow http
ufw allow https
ufw allow from 192.168.1.0/24 to any port 22
ufw enable # answer yes
```

Install packages required for AbuseIO
```
apt-get update
apt-get install software-properties-common
LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php # Press enter to confirm.
apt update
apt-get install composer curl git mariadb-server apache2 libapache2-mod-php8.4 php8.4 php-pear php8.4-dev php8.4-mysql php8.4-curl php8.4-intl php8.4-bcmath php8.4-mbstring php8.4-zip
```

In addition you will need to install an MTA. The examples provided are based on postfix which is defaultly used, but you are free to use any MTA (to collection method) you want.  Another solution is using fetchmail, which retrieves mails from a POP3- or IMAP-mailbox, which is described in the advanced user guide. Answering setup with 'Internet Site' and entering a VALID FQDN for the system mail name is a good start. Postfix configuration beyond the minimum operational settings for AbuseIO are out of scope for this documentation.

```
apt-get install postfix
```

Also we want to use SSL. If you have your own certificates, great! and make sure you use them. The installation guide assumes a default Letsencrypt certification:

```
apt-get install certbot python3-certbot python3-certbot-apache
```

### Mailparse
This is a PECL module for PHP that has to be downloaded and compiled before you can use it. The PEAR installation will fail claiming mbstring is not installed, however this is a bug. To prevent this we will install the latest version from source:

```
apt-get -y install gcc g++ make autoconf libc-dev pkg-config
apt-get -y install re2c

cd /tmp
wget https://pecl.php.net/get/mailparse-3.1.9.tgz
tar -zxvf mailparse-3.1.9.tgz
cd  mailparse-3.1.9
phpize
./configure
sed -i 's/^\(#error .* the mbstring extension!\)/\/\/\1/' mailparse.c
make
make install
echo "extension=mailparse.so" > /etc/php/8.4/mods-available/mailparse.ini
phpenmod mailparse

```

Then confirm if the PHP module has been installed (if you see output, you are good):
```
php -i | grep -i 'mailparse support' # Should report enabled
php -m  | grep mailparse # should output mailparse
```

Install latest version of composer

```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'c8b085408188070d5f52bcfe4ecfbee5f727afa458b2573b8eaaf77b3419b0bf2768dc67c86944da1544f06fa544fd47') { echo 'Installer verified'.PHP_EOL; } else { echo 'Installer corrupt'.PHP_EOL; unlink('composer-setup.php'); exit(1); }"
php composer-setup.php
php -r "unlink('composer-setup.php');"
mv composer.phar /usr/bin/composer
apt-mark hold composer
```

Confirm composer version upgrade
```
sudo -u www-data composer --version
```
Expected output 2.8.12 with PHP version 8.4.13 or better.


## Create local user
We're creating local user, 'abuseio', which will be used to run the application.
```
adduser --disabled-password --shell /bin/false --gecos "AbuseIO" abuseio --home /opt/abuseio
usermod -g abuseio abuseio
```

Then add your Apache user and MTA user to the 'abuseio' group.  
Ubuntu defaults would then be:

```
usermod -a -G  abuseio abuseio
usermod -a -G  postfix abuseio
usermod -a -G  www-data abuseio
```
> You will need to restart Apache and Postfix in this example to make your changes active!
> When you're running php-fpm as www-data, you need to restart that as well.
> If you follow this documentation and their defaults, then the last steps are to restart these, so you do not have to do this now.

## Post-install requirements

### Protect MySQL

Secure the mariadb (mysql) installation.

```
mysql_secure_installation
```

Follow all the default settings (Yes to all) and set a MySQL root password (and don't forget it)

# Installation

You can install AbuseIO by downloading a tarball or installing with Composer. Either way will work fine.

> Keep note of the following:  
> - Updating/installing packages with composer might require a GitHub account and a generated token.
> - You should __NOT__ run the `composer update` command, unless you know exactly what you are doing.

## Install from GITHUB
```
cd /opt
mv abuseio abuseio.old
git clone --single-branch --branch 5.0 https://github.com/AbuseIO/AbuseIO.git abuseio
chown -R abuseio:abuseio abuseio
cd abuseio
sudo -u abuseio bash
composer install
exit
```

If you get ANY errors (red) or warnings (yellow) you should investigate the output. 

### Developer extra

If you are a developer and need to push changes back, change HTTP into SSH origin:
```
git remote set-url origin git@github.com:AbuseIO/AbuseIO.git
```


## Permissions
Some parts of the installation run as the root user. Since the application will run as user 'abuseio', we need to set some permissions.

```
cd /opt/abuseio
chown -R abuseio:abuseio .
chmod -R 770 storage/
chmod 770 bootstrap/cache/
```


## Setup framework daemons (Systemd)

```
cp -vr /opt/abuseio/extra/etc/systemd/* /etc/systemd/
systemctl daemon-reload
```

## MTA Delivery

### Postfix
Configure delivery using transport maps

> Make sure the domain you want to use (e.g.: 'isp.local') is in your local domains. The 'isp.local' reference is just a placeholder
> which you should replace with your own domain. Please make sure you can actually receive e-mail on this domain (e.g. MX records,
> open firewall, etc). By using this transport any e-mail send to notifier@isp.local will be pushed (pipe) to the AbuseIO Framework.

Create file /etc/postfix/transport:
```
echo "notifier@isp.local notifier:" >> /etc/postfix/transport
postmap /etc/postfix/transport
```

Set the transport map in the configuration:
```
postconf -e transport_maps=hash:/etc/postfix/transport
```

/etc/aliases:
```
echo "notifier: notifier@isp.local" >> /etc/aliases
newaliases
```

Add this to /etc/postfix/master.cf:
```
notifier  unix  -       n       n       -       -       pipe
 flags=Rq user=abuseio:abuseio argv=/usr/bin/php -q /opt/abuseio/artisan --env=production email:receive

```

Restart postfix:
```
/etc/init.d/postfix restart
```

### Fetchmail
Configure delivery using a POP3- or IMAP-mailbox. This is not needed if you configured Postfix!

Create a fetchmail configuration for your local abuseio user by creating a file ```.fetchmailrc``` in your local users home-directory with the following content:
```
poll your.mail.server proto imap user "your-username" pass "your-password" mda "/usr/bin/php -q /opt/abuseio/artisan --env=production email:receive"
```

Start fetchmail as a deamon for your local user, which checks it's mailbox every 5 minutes (300 seconds):
```bash
su -c "fetchmail -d 300 -s" abuseio
```

## Webserver

### Apache httpd
Setting up a simple virtual host for AbuseIO.
> It is recommended to setup a virtual host with SSL enabled.

Enable modules:
```
a2enmod rewrite
a2enmod headers
```

Create file /etc/apache2/sites-available/abuseio.conf containing:
```
<VirtualHost _default_:80>
  ServerName abuseio.yourdomain.tld
  ServerAdmin webmaster@localhost
  DocumentRoot /opt/abuseio/public

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

  Header always add Strict-Transport-Security "max-age=15552000; includeSubDomains"
  Header always add X-Content-Type-Options "nosniff"
  Header always add X-Frame-Options "SAMEORIGIN"
  Header always add X-XSS-Protection "1; mode=block"

  <Directory /opt/abuseio/public/>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
  </Directory>
</VirtualHost>
```

Enable the website:

```
a2ensite abuseio
a2dissite 000-default.conf
service apache2 reload
```

Setup SSL via Letsencrypt, run Certbot and follow the instructions. When asked use option 2:Redirect

```
certbot
```

## Database setup

### MySQL
Create a database and a user with permissions to the database. This example will use the local database server.

```
mysql -Be "CREATE DATABASE abuseio"
mysql -Be "CREATE USER 'abuseio'@'localhost' IDENTIFIED BY 'CHANGE-THIS-TO-STRONG-PASSWORD'"
mysql -Be "GRANT ALL on abuseio.* to 'abuseio'@'localhost'"
mysql -Be "FLUSH PRIVILEGES"
```

# Configuration

All these things should be done as user 'abuseio' from within the folder /opt/abuseio.

## Environment settings
 
The .env file contains your base configuration and must be set correctly because you will be setting the application's configuration. An example of the file:

```bash
APP_ENV=production
APP_DEBUG=false
APP_KEY=xxx
APP_ID=xxx
APP_URL='http://localhost/'

DB_DRIVER=mysql
DB_HOST=localhost
DB_DATABASE=abuseio
DB_USERNAME=abuseio
DB_PASSWORD=CHANGE-THIS-TO-STRONG-PASSWORD

CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_DRIVER=database

MAIL_DRIVER=smtp
MAIL_HOST=localhost
MAIL_PORT=25
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=false
MAIL_OVERRIDE=false

GDPR_ANONYMIZE_DOMAIN=abuseio.test
```

## Initializing the database

```
sudo -u abuseio bash
cd /opt/abuseio
php artisan migrate
exit
```

If you want some demo data to play with, you should run the following commands:
```bash
sudo -u abuseio bash
php artisan db:seed
extra/notifier-samples/runall-noqueue
exit
```

## Creating an admin user for the GUI

The default installation does not create an admin user unless you seed the demo data. You can create an admin user manually using these commands.

```
sudo -u abuseio bash
cd /opt/abuseio
php artisan user:create admin@isp.local
php artisan role:assign --role admin --user admin@isp.local
exit
```

The user:create command will use default settings for any optional arguments. If the --password option is missing then a randomly generated password will be assigned.

## Start the backend processes

Start the framework daemons, after databases have been initialised:

```bash
systemctl daemon-reload
systemctl enable abuseio_queue_collector
systemctl enable abuseio_queue_email_incoming
systemctl enable abuseio_queue_email_outgoing
systemctl enable abuseio_queue_delegation
systemctl start abuseio_queue_collector
systemctl start abuseio_queue_email_incoming
systemctl start abuseio_queue_email_outgoing
systemctl start abuseio_queue_delegation
systemctl status abuseio_queue*
```

## Cronjobs

Add a crontab for the user 'abuseio'.  
This scheduler job needs to run every minute and will be kicking off internal jobs at the configured intervals from the main configuration. Example:

Run: `crontab -e -u abuseio`
```
* * * * * php /opt/abuseio/artisan schedule:run >> /dev/null 2>&1
```

## Setup local resolving

Some parsers produce high amounts of DNS queries, so you're better off using a local resolver, such as bind.
Once bind is installed, you only need to update your resolver configuration to use 127.0.0.1 as the first resolver. Make sure you leave your original resolvers as the second and third options. In most distributions, you will simply add 127.0.0.1 as the first line in /etc/resolv.conf. In newer versions of Ubuntu, resolvers are configured in /etc/network/interfaces.

## Core configuration

Once completed there are a few settings you will need to configure. First off, be aware that by default your
queue runners are running in --daemon mode (the services from supervisord or systemd). This is great for reducing CPU load, and it 
is a lot faster. However, configuration changes are only read when the supervisord/systemd services start. So, if you change the
configuration then you will need to restart the supervisord services too!

Copy `/opt/abuseio/config/main.php` to the chosen environment folder `/opt/abuseio/config/$ENV/`.
For example, if you want to configure you production environment do:
```
 cp /opt/abuseio/config/main.php /opt/abuseio/config/production/main.php
```
Then setup stuff like the sender of your e-mails, where to bounce, etc in the file `/opt/abuseio/config/$ENV/main.php`.

Also if you have changed the username and/or group where you run AbuseIO under, you will need to update the
config `/opt/abuseio/config/app.php` as well.

Creating a copy in the $ENV folder with the example above allows you to override the defaults specified in the original file. Only the variables in your override config which you modify will actually be used. If the default config contains an option called 'bla', and you remove it from your override config, then the default option will be used. So, you are not required to copy the entire file, but you can just copy the elements needed into a new file. Using a full copy can help simplify your configuration.

## Collector configuration

By default we have included all the known (stable) collectors for AbuseIO which are *disabled* by default. You will need to configure them
before you can use them, as information is needed (e.g. API keys, URLs, etc) before it can work.

These collectors can be found in your installation directory in the subfolder './vendor/abuseio/collector-*/'. In the collector folder
you find the code and its default configuration. Do NOT change the default configuration here, if you want to override the 
default configuration then make a copy the config file (e.g. Rbl.php) to your local config repository 
(./config/production/collector) and edit that file.

## Parser configuration

By default we have included all the known (stable) parsers for AbuseIO which are 
d by default. These parsers can be found in your
installation directory in the subfolder './vendor/abuseio/parser-*/'. In the parser folder you find the code and its default configuration.
do NOT change the default configuration here, if you want to override the default configuration then make a copy the config file (e.g. 
Spamcop.php) to your local config repository (./config/production/parsers) and edit that file.

In the supplied default configuration you will find that all the hooks/triggers might not be fully complete, as the developers have other
sources than you. If you find additional senders, matches, etc then consider letting us know so we can update the default configuration.

The default configuration should get you started and with the sender- and body mapping the parser tells AbuseIO for which sources it can be
used. For example: Spamcop has in its configuration a sender mapping for '/@reports.spamcop.net/'. If the notifier address receives an 
e-mail from that address that matches to this regex then that Spamcop parser is used. Please be very carefull when editing these mappings
and make sure you use the correct regular expression and do not create duplicates.

## Receiving abuse notifications (using notifier)

Once you have completed all the steps in the installation document you can start receiving abuse notifications from reports. You are 
required to setup some kind of forwarding so that these reports will end up in AbuseIO.

You have created an e-mail address 'notifier@isp.local' (which is a placeholder, which really would be something like notifier@amazon.com)
where you can receive your reports to. Usually you would get your reports in abuse@isp.local (or abuse@amazon.com) and you will need to
forward/redirect (which modifying the message!) the e-mails you want handled by AbuseIO to the notifier@isp.local address.

## Failed Jobs

If the notifier (or collector) fails, then the job that was running it will be put into the failed queue. You can check the logs, fix
the problem and retry that job (or delete it) if needed. 
