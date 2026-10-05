# Fax API Integration Guide -  Sending, Callbacks, and Receiving Inbound Faxes

**Audience**: API developers integrating fax send/receive functionality into a platform.

**Auth Method**: `x-api-key`  

**Response Format**: JSON

**Base URL Pattern**: `https://apih.westfax.com/REST/{MethodName}/json`


---

## A Word Before You Start - Important.
  

The fax API is RESTful, HTTP POST-based, and responds with JSON. There are multi-part methods because you will be uploading files. If you aren't familiar with that please let us know and we'll help.

Also — **use your `x-api-key` header for authentication**. Every call. We will supply the x-api-key and the test account details separately.

There are two types of webhooks. **Inbound** and **Outbound**.

**Inbound webhooks** can be set by WestFax or by developers (see the *Managing Inbound Webhooks Guide*).

Inbound webhooks will provide a prodid, jobid and direction parameter as you'll learn below. Using this data you can retrieve metadata and fax documents using your api key.

**Outbound webhooks** are set when a fax job is sent. When you send your fax you will specify an outbound callback address (as we detail below) and when the fax job is complete (sent or not sent/busy) that callback will fire and you will receive the same three parameters as inbound webhooks. With those parameters you will be able to retrieve the metadata and detailed status information about that fax document transmission.


---

## Authentication

  
All API requests must include your API key as an HTTP header.

  
```

x-api-key: YOUR_API_KEY_HERE

```
  

You also need a `ProductId` — a UUID that identifies the specific fax line (product) you're operating on. Think of it as the fax line handle. You'll have this provisioned ahead of time.


---  

## Overview of the Flow

### Outbound (Sending)
  

```
Your App
│
├──► POST /Fax_SendFax ──────────────────────► WestFax API
│ (file + metadata + CallbackUrl)                     │
│                                                     │
│ ◄── 202 Accepted + JobId ───────────────────────────┘
│
│ [fax is transmitted in the background]
│
└──◄── POST {CallbackUrl} ◄──────────────────── WestFax API
(job completed notification)
```

  
### Inbound (Receiving)

 
```
Remote Sender ──► Fax Line ──► WestFax API
                                         │    
POST {CallbackUrl}                      ◄┘
     │
Your App parses callback
     │
POST /Fax_GetFaxDocuments ──► WestFax API
                                 │
◄── Base64 fax document(s) ──────┘
```
  

---

  
## Part 1: Sending a Fax with a Callback

  ### Endpoint

```bash

POST https://apih.westfax.com/REST/Fax_SendFax/json
Content-Type: multipart/form-data
x-api-key: YOUR_API_KEY_HERE

```

  
This is a `multipart/form-data` POST because you're uploading files alongside metadata. 

  
### Request Parameters

  

| Parameter       | Required | Type     | Description                                                                                      |
| --------------- | -------- | -------- | ------------------------------------------------------------------------------------------------ |
| `x-api-key`     | Yes      | Header   | Your x-api-key                                                                                   |
| `ProductId`     | **Yes**  | UUID     | The fax line (product) identifier                                                                |
| `Numbers1`      | **Yes**  | string   | First recipient fax number (10-digit US/CA)                                                      |
| `Numbers2..N`   | No       | string   | Additional recipients (`Numbers2`, `Numbers3`, ...)                                              |
| `Files1`        | **Yes**  | file     | First document to fax (PDF, DOCX, TIFF, etc.)                                                    |
| `Files2..N`     | No       | file     | Additional documents (`Files2`, `Files3`, ...)                                                   |
| `JobName`       | **Yes**  | string   | A label for this job (can be empty string `""`)                                                  |
| `Header`        | **Yes**  | string   | Fax header text (can be empty string `""`)                                                       |
| `BillingCode`   | **Yes**  | string   | Billing reference (can be empty string `""`)                                                     |
| `CallBackUrl`   | No       | string   | **Your HTTPS endpoint** to receive the job-complete notification **(See callback format below)** |
| `CSID`          | No       | string   | Calling Subscriber ID (what the remote fax machine sees as Caller ID)                            |
| `ANI`           | No       | string   | Automatic Number Identification override. **See Enterprise note below on setting this value**    |
| `StartDate`     | No       | datetime | Schedule future delivery (omit for immediate)                                                    |
| `FaxQuality`    | No       | string   | `"Fine"` (default) or `"Normal"`                                                                 |
| `FeedbackEmail` | No       | string   | Email address for delivery notification                                                          |


##  Callback Format

> **On `CallbackUrl`**:  Supply a publicly reachable HTTPS endpoint, and when the fax job completes (success, failure, or otherwise), WestFax will POST or GET the job result to that URL. You get near-real-time notification without polling. Use it.

  The format should be: (GET OR POST. Your preference)
[METHOD]https://mywebhook.com/process?jobid={@jobId}&prodid={@prodId}&dir={@dir}

This will result in:

GET or POST - Depending on what you specificed in the [METHOD]
https://mywebhook.com/process?jobid=0000-0000-...&prodid=0000-0000-...&dir=outbound

## ANI Override (Enterprise Customers)
The ANI is the authoritative number the document is originating. In most instances the ANI is set to the actual fax number in the following format: 0000000000. No dashs or (). 

If you are an enterprise integration partner you can put values in the ANI of numbers that are not on the account. This is useful if you are using WestFax as a backup platform or sending on behalf of your customers or clients. 

For instance. If you are sending a fax for xyz clinic and their fax number is 202-555-1212 you would set the ANI to 2025551212 and the CSID parameter could be the (202) 555-1212 or XYZ Clinic. The CSID field can accept short text values. ***Best practice is to use the number for both fields to ensure optimal delivery.***

### cURL Example

  

```bash
curl -X POST "https://apih.westfax.com/REST/Fax_SendFax/json" \
-H "x-api-key: your-api-key-here" \
-F "ProductId=0000000000-e5f6-7890-abcd-0000000000" \
-F "Numbers1=8005551234" \
-F "Numbers2=8005555678" \
-F "JobName=Invoice_Q4_2025" \
-F "Header=Acme Corp Fax" \
-F "BillingCode=DEPT-FINANCE" \
-F "FaxQuality=Fine" \
-F "CSID=Acme Corp 202-555-0000" \
-F "ANI=2025550000" \
-F "CallbackUrl=[GET]https://example.com/webhooks/fax/out?prodId={prodId}&jobid={@jobId}" \
-F "Files1=@/path/to/invoice.pdf;type=application/pdf"
```


### Response


```json
{
"Success": true,
"ErrorString": "",
"InfoString": "",
"Result": "f7e8d9c0-b1a2-3456-cdef-789012345678"
}
```

  
Note:The ErrorString and InfoString only return if there is an issue.

| Field         | Description                                                           |
| ------------- | --------------------------------------------------------------------- |
| `Success`     | Boolean. `true` = accepted, `false` = something went sideways         |
| `Result`      | The outbound **JobId** (UUID string) — hang onto this, you'll need it |
| `ErrorString` | Human-readable error message when `Success` is `false`.               |
 

> **On failure**: Check `ErrorString`. Common causes include an invalid `ProductId`, a malformed fax number, an unsupported file type, or an account with insufficient credits. The API won't just silently swallow your bad request — it'll tell you.

  

---

## Part 2: Handling the Outbound Fax Callback

When the fax job completes, WestFax POSTs OR GETSs to your `CallbackUrl` with a `application/x-www-form-urlencoded` body. Your endpoint needs to:

1. Accept `POST` requests
2. Parse the form-encoded body
3. Return HTTP `200 OK` promptly

**Do not** do heavy lifting synchronously in the callback handler. Log it, enqueue it, do your actual processing asynchronously. The callback system has a timeout. If you start calling other APIs or running database queries before you send that `200`, that could cause us to resend the callback over and over. 
  
### Callback Payload Fields

| Field       | Type   | Description                                                  |     |
| ----------- | ------ | ------------------------------------------------------------ | --- |
| `x-api-key` | Header | - Your x-api-key                                             |     |
| `jobId`     | UUID   | The job identifier (same as the `Result` from `Fax_SendFax`) |     |
| `prodid`    | UUID   | The fax line that sent the job                               |     |
| `dir`       | string | `"Outbound"` for sent faxes                                  |     |
  
With the three data elements you need to call Fax_GetFaxDescriptionsUsingIds. 
You use the x-api-key and the jobid, prodid and dir values to replace these values.

``` bash
Call example:
curl --location 'https://apih.westfax.com/REST/Fax_GetFaxDescriptionsUsingIds/json' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'x-api-key: {APIKEY}' \
--form 'Cookies="false"' \
--form 'ProductId="{prodid}"' \
--form 'FaxIds1="{\"Id\":\"{@jobid}\",\"Direction\":\"{dir}\"}"'
```

Results:
``` json {

"Success": true,
"Result": [
{
	"FaxCallInfoList": 
	[
		{
		"CallId": "000000000000-3e7a-490c-930b-000000000000",
		"CompletedUTC": "2026-01-22T22:43:19Z",
		"TermNumber": "9702895979",
		"OrigNumber": "9702895979",
		"TermCSID": "(970) 289-5979",
		"OrigCSID": "(970) 289-5979",
		"Result": "Sent", //This is individual fax job status.
		"CallPageCount": 1,
		"FilterFlag": 0
		},
		{...} //more calls possible here.
	],

"Id": "000000000000-0a80-433e-af00-000000000000",
"Date": "2026-01-22T22:43:19Z",
"Direction": "Outbound",
"Tag": "Retrieved",
"PageCount": 1,
"FaxQuality": "Normal",
"Status": "Complete", //This is the overall status. Can be 'Complete' with all failed fax jobs.
"JobName": "Test Job",
"CreatedBy": "doug@westfax.com",
"LoginId": "00000000-0000-0000-0000-000000000000",
"DocPageCount": 0,
"FolderId": "00000000-0000-0000-0000-000000000000",
"FilesAvailable": false
}]}
```

### Job 'Status' from Fax_GetFaxDescriptionsUsingIds  

| State      | Meaning                                                                                                                                                                                                        |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Complete` | Job is done. Check individual call results for success/failure per recipient. This doesn't mean the fax was sent but rather the fax process has completed. Using the additional API call will provide details. |
| `Failed`   | Something went wrong at the job level. Contact support.                                                                                                                                                        |
|            |                                                                                                                                                                                                                |


### Per-Call 'Result' Values
There can be multiple fax recipients in a call so you need to check all 'Result' elements.

| Result                        | Meaning                                                                                |
| ----------------------------- | -------------------------------------------------------------------------------------- |
| `Sent`                        | Successfully delivered.                                                                |
| `NoAnswer`                    | Rang, nobody answered. Retries exhausted.                                              |
| `Busy`                        | Busy signal. Retries exhausted.                                                        |
| `NoFaxDevice`                 | Connected, but no fax handshake. Wrong number?                                         |
| `ConnectionInterrupt`         | Connected, partial send, then disconnected. The recipient may have a partial document. |
| `BadNumber` / `InvalidNumber` | Number format is wrong or the number is unreachable.                                   |
| `Cancelled`                   | This specific call was cancelled.                                                      |



### Verifying the Callback (Security Note)
  
Your callback URL is public. Anyone who knows it can POST to it. Consider:

- **HTTPS only** — non-negotiable in production
- **Validate the `JobId`** against your own records (did you actually submit a job with that ID?)
- **IP allowlisting** if WestFax provides a known source IP range. Let me know if you want to do this and we'll include the IP range.
- You can add your own parameter pairs in the callback i.e. &securityKey=somevalue if you want to add additional validators 

Note: Many ask why we don't just return the result in the callback. The reason is that a fax can be sent to one or many destinations. Some users send compliance faxes to hundred or thousands of customers. We send the Callback when the fax job is complete. in a fax job you could have 2 documents going to 4 organizations. 1 could fail because the number is disconnected, another job could be busy and 2 successfully sent. This method allows for explicit and clean status on every call. Most customer send one fax document to one destination but it is built for high capacity.
  
---

## Part 3: Receiving Inbound Faxes

When an inbound fax arrives on your fax line, WestFax will POST a notification to a pre-configured callback URL on your account (set up during provisioning, or configurable via the management API). The flow from there is:


1. Receive the inbound notification callback
2. Extract the `jobId` and `ProdId`
3. Call `Fax_GetFaxDescriptionsUsingIds` to get metadata (sender, pages, timestamp)
4. Call `Fax_GetFaxDocuments` to download the actual fax image
  

### Step 1: The Inbound Callback

Similar to the outbound callback, WestFax POSTs to your registered inbound webhook URL.

```

POST /webhooks/fax/inbound
Content-Type: application/x-www-form-urlencoded

jobid=00000000-a7b8-9012-cdef-00000000
&prodid=00000000-e5f6-7890-abcd-00000000
&dir=Inbound

```

  

Your response: `HTTP 200 OK`. Same rules apply as outbound — fast, idempotent, enqueue and move on.

We can set the key pairs to whatever name you like. prodid and jobid are just the standard parameters we use.
  

---
  

### Step 2: Get Fax Descriptions (Metadata)


Once you have the `FaxId` and `ProductId` from the callback, fetch the full fax metadata. Note, some fields are no longer used but provided to support legacy integrations. We'll note those below.

#### Endpoint

```

POST https://apih.westfax.com/REST/Fax_GetFaxDescriptionsUsingIds/json
Content-Type: application/x-www-form-urlencoded
x-api-key: YOUR_API_KEY_HERE

```

  
#### Request Parameters

| Parameter    | Required | Type   | Description                        |
| ------------ | -------- | ------ | ---------------------------------- |
| `x-api-key`  | Yes      | Header | Your x-api-key                     |
| `ProductId`  | **Yes**  | UUID   | The fax line that received the fax |
| `FaxIds1`    | **Yes**  | UUID   | The fax ID from the callback       |
| `FaxIds2..N` | No       | UUID   | Additional fax IDs (batch lookup)  |

#### cURL Example

  
``` bash
Call example:
curl --location 'https://apih.westfax.com/REST/Fax_GetFaxDescriptionsUsingIds/json' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'x-api-key: {APIKEY}' \
--form 'Cookies="false"' \
--form 'ProductId="{prodid}"' \
--form 'FaxIds1="{\"Id\":\"{@jobid}\",\"Direction\":\"{dir}\"}"'
```

Results: (Will always just be a one item array of items in FaxCallInfoList)
``` json {
{
"Success": true,
"Result": [
			{

			"FaxCallInfoList": [

				{
				"CallId": "0000000000-06b0-4f9d-9c4b-0000000000",
				"CompletedUTC": "2026-02-19T20:20:21Z",
				"TermNumber": "9702895979",  //To Number
				"OrigNumber": "9702895979", //From Number
				"TermCSID": "(970) 289-5979", //To CallerID
				"OrigCSID": "(970) 289-5979", //From CallerId
				"Result": "Success", //Inbound will always be Success
				"CallPageCount": 11, //Page Count
				"FilterFlag": 0
				}

			],

	"Id": "0000000000-6fd4-4d2d-8b54-0000000000",
	"Date": "2026-02-19T20:20:21Z",
	"Direction": "Inbound",
	"Tag": "None",
	"PageCount": 11,
	"FaxQuality": "Fine",
	"Status": "Complete",
	"Reference": "0000000000-6fd4-4d2d-8b54-0000000000",
	"JobName": "Fax from (970) 289-5979: Referral Note and Lab Rep",
	"CreatedBy": "System",
	"LoginId": "00000000-0000-0000-0000-000000000000",
	"CreatedVia": "InboundCall",
	"DocPageCount": 0, -> Legacy
	"FilterValue": "None",
	"FolderId": "00000000-0000-0000-0000-000000000000",
	"FilesAvailable": true

}]}
```



---

  

### Step 3: Download the Fax Document


Now that you've confirmed the fax exists and metadata looks right, pull the actual document. 
#### Endpoint
 

```bash

POST https://apih.westfax.com/REST/Fax_GetFaxDocuments/json
Content-Type: application/x-www-form-urlencoded
x-api-key: YOUR_API_KEY_HERE

```

  
#### Request Parameters

  

| Parameter    | Required | Type   | Description                       |
| ------------ | -------- | ------ | --------------------------------- |
| `x-api-key`  | Yes      | Header | Your x-api-key                    |
| `ProductId`  | **Yes**  | UUID   | The fax line identifier           |
| `FaxIds1`    | **Yes**  | UUID   | The fax document ID               |
| `FaxIds2..N` | No       | UUID   | Additional IDs for batch download |
| `Format`     | No       | string | Output format (see table below)   |



#### Supported Formats

| Format Value | Description                             |
| ------------ | --------------------------------------- |
| `pdf`        | PDF document **(default, recommended)** |
| `tiff`       | TIFF image                              |

 
#### cURL Example

  
```bash

curl --location 'https://apih.westfax.com/REST/Fax_GetFaxDocuments/json' \
--header 'x-api-key: APIKEY' \

--form 'ProductId="00000000-a8ac-4a77-8007-00000000"' \

--form 'FaxIds1="{\"Id\":\"00000000-6fd4-4d2d-8b54-00000000\",\"Direction\":\"Inbound\"}"' \

--form 'Format="pdf"'
```

  #### Response

The response wraps the document data in the standard result envelope:

  

```json
{
"Success": true,
"Result": [

{

"FaxFiles": [

	{
	"ContentType": "application/pdf",
	"ContentLength": 228693,	
	"FileContents": "JVBERi0xLjQKJdP0zOEKJS...", //BASE64
	}

],
"Id": "00000000-6fd4-4d2d-8b54-00000000",
"Direction": "Inbound",
"Date": "2026-02-19T20:20:21Z",
"Status": "Ok",
"Format": "pdf",
"PageCount": 11

}]}

```

  
Relevant fields:

| Field          | Description                     |
| -------------- | ------------------------------- |
| `Id`           | Fax identifier (UUID)           |
| `FileContents` | **Base64-encoded** file content |
| `Date`         | Date / Time recieved            |
| `Format`       | Format of the returned file     |
| `PageCount`    | Number of pages                 |

  

Decode `FileContents` from Base64 and write the bytes to your storage of choice. Done — you have the fax.


## Error Handling


All API responses follow the same envelope structure:


```json

{

"Success": false,
"ErrorString": "A descriptive error message explaining what went wrong",
"InfoString": "Optional additional context",
"Result": null
}

```
  


  
**Common error scenarios:**

| Scenario             | Typical `ErrorString`                        | Check                                                                      |
| -------------------- | -------------------------------------------- | -------------------------------------------------------------------------- |
| Bad API key          | `"Authorization_Failed_BadUsernamePassword"` | Check your key.                                                            |
| Invalid `ProductId`  | `"Authorization_Failed_UserRoleLevel"`       | Permissions likely. Check that API Key has access in home portal.          |
| Malformed fax number | `"Api_Failed_MissingDialList"`               | check the format of the outbound number. Use nnnnnnnnnn to keep it simple. |
|                      |                                              |                                                                            |

  ---

  
## API Endpoint Quick Reference

  

| Method | Endpoint URL                                   | Purpose                                |
| ------ | ---------------------------------------------- | -------------------------------------- |
| `POST` | `.../REST/Fax_SendFax/json`                    | Submit a fax for transmission          |
| `POST` | `.../REST/Fax_GetFaxDescriptionsUsingIds/json` | Get fax metadata by specific IDs       |
| `POST` | `.../REST/Fax_GetFaxDocuments/json`            | Download fax document content (Base64) |

Base URL for all: `https://apih.westfax.com/REST/{MethodName}/json`


---  

## Integration Checklist

  

Before you call this done, make sure you've covered the basics:

  

- [ ] **API key is in the `x-api-key` header** — not in the POST body, not in the URL query string
- [ ] **Callback endpoint is HTTPS** — not HTTP
- [ ] **Callback handler returns `200 OK` within the timeout window** — do async processing, not synchronous
- [ ] **Callback handler is idempotent** — receiving the same event twice doesn't cause duplicate records
- [ ] **You're storing the `JobId`** returned from `Fax_SendFax` to correlate with callbacks and status checks
- [ ] **You're checking `Success: true`** before attempting to use `Result`. 
- [ ] **Base64 decoding of `FileContents` is handled correctly** — Many have just written base64 to disk and that doesn't work as one can imagine.
  
---

  
