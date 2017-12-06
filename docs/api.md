# API Documentation

**Documentation currently not yet finished !!**

AbuseIO 4.1+ implements a [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer) CRUD API for most of its models.

## How to use

### Config settings
    main.api.enabled (main.php)
The API is enabled by default, setting this option to false will disable the API.

    APP_URL (.env) 
    app.url (app.php)
Specifies the base url on which the application is running, this url will also be used by the API.

### Endpoint

All the API methods are reachable on the base endpoint

    http(s)://{APP_URL}/api/v1/{MODEL}

### Making a request

When making an API request you to send an API token, each account has it's own token, so you can use the api as a specific account.
**Important**: currently this is only implemented in the ticket model, all other models need to use the system accounts token

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
| GET    | http(s)://{APP_URL}/api/v1/accounts      | List all accounts                   |
| GET    | http(s)://{APP_URL}/api/v1/accounts/{id} | Show the account with id {id}       |
| POST   | http(s)://{APP_URL}/api/v1/accounts      | Create a new account                |
| PUT    | http(s)://{APP_URL}/api/v1/accounts/{id} | Update the account with id {id}     |
| DELETE | http(s)://{APP_URL}/api/v1/accounts/{id} | Delete the account with id {id}     |

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

     $ curl -X POST -H "Content-Type: application/json" -H "X-API-TOKEN: 4f12988c-a861-4b70-bba8-85a1bf0b892d" --data "@account.json" http://localhost:8080/api/v1/accounts
     {"data":{"id":6,"name":"account1","description":"My account","brand_id":1,"disabled":false},"message":{"code":"SUCCESSFULL","message":"success","http_code":200,"success":true}}
     
Updating the created account
    
     $ cat update_account.json
     {
        "name":"updated"
     }
     
     $ curl -X PUT -H "Content-Type: application/json" -H "X-API-TOKEN: 4f12988c-a861-4b70-bba8-85a1bf0b892d" --data "@update_account.json" http://localhost:8080/api/v1/accounts/6
     {"data":{"id":6,"name":"updated","description":"My account","brand_id":1,"disabled":false},"message":{"code":"SUCCESSFULL","message":"success","http_code":200,"success":true}}
     
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
| GET    | http(s)://{APP_URL}/api/v1/brands      | List all brands                   |
| GET    | http(s)://{APP_URL}/api/v1/brands/{id} | Show the brand with id {id}       |
| PUT    | http(s)://{APP_URL}/api/v1/brands/{id} | Update the brand with id {id}     |
| DELETE | http(s)://{APP_URL}/api/v1/brands/{id} | Delete the brand with id {id}     |

Updating a brand:
      
     $ cat update_brand.json
     {
        "name": "test",
        "company_name": "test test"
     }
     
     $ curl -X PUT -H "Content-Type: application/json" -H "X-API-TOKEN: 4f12988c-a861-4b70-bba8-85a1bf0b892d" --data "@test.json" http://localhost:8080/api/v1/brands/1
     {"data":{"id":1,"name":"test","company_name":"testtest","created_at":"2017-11-22 13:34:31"},"message":{"code":"SUCCESSFULL","message":"success","http_code":200,"success":true}}
     
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
| GET    | http(s)://{APP_URL}/api/v1/contacts      | List all contacts                   |
| GET    | http(s)://{APP_URL}/api/v1/contacts/{id} | Show the contact with id {id}       |
| POST   | http(s)://{APP_URL}/api/v1/contacts      | Create a new contact                |
| PUT    | http(s)://{APP_URL}/api/v1/contacts/{id} | Update the contact with id {id}     |
| DELETE | http(s)://{APP_URL}/api/v1/contacts/{id} | Delete the contact with id {id}     |

 
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
    
    $ curl -X POST -H "Content-Type: application/json" -H "X-API-TOKEN: 0043dbd9-fd59-4c16-94d8-3fb4ff4f8bfc  --data "@contact.json" http://localhost:8080/api/v1/contacts
    {"data":{"id":64,"reference":"reference_5a15ec16b5627","name":"Eunice Marks Jr.","email":"chandler82@example.com","api_host":"http:\/\/www.zboncak.info\/rerum-aperiam-et-libero-possimus-assumenda-similique","auto_notify":false,"enabled":true,"account_id":1},"message":{"code":"SUCCESSFULL","message":"success","http_code":200,"success":true}}

Updating a contact

    $ cat contact_update.json
    {
      "reference":"askjdakjdajhdasjd",
      "name":"James Marks Jr.",
      "email":"joey90@example.com"
    }
    
    curl -X PUT -H "Content-Type: application/json" -H "X-API-TOKEN: 0043dbd9-fd59-4c16-94d8-3fb4ff4f8bfc"  --data "@contact_update.json" http://localhost:8080/api/v1/contacts/3
    {"data":{"id":3,"reference":"askjdakjdajhdasjd","name":"James Marks Jr.","email":"joey90@example.com","api_host":"http:\/\/koepp.com\/quibusdam-ea-aut-qui-magnam.html","auto_notify":false,"enabled":false,"account_id":1},"message":{"code":"SUCCESSFULL","message":"success","http_code":200,"success":true}}


### Domains
todo

### Incidents
todo

### Netblocks
todo

### Notes
todo

### Tickets
todo

### Users
todo