# Installation Guide AbuseIO 4.1

# System Requirements

+ 64-bit Linux based distribution
+ MTA (Postfix 2.9.1+, Exim 4.76+) or Fetchmail
+ Web server software (Apache 2.22+ or Nginx 1.1.19+)
+ Database backend (MySQL 5.5+, Postgres 9.1+)
+ PHP 5.5.9+ (Both CLI as apache module)
+ (__optional__) Local resolving nameserver (Bind, pDNSRecursor) ([more info](#resolving))

# Automated installation

This documentation will guide your thru a basic installation. However if you like to do automation with Docker or Ansible, you might
want to check out the following repositories:

Ansible:
```
http://www.github.com/abuseio/abuseio-ansible/
```

Docker:
```
http://www.github.com/abuseio/abuseio-docker/
```

See docker installation and configuration instructions below:

[a relative link](installation_via_docker.md)


# Preparation

## Pre-install Requirements

### Ubuntu
```
apt-get install curl git mysql-server apache2 apache2-utils libapache2-mod-php7.0 php7.0 php-pear php7.0-dev php7.0-mcrypt php7.0-mysql php7.0-pgsql php7.0-curl php7.0-intl php7.0-bcmath php7.0-mbstring php7.0-zip
```
If you intend to use the supervisor instead of systemd, you will need to install it:

```
apt-get install supervisor
```

In addition you will need to install an MTA. The examples provided are based on postfix, but you are free to use any MTA (to collection method) you want.

```
apt-get install postfix
```

In addition you will can install an MTA. The examples provided are based on postfix, but you are free to use any MTA (to collection method) you want. Another solution is using fetchmail, which retrieves mails from a POP3- or IMAP-mailbox.

### CentOS
Still a work in progress, but minimal:
```
php-bcmath, supervisor
```
Please keep in mind AbuseIO was developed on Ubuntu/Debian systems and there might be some assumptions in this installation document. Please keep us posted if you find such situations, so we can improve the documentation!

### Composer
Although Composer is not required, we highly recommend that you install Composer as it allows you to easily update certain parts of the system. You can install AbuseIO without Composer by downloading the premade .TAR archive from our website.

Download the latest version of [Composer](https://getcomposer.org/) and make it accessible system-wide.
```
cd /tmp
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
chmod 755 /usr/local/bin/composer
chown root:root /usr/local/bin/composer
```


### Mailparse
This is a PECL module for PHP that has to be downloaded and compiled before you can use it.
If you're running PHP7 or later, run:
```
pecl install mailparse
```
If you run into this compiler error:
```
/tmp/pear/temp/mailparse/mailparse.c:34:2: error: #error The mailparse extension requires the mbstring extension!
 #error The mailparse extension requires the mbstring extension!
  ^
Makefile:200: recipe for target 'mailparse.lo' failed
make: *** [mailparse.lo] Error 1
ERROR: `make' failed`
```
You should do:
```
vi /usr/include/php/20151012/ext/mbstring/libmbfl/mbfl/mbfilter.h
```
Then add the following lines right under `#define MBFL_MBFILTER_H`:
```c
#undef HAVE_MBSTRING
#define HAVE_MBSTRING 1
```
And rerun the pecl install command.

If you're running PHP5.6 or older, run:

```
pecl install mailparse-2.1.6
```
On some systems, the above command fails. If it does, try adding -Z after 'install'.

```
echo "extension=mailparse.so" > /etc/php/7.0/mods-available/mailparse.ini
phpenmod mailparse
phpenmod mcrypt
```
> Note: Replace "php5" in the above command with "php/7.0" if you're running PHP7.

## Create local user
We're creating local user, 'abuseio', which will be used to run the application.
```
adduser abuseio
```

Please make sure that your abuseio user has the default group into abuseio. Apparently it might be abuseio:users instead:
```
usermod -g abuseio abuseio
```

Then add your Apache user and MTA user to the 'abuseio' group.  
Ubuntu defaults would then be:

```
addgroup abuseio abuseio
addgroup postfix abuseio
addgroup www-data abuseio
```
> You will need to restart Apache and Postfix in this example to make your changes active!
> When you're running php-fpm as www-data, you need to restart that as well.

## Post-install requirements

### MySQL 5.7+ default change

If you are running MySQL 5.7 or higher, the default value for a ZERO_DATE is no longer accepted. Laraval
still uses ZERO_DATE instead of NULL and running database migration will result into errors! Therefor you
_MUST_ disable strict mode:

_/etc/mysql/conf.d/disable_strict_mode.cnf_
```
[mysqld]
sql_mode=IGNORE_SPACE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

restart MySQL after:

```
systemctl restart mysql
```

# Installation

You can install AbuseIO by downloading a tarball or installing with Composer. Either way will work fine.

> Keep note of the following:  
> - Updating/installing packages with composer might require a GitHub account and a generated token.
> - You should __NOT__ run the `composer update` command, unless you know exactly what you are doing.

## Install from tarball
```
cd /opt
wget https://abuse.io/releases/abuseio-latest.tar.gz
tar zxf abuseio-latest.tar.gz
```

## Install with Composer
Install the latest stable version:
```
cd /opt
composer create-project abuseio/abuseio
```

Install the latest version:
```
cd /opt
composer create-project abuseio/abuseio --stability=beta (options are: stable, RC, beta, alpha, dev)
```

## Permissions
Some parts of the installation run as the root user. Since the application will run as user 'abuseio', we need to set some permissions.

```
cd /opt/abuseio
chown -R abuseio:abuseio .
chmod -R 770 storage/
chmod 770 bootstrap/cache/
```


## Copy logrotate and rsyslog configs
> Note: The user/group 'syslog:adm' is prorably a Debianism, and could differ for your Linux distribution!
```
mkdir /var/log/abuseio
chown syslog:adm /var/log/abuseio
cp -vr /opt/abuseio/extra/etc/logrotate.d/* /etc/logrotate.d/
cp -vr /opt/abuseio/extra/etc/rsyslog.d/* /etc/rsyslog.d/
service rsyslog restart
```

> Important: Do NOT use supervisor AND systemd at the same time.

## If you're using systemd:
### On Debian / Ubuntu-likes
```
cp -vr /opt/abuseio/extra/etc/systemd/* /etc/systemd/
systemctl daemon-reload
```

There's a slight variation in the names of required services on different Linux distributions.
On CentOS, make sure the 'Requires=' lines in the service files point to the correct service names or the applications will fail to start.

### On CentOS / RedHat-likes
```
cp -vr /opt/abuseio/extra/etc/systemd/* /etc/systemd/
sed -i -e 's/mysql/mysqld/' -e 's/apache2/httpd/' /etc/systemd/system/abuseio*.service
systemctl daemon-reload
```

> Important: Do NOT use supervisor AND systemd at the same time.

## If you're using supervisor:
```
cp -vr /opt/abuseio/extra/etc/supervisor /etc/
supervisorctl reread
/etc/init.d/supervisor restart
supervisorctl stop abuseio_queue_collector
supervisorctl stop abuseio_queue_email_incoming
supervisorctl stop abuseio_queue_email_outgoing
supervisorctl stop abuseio_queue_delegation
```

> Important: Leave these supervisor or systemd jobs stopped until you completed the entire installation process,
or you might get a lot of errors in your logs!

> Important: The supervisord worker threads run in daemon mode. This will allow the framework to
be cached and will save a lot of CPU cycles. However if you edit the code in _ANY_ way you will need
to restart these daemons to prevent jobs from failing. A better option would be to stop the daemons before making a code change, and starting them after the change is complete.

> Note: If you get messages on 'hanging' jobs its most likely these supervisor jobs are not running.
Please make sure you see running processes from the configured supervisor jobs before submitting
a bug report.

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
Configure delivery using a POP3- or IMAP-mailbox.

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

If you are migrating from version 3.x you can add use the ash-abuseio.domain.tld with documentroot /opt/abuseio/public/legacy to provide a nice redirection by converting tokens.

```
a2ensite abuseio
service apache2 reload
```

### Nginx
This is an example configuration for AbuseIO via Nginx with PHP-fpm. Change it to suit your own setup.  
Make sure the PHP-fpm processes run as user abuseio.

Create file /etc/nginx/sites-available/abuseio containing:
```
server {
  listen 80;
  server_name abuseio.myserver.tld;
  root /opt/abuseio/public;
  index index.php;

  location ~ \.php$ {
    try_files $uri =404;
    fastcgi_pass unix:/var/run/fpm_abuseio.socket;
    fastcgi_index index.php;
    include fastcgi_params;
  }

  location / {
    try_files $uri $uri/ /index.php?q=$uri&$args;
  }
}
```
```
ln -s /etc/nginx/sites-available/abuseio /etc/nginx/sites-enabled/001-abuseio
service nginx reload
```


## Database setup

### MySQL
Create a database and a user with permissions to the database. This example will use the local database server.

```
mysqladmin -p create abuseio
mysql -p -Be "CREATE USER 'abuseio'@'localhost' IDENTIFIED BY '<password>'"
mysql -p -Be "GRANT ALL on abuseio.* to 'abuseio'@'localhost'"
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

DB_DRIVER=mysql
DB_HOST=localhost
DB_DATABASE=abuseio
DB_USERNAME=abuseio
DB_PASSWORD=<password>

CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_DRIVER=database
```

## Initializing the database

```bash
cd /opt/abuseio
php artisan migrate
```

If you want some demo data to play with, you should run the following commands:
```bash
php artisan db:seed
extra/notifier-samples/runall-noqueue
```

## Creating an admin user for the GUI

The default installation does not create an admin user unless you seed the demo data. You can create an admin user manually using these commands.

```
cd /opt/abuseio
php artisan user:create admin@isp.local
php artisan role:assign --role admin --user admin@isp.local
```

The user:create command will use default settings for any optional arguments. If the --password option is missing then a randomly generated password will be assigned.

## Start the supervisor processes again

Start the framework daemons, after databases have been initialised:

```bash
supervisorctl start abuseio_queue_collector
supervisorctl start abuseio_queue_email_incoming
supervisorctl start abuseio_queue_email_outgoing
supervisorctl start abuseio_queue_delegation
```

or with systemd:

```bash
systemctl start abuseio_queue_collector
systemctl start abuseio_queue_email_incoming
systemctl start abuseio_queue_email_outgoing
systemctl start abuseio_queue_delegation
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

By default we have included all the known (stable) parsers for AbuseIO which are enabled by default. These parsers can be found in your
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
