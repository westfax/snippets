# Contacts API (SDK reference)

**Base URL pattern:** `https://api3.westfax.com/REST/{MethodName}/json`
**Method:** `POST` (multipart/form-data or form-urlencoded)
**Response Format:** JSON

---

## Authentication

Use the `x-api-key` header on every call. Request a key from your WestFax contact.

| Location | Field | Value |
| --- | --- | --- |
| Header | `x-api-key` | Your API key |
| Body | `ProductId` | Guid — the fax line (required for line-scoped calls; see per-endpoint notes) |
| Body | `Cookies` | `false` (optional, pass if your client requires it) |

Username/password auth (`UserName` / `Password` in the body) is still accepted for legacy SDK integrations, but `x-api-key` is preferred — scoped, revocable, and keeps credentials out of the request body.

**Bad key response:**

```json
{
    "Success": false,
    "ErrorString": "Authorization_Failed_BadUsernamePassword",
    "InfoString": "Authorization Failed (Bad Username or Password)"
}
```

---

## Contact visibility (`Type`)

Contacts have three visibility scopes. The naming is the part that trips people up — especially `Public`.

| `Type` | Scope | Who sees it |
| --- | --- | --- |
| `Global` | Entire account | All users, all fax lines |
| `Public` | One fax line (`ProductId`) | All users of that specific fax line |
| `Private` | One user | Only the owning user (set via `OwnerId`) |

**On the name `Public`:** it does *not* mean "public to the internet" or "public to the whole account." It means "shared among the users of this one fax line." If you want account-wide sharing, use `Global`. If that distinction matters to your UI, surface it explicitly to your end users.

**On `Private` + admin:** when an admin key creates a `Private` contact without specifying `OwnerId`, the contact is owned by the admin and the intended user will never see it. Always set `OwnerId` to the target user's Guid when creating private contacts on someone else's behalf.

---

## No update endpoint

There is **no** `Contact_UpdateContact`. To change any field on an existing contact:

1. Call `Contact_SaveContact` with the new values (creates a new record).
2. Call `Contact_DeleteContact` with the old record's `Id`.

Do it in that order so you don't leave the user without a contact if step 1 fails.

---

## List contacts

### `Contact_GetContactList`

Role-based list — returns what the authenticated key is allowed to see.

```bash
curl --location 'https://api3.westfax.com/REST/Contact_GetContactList/json' \
  --header 'x-api-key: YOUR_API_KEY' \
  --form 'Cookies="false"'
```

**Response (trimmed, one contact per `Type` shown):**

```json
{
    "Success": true,
    "Result": [
        {
            "Id": "11111111-1111-1111-1111-111111111111",
            "EmailValidated": false,
            "Fax": "(555) 100-0001",
            "FirstName": "Jane",
            "LastName": "Doe",
            "CompanyName": "Acme Health",
            "Type": "Global",
            "OwnerId": "00000000-0000-0000-0000-000000000001",
            "OwnerDisplayName": "ACCT-00000001"
        },
        {
            "Id": "22222222-2222-2222-2222-222222222222",
            "Email": "john@example.org",
            "EmailValidated": false,
            "Fax": "5551000002",
            "FirstName": "John",
            "LastName": "Smith",
            "CompanyName": "Example Clinic",
            "Phone": "5551000099",
            "Type": "Public",
            "OwnerId": "00000000-0000-0000-0000-000000000002",
            "OwnerDisplayName": "Shared Line"
        },
        {
            "Id": "33333333-3333-3333-3333-333333333333",
            "EmailValidated": false,
            "Fax": "(555) 100-0003",
            "FirstName": "Sandra",
            "LastName": "Lopez",
            "Type": "Private",
            "OwnerId": "00000000-0000-0000-0000-000000000003",
            "OwnerDisplayName": "user@example.com"
        }
    ]
}
```

**Use the `Id` from this response** as input to `Contact_SaveContact` (for edits, via delete-then-add) and `Contact_DeleteContact`.

### Other list helpers

| Use case | Method | Notes |
| --- | --- | --- |
| All contacts (full dump) | `Contact_GetAllContactList` | |
| Send-fax picker (paged) | `Contact_GetContactList_ForSend_PagedSearch` | Requires `ProductId`. `MethodParams`: `page`, `count`, optional `searchString`, `filters`, `includeOnlyContactsWithEmail` |
| Admin paged search | `Contact_GetContactList_ForAdmin_PagedSearch` | See below |

---

## Admin paged search

### `Contact_GetContactList_ForAdmin_PagedSearch`

Paged list with optional text filter and visibility filter. Indexed `MethodParams1..N` form keys, each value is a JSON object `{"Name":"...","Value":"..."}`.

| `Name` | `Value` | Required |
| --- | --- | --- |
| `page` | Page number (1-indexed) | Yes |
| `count` | Page size | Yes |
| `searchString` | Free-text filter | No |
| `filters` | Comma-separated visibility names: `Global`, `Public`, `Private` | No |

```bash
curl --location 'https://api3.westfax.com/REST/Contact_GetContactList_ForAdmin_PagedSearch/json' \
  --header 'x-api-key: YOUR_API_KEY' \
  --form 'Cookies="false"' \
  --form 'MethodParams1="{\"Name\":\"page\",\"Value\":\"1\"}"' \
  --form 'MethodParams2="{\"Name\":\"count\",\"Value\":\"50\"}"' \
  --form 'MethodParams3="{\"Name\":\"filters\",\"Value\":\"Global,Public\"}"' \
  --form 'MethodParams4="{\"Name\":\"searchString\",\"Value\":\"acme\"}"'
```

**Response:** `ApiResult<Page<ContactItem>>` — `Result.Items` holds the contacts, paging metadata on `Result`.

---

## Create a contact

### `Contact_SaveContact`

Serialize the contact as a JSON string in the `CrmContact` form field. Leave `Id` empty to create; the server will assign one.

**Form fields:**

| Field        | Required | Notes                                            |
| ------------ | -------- | ------------------------------------------------ |
| `ProductId`  | Yes      | The fax line context                             |
| `CrmContact` | Yes      | JSON-serialized `ContactItem` (see fields below) |
| `Cookies`    | No       | `false` if your client needs it                  |

**Success response (same for all three types):**

```json
{ "Success": true, "Result": true }
```

### Create a `Public` contact (shared on a fax line)

Every user of the line identified by `ProductId` will see it.

```bash
curl --location 'https://api3.westfax.com/REST/Contact_SaveContact/json' \
  --header 'x-api-key: YOUR_API_KEY' \
  --form 'Cookies="false"' \
  --form 'ProductId="0000000000-a8ac-4a77-8007-0000000000"' \
  --form 'CrmContact="{\"FirstName\":\"John\",\"LastName\":\"Test32\",\"Fax\":\"3032993339\",\"CompanyName\":\"Acme, LLC\",\"Email\":\"JTest@acme.org\",\"Phone\":\"3332223333\",\"Type\":\"Public\"}"'
```

### Create a `Private` contact (owned by a specific user)

Admins creating on behalf of a user must set `OwnerId` to that user's Guid — otherwise the contact gets owned by the admin and the target user won't see it.

```bash
curl --location 'https://api3.westfax.com/REST/Contact_SaveContact/json' \
  --header 'x-api-key: YOUR_API_KEY' \
  --form 'Cookies="false"' \
  --form 'ProductId="0000000000-a8ac-4a77-8007-0000000000"' \
  --form 'CrmContact="{\"FirstName\":\"John\",\"LastName\":\"Test32\",\"Fax\":\"3032993339\",\"CompanyName\":\"Acme, LLC\",\"Email\":\"JTest@acme.org\",\"Phone\":\"3332223333\",\"Type\":\"Private\",\"OwnerId\":\"a1b2c3d4-e5f6-7890-abcd-ef1234567890\"}"'
```

If the end user is authenticating with their own key, `OwnerId` can be omitted — it defaults to the caller.

### Create a `Global` contact (account-wide)

```bash
curl --location 'https://api3.westfax.com/REST/Contact_SaveContact/json' \
  --header 'x-api-key: YOUR_API_KEY' \
  --form 'Cookies="false"' \
  --form 'ProductId="0000000000-a8ac-4a77-8007-0000000000"' \
  --form 'CrmContact="{\"FirstName\":\"John\",\"LastName\":\"Test32\",\"Fax\":\"3032993339\",\"CompanyName\":\"Acme, LLC\",\"Email\":\"JTest@acme.org\",\"Phone\":\"3332223333\",\"Type\":\"Global\"}"'
```

---

## Update a contact (delete-then-add)

There is no update endpoint. To "edit" a contact, create a new one with the corrected fields and delete the old one.

```bash
# 1. Create the replacement
curl --location 'https://api3.westfax.com/REST/Contact_SaveContact/json' \
  --header 'x-api-key: YOUR_API_KEY' \
  --form 'Cookies="false"' \
  --form 'ProductId="0000000000-a8ac-4a77-8007-0000000000"' \
  --form 'CrmContact="{\"FirstName\":\"John\",\"LastName\":\"Smith\",\"Fax\":\"3032993339\",\"CompanyName\":\"Acme, LLC\",\"Email\":\"jsmith@acme.org\",\"Phone\":\"3332223333\",\"Type\":\"Global\"}"'

# 2. Delete the old record (Id from the list endpoint)
curl --location 'https://api3.westfax.com/REST/Contact_DeleteContact/json' \
  --header 'x-api-key: YOUR_API_KEY' \
  --form 'Cookies="false"' \
  --form 'CrmContact="{\"Id\":\"22222222-2222-2222-2222-222222222222\",\"Type\":\"Global\"}"'
```

If step 1 fails, don't run step 2. The old contact is still there and still usable.

---

## Delete a contact

### `Contact_DeleteContact`

Get the `Id` from any list endpoint (`Contact_GetContactList`, `Contact_GetAllContactList`, or the paged search variants). Include the `Type` of the contact being deleted.

```bash
curl --location 'https://api3.westfax.com/REST/Contact_DeleteContact/json' \
  --header 'x-api-key: YOUR_API_KEY' \
  --form 'Cookies="false"' \
  --form 'CrmContact="{\"Id\":\"0000000000-0000-0000-0000-0000000000\",\"Type\":\"Global\"}"'
```

**Response:**

```json
{ "Success": true, "Result": true }
```

For `Private` contacts: authenticate with the owning user's key, or use an admin key where the contact's `OwnerId` is already set to that user.

---

## `ContactItem` fields

JSON property names match the `ContactItem` object:
***Only field that is required is the `Fax` field.***

| Field              | Type    | Notes                                                            |
| ------------------ | ------- | ---------------------------------------------------------------- |
| `Id`               | `Guid`  | Omit / empty on create; required on delete                       |
| `Title`            | string  | e.g. "Dr.", "Detective"                                          |
| `FirstName`        | string  |                                                                  |
| `LastName`         | string  |                                                                  |
| `CompanyName`      | string  |                                                                  |
| `Email`            | string  |                                                                  |
| `Fax`              | string  | Accepted formats: `3032993339`, `303-299-3339`, `(303) 299-3339` |
| `MobilePhone`      | string  |                                                                  |
| `Phone`            | string  |                                                                  |
| `Type`             | string  | `Global` \| `Public` \| `Private`                                |
| `OwnerId`          | `Guid?` | Required for admin-created `Private` contacts                    |
| `OwnerDisplayName` | string  | Read-only in responses                                           |
| `EmailValidated`   | bool    | Read-only in responses                                           |

---

## Error responses

All errors follow the standard envelope:

```json
{
    "Success": false,
    "ErrorString": "Short machine-readable code",
    "InfoString": "Human-readable explanation"
}
```

Common ones:

| `ErrorString` | Likely cause |
| --- | --- |
| `Authorization_Failed_BadUsernamePassword` | Bad or revoked `x-api-key` |
| `Authorization_Failed_UserRoleLevel` | Key lacks permission for this endpoint or `ProductId` |

Always check `Success` before reading `Result`.
