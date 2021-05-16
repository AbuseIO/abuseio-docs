# Notes before you start

Obviously, create a backup before you start.

In addition if you upgrade (or have upgrade) onto MySQL 5.7 or higher you need to disable strict mode. If you do
not disable strict mode you will run into problems with the database  migration as laravel stull users ZERO_DATES
instead of 'null' which is now the MySQL default. Instructions are listed in the installation guide.

# Before you start

Update your system to the lastest OS/Package version while you are it!

# Requirements

If you are running 4.0, there are no additional requirements.

# Upgrading from 4.0 to 4.1

```bash
Upgrading from 4.0 to 4.1

echo "!!! MAKE BACKUPS !!!"
su - abuseio

cd /opt
cp -r abuseio abuseio.40-backup
mysqldump abuseio > /opt/abuseio.40/abuseio.sql

git clone --branch 4.1 https://github.com/AbuseIO/AbuseIO.git abuseio-new
cd /opt/abuseio-new
rsync /opt/abuseio/.env /opt/abuseio-new/
rsync -var /opt/abuseio/config/production/ /opt/abuseio-new/config/production/
rsync -var /opt/abuseio/config/development/ /opt/abuseio-new/config/development/
rsync -var /opt/abuseio/config/testing/ /opt/abuseio-new/config/testing/
rsync -var /opt/abuseio/storage/ /opt/abuseio-new/storage/
chown -R abuseio:abuseio /opt/abuseio-new
cd /opt/abuseio-new
php artisan migrate

go live:

cd /opt
mv abuseio abuseio.40-old
mv abuseio-new abuseio
php artisan view:clear
php artisan cache:clear
rm storage/framework/sessions/*
chown -R abuseio:abuseio /opt/abuseio
```


# Post installation

Once your migration has been completed without errors (and you will notice them because they are big and red) you 
can start up all the services again (supervisord or systemd, webserver, mta, etc).

If you are still using supervisor, consider moving onto systemd, instructions are listed in the installation guide.

