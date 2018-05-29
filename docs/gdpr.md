# GDPR documentation

AbuseIO can create a report for all data containing an emailaddress, this report will include contact- and ticketdata related to the specified email address.
You can also use AbuseIO to anonymize this data, when a Ticket or Contact is anonymized, all personal data will be hashed.
Both these functions are exported to the console and api and can be called using php artisan or a simple api call

### Console commands

You can use php artisan to create a report and to anonymize all contacts / tickets for an email address
    
    $ php artisan gdpr:report example@example.org             # Create a report with all data related to the email example@example.org
    $ php artisan gdpr:anonymize example@example.org --yes    # Anonymize all personal contact data

### API calls
Only the anonymize methods are exported through the api, you can query tickets and contacts through the search api call.
For specifics see the [API Documentation](api.md)

There is an anonymize API call for both Tickets and Contacts, however you can also call the gdpr/anonymize API call, which will anonymize both Tickets and Contacts in one call.

### UI
not yet implemented