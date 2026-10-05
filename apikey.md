
## Purpose

This API call generates an **API key** for a specified account. The resulting API key determines which fax lines (products) the caller can access based on assigned permissions.


---
## Endpoint

**POST**

https://api3.westfax.com/REST/Security_CreateApiKey/json

---
## Request Parameters

| Parameter  | Required | Description                                           |
| ---------- | -------- | ----------------------------------------------------- |
| Username   | Yes      | Customer’s WestFax username                           |
| Password   | Yes      | Customer’s WestFax password                           |
| AccountId  | Yes      | Account ID provided by WestFax                        |
| ApiKeyInfo | Yes      | JSON object defining the API key name and permissions |

---

## Example Request

```bash

curl --location 'https://api3.westfax.com/REST/Security_CreateApiKey/json' \

--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'Username=user@user.com' \
--data-urlencode 'Password=passwordX' \
--data-urlencode 'AccountId=000000000-e660-49ce-80f6-0000000000' \
--data-urlencode 'ApiKeyInfo={
"Name": "Enterprise",
"Acls": [
			{
				"EntityId": "000000000-e660-49ce-80f6-0000000000",
				"ACLType": "Account",
				"UserRoleLevel": "Admin"
			}
		]
}'

```

  

## Example Response

```json

{
"Success": true,
"Result": 
		{
		"Id": "0000000-e289-44db-a28c-0000000000000",
		"Name": "Enterprise",
		"Acls": [
			{
			"SrcAclId": "0000000-2559-4d31-9c1d-0000000000000",
			"Inherited": false,
			"ACLType": "Account",
			"UserRoleLevel": "Admin",
			"EntityName": "WestFax Demo",
			"EntityId": "0000000-e660-49ce-80f6-0000000000000"
			},
			{
			"SrcAclId": "0000000-2559-4d31-9c1d-0000000000000",
			"Inherited": true,
			"ACLType": "Product",
			"UserRoleLevel": "Admin",
			"EntityName": "Test #2 (FaxForward)",
			"EntityId": "0000000-768c-4f06-b978-0000000000000"
			},
			{
			"SrcAclId": "0000000-2559-4d31-9c1d-0000000000000",
			"Inherited": true,
			"ACLType": "Product",
			"UserRoleLevel": "Admin",
			"EntityName": "Test #3 (FaxForward)",
			"EntityId": "0000000-e54a-4c7a-b13b-fe7be351f5e7"
			}
		],
	"ExpirationUtc": "9999-12-31T23:59:59Z",
	"Secret": "APIKEYAPIKEY123456789001234567890APIKEYAPIKEYAPIKEY"
	}
}

```

  

The value returned in *Result.Secret* is the API key.

  ```bash

x-api-key: <Result.Secret>

```

### Making API calls with the x-api-key

```bash
curl --location 'https://api3.westfax.com/REST/Fax_GetFaxDescriptionsUsingIds/json' \

--header 'Content-Type: application/x-www-form-urlencoded' \

--header 'x-api-key: APIKEYAPIKEY123456789001234567890APIKEYAPIKEYAPIKEY' \

--form 'Cookies="false"' \

--form 'ProductId="0000000000-a8ac-4a77-8007-00000000000"' \

--form 'FaxIds1="{\"Id\":\"0000000000-0add-4d23-9087-0000000000\",\"Direction\":\"Inbound\"}"'
```
