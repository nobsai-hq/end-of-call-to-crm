# How to test this template

The workflow JSON is written to import into n8n. It has not yet been run against a live Vapi number or a live GoHighLevel sub-account. Use these payloads in a test n8n instance first.

All phone numbers below are fake 555 numbers.

## 1. Import

In n8n, Import from File, pick `workflow.json`. The canvas should show nine nodes, including a sticky note. Turn the workflow on (Active) so the seen-call list can save. Leave `ghl_location_id` as `REPLACE_ME` unless you intentionally want to write to a test sub-account.

Copy the Production URL from the **End of Call** node.

## 2. Simple test payload

Send this twice. Use the same `call_id` both times.

```
POST <production-url>
Content-Type: application/json

{
  "call_id": "test-call-001",
  "caller_phone": "+15550109001",
  "caller_name": "Test Caller",
  "dialed_number": "+15550101000",
  "summary": "Wants a Friday haircut"
}
```

Expected:

- First request: HTTP 200 from the webhook, `is_duplicate` is false, `write_to_ghl` is false while the location ID is still `REPLACE_ME`.
- Second request, same `call_id`: HTTP 200, `is_duplicate` is true, `skip_reason` is `call_id already processed`. No CRM write.

## 3. Missing call ID

Send `{ "caller_phone": "+15550109001" }` with no `call_id`. Expected: `valid` is false, `skip_reason` starts with `missing`, no CRM write.

## 4. Vapi-shaped payload

```
{
  "message": {
    "type": "end-of-call-report",
    "call": { "id": "test-vapi-001" },
    "customer": { "number": "+15550109002", "name": "Alex" },
    "phoneNumber": { "number": "+15550101000" },
    "analysis": {
      "summary": "Asked about Saturday hours",
      "structuredData": { "name": "Alex" }
    }
  }
}
```

Expected: `source_provider` is `vapi`, `call_id` is `test-vapi-001`.

## 5. Twilio-shaped payload

```
{
  "CallSid": "CAxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "CallStatus": "completed",
  "From": "+15550109003",
  "To": "+15550101000"
}
```

Expected: `source_provider` is `twilio`, `call_id` is the CallSid.

## Not tested yet

- A real Vapi assistant posting a live end-of-call report
- A real Twilio number posting a hangup callback
- A real GoHighLevel contact upsert
- Two copies arriving at the exact same moment (the static-data write is not a database transaction)

If you run those tests, add the results here.
