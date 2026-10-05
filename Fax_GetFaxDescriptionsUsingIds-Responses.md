`# `Fax_GetFaxDescriptionsUsingIds` — Example Responses

  

Full, realistic `Fax_GetFaxDescriptionsUsingIds` JSON responses for every status/result combination documented in [`Fax_GetFaxDescriptionsUsingIds_Status_Result_Reference.md`](Fax_GetFaxDescriptionsUsingIds_Status_Result_Reference.md). Built off a real captured response's field shape — pick the block(s) that apply and drop the rest.

  

**A note on field confidence** — most fields/values below are confirmed directly against the SDK source (`Direction`, `FaxQuality`, `Status`/`FaxStatus`, `FaxCallInfoList[].Result`, and `Tag`/`FilterValue`, whose four real values — `None` = unread, `Retrieved` = read, `Removed` = moved to Deleted Items, `Deleted` = permanently gone — are documented inline at `FaxInterfaceRaw.cs:1805-1807`). Two fields on the live wire aren't modeled in the current SDK at all, so treat their example values as **illustrative, not guaranteed**:

- `ProcessingState` — only value ever observed is `"Processing_Visible"`; no enum for it exists in the SDK.

- `CreatedVia` — free-text field, no fixed catalog in the SDK. `"InboundCall"` is confirmed (from your sample); `"API"` used below for outbound examples is a plausible placeholder, not a confirmed literal.

  

If a customer needs those two pinned down precisely, that has to come from WestFax directly rather than the SDK.

  

---

  

## 1. Inbound fax — received successfully (`Complete`)

  

```json

{

"Success": true,
"Result": [

			{

				"FaxCallInfoList": 
				[
				{
				"CallId": "00000000-65ce-482f-bc31-00000000",			
				"CompletedUTC": "2026-09-09T23:19:22Z",
				"TermNumber": "9702895979",
				"OrigNumber": "8005551234",
				"TermCSID": "(970) 289-5979",
				"OrigCSID": "ACME REMOTE SENDER",
				"Result": "Success",
				"CallPageCount": 1,
				"FilterFlag": 0
				}
				],

	"Id": "00000000-37b7-493c-9301-00000000",
	"Date": "2026-09-09T23:19:22Z",
	"Direction": "Inbound",
	"Tag": "None",
	"PageCount": 1,
	"FaxQuality": "Fine",
	"Status": "Complete",
	"Reference": "00000000-37b7-493c-9301-00000000",
	"JobName": "Fax from (970) 289-5979: Test Faxes",
	"CreatedBy": "System",
	"LoginId": "00000000-0000-0000-0000-000000000000",
	"CreatedVia": "InboundCall",
	"DocPageCount": 0,
	"FilterValue": "None",
	"FolderId": "00000000-0000-0000-0000-000000000000",
	"FilesAvailable": true,
	"ProcessingState": "Processing_Visible"
	}
	]
}

```

  

## 2. Inbound fax — already retrieved/read

  

Same fax as above, but the caller already downloaded it via `Fax_GetFaxDocuments` (or marked it read). Only `Tag`/`FilterValue` change.

  

```json

{

"Success": true,

"Result": [

{

"FaxCallInfoList": [

{

"CallId": "00000000-65ce-482f-bc31-00000000",

"CompletedUTC": "2026-09-09T23:19:22Z",

"TermNumber": "9702895979",

"OrigNumber": "8005551234",

"TermCSID": "(970) 289-5979",

"OrigCSID": "ACME REMOTE SENDER",

"Result": "Success",

"CallPageCount": 1,

"FilterFlag": 1

}

],

"Id": "00000000-37b7-493c-9301-00000000",

"Date": "2026-09-09T23:19:22Z",

"Direction": "Inbound",

"Tag": "Retrieved",

"PageCount": 1,

"FaxQuality": "Fine",

"Status": "Complete",

"Reference": "00000000-37b7-493c-9301-00000000",

"JobName": "Fax from (970) 289-5979: Test Faxes",

"CreatedBy": "System",

"LoginId": "00000000-0000-0000-0000-000000000000",

"CreatedVia": "InboundCall",

"DocPageCount": 0,

"FilterValue": "Retrieved",

"FolderId": "00000000-0000-0000-0000-000000000000",

"FilesAvailable": true,

"ProcessingState": "Processing_Visible"

}

]

}

```

  

---

  

## 3. Outbound fax — delivered successfully (`Sent`)

  

```json

{

"Success": true,

"Result": [

{

"FaxCallInfoList": [

{

"CallId": "00000000-3c9e-4a7b-9f10-00000000",

"CompletedUTC": "2026-09-11T14:45:10Z",

"TermNumber": "8005559999",

"OrigNumber": "8005551234",

"TermCSID": "Acme Corp Fax",

"OrigCSID": "Your Company Fax",

"Result": "Sent",

"CallPageCount": 3,

"FilterFlag": 0

}

],

"Id": "00000000-a7b8-9012-cdef-00000000",

"Date": "2026-09-11T14:44:52Z",

"Direction": "Outbound",

"Tag": "None",

"PageCount": 3,

"FaxQuality": "Fine",

"Status": "Complete",

"Reference": "00000000-a7b8-9012-cdef-00000000",

"JobName": "Invoice_Q3_2026",

"CreatedBy": "jsmith@example.com",

"LoginId": "00000000-9e0f-4123-a456-00000000",

"CreatedVia": "API",

"DocPageCount": 3,

"FilterValue": "None",

"FolderId": "00000000-0000-0000-0000-000000000000",

"FilesAvailable": true,

"ProcessingState": "Processing_Visible"

}

]

}

```

  

## 4. Outbound fax — failed, no answer (`NoAnswer`)

  

```json

{

"Success": true,

"Result": [

{

"FaxCallInfoList": [

{

"CallId": "00000000-5b6c-4d7e-8f90-00000000",

"CompletedUTC": "2026-09-11T15:02:33Z",

"TermNumber": "8005559998",

"OrigNumber": "8005551234",

"TermCSID": "",

"OrigCSID": "Your Company Fax",

"Result": "NoAnswer",

"CallPageCount": 0,

"FilterFlag": 0

}

],

"Id": "00000000-b8c9-0123-def4-00000000",

"Date": "2026-09-11T14:59:01Z",

"Direction": "Outbound",

"Tag": "None",

"PageCount": 3,

"FaxQuality": "Fine",

"Status": "Complete",

"Reference": "00000000-b8c9-0123-def4-00000000",

"JobName": "Invoice_Q3_2026",

"CreatedBy": "jsmith@example.com",

"LoginId": "00000000-9e0f-4123-a456-00000000",

"CreatedVia": "API",

"DocPageCount": 3,

"FilterValue": "None",

"FolderId": "00000000-0000-0000-0000-000000000000",

"FilesAvailable": true,

"ProcessingState": "Processing_Visible"

}

]

}

```

  

## 5. Outbound fax — failed, busy (`Busy`)

  

```json

{

"Success": true,

"Result": [

{

"FaxCallInfoList": [

{

"CallId": "00000000-6c7d-4e8f-9012-00000000",

"CompletedUTC": "2026-09-11T15:10:07Z",

"TermNumber": "8005559997",

"OrigNumber": "8005551234",

"TermCSID": "",

"OrigCSID": "Your Company Fax",

"Result": "Busy",

"CallPageCount": 0,

"FilterFlag": 0

}

],

"Id": "00000000-c9d0-1234-ef56-00000000",

"Date": "2026-09-11T15:06:44Z",

"Direction": "Outbound",

"Tag": "None",

"PageCount": 3,

"FaxQuality": "Fine",

"Status": "Complete",

"Reference": "00000000-c9d0-1234-ef56-00000000",

"JobName": "Invoice_Q3_2026",

"CreatedBy": "jsmith@example.com",

"LoginId": "00000000-9e0f-4123-a456-00000000",

"CreatedVia": "API",

"DocPageCount": 3,

"FilterValue": "None",

"FolderId": "00000000-0000-0000-0000-000000000000",

"FilesAvailable": true,

"ProcessingState": "Processing_Visible"

}

]

}

```

  

## 6. Outbound fax — failed, no fax device detected (`NoFaxDevice`)

  

```json

{

"Success": true,

"Result": [

{

"FaxCallInfoList": [

{

"CallId": "00000000-7d8e-4f90-a123-00000000",

"CompletedUTC": "2026-09-11T15:20:19Z",

"TermNumber": "8005559996",

"OrigNumber": "8005551234",

"TermCSID": "",

"OrigCSID": "Your Company Fax",

"Result": "NoFaxDevice",

"CallPageCount": 0,

"FilterFlag": 0

}

],

"Id": "00000000-d0e1-2345-f678-00000000",

"Date": "2026-09-11T15:17:52Z",

"Direction": "Outbound",

"Tag": "None",

"PageCount": 3,

"FaxQuality": "Fine",

"Status": "Complete",

"Reference": "00000000-d0e1-2345-f678-00000000",

"JobName": "Invoice_Q3_2026",

"CreatedBy": "jsmith@example.com",

"LoginId": "00000000-9e0f-4123-a456-00000000",

"CreatedVia": "API",

"DocPageCount": 3,

"FilterValue": "None",

"FolderId": "00000000-0000-0000-0000-000000000000",

"FilesAvailable": true,

"ProcessingState": "Processing_Visible"

}

]

}

```

  

## 7. Outbound fax — failed, invalid/unreachable number (`InvalidNumber`)

  

```json

{

"Success": true,

"Result": [

{

"FaxCallInfoList": [

{

"CallId": "00000000-8e9f-4012-b345-00000000",

"CompletedUTC": "2026-09-11T15:25:44Z",

"TermNumber": "8005559995",

"OrigNumber": "8005551234",

"TermCSID": "",

"OrigCSID": "Your Company Fax",

"Result": "InvalidNumber",

"CallPageCount": 0,

"FilterFlag": 0

}

],

"Id": "00000000-e1f2-3456-a789-00000000",

"Date": "2026-09-11T15:24:10Z",

"Direction": "Outbound",

"Tag": "None",

"PageCount": 3,

"FaxQuality": "Fine",

"Status": "Complete",

"Reference": "00000000-e1f2-3456-a789-00000000",

"JobName": "Invoice_Q3_2026",

"CreatedBy": "jsmith@example.com",

"LoginId": "00000000-9e0f-4123-a456-00000000",

"CreatedVia": "API",

"DocPageCount": 3,

"FilterValue": "None",

"FolderId": "00000000-0000-0000-0000-000000000000",

"FilesAvailable": true,

"ProcessingState": "Processing_Visible"

}

]

}

```

  

## 8. Outbound fax — connection interrupted mid-transmission (`ConnectionInterrupt`)

  

```json

{

"Success": true,

"Result": [

{

"FaxCallInfoList": [

{

"CallId": "00000000-9f01-4123-c456-00000000",

"CompletedUTC": "2026-09-11T15:33:29Z",

"TermNumber": "8005559994",

"OrigNumber": "8005551234",

"TermCSID": "Recipient Fax Machine",

"OrigCSID": "Your Company Fax",

"Result": "ConnectionInterrupt",

"CallPageCount": 1,

"FilterFlag": 0

}

],

"Id": "00000000-f2a3-4567-b890-00000000",

"Date": "2026-09-11T15:31:58Z",

"Direction": "Outbound",

"Tag": "None",

"PageCount": 3,

"FaxQuality": "Fine",

"Status": "Complete",

"Reference": "00000000-f2a3-4567-b890-00000000",

"JobName": "Invoice_Q3_2026",

"CreatedBy": "jsmith@example.com",

"LoginId": "00000000-9e0f-4123-a456-00000000",

"CreatedVia": "API",

"DocPageCount": 3,

"FilterValue": "None",

"FolderId": "00000000-0000-0000-0000-000000000000",

"FilesAvailable": true,

"ProcessingState": "Processing_Visible"

}

]

}

```

  

## 9. Outbound fax — cancelled before completion

  

```json

{

"Success": true,

"Result": [

{

"FaxCallInfoList": [

{

"CallId": "00000000-0123-4456-d789-00000000",

"CompletedUTC": "2026-09-11T15:40:00Z",

"TermNumber": "8005559993",

"OrigNumber": "8005551234",

"TermCSID": "",

"OrigCSID": "Your Company Fax",

"Result": "Cancelled",

"CallPageCount": 0,

"FilterFlag": 0

}

],

"Id": "00000000-a3b4-5678-c901-00000000",

"Date": "2026-09-11T15:38:12Z",

"Direction": "Outbound",

"Tag": "None",

"PageCount": 3,

"FaxQuality": "Fine",

"Status": "Cancelled",

"Reference": "c9d0e1f2-a3b4-5678-c901-901234567890",

"JobName": "Invoice_Q3_2026",

"CreatedBy": "jsmith@example.com",

"LoginId": "00000000-9e0f-4123-a456-00000000",

"CreatedVia": "API",

"DocPageCount": 3,

"FilterValue": "None",

"FolderId": "00000000-0000-0000-0000-000000000000",

"FilesAvailable": false,

"ProcessingState": "Processing_Visible"

}

]

}

```

  

## 10. Outbound fax — still in progress / dialing (`Production`)

  

Transient — only seen if you query while the job is actively being worked. `FaxCallInfoList` may be empty or show an in-flight attempt with no `CompletedUTC` yet.

  

```json

{

"Success": true,

"Result": [

{

"FaxCallInfoList": [

{

"CallId": "00000000-2345-4567-e890-00000000",

"CompletedUTC": "0001-01-01T00:00:00Z",

"TermNumber": "8005559992",

"OrigNumber": "8005551234",

"TermCSID": "",

"OrigCSID": "Your Company Fax",

"Result": "Sending",

"CallPageCount": 0,

"FilterFlag": 0

}

],

"Id": "00000000-b4c5-6789-d012-00000000",

"Date": "2026-09-11T15:45:00Z",

"Direction": "Outbound",

"Tag": "None",

"PageCount": 3,

"FaxQuality": "Fine",

"Status": "Production",

"Reference": "00000000-b4c5-6789-d012-00000000",

"JobName": "Invoice_Q3_2026",

"CreatedBy": "jsmith@example.com",

"LoginId": "00000000-9e0f-4123-a456-00000000",

"CreatedVia": "API",

"DocPageCount": 3,

"FilterValue": "None",

"FolderId": "00000000-0000-0000-0000-000000000000",

"FilesAvailable": false,

"ProcessingState": "Processing_Visible"

}

]

}

```

  

## 11. Outbound fax — multiple attempts on one job (retry, then delivered)

  

Some jobs retry a busy/no-answer number before ultimately succeeding — `FaxCallInfoList` then holds more than one entry for the same job.

  

```json

{

"Success": true,

"Result": [

{

"FaxCallInfoList": [

{

"CallId": "00000000-4567-4890-f012-00000000",

"CompletedUTC": "2026-09-11T15:50:11Z",

"TermNumber": "8005559991",

"OrigNumber": "8005551234",

"TermCSID": "",

"OrigCSID": "Your Company Fax",

"Result": "Busy",

"CallPageCount": 0,

"FilterFlag": 0

},

{

"CallId": "00000000-6789-4012-a345-00000000",

"CompletedUTC": "2026-09-11T15:55:47Z",

"TermNumber": "8005559991",

"OrigNumber": "8005551234",

"TermCSID": "Acme Corp Fax",

"OrigCSID": "Your Company Fax",

"Result": "Sent",

"CallPageCount": 3,

"FilterFlag": 0

}

],

"Id": "00000000-c5d6-7890-e123-00000000",

"Date": "2026-09-11T15:48:30Z",

"Direction": "Outbound",

"Tag": "None",

"PageCount": 3,

"FaxQuality": "Fine",

"Status": "Complete",

"Reference": "00000000-c5d6-7890-e123-00000000",

"JobName": "Invoice_Q3_2026",

"CreatedBy": "jsmith@example.com",

"LoginId": "00000000-9e0f-4123-a456-00000000",

"CreatedVia": "API",

"DocPageCount": 3,

"FilterValue": "None",

"FolderId": "00000000-0000-0000-0000-000000000000",

"FilesAvailable": true,

"ProcessingState": "Processing_Visible"

}

]

}

```

  

---

  

## 12. Batch lookup — mixed inbound/outbound IDs in one call

  

This is the normal shape when `FaxIds1..N` requests several different faxes at once (matches how the endpoint is meant to be used for batch lookups).

  

```json

{

"Success": true,

"Result": [

{

"FaxCallInfoList": [

{

"CallId": "00000000-65ce-482f-bc31-00000000",

"CompletedUTC": "2026-09-09T23:19:22Z",

"TermNumber": "9702895979",

"OrigNumber": "8005551234",

"TermCSID": "(970) 289-5979",

"OrigCSID": "ACME REMOTE SENDER",

"Result": "Success",

"CallPageCount": 1,

"FilterFlag": 0

}

],

"Id": "00000000-37b7-493c-9301-00000000",

"Date": "2026-09-09T23:19:22Z",

"Direction": "Inbound",

"Tag": "None",

"PageCount": 1,

"FaxQuality": "Fine",

"Status": "Complete",

"Reference": "00000000-37b7-493c-9301-00000000",

"JobName": "Fax from (970) 289-5979: Test Faxes",

"CreatedBy": "System",

"LoginId": "00000000-0000-0000-0000-000000000000",

"CreatedVia": "InboundCall",

"DocPageCount": 0,

"FilterValue": "None",

"FolderId": "00000000-0000-0000-0000-000000000000",

"FilesAvailable": true,

"ProcessingState": "Processing_Visible"

},

{

"FaxCallInfoList": [

{

"CallId": "00000000-3c9e-4a7b-9f10-00000000",

"CompletedUTC": "2026-09-11T14:45:10Z",

"TermNumber": "8005559999",

"OrigNumber": "8005551234",

"TermCSID": "Acme Corp Fax",

"OrigCSID": "Your Company Fax",

"Result": "Sent",

"CallPageCount": 3,

"FilterFlag": 0

}

],

"Id": "00000000-a7b8-9012-cdef-00000000",

"Date": "2026-09-11T14:44:52Z",

"Direction": "Outbound",

"Tag": "None",

"PageCount": 3,

"FaxQuality": "Fine",

"Status": "Complete",

"Reference": "00000000-a7b8-9012-cdef-00000000",

"JobName": "Invoice_Q3_2026",

"CreatedBy": "jsmith@example.com",

"LoginId": "00000000-9e0f-4123-a456-00000000",

"CreatedVia": "API",

"DocPageCount": 3,

"FilterValue": "None",

"FolderId": "00000000-0000-0000-0000-000000000000",

"FilesAvailable": true,

"ProcessingState": "Processing_Visible"

},

{

"FaxCallInfoList": [

{

"CallId": "00000000-5b6c-4d7e-8f90-00000000",

"CompletedUTC": "2026-09-11T15:02:33Z",

"TermNumber": "8005559998",

"OrigNumber": "8005551234",

"TermCSID": "",

"OrigCSID": "Your Company Fax",

"Result": "NoAnswer",

"CallPageCount": 0,

"FilterFlag": 0

}

],

"Id": "00000000-b8c9-0123-def4-00000000",

"Date": "2026-09-11T14:59:01Z",

"Direction": "Outbound",

"Tag": "None",

"PageCount": 3,

"FaxQuality": "Fine",

"Status": "Complete",

"Reference": "00000000-b8c9-0123-def4-00000000",

"JobName": "Invoice_Q3_2026",

"CreatedBy": "jsmith@example.com",

"LoginId": "00000000-9e0f-4123-a456-00000000",

"CreatedVia": "API",

"DocPageCount": 3,

"FilterValue": "None",

"FolderId": "00000000-0000-0000-0000-000000000000",

"FilesAvailable": true,

"ProcessingState": "Processing_Visible"

}

]

}

```

  

---

  

## 13. Valid call, no matching faxes found

  

The request authenticated fine, but none of the supplied `FaxIds` resolved to a fax visible under `ProductId` (wrong product, already-purged fax, or a well-formed-but-unrecognized GUID).

  

```json

{

"Success": true,

"Result": []

}

```

  

---

  

## 14. Failure — authentication rejected

  

```json

{

"Success": false,

"ErrorString": "Authentication failed",

"InfoString": "",

"Result": null

}

```

  

## 15. Failure — invalid/inaccessible `ProductId`

  

```json

{

"Success": false,

"ErrorString": "Product not found",

"InfoString": "",

"Result": null

}

```

  

## 16. Failure — transport/network error (no server round-trip)

  

Synthesized locally by the SDK when the HTTP call itself fails (timeout, DNS failure, connection refused) — never reached the WestFax server. `InfoString` is always exactly `"Error Calling API."` in this case, which is the tell that distinguishes it from an actual server-side rejection.

  

```json

{

"Success": false,

"ErrorString": "The operation has timed out",

"InfoString": "Error Calling API."

}

````
