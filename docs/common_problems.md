# Common AbuseIO problems / FAQ

## 1. Supervisord services not running (properly)

**Symptoms:**

- You get incoming e-mails that are not processed, although they were logged as received
- AND/OR You are missing data from tickets you are sure have been mailed to the system
- AND/OR You see a lot of jobs when running `php artisan queue:list`
- AND/OR You get an alert of jobs being stuck in the queue

**Possible cause:**

Supervisord services not running. In most cases, the supervisor was asked to start after installation, but it was not able to since the database was not fully installed yet.

Once an e-mail has been received, it is put into a queue to handle and the MTA part has been completed. The MTA can return an 200 OK message to the sender. If the supervisord queue daemons are not running, no jobs are picked up from the queue and remain there until worker process has been started.

**Solution:**

Restart the supervisord services (currently 3) or the entire supervisord daemon, which is safe to do if this is system
is dedicated to AbuseIO. After restarting it you will see in the /var/log/abuseio/queue* files the workers to pick up
work and start handling the data from the received e-mails.


## 2. Previous configuration remains active after updating configuration files

**Symptoms:**

- Your configuration options that have changed still reflect the old settings
- AND/OR Parsers with changed configuration (e.g. enabling/disabling them) still reflect the old settings

**Possible cause:**

Laravel Framework has not been reloaded.

To conserve CPU/memory, the framework and its configuration are only loaded once for all the daemons. After changing a configuration you will need to reload the framework to make the new configuration 'active'.

**Solution:**

Restart the supervisor processes.

```
supervisorctl restart abuseio_queue_collector
supervisorctl restart abuseio_queue_email_incoming
supervisorctl restart abuseio_queue_email_outgoing
```
