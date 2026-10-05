# `Fax_GetF2EFaxDescriptions_PagedSearch`

  

Paged/filtered list of fax descriptions for a Fax2Email product.

  

- **URL:** `https://api2.westfax.com/REST/Fax_GetF2EFaxDescriptions_PagedSearch/json`

- **Method:** `POST` (form body, `application/x-www-form-urlencoded`)

- **Auth:** `UserName` + `Password`, **or** `ApiKey`

- **Required:** `ProductId` (Guid), `MethodParams1` = `page`, `MethodParams2` = `count`

  

## Body fields (top level)

  

| Field              | Required | Notes                                                        |
| ------------------ | -------- | ------------------------------------------------------------ |
| `UserName`         | one of   | Use with `Password`                                          |
| `Password`         | one of   |                                                              |
| `ApiKey`           | one of   | Replaces user/pass                                           |
| `ProductId`        | yes      | Guid of the F2E product                                      |
| `Cookies`          | no       | `false` (default)                                            |
| `FaxDirection`     | yes      | `Inbound` or `Outbound`                                      |
| `StartDate`        | no       | Local date/time, e.g. `4/1/2026 12:00:00 AM`                 |
| `EndDate`          | no       | Local date/time                                              |
| `FilterList1..N`   | no       | Status filter strings (e.g. `Success`, `Failure`, `Busy`)    |
| `CategoryList1..N` | no       | Category name strings                                        |
| `StringParams1..N` | no       | SDK also re-sends categories here when `CategoryList` is set |

  

## `MethodParams1..N` (each value is JSON `{"Name":"...","Value":"..."}`)

  

| `Name`              | `Value`                             | Required |
| ------------------- | ----------------------------------- | -------- |
| `page`              | `1`, `2`, ...                       | yes      |
| `count`             | items per page (e.g. `25`)          | yes      |
| `ownerLoginId`      | Guid — restrict to one user's faxes | no       |
| `numberMatch`       | digits to match on number           | no       |
| `stringMatch`       | free text to match                  | no       |
| `matchType`         | `Or` (default) or `And`             | no       |
| `IncludeCategories` | `true`                              | no       |


## Response
  

`ApiResult<Page<FaxDescItem>>`:

  

```json

{

"Success": true,

"Result": {

"TotalItems": 137,

"PageNumber": 1,

"ItemsPerPage": 25,

"Items": [ { "FaxId": "...", "Direction": "Inbound", "...": "..." } ]

}

}

```

  

---

  

## Curl — minimal (user/pass, inbound, page 1)

  

```bash

curl -X POST "https://api2.westfax.com/REST/Fax_GetF2EFaxDescriptions_PagedSearch/json" \

--data-urlencode "UserName=you@example.com" \

--data-urlencode "Password=YourPassword" \

--data-urlencode "ProductId=00000000-0000-0000-0000-000000000000" \

--data-urlencode "Cookies=false" \

--data-urlencode "FaxDirection=Inbound" \

--data-urlencode 'MethodParams1={"Name":"page","Value":"1"}' \

--data-urlencode 'MethodParams2={"Name":"count","Value":"25"}' 

```

  

## Curl — ApiKey instead of user/pass

  

```bash

curl -X POST "https://api2.westfax.com/REST/Fax_GetF2EFaxDescriptions_PagedSearch/json" \

--data-urlencode "ApiKey=YOUR_API_KEY" \

--data-urlencode "ProductId=00000000-0000-0000-0000-000000000000" \

--data-urlencode "FaxDirection=Inbound" \

--data-urlencode 'MethodParams1={"Name":"page","Value":"1"}' \

--data-urlencode 'MethodParams2={"Name":"count","Value":"25"}'

```

  


  

## Curl — filter by status + categories, owner-scoped

  

```bash

curl -X POST "https://api2.westfax.com/REST/Fax_GetF2EFaxDescriptions_PagedSearch/json" \

--data-urlencode "ApiKey=YOUR_API_KEY" \

--data-urlencode "ProductId=00000000-0000-0000-0000-000000000000" \

--data-urlencode "FaxDirection=Inbound" \

--data-urlencode "FilterList1=Success" \

--data-urlencode "FilterList2=Failure" \

--data-urlencode "CategoryList1=Billing" \

--data-urlencode "CategoryList2=Legal" \

--data-urlencode "StringParams1=Billing" \

--data-urlencode "StringParams2=Legal" \

--data-urlencode 'MethodParams1={"Name":"page","Value":"1"}' \

--data-urlencode 'MethodParams2={"Name":"count","Value":"25"}' \

--data-urlencode 'MethodParams3={"Name":"ownerLoginId","Value":"11111111-2222-3333-4444-555555555555"}' \

--data-urlencode 'MethodParams4={"Name":"matchType","Value":"And"}' \

--data-urlencode 'MethodParams5={"Name":"IncludeCategories","Value":"true"}'

```

  

## Notes / gotchas

  

- `MethodParamsN` values must be JSON (not just the `Value`) — quote them in single quotes so the shell doesn't eat the JSON.

- `FilterList`, `CategoryList`, `StringParams` are **1-indexed** (`FilterList1`, `FilterList2`, ...).

- If `StartDate`/`EndDate` are omitted, the API applies its default window.

- Page metadata is on `Result` (`TotalItems`, `PageNumber`, `ItemsPerPage`) — use it to iterate pages.

- `FaxDirection` is required at the top level, not in `MethodParams`.
