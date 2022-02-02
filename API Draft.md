# API Draft
Version: 0.1  
This document is subject for future changes.  
Check json description in json schema.  

## Init method
Initializes background check. If success server respåond with 200 HTTP Code, otherwise JSON from Appendix 2 

**POST /api/integration/init**

PAYLOAD (json)

```javascript
{  
"requestId": "any_unique_id",  
"customerId": "CUSTOMER_id", 
"customerTitle": "COMPANY NAME",
"type": "legal_analysis",
"personalNr": "XXXXXXXX-XXXX", 
"candidatePhone": "",  
"candidateEmail": "",  
"callbackUrl": "CALLBACK URL", 
"additionalData": { 
       "key": "value" 
}, 
"testMode": false, 
"testVerificationPersonalNr": "XXXXXXXX-XXXX" 
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
      "description" : "Unique id generated on caller side. For example UUID. varchar(255)",
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
    "testVerificationPersonalNr": {  
      "type": "string" , 
      "pattern": "^\\d{8}-\\d{4}$",
      "description": "To finish process candidate must validate request with BankId. This value can be used by developer to be able to validate background check for a test person."
    }  
  },  

  "required": [  
    "requestId",  
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
   "errors": [ 
      { 
         "type": "validation", 
         "data": "FIELD IF validation", 
         "text": {
             "sv": "TEXT IN SWEDISH",
             "en": "TEXT IN ENGLISH"
         }
      } 
   ] 
} 
```

ERROR RESPONSE SCHEME (json scheme)

```javascript
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "properties": {
    "errors": {
      "type": "array",
      "items": [
        {
          "type": "object",
          "properties": {
            "type": {
              "type": "string",
              "enum": ["validation", "general"]
            },
            "data": {
              "type": "string",
              "description": "Provides additional information. For 'validation' it provides field key."
            },
            "text": {
              "type": "object"
            }
          },
          "required": [
            "type",           
            "text"
          ]
        }
      ]
    }
  },
  "required": [
    "errors"
  ] 
}
```

## Status
Status response and callback payload have 100% same format. 
If callbackUrl is set, Verifiera will test if callback url returns HTTP 200 code for status update with status “test” during Init call.  

**GET /api/integration/status/REQUEST_ID**

PAYLOAD (json)
```javascript
{ 
    "requestId": "REQUEST_ID", 
    "requestCreatedAt": "2018-11-13T20:20:39+00:00", // date when init was called 
    "statusChangeAt": "2018-11-13T20:20:39+00:00", // date when init was called 
    "candidateResponseAt": "2018-11-13T20:20:39+00:00", // date when candidate responded to approval request 
    "status": "waiting_recipient_approval|waiting_security_approval| candidate_not_approved | candidate_approved |test", 
    "displayString": { 
 	    "sv" : "STRING TO DISPLAY IN SWEDISH", 
        "en" : "STRING TO DISPLAY IN ENGLISH", 
    } 
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
      "type": "object",
      "properties": {
        "sv": {
          "type": "string"
        },
        "en": {
          "type": "string"
        }
      },
      "required": [
        "sv",
        "en"
      ]
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

##Callback 

Callback sent as POST request with payload described in Status. Your server must respond with 2xx HTTP code, otherwise our server will make 3 retries with following windows: 1 minute, 1 hour, 1 day. 
