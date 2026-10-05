# `Fax_GetFaxDescriptionsUsingIds` — Status & Result Reference

  

**Endpoint**: `POST https://api2.westfax.com/REST/Fax_GetFaxDescriptionsUsingIds/json`

**SDK entry points**:

This document enumerates every status/result combination that can realistically occur when calling this endpoint, based on the actual enum definitions and parsing logic in the SDK, not just the "happy path" shown in the integration guide.
  

---

  

## 1. Envelope-Level Outcomes

  

Every response is wrapped in the standard `ApiResult` envelope:

  

```json

{ "Success": bool, "ErrorString": string, "InfoString": string, "Result": ... }

```

  

| #   | `Success` | `Result`                                                   | When this happens                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| --- | --------- | ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| E1  | `true`    | Non-empty array of `FaxDesc` items                         | One or more of the requested `FaxIds` resolved to faxes the caller can see.                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| E2  | `true`    | Empty array `[]`                                           | Call was well-formed and authenticated, but none of the supplied `FaxIds` matched a fax visible under `ProductId` (wrong product, already-purged fax, or bogus-but-well-formed GUID).                                                                                                                                                                                                                                                                                                                           |
| E3  | `true`    | Empty array `[]` — **but only at the typed-wrapper level** | The raw HTTP call succeeded and `Success:true` came back with data, but the SDK's `FaxDesc` conversion threw (see §5.1) and the wrapper's `catch` silently discarded the whole batch, returning `Success:true` with an empty list anyway. **This is a real trap**: `Success:true` does not guarantee `Result.Count` reflects what the server actually returned. Only reachable via `FaxInterface.GetFaxDescriptions` (the raw/string-returning `FaxInterfaceRaw.GetFaxDescriptions` does not do this).          |
| E4  | `false`   | `null` / absent                                            | Server-side rejection — bad credentials, invalid/inaccessible `ProductId`, malformed request, insufficient balance, etc. `ErrorString` is populated.                                                                                                                                                                                                                                                                                                                                                            |
| E5  | `false`   | `null` / absent                                            | **Client-synthesized failure** — no server round-trip happened at all. If the HTTP transport itself errors (timeout, DNS failure, connection refused, TLS failure), `FaxInterfaceRaw.GetResponseStr` fabricates the JSON locally: `{"Success":false,"ErrorString":"<transport error message>","InfoString":"Error Calling API."}` . Indistinguishable from E4 by shape — only `ErrorString` content and `InfoString == "Error Calling API."` hint that it never reached the server. |

  

### 1.1 Realistic `ErrorString` values for E4 (server-side failure)

  

| Scenario                                                              | Typical `ErrorString`                                                                                                                                                                                                                                          |
| --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Bad/expired API key or username+password                              | `"Authentication failed"`                                                                                                                                                                                                                                      |
| `ProductId` doesn't exist or isn't owned by the authenticated account | `"Product not found"`                                                                                                                                                                                                                                          |
| No `FaxIds1..N` supplied at all (empty `items` list)                  | Likely a parameter-validation error (e.g. "FaxIds required") — the SDK does not pre-validate this;  silently posts zero `FaxIds*` keys if the list is empty, so this is entirely a server-side response. |
| One or more `FaxIds` malformed (not a GUID)                           | Request-level validation error from the server.                                                                                                                                                                                                                |
| Account suspended / out of credits                                    | `"Insufficient balance"` (per the integration guide's shared error table — applies account-wide, not fax-specific, but can surface here too).                                                                                                                  |

  

---

  

## 2. Per-Fax (`FaxDesc`) Field Combinations

  

Each element of a successful `Result` array deserializes from the wire `Status`/`FaxQuality`/`Direction` strings into these enums . **Important**: `FaxDesc(FaxDescItem item)`  uses `Enum.Parse(..., true)` (case-insensitive, no fallback) for `Direction`, `FaxQuality`, and `Status` → `FaxStatus`. If the wire value doesn't match a known member name **at all**, parsing throws, which is what triggers envelope outcome **E3** above.

  

### 2.1 `Direction`

  

| Value      | Meaning                   |
| ---------- | ------------------------- |
| `Inbound`  | Fax received on the line. |
| `Outbound` | Fax sent from the line.   |

  

### 2.2 `FaxQuality`

  

| Value    | Meaning              |
| -------- | -------------------- |
| `Fine`   | High resolution.     |
| `Normal` | Standard resolution. |

  

(Confirmed by test assertion: `detail.FaxQuality == "Normal" || detail.FaxQuality == "Fine"`.)

  

### 2.3 `FaxStatus` (the `Status` field)

  

Full enum, with the authors' own inline commentary (`Enums.cs:162-173`):

  

| Value         | Meaning                                                     | Realistically seen for this endpoint?                                                                                            |
| ------------- | ----------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `Complete`    | Job is done dialing; no longer being worked on.             | **Yes — the overwhelming majority of results.** Both inbound-received and outbound-finished faxes land here.                     |
| `Failed`      | "Probably will never see this. Make the same as completed." | Rare but possible — treat like `Complete` for display purposes.                                                                  |
| `Cancelled`   | Canceled (e.g. via `Fax_CancelFax`) before completion.      | Yes, for outbound faxes canceled before finishing.                                                                               |
| `Submitted`   | Production pipeline, pre-processing.                        | Only if you query an ID immediately after send/receive, before the pipeline has picked it up — a race condition against polling. |
| `Production`  | Production pipeline, generally dialing.                     | Same as above — transient, only visible if you query while a job is actively in flight.                                          |
| `Paused`      | Paused, in production pipeline.                             | Possible for outbound jobs paused mid-processing (e.g. account/billing hold).                                                    |
| `UnSubmitted` | Created but not actually submitted for production.          | Edge case — a job record exists but was never queued.                                                                            |
| `UnAssigned`  | "Generally will not see this state."                        | Essentially never; documented as a should-not-happen internal state.                                                             |
| `Submitter`   | Production pipeline (submission stage).                     | Transient/internal — same caveat as `Production`/`Submitted`.                                                                    |
  

A test assertion narrows the *typical, at-rest* range further: `detail.FaxStatus == FaxStatus.Complete || detail.FaxStatus == FaxStatus.Failed`  — i.e., once a fax is done being worked on and you're querying it normally (not mid-flight), expect only `Complete` or `Failed`. The other five values are only realistic if you're polling a job that's still actively moving through the pipeline.

  


---

  

## 3. Per-Call (`FaxCallInfoList[]`) Result Combinations

  

Each call-leg's raw `Result` string is parsed into `CallResult` via `Enum.TryParse<CallResult>(...)` (case-**sensitive**, no `true` flag) with a manual lowercase-fallback `switch` if that fails. This two-tier parse means **the exact casing the server sends determines which code path resolves the value** — a real, easy-to-miss subtlety.

  

### 3.1 `CallResult` enum (the parsed, typed value you actually get)

  

| Value                 | Meaning                                                                                             |
| --------------------- | --------------------------------------------------------------------------------------------------- |
| `Success`             | Call succeeded (inbound receipt or outbound delivery).                                              |
| `Sending`             | Still in progress / waiting to dial.                                                                |
| `Removed`             | Number blocked/removed (area code block, tier block, dedupe, removal list).                         |
| `BadNumber`           | Malformed number.                                                                                   |
| `Busy`                | Busy signal (only reached via **direct** enum match — see 3.2).                                     |
| `NoAnswer`            | No answer, retries exhausted.                                                                       |
| `NoFaxDevice`         | Connected but no fax handshake.                                                                     |
| `Cancelled`           | This call leg was cancelled.                                                                        |
| `Failed`              | Generic failure bucket (csid/schedule/conversion/connection issues that don't have their own case). |
| `InvalidNumber`       | Number format/reachability invalid.                                                                 |
| `Unknown`             | Server sent a `Result` string that matched **nothing**, direct or fallback.                         |
| `ConnectionInterrupt` | Connected, then dropped mid-transmission.                                                           |

  

### 3.2 Wire value → parsed `CallResult` mapping

  

**Tier 1 — direct match** (wire string equals a `CallResult` member name, case-sensitive): if WestFax sends PascalCase values matching the enum exactly (`"Busy"`, `"NoAnswer"`, `"NoFaxDevice"`, `"Cancelled"`, `"Failed"`, `"InvalidNumber"`, `"Unknown"`, `"ConnectionInterrupt"`, `"BadNumber"`, `"Removed"`, `"Success"`, `"Sending"`), it resolves directly to that same-named value — this is the normal/expected path, and matches the casing shown in the integration guide's own examples.
  
**Tier 2 — fallback switch** (only runs if Tier 1 fails, e.g. non-matching casing or a value with no matching enum member at all):
 

| Raw wire value (lowercased, spaces stripped)             | Resolves to     |
| -------------------------------------------------------- | --------------- |
| `inboundfaxreceived`                                     | `Success`       |
| `sent`                                                   | `Success`       |
| `waitingtodial`                                          | `Sending`       |
| `noanswer`                                               | `NoAnswer`      |
| `busy` *(only if casing didn't match Tier 1's `"Busy"`)* | `NoAnswer`      |
| `invalidcsid`                                            | `Failed`        |
| `invalidschedule`                                        | `Failed`        |
| `faxormodemdetected`                                     | `Failed`        |
| `nofaxdevice` *(fallback path)*                          | `Failed`        |
| `fileerror`                                              | `Failed`        |
| `documentconversionerror`                                | `Failed`        |
| `connectionfailure`                                      | `Failed`        |
| `connectioninterrupt` *(fallback path)*                  | `Failed`        |
| `areacodeblocked`                                        | `Removed`       |
| `tierblocked`                                            | `Removed`       |
| `duplicatenumber`                                        | `Removed`       |
| `duplicatenumbermax`                                     | `Removed`       |
| `blocked`                                                | `Removed`       |
| `removallist`                                            | `Removed`       |
| `operatorintercept`                                      | `InvalidNumber` |
| `invalidnumber` *(fallback path)*                        | `InvalidNumber` |
| `cancelled`                                              | `Cancelled`     |
| *(anything else unrecognized)*                           | `Unknown`       |

  

**Practical note**: because Tier 1 is case-sensitive and Tier 2 lowercases, the *same underlying server condition* can map to two different `CallResult` values purely based on the exact casing sent (e.g. `"NoFaxDevice"` → `CallResult.NoFaxDevice` directly, but `"nofaxdevice"` → `CallResult.Failed` via fallback; `"ConnectionInterrupt"` → `CallResult.ConnectionInterrupt` directly, but lowercase → `CallResult.Failed`). Don't assume `CallResult.Failed` vs the more specific value is meaningful without knowing the server's exact casing behavior.

  

### 3.3 `OutboundCallResult` (nullable `DetailedOutboundCallResult`) — outbound only

  

Populated **only when** `Direction == Outbound` **and** the raw `Result` string successfully parses via `DetailedCallResultExtensions.TryFrom` (`Enums.cs:485-504`, wired in at `FaxIdClasses.cs:168`). For `Direction == Inbound`, this field is always `null` — never populated regardless of the raw `Result` value.

  

| Value                                                | Cause                                                                  | Effect                                       | `ToDisplayResult()`        |
| ---------------------------------------------------- | ---------------------------------------------------------------------- | -------------------------------------------- | -------------------------- |
| `Unassigned` (0)                                     | Default/unset.                                                         | —                                            | `"Unknown"`                |
| `Sent`                                               | —                                                                      | Fax successfully sent.                       | `"Sent"`                   |
| `RemovalList`                                        | Number is on a do-not-call/removal list.                               | Call blocked to prevent unwanted contact.    | `"Removed"`                |
| `TierBlocked`                                        | Fax line not permitted to call this tier/region.                       | Fax not attempted.                           | `"Removed"`                |
| `Blocked` *(alias of `TierBlocked`)*                 | Same as `TierBlocked`.                                                 | Same.                                        | `"Removed"`                |
| `InvalidNumber`                                      | Number is invalid.                                                     | Fax not attempted.                           | `"Number Unreachable"`     |
| `InvalidSchedule`                                    | Call schedule invalid.                                                 | Could not call per schedule.                 | `"Failed"`                 |
| `Canceled`                                           | Fax was canceled (before or during the call).                          | —                                            | `"Canceled"`               |
| `DocumentConversionError`                            | Document couldn't convert to fax TIFF.                                 | Fax not attempted.                           | `"Conversion Error"`       |
| `InvalidCsid`                                        | CSID configuration invalid.                                            | Fax not attempted.                           | `"Failed"`                 |
| `WaitingToDial`                                      | Fax is in progress.                                                    | —                                            | `"Sending"`                |
| `Busy` *(alias of `NoAnswer`)*                       | —                                                                      | —                                            | `"Busy"`                   |
| `NoAnswer`                                           | None of the attempts were answered.                                    | Fax not sent.                                | `"No Answer"`              |
| `NoFaxDevice`                                        | No fax connection established (not a fax machine, or poor connection). | Fax not sent.                                | `"No Fax Device"`          |
| `ConnectionInterrupt`                                | Call disconnected before full transmission.                            | Fax may be partially received or not at all. | `"Connection Interrupted"` |
| `FaxOrModemDetected` *(alias of `NoFaxDevice`)*      | Same as `NoFaxDevice`.                                                 | Same.                                        | `"No Fax Device"`          |
| `ConnectionFailure`                                  | Issue connecting to recipient.                                         | Fax not sent.                                | `"Failed"`                 |
| `OperatorIntercept`                                  | Number unreachable/deactivated (possibly recently ported).             | Fax not sent.                                | `"Number Unreachable"`     |
| `UnreachableNumber` *(alias of `OperatorIntercept`)* | Same.                                                                  | Same.                                        | `"Number Unreachable"`     |
| `FileError` *(alias of `DocumentConversionError`)*   | Same.                                                                  | Same.                                        | `"Conversion Error"`       |
| `AreaCodeBlocked` *(alias of `RemovalList`)*         | Same.                                                                  | Same.                                        | `"Removed"`                |
| `DuplicateNumber`                                    | Number appeared multiple times in recipient list.                      | Duplicate removed; only one copy sent.       | —                          |
| `DuplicateNumberMax` *(alias of `DuplicateNumber`)*  | Same.                                                                  | Same.                                        | —                          |

  

`TryFrom` also has its own fallback quirks worth knowing: `"inboundfaxreceived"` → `Sent` (so an inbound call-result string, if it ever ended up here, would map to `Sent`), and the common misspelling `"cancelled"` → `Canceled`.

  

---

  

## 4. `FilterFlag` on each call (bitfield, rarely all set at once)

  

`FaxCallInfo.FilterFlag` is a `[Flags]`-style int, not a single state — realistic combinations are any bitwise union of:

  

| Flag            | Bit   | Meaning                                   |
| --------------- | ----- | ----------------------------------------- |
| `None`          | `0`   | No flags — the default/most common state. |
| `Read`          | `1`   | Read vs. unread.                          |
| `Web_Retrieved` | `2`   | Retrieved via Fax Console.                |
| `FT_Retrieved`  | `4`   | Retrieved via Fax Tools.                  |
| `CFT_Retrieved` | `8`   | Retrieved via CloudFaxToolkit.            |
| `RFC_Retrieved` | `16`  | Retrieved via RFC/FaxConnector.           |
| `Web_Removed`   | `32`  | Deleted via Fax Console.                  |
| `FT_Removed`    | `64`  | Deleted via API/Fax Tools.                |
| `CFT_Removed`   | `128` | Deleted via CloudFaxToolkit.              |
| `RFC_Removed`   | `256` | Deleted via RFC.                          |

`Retrieved` and `Removed` are pre-combined convenience masks (bitwise-OR of their four respective sub-flags), not additional standalone states.
 

---

  

## 5. Known SDK-Level Parsing Failure Modes

  

### 5.1 Silent result-wipe on enum parse failure (typed wrapper only)

  

`FaxInterface.GetFaxDescriptions` wraps the whole per-item conversion in a blanket `try { ... } catch { ret.Result = new List<IFaxId>(); }`. If **any single item** in the batch has a `Status`, `FaxQuality`, or `Direction` string that doesn't match a known enum member (typo, new server-side status not yet added to the SDK, etc.), the **entire batch's** `Result` silently becomes an empty list — while `Success` still reports `true`. This is a realistic and easy-to-hit failure mode whenever the server introduces a new status value the SDK hasn't been updated to parse. The raw string-returning `FaxInterfaceRaw.GetFaxDescriptions` does not have this problem — it hands back the untouched JSON.

  

### 5.2 Mixed-batch requests

  

Because `FaxIds1..N` supports batch lookup, a single call can legitimately mix Inbound and Outbound IDs, and IDs across different statuses (`Complete`, `Cancelled`, etc.) in one `Result` array — this is the normal, expected shape for a multi-ID request, not an edge case.

  

---

  

## 6. Summary: Realistic Top-Level Outcome Matrix

  

| Outcome                            | `Success` | `Result`                     | `FaxStatus` seen                         | `CallResult` seen                                                                                    | Notes                                                                                                        |
| ---------------------------------- | --------- | ---------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Normal inbound fetch               | `true`    | 1 item, `Direction=Inbound`  | `Complete` (typically)                   | `Success` (from `InboundFaxReceived`/`Sent`)                                                         | 1 call info entry.                                                                                           |
| Normal outbound fetch, delivered   | `true`    | 1 item, `Direction=Outbound` | `Complete`                               | `Success`                                                                                            | `OutboundCallResult = Sent`.                                                                                 |
| Outbound fetch, delivery failed    | `true`    | 1 item, `Direction=Outbound` | `Complete` or `Failed`                   | `NoAnswer` / `Busy` / `NoFaxDevice` / `Failed` / `InvalidNumber` / `Removed` / `ConnectionInterrupt` | `OutboundCallResult` gives the detailed reason.                                                              |
| Outbound fetch, still in flight    | `true`    | 1 item                       | `Submitted` / `Production` / `Submitter` | `Sending`                                                                                            | Transient — poll again later.                                                                                |
| Outbound fetch, cancelled          | `true`    | 1 item                       | `Cancelled`                              | `Cancelled`                                                                                          |                                                                                                              |
| Batch, mixed directions/statuses   | `true`    | N items, heterogeneous       | any mix of above                         | any mix of above                                                                                     | Normal for multi-ID requests.                                                                                |
| Valid call, nothing found          | `true`    | `[]`                         | —                                        | —                                                                                                    | IDs didn't resolve under this `ProductId`.                                                                   |
| SDK parse failure on ≥1 item       | `true`    | `[]` (wrapper only)          | —                                        | —                                                                                                    | Server actually returned data; SDK silently dropped it. Only in typed `FaxInterface`, not `FaxInterfaceRaw`. |
| Auth/permission/validation failure | `false`   | `null`                       | —                                        | —                                                                                                    | `ErrorString` populated (`"Authentication failed"`, `"Product not found"`, etc.).                            |
| Transport/network failure          | `false`   | `null`                       | —                                        | —                                                                                                    | Client-synthesized; `InfoString == "Error Calling API."`.                                                    |

  

---

