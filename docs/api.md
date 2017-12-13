# API Documentation

**This is a living document and can be updated**

AbuseIO 4.1+ implements an **experimental!!** [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer) CRUD API for most of its models.
The API is good enough to work, but isn't completely stable, sometimes you'll get some ugly errors 

## How to use

### Config settings
    main.api.enabled (main.php)
The API is enabled by default, setting this option to false will disable the API.

    APP_URL (.env) 
    app.url (app.php)
Specifies the base url on which the application is running, this url will also be used by the API.

### Endpoint

All the API methods are reachable on the base endpoint

    http://{APP_URL}/api/v1/{MODEL}

### Making a request

When making an API request you to send an API token, each account has it's own token, so you can use the api as a specific account.

**Important: currently this is only implemented in the ticket model, all other models need to use the system accounts token**

Note: At the moment we haven't implemented pagination, for Tickets you can emulate this in a search query. See the Tickets section for more information.

#### Setting the API token
You need to send the token using a custom header "X-API-TOKEN", which contains the API token of the account (in most cases this will be the system account)

All data send to and received from the request/response is in JSON.

#### Responses
All responses are in JSON and have two parts, the 'data' part and the 'message' part.

    {
      "data":
      {
        "id":6,
        "name":"account1",
        "description":"My account",
        "brand_id":1,
        "disabled":false
      },
      "message":
      {
        "code":"SUCCESSFULL",
        "message":"success",
        "http_code":200,
        "success":true
      }
    }

The data part contains one or more returned model objects on success and is empty when the call errors.
The message part contains the result message (if it was a success or a specific error message), the resulting http_code and a success status.

## Supported Models

### Accounts

#### Model

    account:
    [
      "id" => 1,
      "name" => "Default",
      "description" => "Default system account",
      "disabled" => 0,
      "brand_id" => 1,
      "systemaccount" => 1,
    ]

#### Available methods

| Method | Endpoint                                 | Description                         |
|--------|------------------------------------------|-------------------------------------|
| GET    | http://{APP_URL}/api/v1/accounts      | List all accounts                   |
| GET    | http://{APP_URL}/api/v1/accounts/{id} | Show the account with id {id}       |
| POST   | http://{APP_URL}/api/v1/accounts      | Create a new account                |
| PUT    | http://{APP_URL}/api/v1/accounts/{id} | Update the account with id {id}     |
| DELETE | http://{APP_URL}/api/v1/accounts/{id} | Delete the account with id {id}     |

When updating or creating an Account you need to send raw json data with the object fields

Creating a new account from the file account.json

     $ cat account.json
     {
        "name":"account1",
        "description":"My account",
        "disabled":0,
        "brand_id":1,
        "systemaccount":0
     }

     $ curl -X POST -H "Content-Type: application/json" \
       -H "X-API-TOKEN: 4f12988c-a861-4b70-bba8-85a1bf0b892d" \
       --data "@account.json" http://localhost:8080/api/v1/accounts
       
     {
       "data":
       {
         "id":6,
         "name":"account1",
         "description":"My account",
         "brand_id":1,
         "disabled":false
       },
       "message":
       {
         "code":"SUCCESSFULL",
         "message":"success",
         "http_code":200,
         "success":true
       }
     }
     
Updating the created account
    
     $ cat update_account.json
     {
        "name":"updated"
     }
     
     $ curl -X PUT -H "Content-Type: application/json" \
       -H "X-API-TOKEN: 4f12988c-a861-4b70-bba8-85a1bf0b892d" \
       --data "@update_account.json" http://localhost:8080/api/v1/accounts/6
       
     {
       "data":
       {
         "id":6,
         "name":"updated",
         "description":"My account",
         "brand_id":1,
         "disabled":false
       },
       "message":
       {
         "code":"SUCCESSFULL",
         "message":"success",
         "http_code":200,
         "success":true
       }
     }
     
### Brands

#### Model
     brand:
     [
       "name":"AbuseIO",
       "company_name":"AbuseIO"
     ]
     
#### Available methods

Note: create brand isn't yet implemented

| Method | Endpoint                                 | Description                         |
|--------|------------------------------------------|-------------------------------------|
| GET    | http://{APP_URL}/api/v1/brands      | List all brands                   |
| GET    | http://{APP_URL}/api/v1/brands/{id} | Show the brand with id {id}       |
| PUT    | http://{APP_URL}/api/v1/brands/{id} | Update the brand with id {id}     |
| DELETE | http://{APP_URL}/api/v1/brands/{id} | Delete the brand with id {id}     |

Updating a brand:
      
     $ cat update_brand.json
     {
        "name": "test",
        "company_name": "test test"
     }
     
     $ curl -X PUT -H "Content-Type: application/json" \
       -H "X-API-TOKEN: 4f12988c-a861-4b70-bba8-85a1bf0b892d" \
       --data "@test.json" http://localhost:8080/api/v1/brands/1
       
     {
       "data":
       {
         "id":1,
         "name":"test",
         "company_name":"testtest",
         "created_at":"2017-11-22 13:34:31"
       },
       "message":
       {
         "code":"SUCCESSFULL",
         "message":"success",
         "http_code":200,
         "success":true
       }
     }
     
### Contacts

#### Model
     contact:
     [
        "id" => 1,
        "account_id" => 1,
        "reference" => "reference_5a157ceecf16b",
        "name" => "Eunice Marks Sr.",
        "email" => "chandler22@example.com",
        "api_host" => "http://www.zboncak.info/rerum-aperiam-et-libero-possimus-assumenda-similique",
        "enabled" => 0,
        "token" => null,
     ] 
    
#### Available methods

| Method | Endpoint                                 | Description                         |
|--------|------------------------------------------|-------------------------------------|
| GET    | http://{APP_URL}/api/v1/contacts      | List all contacts                   |
| GET    | http://{APP_URL}/api/v1/contacts/{id} | Show the contact with id {id}       |
| POST   | http://{APP_URL}/api/v1/contacts      | Create a new contact                |
| PUT    | http://{APP_URL}/api/v1/contacts/{id} | Update the contact with id {id}     |
| DELETE | http://{APP_URL}/api/v1/contacts/{id} | Delete the contact with id {id}     |

 
Creating a new contact:

    $ cat contact.json
    {
      "reference":"reference_5a15ec16b5627",
      "name":"Eunice Marks Jr.",
      "email":"chandler82@example.com",
      "api_host":"http:\/\/www.zboncak.info\/rerum-aperiam-et-libero-possimus-assumenda-similique",
      "auto_notify":false,
      "enabled":true,
      "account_id":1
    }
    
    $ curl -X POST -H "Content-Type: application/json" \
      -H "X-API-TOKEN: 0043dbd9-fd59-4c16-94d8-3fb4ff4f8bfc" \ 
      --data "@contact.json" http://localhost:8080/api/v1/contacts
      
    {
      "data":
      {
        "id":64,
        "reference":"reference_5a15ec16b5627",
        "name":"Eunice Marks Jr.",
        "email":"chandler82@example.com",
        "api_host":"http:\/\/www.zboncak.info\/rerum-aperiam-et-libero-possimus-assumenda-similique",
        "auto_notify":false,
        "enabled":true,
        "account_id":1
      },
      "message":
      {
        "code":"SUCCESSFULL",
        "message":"success",
        "http_code":200,
        "success":true
      }
    }

Updating a contact

    $ cat contact_update.json
    {
      "reference":"askjdakjdajhdasjd",
      "name":"James Marks Jr.",
      "email":"joey90@example.com"
    }
    
    $ curl -X PUT -H "Content-Type: application/json" \
      -H "X-API-TOKEN: 0043dbd9-fd59-4c16-94d8-3fb4ff4f8bfc" \
      --data "@contact_update.json" http://localhost:8080/api/v1/contacts/3
      
    {
      "data":
      {
        "id":3,
        "reference":"askjdakjdajhdasjd",
        "name":"James Marks Jr.",
        "email":"joey90@example.com",
        "api_host":"http:\/\/koepp.com\/quibusdam-ea-aut-qui-magnam.html",
        "auto_notify":false,
        "enabled":false,
        "account_id":1
      },
      "message":
      {
        "code":"SUCCESSFULL",
        "message":"success",
        "http_code":200,
        "success":true
      }
    }


### Domains

#### Model
     domain:
     [
       id: 1,
       name: "john-doe.tld",
       contact_id: 1,
     ] 
    
#### Available methods

| Method | Endpoint                                 | Description                         |
|--------|------------------------------------------|-------------------------------------|
| GET    | http://{APP_URL}/api/v1/domains      | List all domains                   |
| GET    | http://{APP_URL}/api/v1/domains/{id} | Show the domain with id {id}       |
| POST   | http://{APP_URL}/api/v1/domains      | Create a new domain                |
| PUT    | http://{APP_URL}/api/v1/domains/{id} | Update the domain with id {id}     |
| DELETE | http://{APP_URL}/api/v1/domains/{id} | Delete the domain with id {id}     |

 
Creating a new domain:

    $ cat domain.json
    {
      "name":"example.com",
      "contact_id":1
    }
    
    $ curl -X POST -H "Content-Type: application/json" \
      -H "X-API-TOKEN: 0043dbd9-fd59-4c16-94d8-3fb4ff4f8bfc" \ 
      --data "@domain.json" http://localhost:8080/api/v1/domains
     
    {
      "data":
      {
        "id":4,
        "name":"example.com",
        "contact":
        {
          "id":1,
          "reference":"reference_5a27f66ce27f5",
          "name":"Arnold Schmeler V",
          "email":"sigurd.barrows@example.com",
          "api_host":"http:\/\/hermann.net\/minus-deleniti-facilis-impedit-sit-cum-voluptas-esse-rem",
          "auto_notify":false,
          "enabled":false,
          "account_id":1
        },
        "enabled":false
      },
      "message":
      {
        "code":"SUCCESSFULL",
        "message":"success",
        "http_code":200,
        "success":true
      }
    }  
    
Updating a contact

    $ cat domain_update.json
    {
      "contact_id":2
    }
    
    $ curl -X PUT -H "Content-Type: application/json" \
      -H "X-API-TOKEN: 0043dbd9-fd59-4c16-94d8-3fb4ff4f8bfc" \
      --data "@domain_update.json" http://localhost:8080/api/v1/domains/4
      
    {
      "data":
      {
        "id":4,
        "name":"example.com",
        "contact":
        {
          "id":2,
          "reference":"reference_5a27f66ce3264",
          "name":"Miss Adelia Conroy",
          "email":"blanda.adan@example.net",
          "api_host":"http:\/\/www.kiehn.com\/mollitia-repellendus-porro-nesciunt-nam-qui-tempore-maxime.html",
          "auto_notify":false,
          "enabled":true,
          "account_id":1
        },
        "enabled":false
      },
      "message":
      {
        "code":"SUCCESSFULL",
        "message":"success",
        "http_code":200,
        "success":true
      }
    }
        
Note: As you can see in both outputs, the returned domain json also includes the contact it belongs to.

### Incidents

Incidents are used as a internal model and aren't saved to the database. They are converted to tickets and/or events.
Because the models aren't saved to the database, you can only create an incident.
This model is used by AbuseIO to delegate events to other child instances.

#### Model

    [
      "source" => "google",
      "source_id" => 1,
      "ip" => "127.0.0.1",
      "domain" => null,
      "timestamp" => 0,
      "class" => "OPEN_NTP_SERVER",
      "type" => "ABUSE",
      "information" => "{}",
      "remote_api_token" => null,
      "remote_api_url" => null,
      "remote_ticket_id" => null,
      "remote_ash_link" => null
    ]

#### Available methods

| Method | Endpoint                                 | Description                         |
|--------|------------------------------------------|-------------------------------------|
| POST   | http://{APP_URL}/api/v1/incidents      | Create a new incident                |

Creating a new incident:

    $ cat incident.json
    {
      "source":"google",
      "source_id":1,
      "ip":"127.0.0.1",
      "domain":null,
      "timestamp":0,
      "class":"OPEN_NTP_SERVER",
      "type":"ABUSE",
      "information":"{}",
      "remote_api_token":null,
      "remote_api_url":null,
      "remote_ticket_id":null,
      "remote_ash_link":null
    }
   
    $ curl -X POST -H "Content-Type: application/json" \
      -H "X-API-TOKEN: 0043dbd9-fd59-4c16-94d8-3fb4ff4f8bfc" \ 
      --data "@incident.json" http://localhost:8080/api/v1/incidents
      
    {
      "data":
      {
        "source":"google",
        "source_id":true,
        "ip":"127.0.0.1",
        "domain":"",
        "timestamp":0,
        "class":"OPEN_NTP_SERVER",
        "type":"ABUSE",
        "information":"{}",
        "remote_api_url":"",
        "remote_api_token":"",
        "remote_ticket_id":"",
        "remote_ash_link":""
      },
      "message":
      {
        "code":"SUCCESSFULL",
        "message":"success",
        "http_code":200,
        "success":true
      }
    }
       
### Netblocks

#### Model
    
    netblock:
    [
      "first_ip" => "172.16.10.13",
      "last_ip" => "172.16.10.13",
      "description" => "Dedicated IP address for John's server",
      "contact_id" => 1,
      "enabled" => true
    ]

#### Available methods

| Method | Endpoint                                 | Description                         |
|--------|------------------------------------------|-------------------------------------|
| GET    | http://{APP_URL}/api/v1/netblocks      | List all netblocks                   |
| GET    | http://{APP_URL}/api/v1/netblocks/{id} | Show the netblock with id {id}       |
| POST   | http://{APP_URL}/api/v1/netblocks      | Create a new netblock                |
| PUT    | http://{APP_URL}/api/v1/netblocks/{id} | Update the netblock with id {id}     |
| DELETE | http://{APP_URL}/api/v1/netblocks/{id} | Delete the netblock with id {id}     |

Creating a new netblock:

    $ cat netblock.json
    {
      "first_ip":"172.16.13.13",
      "last_ip":"172.16.13.13",
      "description":"Dedicated IP address for John's server",
      "contact_id":1,
      "enabled":true
    }

    $ curl -X POST -H "Content-Type: application/json" \
      -H "X-API-TOKEN: 0043dbd9-fd59-4c16-94d8-3fb4ff4f8bfc" \ 
      --data "@netblock.json" http://localhost:8080/api/v1/netblocks

    {
      "data":
      {
        "first_ip":"172.16.13.13",
        "last_ip":"172.16.13.13",
        "description":"Dedicated IP address for John's server",
        "contact":
        {
          "id":1,
          "reference":"reference_5a27f66ce27f5",
          "name":"Arnold Schmeler V",
          "email":"sigurd.barrows@example.com",
          "api_host":"http:\/\/hermann.net\/minus-deleniti-facilis-impedit-sit-cum-voluptas-esse-rem",
          "auto_notify":false,
          "enabled":false,
          "account_id":1
        },
        "enabled":true
      },
      "message":
      {
        "code":"SUCCESSFULL",
        "message":"success",
        "http_code":200,
        "success":true
      }
    }
   
Updating a netblock:
 
    $ cat netblock_update.json
    {
      "contact_id":2
    }

    $ curl -X PUT -H "Content-Type: application/json" \
      -H "X-API-TOKEN: 0043dbd9-fd59-4c16-94d8-3fb4ff4f8bfc" \ 
      --data "@netblock_update.json" http://localhost:8080/api/v1/netblocks/1
      
    {
      "data":
      {
        "first_ip":"172.16.10.13",
        "last_ip":"172.16.10.13",
        "description":"Dedicated IP address for John's server",
        "contact":
        {
          "id":2,
          "reference":"reference_5a27f66ce3264",
          "name":"Miss Adelia Conroy",
          "email":"blanda.adan@example.net",
          "api_host":"http:\/\/www.kiehn.com\/mollitia-repellendus-porro-nesciunt-nam-qui-tempore-maxime.html",
          "auto_notify":false,
          "enabled":true,
          "account_id":1
        },
        "enabled":true
      },
      "message":
      {
        "code":"SUCCESSFULL",
        "message":"success",
        "http_code":200,
        "success":true
      }
    }
    
Note: Just like Domains, Netblocks also returns the contact data instead of just an id.
    
### Notes
Currently you can only retrieve notes.

#### Model
    
    note:
    [
      "ticket_id" => 1,
      "submitter" => "lharris",
      "text" => "Iusto consequatur alias cumque explicabo.",
      "hidden" => true,
      "viewed" => true
    ]
     
#### Available methods

| Method | Endpoint                                 | Description                         |
|--------|------------------------------------------|-------------------------------------|
| GET    | http://{APP_URL}/api/v1/notes      | List all notes                   |
| GET    | http://{APP_URL}/api/v1/notes/{id} | Show the note with id {id}       |


### Tickets

The Ticket model is use by AbuseIO for syncing between instances, therefor it has next to the normal CRUD some extra API methods.
Tickets are the main models in AbuseIO, so it is very possible that an AbuseIO instances has over 100,000 tickets, to retrieve these tickets you
can query the to list all tickets, but that will take a while. So the idea is to query for a specific subset (eg. all tickets of a specific 
class, type, contact or a combination) by using the search method.

#### Model

    ticket:
    [
      "id" => 1,
      "ip" => "127.0.0.1",
      "domain" => "borer.net",
      "class_id" => "NOTICE_AND_TAKEDOWN_REQUEST",
      "type_id" => "ABUSE",
      "ip_contact_account_id" => 1,
      "ip_contact_reference" => "reference_5a27f66ce3264",
      "ip_contact_name" => "Miss Adelia Conroy",
      "ip_contact_email" => "blanda.adan@example.net",
      "ip_contact_api_host" => "http => \/\/www.kiehn.com\/mollitia",
      "ip_contact_auto_notify" => false,
      "ip_contact_notified_count" => 0,
      "domain_contact_account_id" => 1,
      "domain_contact_reference" => "reference_5a27f66ce27f5",
      "domain_contact_name" => "Arnold Schmeler V",
      "domain_contact_email" => "sigurd.barrows@example.com",
      "domain_contact_api_host" => "http => \/\/hermann.net\/minus",
      "domain_contact_auto_notify" => false,
      "domain_contact_notified_count" => 0,
      "status_id" => "OPEN",
      "contact_status_id" => "OPEN",
      "last_notify_count" => 1,
      "last_notify_timestamp" => 1225765055,
      "event_count" => 1,
      "note_count" => 23,
      "ash_token_ip" => "be94efece2133d5384f9565e9dba5b53",
      "ash_token_domain" => "cad1b0024d8d4bcd90a1f1e97819750f",
      "remote_api_token" => "",
      "api_token" => "0cd11e7e-c406-481c-8a73-724bc3da1a39",
      "remote_api_url" => "",
      "remote_ticket_id" => "",
      "remote_ash_link" => ""
    ]

#### Available methods

| Method | Endpoint                                 | Description                         |
|--------|------------------------------------------|-------------------------------------|
| GET    | http://{APP_URL}/api/v1/tickets      | List all tickets                   |
| GET    | http://{APP_URL}/api/v1/tickets/{id} | Show the ticket with id {id}       |
| POST   | http://{APP_URL}/api/v1/tickets      | Create a new ticket                |
| PUT    | http://{APP_URL}/api/v1/tickets/{id} | Update the ticket with id {id}     |
| DELETE | http://{APP_URL}/api/v1/tickets/{id} | Delete the ticket with id {id}     |
| POST   | http://{APP_URL}/api/v1/tickets/search| Search for tickets |
| GET    | http://{APP_URL}/api/v1/tickets/{id}/notify | Send out notifications to the contacts of the ticket with id {id} |
| POST   | http://{APP_URL}/api/v1/tickets/syncstatus| Sync the status with the matching ticket |
| POST   | http://{APP_URL}/api/v1/tickets/synccontactstatus| Sync the contact status with the matching ticket |

Note: you can create a new ticket using the create method, but this will make a ticket without events, it is preferred that tickets are created using incidents.

As you can see we have some extra methods, next to the normal CRUD methods. Syncstatus and synccontactstatus are used by delegation to sync the specific statuses and
are out of scope for this document. In short only use it when you know what to do :)

#### Searching tickets

You can search for tickets in the AbuseIO API using the search method, if you POST a JSON with your search criteria it will only return the mathching tickets.

##### Search criteria

You can specify multiple criteria, note that all criteria will be implemented with AND clauses. 
The search criteria must have a column and a value field, an operator field is optional, when not specified is will use EQUAL as operator
The column names are the properties from the Ticket model as described earlier.

| Supported operators | Description |
|---------------------|-------------|
| '<' | Less than |
| '>' | More than |
| '=' | Equal |
| '!=' | Not equals |
| '<=' | Less or equal |
| '>=' | equal of more |

##### Sorting

You can sort, offset and limit the returned collection of tickets by using the specifying the following fields in the JSON

| Field   | Possible Values |Description |
|---------|-----------------|-------------|
| orderby | property name | Order by the property|
| desc    | boolean | Sort direction (descending or ascending (default))|
| offset  | N | Offset the result by N Tickets |
| limit   | N | Return max N Tickets | 

    $ cat search.json
    {
      "criteria":
      [
        {
          "column":"type_id",
          "value":"ABUSE"
        },
        {
          "column":"status_id",
          "value":"OPEN"
        },
        {
          "column":"last_notify_timestamp",
          "operator":">",
          "value":1416486944
        }
      ],
      "orderby":"event_count",
      "desc":true,
      "offset":1,
      "limit":1
    }
   
    $ curl -X POST -H "Content-Type: application/json" \
    -H "X-API-TOKEN: 9d48e4cb-2936-4156-a5a2-c794e0c06b47" \
    --data "@search.json" http://localhost:8080/api/v1/tickets/search
    
    {
      "data":
      [  
        {
          "id":4,
          "ip":"224.93.220.162",
          "domain":"kreiger.com",
          "class_id":"OPEN_SMB_SERVER",
          "type_id":"ABUSE",
          "ip_contact_account_id":1,
          "ip_contact_reference":"reference_5a30dddbcf774",
          "ip_contact_name":"Carissa O'Keefe",
          "ip_contact_email":"della.russel@example.org",
          "ip_contact_api_host":"http:\/\/jacobi.com\/id",
          "ip_contact_auto_notify":false,
          "ip_contact_notified_count":0,
          "domain_contact_account_id":1,
          "domain_contact_reference":"reference_5a30dddbd00cf",
          "domain_contact_name":"Dr. Ricardo Spencer",
          "domain_contact_email":"legros.barton@example.org",
          "domain_contact_api_host":"http:\/\/www.shanahan.com\/",
          "domain_contact_auto_notify":false,
          "domain_contact_notified_count":0,
          "status_id":"OPEN",
          "contact_status_id":"OPEN",
          "last_notify_count":1,
          "last_notify_timestamp":1426517356,
          "event_count":1,
          "note_count":1,
          "ash_token_ip":"af4d9810e012f420791317eae004d289",
          "ash_token_domain":"9fb651ab92952431df48919fc2a16295",
          "remote_api_token":"",
          "api_token":"efee328b-97ef-4a6c-8819-c2362e386ead",
          "remote_api_url":"",
          "remote_ticket_id":"",
          "remote_ash_link":"",
          "events":
          [
            {
              "id":42,
              "ticket_id":4,
              "evidence_id":42,
              "source":"Meghan Schamberger",
              "timestamp":1513152014,
              "information":"{}",
              "evidence":
              {
                "id":42,
                "filename":"mailarchive\/20171213\/5a30de0e07f5f_messageid",
                "sender":"Alyson Mills",
                "subject":"Cum quae excepturi non aut doloribus debitis."
              }
            }
          ],
          "notes":
          [
            {
              "id":43,
              "ticket_id":4,
              "submitter":"braulio.abshire",
              "text":"Molestias tenetur alias id alias dolores recusandae aliquid.",
              "hidden":true,
              "viewed":false
            }
          ]
        }
      ],
      "message":
      {
        "code":"SUCCESSFULL",
        "message":"success",
        "http_code":200,
        "success":true
      }
    } 
   
In the above example we query for the second ticket (offset 1 and limit 1) with the status 'OPEN' of type 'ABUSE', that hasn't been notified after timestamp 1426517356.
Notice that the returned ticket also returns its events and notes. 

#### Notifying the contacts of a ticket

With the use of the API you can notify all the contacts of the ticket.

    $ curl -X GET -H "Content-Type: application/json" \
    -H "X-API-TOKEN: 9d48e4cb-2936-4156-a5a2-c794e0c06b47" \
    http://localhost:8080/api/v1/tickets/1/notify
    
    {
      "data":
      {
        "id":1,
        "ip":"224.93.220.162",
        "domain":"kreiger.com",
        "class_id":"OPEN_SMB_SERVER",
        
        ...
        
        "remote_ticket_id":"",
        "remote_ash_link":"",
        "events": [...],
        "notes": [...]
       } 
    }
    "message":
    {
      "code":"SUCCESSFULL",
      "message":"success",
      "http_code":200,
      "success":true
    }
    
### Users

Currently you can only list and show an user

#### Model

    user:
    [
      id":1,
      "first_name":"System",
      "last_name":"Admin",
      "full_name":"System Admin",
      "email":"admin@isp.local",
      "account_id":1,
      "locale":"en",
      "disabled":false
    ]

#### Available methods

| Method | Endpoint                                 | Description                         |
|--------|------------------------------------------|-------------------------------------|
| GET    | http://{APP_URL}/api/v1/users      | List all users                   |
| GET    | http://{APP_URL}/api/v1/users/{id} | Show the user with id {id}       |

