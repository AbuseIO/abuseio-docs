# Common AbuseIO problems / FAQ

## 1. Supervisord services not running (properly)

Sympthoms: 

- You will get incoming e-mails, their logged as being received but the e-mail is not processed 
- AND/OR You are missing data from tickets you are sure have been mailed onto the system
- AND/OR You will see a lot of jobs when running `php artisan queue:list`
- AND/OR You get a alert of jobs being stuck in the queue

Possible cause:

Supervisord services not running. Most likely after installation the supervisor was asked to start, however
because the database has not been fully installed yet the daemon did not start.

Once an e-mail has been received it is put into a queue to handle and the MTA part has been completed. The MTA can 
return an 200 OK message to the sender. If the supervisord queue daemons are not running, no jobs are picked up from
the queue and remain there until worker process has been started.

Solution:

Restart the supervisord services (currently 3) or the entire supervisord daemon, which is safe to do if this is system
is dedicated to AbuseIO. After restarting it you will see in the /var/log/abuseio/queue* files the workers to pick up
work and start handling the data from the received e-mails.


## 2. Still having the old configuration active after changing config files

Sympthoms: 

- your configuration options that have changed still reflect the old settings
- AND/OR parsers with changed configuration (e.g. to enable them) still reflect the old settings

Possible cause:

Laravel Framework has not been reloaded. 

To save a lot of CPU/Memory the framework and its configuration is only loaded up once for all the daemons. After changing a configuration you will need to reload the framework to make the new configuration 'active'.

Solution:

Restart (all) the supervisor processes of AbuseIO, for example:

supervisorctl restart abuseio_queue_collector
supervisorctl restart abuseio_queue_email_incoming
supervisorctl restart abuseio_queue_email_outgoing
