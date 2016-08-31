# How incoming data is received

The system has several methods of receiving abuse events:

1. By e-mail directly, using the MDA delivery (pipe) directly into the Artisan handler
2. By e-mail indirectly, via logging into a mailbox and collecting the e-mails that can be parsed
3. By RPC push notifications
4. By collecting data from remote hosts, e.g. HTTP(s) or RSS subscriptions
5. By directly inputting data from the CLI, which is handy if you have your own local scantools

Once the data has been received the datafile will be saved onto the archive and a processor job is queued to change
this raw data into tickets and events we can then use for notifications. The processor job will be calling each step
in the process to end up with a parsed data set in the database.

# The Processor job

## Dissecting the e-mail / raw data

In most cases you will receive an e-mail or raw data. In the first stage, we convert the data into an object, for example a parsedEmail object which allows easy access to specific areas of that object like headers or RFC822 evidence.

## Calling the right handler for the job

Since every notifier and sometimes even individual feeds from the same notifier have different formats, AbuseIO provides specific handlers to convert the data object into events. A received object might contain a single event (e.g. a spam feedback loop) or multiple events (e.g.: a large combined CSV dataset).

Each handler (parser, collector, or otherwise) is its own package. AbuseIO releases provide a full set of all available production-ready handlers. You also have the option of adding, removing or updating individual packages.

Based on all available parser configurations, AbuseIO will determine which handler can be used to handle the dataset. This decision is made by two methods:

- By sender address mapping, which relies on the 'from:' address. In most cases, this information allows us to determine who the notifier is, and which feed it comes from
- By body mapping, which uses a regex to match an area of the raw message

Once a handler (parser) has been selected it will be called to parse the dataset. At this stage, an event object is created for each event found in the dataset. An event object contains:

- **timestamp**: This is the timestamp for when the notifier has seen this event (not necessarily when the data was received)
- **source**: The name of the notifier/parser name that handled the event
- **information**: A multilayered array of data regarding the event
- **class**: The internal classification, using a preset list to be used for showing information texts
- **type**: If the event is informational, abuse or escalation. This triggers different kinds of notifications.
- **ip address**: The IP address of the events
- **domain name**: Optional: The domain name regarding the event
- **uri**: Optional: The URI where the event was found/directed to

## Validations

If you are writing your own parser or collector, you will notice that there is only minimal validation done on that level. That is because AbuseIO handles everything besides parsing within the main framework. This will allow AbuseIO to ensure a parser will provide data we can actually use. In short, a parser should only dissect the information blob into event objects.

Once a parser is done parsing, the events are returned in a standard format. Since each event object should be exactly in the same format, this saves a lot of effort on creating validations for custom parsers.  Next, the validator is called to go through each of the event objects and validate each part of the event. For example the validator will check whether the field 'ip' actually contains a valid IPv4 or IPv6 address.

## Saving evidence data

Once the data has successfully been validated, we can write the evidence to storage. This evidence object is a link containing some basic information (such as sender) and the physical location of the file where it was stored. The ID of the evidence is linked to each of the events that were generated from this evidence.

The raw dataset is written unchanged to the filesystem for reference. This will allow later access to the evidence. Rescans can be forced if needed, as well.

## Saving tickets and events

AbuseIO will check for an existing case using a query in the ticket system based on IP, domain, class, type and IP owner. If this combination of items already has an existing ticket then the event will be added onto this existing ticket. In other cases, it will create a new ticket with the event attached to it.

Here we will split up the event into two sections:

- **ticket data**: Static data, like the IP address, class, etc that will not change
- **event data**: Dynamic data, that mainly contains the data blob of information from the notifier

After splitting these sections we will enrich the data with contact information for the IP owner and/or domain owner. This data will be added onto the ticket data and will be static from that point on. The reason that this data remains static is because the same IP may be reassigned to a different owner (for example, if DHCP is used).

## Sending out notifications

!!! note "Note about domains"
    Notifications are provided to the owner of the IP, since the abuse originates with the IP address. If a domain name is referenced in the ticket, then AbuseIO will notify the domain owner as well. This will result in two different of behaviours.

    If the IP owner changes (or domain, class, etc) the system will consider it a new incident and will create a new ticket. If the domain owner changes, then the existing ticket will remove the old domain contact and replaces it with the new contact. Changing the domain contact will also result in changing the access token to ASH and only the 'new' owner will have access to the ticket.

There are 3 kinds of notifications that can be sent out either as a single notification or in conjunction with each
other. These notifications are:

- E-mail message: An e-mail with a simple text referring the contact to ASH for more information
- RPC message: a IODEF formatted message for contacts that automate their abuse handling (e.g. resellers)
- External message: a customized notification that is called (e.g. IRC notification, or creating a blackhole for the IP via the a login to the router).

Based on the ticket type (Info, Abuse, Escalation) the notifications have configurable intervals. By default we send abuse notifications every 15 minutes and informational notifications every 90 days.

## Data collection errors

Parsers that only set a warning counter that increase when the parsers has hit a snag. Additional warnings are automatically added by parser-common for events like where a parser did not return any event at all.

By default, the handler is set to treat these warnings as errors and will not even try to continue validating or saving the found events. Instead, the original EML file is added onto an e-mail which is sent to the admin to investigate. In most cases this is the result of a notifier changing the format or misconfiguration.

If you want to continue and try to continue validating and saving the event, you can change the setting parser_warnings_are_errors to false.

## Retrying data collection

If a parser had problems and only partially saves the events found in the data set, you can simply just retry the parsing by reintroducing the e-mail onto the system (either by bouncing the EML or using CLI tools) so the parser will try to handle the e-mail again.

You will not have to worry about getting duplicates, as there is a filter on saving events that are an exact match. Only if there is actually other data (e.g. timestamp, or a infoblob value) will the event will be saved.

# Local and remote contact data

The system comes with a built-in database for registering IP owners, netblocks and domains. This local database always has preference above remote data calls. If you have an external database to collect this data from you can leave these sections empty or use them as an override of your remote data call.

For each of the items (contacts, netblocks and domain) you can specify your own backend(s). For example, you could use NIPAP for your IP administration (which contains the customer IDs), and SAP Accounting (which holds your customers' contact information). In this case you can use them both to fill in the contact data.

The integration of remote contact data is explained in its own chapter.

# Changing the default parser/collect configuration

Each parser has its own dedicated configuration. This can be divided into four sections. Lets take a look at the parser for Shadowserver which can be found in ./vendor/abuseio/parser-shadowserver/.

This directory contains the parser in the /src/ directory and the configuration in the /config/ directory. Inside the config directory you will find the Shadowserver.php containing the default configuration (which you should never edit). Instead, you can create a copy of the config/Shadowserver.php into $app/config/$environment/parsers/.

The contents of the config file is simply a single PHP array with the configurable items. You will not have to recreate the entire array, but only need to create the same array keys with values you want to change. The configuration will be merged and your custom configuration will be overlaid onto the existing array.

There are several configurable options built-in. For example, you are able to change the notifier information and the per-feed settings from that notifier without having to change the parser code itself.
