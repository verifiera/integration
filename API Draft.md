
# API Draft
Version: 0.4

## Changes

**Changes in 0.4**
Added status "approval_request_rejected"  
Added "testCallback" in Init request
 
## Errors handling

HTTP code are used to indicate errors

We have following status codes:  
500 - no json, fatal error  
403 - Forbidden - you do not have access to this endpoint 401 - Unauthorized - wrong credentials   
400 - Bad Request - Validation error   
Bad request codes:   
	0 - wrong personal_id format      
	1 - incorrect json   
	2 - requested item is not approved by recipient (/api/integration/commit)  
500 - non fatal error, wrong use of api, for example API request parameter not exists   

## Background check process description.
1. Process start (init)
2. Candidate approves or denies request (candidate-response [this request if only for dev purposes. no intended to be used in prod]) 
3. If candidate approved then you need to tell our system to register background check purchase. In our system customer do it implicitly when open report. But we think for your case it should be done automatically, for example during callback. If callback won't work probably it would be nice to have a button in such case so user could purchase report implicitly. (commit)
4. If person is "green", then everything is fine. But if it is not approved by our system, it goes to security officer. (security-response [only for dev] ) 
5. When security officer gives response result is ready to be shown

Anytime you can get current status (status).

 There are following statuses: 
 - waiting_recipient_approval
 - approval_request_rejected
 - ready_to_commit
 - waiting_security_approval
 - candidate_not_approved
 - candidate_approved
 - unknown
 - test  (in future we could check callback address during init call, so this status will come to callback)

## Status flows

- waiting_recipient_approval > approval_request_rejected  
- waiting_recipient_approval > ready_to_commit > candidate_approved  
- waiting_recipient_approval > ready_to_commit > waiting_security_approval > candidate_approved  
- waiting_recipient_approval > ready_to_commit > waiting_security_approval > candidate_not_approved

## Init method
Initializes a background check. If success - server responds with 200 HTTP Code.
requestId is optional, if you skip it server will generate it for you,
additionalData currently not saved. Will fix later if needed.


**POST /api/integration/init**

PAYLOAD (json)

```javascript
{  
"requestId": "any_unique_id",  
"customerId": "CUSTOMER_id", 
"customerTitle": "COMPANY NAME",
"referenceCustomer": "Recruteringsföretag AB",
"type": "legal_analysis",
"personalNr": "XXXXXXXX-XXXX", 
"candidatePhone": "",  
"candidateEmail": "",  
"callbackUrl": "CALLBACK URL", 
"additionalData": { 
       "key": "value" 
}, 
"testMode": false, 
"testVerificationPersonalNr": "XXXXXXXX-XXXX",
"testCallback": false
} 
```

PAYLOAD SCHEMA (json schema)
```javascript
{  
  "$schema": "http://json-schema.org/draft-04/schema#",  
  "type": "object",  
  "properties": {  
    "requestId": {  
      "type": "string",
      "description" : "Unique id. Could generated on caller side or on hour side if value is absent.",
      "maxLength": 255
    },  
    "customerId": {  
      "type": "string",
      "description" : "This value is provided by Verifiera. This value is an encrypted internal id which identifies Verifiera's customer. Every customer on integrators side will have unique customerId. We will use this value to connect request to particular customer in our system. Customer request this value from us and put in integrator's system.  ",
      "maxLength": 255
    },  
    "customerTitle": {  
      "type": "string",
      "description" : "This value will be shown on approval sent to candidate. Usually it will be company name of customerId.",
      "maxLength": 255
    },  
     "referenceCustomer": {  
      "type": "string",
      "description" : "Customer of customer.",
      "maxLength": 255
    },  
    "type": {  
      "type": "string"  ,
       "description" : "This is constant value.",
       enum: ['legal_analysis']
    },  
    "personalNr": {  
      "type": "string",  
      "pattern": "^\\d{8}-\\d{4}$",
       "description" : "Personal ID of candidate (personnummer). In testMode use only 19101010-1010 or you get en error.",
    },  
    "candidatePhone": {  
      "type": "string", 
      "pattern": "^07[\\d ]{8}$|^\\+[\\d ]+$" ,
        "description" : "Candadate's mobile phone. This value will be used to send approval. One of candidatePhone or candidateEmail should have value or both. If there no values in both candidatePhone and candidateEmail system will respond with validation error.",
    },  
    "candidateEmail": {  
      "type": "string",
       "description" : "Candadate's email. This value will be used to send approval. One of candidatePhone or candidateEmail should have value or both. If there no values in both candidatePhone and candidateEmail system will respond with validation error.",
    },  
    "callbackUrl": { 
      "type": "string",
      "description": "The Callback Url will be used to send status updates. During Init request this callback will be checked with 'test' status and your server should respond with 200 http code otherwise system send 'general' error"
    },  
    "additionalData": {  
      "type": "object",  
      "description": "Arbitary data according to customer needs. The keys of this object should be agreed with verifiera so we could provide UI texts and search capabilities."
    },  
    "testMode": {  
      "type": "boolean" ,
      "description": "testMode is used during development. test mode provides following features: validate if personalId has test person ID (19101010-1010), does not register billable action on customer account, allow to use testVerificationPersonalNr"
    },  
     "testCallback": {  
      "type": "boolean" ,
      "description": "Our server makes test request to callback address during Init call. Default is false.  testCallback is true and callback url does not anwer with HTTP code 200 you get exception during Init."
    },  
    "testVerificationPersonalNr": {  
      "type": "string" , 
      "pattern": "^\\d{8}-\\d{4}$",
      "description": "To finish process candidate must validate request with BankId. This value can be used by developer to be able to validate background check for a test person."
    }  
  },  

  "required": [  
    "customerId",  
    "customerTitle",  
    "type",  
    "personalNr",  
    "candidatePhone",
    "candidateEmail",  
  ]  
} 
```

ERROR RESPONSE (json)

```javascript
{
"success": false,
"error": true,
"name": "Exception",
"message": "Incorrect personal_id format. Use (XX)XXXXXX-XXXX",
"code": 0,
"type": "yii\\base\\UserException"
}
```


## Commit
After approval request if approved by candidate integrator must Commit request. During commit Verifiera register purchase. This request can be done in callback or additionally by user.

**POST /api/integration/commit?request_id=REQUEST_ID**

## Status
Status response and callback payload have 100% same format. 
If callbackUrl is set, Verifiera will test if callback url returns HTTP 200 code for status update with status “test” during Init call.  

**GET /api/integration/status?request_id=REQUEST_ID**

PAYLOAD (json)
```javascript
{ 
    "requestId": "REQUEST_ID", 
    "requestCreatedAt": "2018-11-13T20:20:39+00:00", // date when init was called 
    "statusChangeAt": "2018-11-13T20:20:39+00:00", // date when init was called 
    "candidateResponseAt": "2018-11-13T20:20:39+00:00", // date when candidate responded to approval request 
    "status": "waiting_recipient_approval|waiting_security_approval| candidate_not_approved | candidate_approved |test", 
    "displayString": "String in swedish"
} 
```

PAYLOAD SCHEMA (json schema)
```javascript
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "properties": {
    "requestId": {
      "type": "string"
    },
    "requestCreatedAt": {
      "type": "date-time"
    },
    "statusChangeAt": {
      "type": "date-time"
    },
    "candidateResponseAt": {
      "type": "date-time"
    },
    "status": {
      "type": "string",
      "enum": ["waiting_recipient_approval","waiting_security_approval","candidate_not_approved","candidate_approved","test"]
    },
    "displayString": {
      "type": "string" 
     
    }
  },
  "required": [
    "requestId",
    "requestCreatedAt",
    "candidateResponseAt",
    "status",
    "displayString"
  ]
}
```

## Callback 

Callback sent as POST request with payload described in Status. Your server must respond with 2xx HTTP code, otherwise our server will make 3 retries with following windows: 1 minute, 1 hour, 1 day.

## Development aid

To make development easer there are additional calls to go through process.

/api/integration/candidate-response?request_id=REQUEST_id&answer=(yes|no)  
/api/integration/security-response?request_id=60620a1639d2173&answer=(accept|reject)
