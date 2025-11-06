# Notes before you start

## Stategy summery

- Create a new and clean installation of 5.x
- Transfer the database from 4.3 into 5.0
- Run migrations

# Making a new installation

Create a new system based on the default installation guide. You are advised to seed the database and create a new admin user for testing the new installation. When we restore your 4.x database this will be overwritten with your 4.3 install.

This is needed because you are likely still running Ubuntu 18.04 with old versions, and upgrading is not only more time consuming ... having a clean start is better at this stage. It also allows you to easily rollback if problems arise and you need to change back.

# Backup

Obviously, create a backup before you start. You will need to make a copy on your old system (locally):

- a copy of /opt/abuseio, into /opt/abuseio-old/
- mysql backup dump of the abuseio database, into /opt/abuseio-old/database.sql
- changes done to postfix, systemd, etc. if you are not using the defaults (these are out of scope for this guide)

Then rsync /opt/abuseio-old onto your new system, also into /opt/abuseio-old.

# Upgrading from 4.3 to 5.0

These steps are following the install guide (see abuse)

```
rsync /opt/abuseio-old/.env /opt/abuseio/
rsync -var /opt/abuseio-old/config/production/ /opt/abuseio/config/production/
rsync -var /opt/abuseio-old/config/development/ /opt/abuseio/config/development/
rsync -var /opt/abuseio-old/config/testing/ /opt/abuseio/config/testing/
rsync -var /opt/abuseio-old/storage/ /opt/abuseio/storage/
chown -R abuseio:abuseio /opt/abuseio
mysql --database abuseio < /opt/abuseio-old/database.sql
```

Update your .env :
- DB_DRIVER=mysql
+ DB_DRIVER=mariadb

```
sudo -u abuseio /bin/bash
cd /opt/abuseio
php artisan migrate
php artisan view:clear
php artisan cache:clear
rm storage/framework/sessions/*
```

# Post installation

Once your migration has been completed without errors (and you will notice them because they are big and red) you 
can start up all the services again (supervisord or systemd, webserver, mta, etc).

If you are still using supervisor, consider moving onto systemd, instructions are listed in the installation guide.

