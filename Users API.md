## Request

```
POST https://verifiera.se/api/users?dry-run=false   
Authentication: Bearer TOKEN
```

**Params**  
dry-run – required, false or true. Use dry-run=true to view changes before applying them. You can see if the request was executed by reading the "executed" value in the response.

**Example of request body:**   
```raw
user@test.com   
user2@test.com;AI,Ps,Fs,A   
user3@test.com
```
 

Example of response 
```json
{
    "status": 200,
    "success": true,
    "error": false,
    "created": [
          "user@test.com"
    ],
    "updated": {
        "user2@test.com": [
            "AI",
            "Ps",
            "Fs",
            "A"
        ]
    },
    "deleted": [
         "user4@test.com"
     ],
    "executed": true
}
```
 

## Description
### Users
The request body should contain a list of usernames delimited with a new line. You must include all users in the request; users absent from the list will be marked as deleted. 
However, returning them to the list will restore deleted users without any loss.   
### Permissions
Additionally, you can add permissions that need to be applied to the user by putting them after the “;” delimiter. 
If permissions are absent, default account permission will be used. 

The following permissions are accepted.
```
        'Ps'  => Personsök,
        'Fs'  => Företagsök,
        'Ds'  => Dokumentsök,
        'Im'  => Import,
        'Bv'  => Bevakning,
        'Bkg' => Bakgrundskontroll,
        'Lr'  => Lärare,
        'Lv'  => Läkare,
        'A'   => Analyssmall,
        'AI'  => Analyssmallinställningar
```

## Additional info
API following general logic from [Verifiera API](https://verifiera.se/docs/api), for example, Errors handling.  
API also can return responses in XML format; info is provided in general API docs.
