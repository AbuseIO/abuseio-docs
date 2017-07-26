# Common AbuseIO problems / FAQ

## 1. Services not running (properly)

**Symptoms:**

- You get incoming e-mails that are not processed, although they were logged as received
- AND/OR You are missing data from tickets you are sure have been mailed to the system
- AND/OR You see a lot of jobs when running `php artisan queue:list`
- AND/OR You get an alert of jobs being stuck in the queue

**Possible cause:**

Services not running, like the systemd, supervisor or other service method used during installation. In most cases, the service was asked to start after installation, but it was not able to since the database was not fully installed yet.

Once an e-mail has been received, it is put into a queue to handle and the MTA part has been completed. The MTA can return an 200 OK message to the sender. If the supervisord queue daemons are not running, no jobs are picked up from the queue and remain there until worker process has been started.

**Solution:**

Restart the services (currently 3) or the entire supervisord daemon, which is safe to do if this is system
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

## 3. Notifications are not being sent out

**Symptoms:**

- You have a ticket with valid incidents
- AND the ticket has not send out any notifications
- AND the ticket contains valid contact information, including auto_notications to be enabled
- AND manually triggering notifications work fine

**Possible cause:**

You are using data (or e.g. the provided samples in /extra/notifier-samples/) which hold data which is older then the permitted notification threshold.

This is a configuration setting (main configuration, under notifications.min_lastseen) which defaults to 14 days. This would mean that if the ticket holds only incidents older then 14 days it would continue to collect data but not notify as its considered to old to report. This is expected behaviour to prevent alerts being send out that are useless. If the problem still exist at some point a incident would be reported within the threshold for notifcations.

**Solution:**

Non are required, as this is normal behaviour. However when testing the system and the need to use older samples, e.g. from the supplied samples directory, you will need to change this threshold e.g. to 10 years. But be aware you should set this back when going production. Best way to go is to set your env to 'testing' and set the config/testing/main.php with the desired settings. Do not use this in production unless you want to notify old data!

Another options would be to update the samples to reflect newer dates, however that would be more work.
