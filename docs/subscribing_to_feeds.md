# Subscribing to Abuse Feeds

This document will help how you to subscribe to abuse feeds and is in no way meant to be a walk-through, thus some thinking and common sense is required.

# Preface

In general this document is meant for Internet Providers and large networks (Network Operators) as a guide into collecting data from abuse feeds. Unless you actually _own_ the IP Space and/or network (ASN) it will be very hard to register these feeds. If you do not own the IP space please consult your internet provider first before trying to register for any kind of feeds.

# Getting started

## Definitions

ASN: Autonomous System Number
PI: Provider Independent IP Space
PA: Provider Assigned IP Space
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

Our company is operating under the Network ASN AS-ABUSEIO / AS112911.

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

For both the ASN as IP registration you can use the REGISTRATION-TEMPLATE and
send it to request_report@shadowserver.org

### Netcraft

For both the ASN as IP registration you can use the REGISTRATION-TEMPLATE and
send it to takedown@netcraft.com.

Please note that Netcraft is a commercial company and will only send
notifications about phishing and malware attacks where they are performing a
takedown for one their customers. Registering your details will allow them to
simplify contacts and allows your to directly pass the notifications onto AbuseIO.
For a full ISP feed you can contact sales@netcraft.com for details or a trial.

### Google Safe Browsing

Totally free, but you can only register for an ASN feed and requires a Gmail
account to link your company details:

- Create a Gmail account with your company / abusedesk information
- Visit https://www.google.com/safebrowsing/alerts/ and login with that account
- Click 'Add AS', enter your ASN and click continue
- Double check if your number, name and contact matches your company information as it happens frequently that information is outdated!
- Enter the e-mail address under 'notification emails' where you want to receive notifications
- Click on 'send confirmation' to receive the verification e-mail
- Follow the instructions in the confirmation e-mail

### Microsoft JMRP (Junk Mail Reporting Program)

Totally free, but you need an Microsoft account to sign up. 

- Create a Microsoft Live Account  with your company / abusedesk information (can be the same account for Microsoft SNDS)
- Visit https://sendersupport.olc.protection.outlook.com/snds/Jmrp.aspx?view=jj and login with your account.
- Create a new feed

### Microsoft SNDS (Smart Network Data Services)

Totally free, and you can register whole ASN's up to single IP's of your mailservers. It does  require a Live account to link your company details. _IPv6 is not supported at all (!)_ Please note that for some reason the 'whois access' for SNDS is very limited, so it might take a VERY long time before ASN or CIDR requests are approved. Authorization addresses are chosen automatically by an algorithm based on the input requested, either an IP range or an Autonomous System Number (ASN). 

For IP or IP range, it uses two data sources, reverse DNS and WHOIS, each of which can return results independently. For an ASN, SNDS will use WHOIS similarly to how it does for IPs, in that it will authorize any email addresses found in the ASN record maintained by the registrars. Details can be found here: https://postmaster.live.com/snds/faq.aspx#AddressChoosing  

- Create a Microsoft Live Account  with your company / abusedesk information (can be the same account for Microsoft JMRP)
- Visit https://postmaster.live.com/snds and click login. Login with your account.
- Click 'Request Access'
- Enter your ASN (preferred) or your IP(s)
- Now pray if your verification will continue, if not try again ...
- In step 2 select the e-mail address where you receive the validation e-mail
- If you are _NOT_ the IP/Owner you have a textbox where you can enter a description why you need/want this access for the IP/Owner to review. If you are the owner, you can e-mail validate yourself and leave the box empty.
- Click 'Send mail to the chosen address'
- Follow the instructions in the confirmation e-mail

### Spamcop

### SpamExperts

This is a internal feedback loop for SE Customers. Please contact your sales representative
or support desk and ask for a AbuseIO feed. They will provide information how to setup this feed.
