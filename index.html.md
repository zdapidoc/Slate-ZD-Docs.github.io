---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:

includes:
  - errors

search: true
---

# Introduction

This is the official public documentation for The Dispute Service Disputes API.

TDS exposes a REST based API that allows 3rd party companies to use their dispute management service by submitting 
dispute related data that will then be managed by the adjudication team. 

The adjudication results will then be passed over to the dispute requester via Webhook to the provided endpoint.

TDS can also send regular dispute status updates via Webhook if the requester is able to provide an endpoint for it.

# Environments

TDS will expose a staging environment, that suppliers can use to test their integration end-to-end and create test data, and the production
environment where the live version of the API is hosted and live data is flowing.

Enviromnent | Endpoint
------------| --------
Production | TBC
Staging | https://uat-tds.cs87.force.com/services/apexrest/v1/disputes 


# Authentication

In order to submit disputes via the API, clients will need to pass a token with every request. For every new client that
requests access, a new API account will be set up and TDS will provide a set of credentials.

These credentials will include:

`Account ID` - identifies the supplier account and it is the same across all environments

`client_id` - unique identifier for the application, should be different across environments

`client_secret` - key used to sign the token hash, should also different across environments

For every environment that TDS exposes the API(production or sandboxes), although the Account ID will be the same, a
different set of client credentials will be provided.

<aside class="notice">
Account ID and client credentials will be provided by TDS on request.
</aside>

## Token Header

Your token will be passed in the `Token` HTTP Header and should have the following format:

`<account_ID>-<timestamp>-<hmac>` 

`account_ID` - used to track the supplier account ID

`timestamp` - milliseconds since Unix Epoch time; any timestamp within a 5 minute window(between 
client request and the moment that the API processes the request) will be accepted, to cover for a potential
delay between systems

`hmac` - this will be a computed hash of a string containing the client credentials and the timestamp parameters

## Token generation

### Step 1 - generating the url parameter string

Use your client credentials and Unix time to generate a url-param string like below

`client_id=<your_client_id>&client_secret=<your_client_secret>&timestamp=<current_unix_time>`

<aside class="warning">
Remember, the timestamp should be sent in milliseconds.
</aside>

### Step 2 - hashing 

Use SHA1 algorithm to generate a HMAC of the url string using your `client_secret` as the private key.

### Step 3 - map it to Token header

Concatenate your account ID, timestamp and HMAC in a string like below

Finally, use the outputs generated above to form your token like below

`<account_ID>-<timestamp>-<hmac>`

And map it to your `Token` HTTP Header in your HTTP Request.

HTTP Header | Value
----------- | -----
Token | `<account_ID>-<timestamp>-<hmac>`

Field marked with < > should be replaced with the correct outputs.

## Example

Below is an example of token generation.

Parameters | Value
---------- | -----
account_id | supplierX
client_id | iy8oiyuy809piohSd
client_secret | 2e9ifb809!s23e723028ffLL
timestamp | 1576769124000

### Step 1

This will result in the string below:

`client_id=iy8oiyuy809piohSd&client_secret=2e9ifb809!s23e723028ffLL&timestamp=1576769124000`

### Step 2

The SHA1 digest using the client_secret as private key should be:

`b4072b847c5c3658e3f293267363c9c545fa034c`

### Step 3

HTTP Token header will then look like:

`Token`:`supplierX-1576769124000-b4072b847c5c3658e3f293267363c9c545fa034c`

<aside class="notice">
Providing a Token that has a wrong account_id, a timestamp that is not unix time or doesn't fall within a 5 minute 
window delay, or an invalid hmac will result in an unauthorized API call and therefore will be rejected.
</aside>

# Create Disputes

## Introduction

Dispute data submission should be done via JSON payload and will include dispute related context.
Parameter names and field types should comply to the schema provided for the endpoint.

A strict level of validation will be performed on the payload, to check value integrity and ensuring that mandatory 
information is being provided correctly and within the allowed value sets when those apply.

Any attempt to send duplicate dispute, dispute_line, tenancy or evidence objects will cause the payload to be rejected.

## Payload Object Description

The payload should be a JSON object contain the key parameters described below.

Key Name | Mandatory | Object Type | Description
-------- | --------- | ----------- | -----------
agent | No | Object | Agent related to the dispute
branch | No | Object | Agency branch of the agent related to dispute
tenant | Yes | Array of Objects | List of tenants related to dispute
landlord | Yes | Object | Landlord related to dispute
tenancy | Yes | Object | Related Tenancy information such as address and start dates
dispute | Yes | Object | Dispute information
dispute_line | Yes | Array of Objects | List of claim types and amounts for dispute
evidence | Yes | Array of Objects | List of evidence information such as file names and resource url

## Object Schemas

Below are the schemas for the objects described above.

### agent


Parameter | Field Type | Value Set | Mandatory | Description
--------- | ---------- | --------- | --------- | -----------
id | String(244) | - | Yes | unique identifier
branch_id | String(244) | - | Yes| agent's branch unique identifier
firstname | String(40) | - | Yes | first name
lastname | String(80) | - | Yes | surname
phone | String(40) | - | No | land line number
mobile | String(40) | - | No | mobile phone number
email | String(80) | - | Yes | email address
birth_date | Date(YYYY-MM-DD) | - | No | birth date
salutation | Enum(10) | Mr., Ms., Mrs., Dr., Prof., Mx. | No | salutation
job_title | String(128) | - | No | job title
created_date | Date/time(YYYY-MM-DDTHH:mm:ssZ) | - | Yes | resource creation date and time

### branch


Parameter | Field Type | Value Set | Mandatory | Description
--------- | ---------- | --------- | --------- | -----------
id | String(244) | - | Yes | unique identifier
name | String(255) | - | Yes | agencie's branch name
phone | String(40) | - | No | landline number
postcode | String(20) | - | No | address post code
street | String(255) | - | No | address street name
city | String(40) | - | No | city name
country | String(80) | - | No| country name
website | String(255) | - | No | website
created_date | Date/time(YYYY-MM-DDTHH:mm:ssZ) | - | Yes | resource creation date and time

### tenant


Parameter | Field Type | Value Set | Mandatory | Description
--------- | ---------- | --------- | --------- | -----------
id | String(244) | - | Yes | unique identifier
firstname | String(40) | - | Yes | first name
lastname | String(80) | - | Yes | surname
phone | String(40) | - | No | land line number
mobile | String(40) | - | No | mobile phone number
email | String(80) | - | Yes | email address
birth_date | Date(YYYY-MM-DD) | - | No | birth date
salutation | Enum(10) | Mr., Ms., Mrs., Dr., Prof., Mx. | No | salutation
job_title | String(128) | - | No | job title
created_date | Date/time(YYYY-MM-DDTHH:mm:ssZ) | - | Yes | resource creation date and time

### landlord


Parameter | Field Type | Value Set | Mandatory | Description
--------- | ---------- | --------- | --------- | -----------
id | String(244) | - | Yes | unique identifier
firstname | String(40) | - | Yes | first name
lastname | String(80) | - | Yes | surname
phone | String(40) | - | No | land line number
mobile | String(40) | - | No | mobile phone number
email | String(80) | - | Yes | email address
birth_date | Date(YYYY-MM-DD) | - | No | birth date
salutation | Enum(10) | Mr., Ms., Mrs., Dr., Prof., Mx. | No | salutation
job_title | String(128) | - | No | job title
postcode | String(20) | - | No | address post code
street | String(255) | - | No | address street name
city | String(40) | - | No | city name
country | String(80) | - | No| country name
created_date | Date/time(YYYY-MM-DDTHH:mm:ssZ) | - | Yes | resource creation date and time

### tenancy


Parameter | Field Type | Value Set | Mandatory | Description
---------- | ---------- | --------- | --------- | -----------
id | String(244) | - | Yes | unique identifier
postcode | String(20) | - | Yes | address post code
street | String(255) | - | Yes | address street name
city | String(40) | - | Yes | city name
country | String(80) | - | Yes | country name
start_date | Date(YYYY-MM-DD) | - | Yes | start date of tenancy as per tenancy agreement
end_date | Date(YYYY-MM-DD) | - | Yes | expected end date of tenancy as per the tenancy agreement
deposit_amount | Double(16,2) | - | Yes | total value of the deposit
rent_amount | Double(16,2) | - | No |  rent amount payable as per the tenancy agreement.
nr_of_tenants | Integer(3) | - | No| total number of tenants allocated to the tenancy
reference | String(255) | - | Yes | reference for tenancy, policy or deposit
customer_id | String(244) | - | Yes | unique id of Landlord or Branch that paid the deposit, police or fee
customer_type | Enum(255) | Landlord, Branch | Yes | specifies the customer type, Branch or Landlord
created_date | Date/time(YYYY-MM-DDTHH:mm:ssZ) | - | Yes | resource creation date and time

### dispute


Parameter | Field Type | Value Set | Mandatory | Description
--------- | ---------- | --------- | --------- | -----------
id | String(244) | - | Yes | unique identifier
tenancy_id | String(244) | - | Yes | unique id of the dispute's related tenancy
disputed_amount | Double(16,2) | - | Yes | the amount in dispute
deposit_owner_id | String(244) | - | Yes | the id of the deposit owner or beneficiary (landlord or branch)
agent_id | String(244) | - | No | unique id of the agent involved to the dispute
branch_id | String(244) | - | No | id of the agent's branch involved in the dispute 
initiated_by | Enum(255) | Agent, Landlord, Tenant | Yes | the party that raised the dispute
dispute_steps | String(32768) | - | No | steps taken to resolve dispute pior to raising it with TDS
dispute_reasons | String(32768) | - | No| summary of reasons for the dispute
lead_landlord_id | String(244) | - | Yes | id for landlord involved in the dispute
lead_tenant_id | String(244) | - | No | id for the main tenant involved in dispute
total_claim_amount | Double(16,2) | - | Yes | the total amount claimed from the deposit, including both agreed and disputed deductions
created_date | Date/time(YYYY-MM-DDTHH:mm:ssZ) | - | Yes | resource creation date and time

### dispute_line


Parameter | Field Type | Value Set | Mandatory | Description
--------- | ---------- | --------- | --------- | -----------
id | String(244) | - | Yes | unique identifier
dispute_id |String(244) | - | Yes | unique id of parent dispute
type | Enum(255) | Cleaning, Damage, Redecoration, Gardening, Rent, Arrears, Other | Yes | type of claim
tenant_amount_agreed | Double(16,2) | - | No | amount the tenant agrees to be paid to the landlord
claimed_landlord | Double(16,2) | - | Yes | amount claimed by Landlord
landlord_statement | String(32768) | - | No | landlord's justification for the claim
tenant_statement | String(32768) | - | No | tenant's justification for disputing the landlord's claim
created_date | Date/time(YYYY-MM-DDTHH:mm:ssZ) | - | Yes | resource creation date and time

### evidence


Parameter | Field Type | Value Set | Mandatory | Description
--------- | ---------- | --------- | --------- | -----------
id | String(244) | - | Yes | unique identifier
dispute_id | String(244) | - | Yes | unique id of parent dispute
dispute_item_id | String(244) | - |  No | unique id of related dispute line
filename | String(244) | - | Yes | name of the file that has been uploaded
file_size |Integer(10) | - | No | size of the file in bytes
file_type | Enum(10) | See below | Yes | format of the file
file_url | String(255) | - | Yes | link including resource path in storage server
source |Enum(255) | Dispute Application, Dispute Response, Admin | No | mechanism through which the evidence was uploaded
uploader_type | Enum(255) | Agent, Landlord, Tenant | Yes | type of user who uploaded the evidence during the dispute process
created_date | Date/time(YYYY-MM-DDTHH:mm:ssZ) | - | Yes | resource creation date and time

<aside class="notice">
File type can be pdf,doc,docx,txt,rtf,odt,xls,xlsx,ods,png,jpeg,jpg,gif,tiff,tif,bmp,emf,mpg,mpe,avi,mp3,mp4,wmv,wav,
csv,eml,msg
</aside>

## Example

```shell
curl -X POST 'https://uat-tds.cs87.force.com/services/apexrest/v1/disputes' 
-H 'Accept: application/json'
-H 'Content-Type: application/json' 
-H 'AccessToken: ZeroDeposit-1579175515000-7c474385a710551b5d2aaeca2268d6ea7e7721fc' 
-d '{
        "landlord": {
            "id": "landlordId20",
            "email": "test1@test.com",
            "lastname": "Landlord",
            "firstname": "Jack",
            "phone": "07401655677",
            "mobile": "06788987678",
            "birth_date": "1990-01-13",
            "salutation": "Mr.",
            "job_title": "CEO",
            "postcode": "EC3M5DJ",
            "street": "fenchurch",
            "city": "london",
            "country": "UK",
            "created_date": "2019-12-23T16:00:00Z"
        },
        "tenancy": {
            "end_date": "2019-12-23",
            "start_date": "2019-12-23",
            "deposit_amount": 2500.00,
            "country": "UK",
            "city": "London",
            "street": "Faringford Road",
            "postcode": "E15 4DG",
            "customer_id": "landlordId20",
            "reference": "1111333",
            "id": "tenancy68",
            "rent_amount": 10000.15,
            "nr_of_tenants": 3,
            "created_date": "2019-12-23T16:00:00Z"
        },
        "evidence": [
            {
                "file_url": "https://test/evidence1.txt",
                "uploader_type": "Landlord",
                "filename": "evidence1",
                "dispute_id": "dispute68",
                "dispute_item_id": "l26",
                "file_size": "500",
                "source": "Admin",
                "file_type": "txt",
                "id": "ev28",
                "created_date": "2019-12-23T16:00:00Z"
            },
            {
                "file_url": "https://test/evidence2.txt",
                "uploader_type": "Landlord",
                "filename": "evidence2",
                "dispute_id": "dispute68",
                "dispute_item_id": "l26",
                "file_size": "500",
                "source": "Admin",
                "file_type": "txt",
                "id": "ev29",
                "created_date": "2019-12-23T16:00:00Z"
            },
            {
                "file_url": "https://test/evidence3.txt",
                "uploader_type": "Landlord",
                "filename": "evidence3",
                "dispute_id": "dispute68",
                "dispute_item_id": "l26",
                "file_size": "500",
                "source": "Admin",
                "file_type": "txt",
                "id": "ev30",
                "created_date": "2019-12-23T16:00:00Z"
            }
        ],
        "tenant": [
            {
                "email": "test8@test8.com",
                "lastname": "Tenant",
                "firstname": "Lee",
                "id": "t8",
                "phone": "07899855677",
                "mobile": "07988766544",
                "birth_date": "1999-08-23",
                "salutation": "Mr.",
                "job_title": "Consultant",
                "created_date": "2019-12-23T16:00:00Z"
            },
            {
                "email": "test9@test9.com",
                "lastname": "Tenant",
                "firstname": "James",
                "id": "t9",
                "phone": "07899855677",
                "mobile": "07988766544",
                "birth_date": "1999-08-23",
                "salutation": "Mr.",
                "job_title": "Admin",
                "created_date": "2019-12-23T16:00:00Z"
            },
            {
                "email": "test10@test10.com",
                "lastname": "Tenant",
                "firstname": "Darren",
                "id": "t10",
                "phone": "07899855677",
                "mobile": "07988766544",
                "birth_date": "1999-08-23",
                "salutation": "Mr.",
                "job_title": "Consultant",
                "created_date": "2019-12-23T16:00:00Z"
            }
        ],
        "dispute_line": [
            {
                "claimed_landlord": 400.00,
                "type": "Rent",
                "dispute_id": "dispute68",
                "tenant_amount_agreed": 500.55,
                "landlord_statement": "late rent",
                "tenant_statement": "no money left",
                "id": "l26",
                "created_date": "2019-12-23T16:00:00Z"
            },
            {
                "claimed_landlord": 600.00,
                "type": "Damage",
                "dispute_id": "dispute68",
                "tenant_amount_agreed": 100.55,
                "landlord_statement": "damaged counter",
                "tenant_statement": "little damage",
                "id": "l27",
                "created_date": "2019-12-23T16:00:00Z"
            },
            {
                "claimed_landlord": 1000.00,
                "type": "Gardening",
                "dispute_id": "dispute68",
                "tenant_amount_agreed": 50.55,
                "landlord_statement": "damaged garden",
                "tenant_statement": "didnt do it",
                "id": "l28",
                "created_date": "2019-12-23T16:00:00Z"
            }
        ],
        "dispute": {
            "initiated_by": "Landlord",
            "tenancy_id": "tenancy68",
            "disputed_amount": 2000.00,
            "lead_landlord_id": "landlordId20",
            "lead_tenant_id": "t9",
            "total_claim_amount": 2000.00,
            "deposit_owner_id": "branch1",
            "agent_id": "agent20",
            "branch_id": "branch1",
            "dispute_reasons": "tenant left house in a mess",
            "dispute_steps": "tried to agree amount with tenant but didnt work",
            "id": "dispute68",
            "created_date": "2019-12-23T16:00:00Z"
        },
        "agent": {
            "id": "agent20",
            "branch_id": "branch1",
            "firstname": "Joe",
            "lastname": "Bloggs",
            "email": "joe@bloggs.com",
            "phone": "06789765444",
            "mobile": "07501655678",
            "birth_date": "1999-09-24",
            "salutation": "Mr.",
            "job_title": "CFO",
            "created_date": "2019-12-23T16:00:00Z"
        },
        "branch": {
            "id": "branch1",
            "name": "Foxtons Central",
            "postcode": "EC3M5DJ",
            "street": "fenchurch",
            "city": "london",
            "country": "UK",
            "website": "www.website.com",
            "phone": "090888788677",
            "created_date": "2019-12-23T16:00:00Z"
        }
    }'
```

> The above command returns JSON structured like this:

```json
HTTP/1.1  201 Created
{
    "tds_dispute_id": "5008E00000Gu5NsQAJ",
    "success": true
}
``` 

An example of a dispute submission is shown through the shell script on the right side panel. Some definitions are 
explained below.

### HTTP Method
`POST`

### Endpoint

`https://uat-tds.cs87.force.com/services/apexrest/v1/disputes`

### HTTP Headers

Header | Description
------ | -----------
Accept | application/json
Content-Type | application/json
AccessToken | TestAccount-1578320284000-77667d19e566d4744ed0db55b734f63c5c7a5338

### Request Body

See shell script example

Notes:

`Date` fields should be sent in `YYYY-MM-DD` format.

`Date/time` fields should be sent in UTC format like `YYYY-MM-DDTHH:mm:ssZ`.

`Enum` type fields should have values inside the provided set or the payload will be rejected.

### Response

The response body will contain a JSON object with the parameters below.

Parameter | Description
--------- | -----------
success | Boolean indicating whether submission was successful or not
errors | Displayed on failure will include specific message flagging the error(s)
tds_dispute_id | Displayed on success with internal ID for the created dispute

## Error messages

In case the submitted payload is not successful, the returned parameter `errors` can contain the any of the messages 
described below.

Message | Description
------- | -----------
Content Type not supported | The Content-Type header value provided is not supported
Accept not supported | The Accept header value provided is not supported
No access token provided | AccessToken header not provided or blank value
Invalid access token format | AccessToken header value has wrong format
Invalid timestamp | Timestamp not provided or invalid
Timestamp is too old | Timestamp provided is older than 5 minutes
Invalid access token | Provided hmac can not be authenticated
Invalid payload | Payload provided is malformed 
Missing required Objects | Required payload objects have not been provided
Invalid objects | Payload contains objects that are not part of the schema
Invalid type for objects | Payload contains objects that have the wrong type
Missing required parameters | Flags mandatory fields that are missing in dot notation 'object.field'
List can not be empty | Payload contains an empty list for a mandatory object, there should always be at least 1 element
Invalid parameters for object | Payload contains invalid/non existent parameter(s) for the object specified
Invalid value for field | A value that is not within the allowed value set for that field has been provided
Invalid field type for key | A blank or null value has been provided for a mandatory key. Mandatory keys can not be blank
Duplicate {object} id found | A dispute, tenancy, dispute_line, or evidence has already been created with that id
Invalid foreign key for object | The object contains a foreign key reference to an object that is not on the payload



# Webhooks

## Introduction

TDS will communicate status updates and adjudication results via REST based webhooks, where suppliers will be able to 
subscribe and provide the callback endpoints. Subscriptions are done via request to TDS only, at this point.

Once TDS is provided with an endpoint for the callback for a specific webhook action, a subscription will be created and 
shared key will be provided to the supplier. There will always be a unique shared key per endpoint per subscription.

## Headers

TDS webhook requests will always have three specific HTTP headers that will help suppliers verify the source, content
type and delivery unique identifier.

Header | Type | Description
------ | ---------- | -----------
Content-Type | String | Content type of the request, always application/json
TDS-Delivery | String | Unique identifier for the webhook event
TDS-Signature | String | TDS signature using shared key

## Verifying the source

Suppliers will be able to verify the source of the webhook using `TDS-Signature` header.

This will contain the HMAC generated by a SHA1 hash of the payload, signed with the shared key provided by TDS for the
endpoint. Your generated hash should always match this signature for a valid authentication.

## Dispute Status update

The Dispute Status Update webhook allows suppliers to be notified whenever the status of a case changes.

The subscription is not mandatory. List of statuses are shown in the table below.

Status |
------ |
On Hold |
Evidence Gathering TT |
Awaiting Review |
Review Complete |
Adjudication |
Closed |

### Payload schema

```shell
curl -X POST 'https://disputesupplier.com/dispute_status_update' 
-H 'Content-Type: application/json' 
-H 'TDS-Delivery: 5008E00000GwTBjQAN1579786198991' 
-H 'TDS-signature: 5b96954597082eade74430f075c819d27aa66c1e'
-d '{
      "status" : "Review Complete",
      "id" : "dispute71",
      "updated_datetime" : "2020-01-23T13:29:56Z"
    }'
```
> Webhook should expected the following response

```json
HTTP/1.1  200 OK
```

Payload will be sent in JSON format and will contain the following schema.

Parameter | Type | Description
--------- | ---- | -----------
id | String(244) | The primary key for the dispute on the source system
status | String(244) | The new status of the dispute
updated_datetime | Date/time | Datetime of the update

Datetime will be provided in UTC time with format `YYYY-MM-DDTHH:mm:ssZ`.

Example request shown on the right side panel.


## Adjudication Results

The Adjudication results webhook will be triggered off of the report being published by the adjudication team 
and will contain information about the decision on the dispute and amounts to be paid out.

This subscription is mandatory.

### Payload schema

```shell
curl -X POST 'https://disputesupplier.com/adjudication_results' 
-H 'Content-Type: application/json' 
-H 'TDS-Delivery: a0C8E000006o4w5UAA1579785940763' 
-H 'TDS-signature: 4d9c90e059ba262baf07a02f2258680390cee645'
-d '{
      "awarded_to_tenant" : 449.44,
      "report_url" : "https://shareuatna11.springcm.com/Public/DownloadPdf/12341/1a4351cc-0032-ea11-b80a-48df378a7098/299e1a2e-0132-ea11-b80a-48df378a7098",
      "created_date" : "2020-01-20T17:05:29Z",
      "dispute_id" : "dispute68",
      "publish_date" : "2020-01-23T00:00:00Z",
      "awarded_to_agent" : 100.00,
      "awarded_to_landlord" : 450.56
    }'
```
> Webhook should expected the following response

```json
HTTP/1.1  200 OK
```

Payload will be sent in JSON format and will contain the following schema.

Parameter | Type | Description
--------- | ---- | -----------
id | String(244) | TDS report unique identifier
dispute_id | String(244) | Dispute identifier of the source system
publish_date | Date/time | Date/time of when decision was published
awarded_to_tenant | Double(16,2) | Amount awarded to tenants
awarded_to_landlord | Double(16,2) | Amount awarded to landlord
awarded_to_agent | Double(16,2) | Amount awarded to agent
report_url | String(255) | Public url to TDS adjudication report in PDF format. Valid for 1 day
created_date | Date/time | Date/time of report creation

Date/time will be provided in UTC time with format `YYYY-MM-DDTHH:mm:ssZ`.

Example request shown on the right side panel.

<aside class="warning">
The public report link sent is valid for one day only.
</aside>

## On-demand webhook triggering (Staging only)

TDS allows the supplier to trigger the webhooks themselves via API to help with testing your integration - this feature 
is only available in the staging environment.

Calling these endpoints will result in the corresponding webhook being triggered and will send the event to your
specified handler endpoints.

### Authentication

Authentication method is the same as the Disputes API, so you should be able to just reuse what you already have set up.

### Triggering a Dispute Status Update

Manually triggering a status update can be done using the specifications below.


```shell
curl -X POST 'https://uat-tds.cs87.force.com/services/apexrest/webhooktrigger/v1/dispute_status_update' 
-H 'Accept: application/json'
-H 'Content-Type: application/json' 
-H 'AccessToken: ZeroDeposit-1579175515000-7c474385a710551b5d2aaeca2268d6ea7e7721fc' 
-d '{
      "status" : "Review Complete",
      "tds_dispute_id" : "5008E00000HKEVe"
    }'
```
> Response should be presented as below

```json
HTTP/1.1  200 OK
{
    "success": true,
    "errors": null
}
```

 Environment | Endpoint
 ----------- | --------
 Staging | https://uat-tds.cs87.force.com/services/apexrest/webhooktrigger/v1/dispute_status_update
 
 
 ### Request Headers
 
 Header | Description
 ------ | -----------
 Accept | application/json
 Content-Type | application/json
 AccessToken | <token>
 
`HTTP Method` - `POST`

 ### Request Parameters
 
 Parameter | Type | Description
 --------- | ---- | -------------
 tds_dispute_id | String | TDS's id of the dispute returned by API after creation
 status | Enum | New status of the case to update to. Values are: On Hold Evidence Gathering TT, Awaiting Review, Review Complete, Adjudication, Closed

 ### Response Body 
 
Parameter | Description
--------- | -----------
success | Boolean indicating if update was successful or not
errors | Displayed on failure will include specific message flagging the error(s)


See example request in the panel on the right.  

### Triggering an Adjudication Report publishing

Manually triggering an Adjudication Report publishing can be done using the specifications below.

This will create a dummy adjudication report with the results and change the status to 'Published', that will trigger
the webhook.

<aside class="warning">
The report results triggered this way will NOT include a report link, as this is a manual step that integrates with
the document generation engine.
</aside>

```shell
curl -X POST 'https://uat-tds.cs87.force.com/services/apexrest/webhooktrigger/v1/publish_report' 
-H 'Accept: application/json'
-H 'Content-Type: application/json' 
-H 'AccessToken: ZeroDeposit-1579175515000-7c474385a710551b5d2aaeca2268d6ea7e7721fc' 
-d '{
     "tds_dispute_id" : "5008E00000HKEVe"
    }'
```
> Response should be presented as below

```json
HTTP/1.1  200 OK
{
    "success": true,
    "errors": null
}
```

 Environment | Endpoint
 ----------- | --------
 Staging | https://uat-tds.cs87.force.com/services/apexrest/webhooktrigger/v1/publish_report
 
 
 ### Request Headers
 
 Header | Description
 ------ | -----------
 Accept | application/json
 Content-Type | application/json
 AccessToken | <token>
 
 
`HTTP Method` - `POST`

 ### Request Parameters
 
 Parameter | Type | Description
 --------- | ---- | -------------
 tds_dispute_id | String | TDS's id of the dispute returned by API after creation 
 

 ### Response Body 
 
Parameter | Description
--------- | -----------
success | Boolean indicating if update was successful or not
errors | Displayed on failure will include specific message flagging the error(s)



# Contacts

For any issue or query related to any of the functionality documented, please email the admin tiago.lopes@weare4c.com. 


