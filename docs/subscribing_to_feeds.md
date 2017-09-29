# Subscribing to Abuse Feeds

This document will help how you to subscribe to abuse feeds and is in no way meant to be a walk-through, thus some thinking and common sense is required.

# Getting started

## Definitions

ASN: Autonomous System Number
PI: Provider Independent IP Space
PA: Privider Assigned IP Space
RIR:
LIR:

## Examples used

This documents explains how the ASN AS-ABUSEIO will request the feeds for the IP spaces 100.64.0.0/10 and 64:ff9b::/96. AS-ABUSEIO is an EU LIR and member of RIPE (RIR).

All ASN', IP spaces, e-mail addresses listed in definitions, examples and requirements are not real and _must_ be replaced with your own information.

Some examples might have different names / objects in other regions. For more information about ASN, IP Spaces and internet routing related question please contact your RIR or LIR as these are out-of-scope for AbuseIO.

## ASN vs IP registration

In most cases notifiers use an ASN to link a feed. This way all IP's announced from that ASN will resolve to that feed notification address and automates changes when an ISP add's or removes IP Spaces. Simply put for Abuse Feeds the ASN is a collection of IP spaces that is automatically resolved. When subscribing AS-ABUSEIO it will automatically add the addresses 100.64.0.0/10 and 64:ff9b::/96.

Some notifiers allow the option for IP Administrators to directly sign up for a feed. If you do not have an ASN you have two options, directly Subscribing to feeds or asking your ISP that has the feeds to pass them along. In such cases you can request all notifications for 100.64.0.0/10 and 64:ff9b::/96 to be send to abusefeed@isp.local. However most feeds will require an ASN or a minimale IP Space size of a IPv4 /24 or IPv6 /47.

## Requirements

### Checking WHOIS information

Its always good to reflect on your own information published inthe whois if this is correct. Make sure people can find you and direct their queries to the right person/department.

Stating the obvious, WHOIS information should be correct. WHOIS is widely used by all kinds of notifiers so it would be wise to add some information on your 'upper object' or organizational(role) element, for example adding elements like:

```
abuse-mailbox: abuse@isp.local
remarks: Please send abuse complaints ONLY to abuse@isp.local
remarks: Please send Responsible Disclosures ONLY to cert@isp.local (GPG ID 0xAB1337)
remarks: Abuse Policy can be found on http://www.isp.local/ap
remarks: Acceptable Use Policy can be found on http://www.isp.local/aup
remarks: Responsible Disclosure Policy can be found on http://www.isp.local/rd
remarks: Escalations can be reported to abuse-escalations@isp.local or by phone to +31 (0) 112 112 112
```

### Setting IP Abuse Contact

The RIR database allows you you to add an abuse contact.

If you own an ASN please make sure you have the abuse-mailbox element configured on your ASN object as well as on your organizational(role) element.

If you do not own an ASN you can still add the abuse-mailbox element on your IP spaces as well as on your organizational(role) element.

### Creating e-mail aliases

For simplicity its best to create a separate e-mail alias for each feed. This allows you to redirect each feed into AbuseIO, receive a copy in your mailbox for debugging or move then outside AbuseIO if needed.

For example:
```
feed-shadowserver@isp.local -> notifier@abuseio.isp.local
feed-netcraft@isp.local -> notifier@abuseio.isp.local
feed-spamcop@isp.local -> notifier@abuseio.isp.local
```

### Define your IP Spaces in AbuseIO

Create a default account and under that create a default contact ISPLOCAL with your own information and IP spaces (100.64.0.0/10 and 64:ff9b::/96) so that AbuseIO knows that these ranges are your local addresses and differentiate from domains under your control, but resolve to IP's outside your network.

# Registration e-mail template

To make things easier, lets start by creating and combined e-mail template. The below template contains all the information and can be used as a boiler template to send out e-mails to notifiers.

REGISTRATION-TEMPLATE:
```
Hello {$feed},

I am {$person_full_name} and responsible for Abuse Management for the network(s) listed below.

We would request to receive your feed(s) / automated notifications on the dedicated address of feed-{$feed}@isp.local

{start ASN part}

Out company is operating under the Network ASN AS-ABUSEIO / AS112911.

{end ASN part}

{start IP part}

Our company is currently responsible for the following the IP Spaces:

100.64.0.0/10
64:ff9b::/96

{end IP part}

Our Company information:

ISPLOCAL Inc. (ORG-ISPLOCAL-RIPE)
{website URL}
{$address line1}
{$address line2}
{$generic mailbox}
{$phone line1}

Our Abuse contact information:

Automation feed address: feed-{$feed}@isp.local
Human read address: abuse@isp.local
Human phone contact: +31 (0) 112 112 112
Escalation and 24/7 contact: +31 (0) 911 911 911


If you have and questions, feel free to contact me directly.

With regards,

{$person_full_name}
ISPLOCAL Inc.
{$person mail}
{$phone line1}

```

# Available feeds

### Shadowserver

For both the ASN as IP registration you can use the REGISTRATION-TEMPLATE and send it to request_report@shadowserver.org

