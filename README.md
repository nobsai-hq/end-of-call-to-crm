# End of Call to CRM (no duplicates)

A free n8n workflow for local businesses and the agencies that serve them. When an AI phone call or a tracked phone call ends, the result lands in GoHighLevel once. A second copy of the same call does not create a second contact.

Built by Bobby Ve, No BS AI.

Live walkthrough of the idea (browser sandbox, no login): https://end-of-call-to-crm.news-ce2.workers.dev

## The problem

Voice providers and phone systems retry webhooks. Vapi can also fire an end-of-call report more than once, for example on a transfer. If your n8n workflow writes to a CRM on every request, you get duplicate contacts, duplicate notes, and a messy demo.

People ask this often: how do I stop n8n from creating a second record when a webhook is retried?

## What it does

1. A webhook receives the end-of-call event and answers HTTP 200 right away, so Vapi or Twilio stop retrying.
2. The workflow reads a call ID, the caller's number, the number that was dialed, a name if one was provided, and a short summary.
3. It remembers call IDs it has already processed. The same call ID is skipped.
4. A new call can upsert a GoHighLevel contact by phone, with tags, and can ping an optional owner webhook.

It accepts three payload shapes:

- A Vapi `end-of-call-report`
- A Twilio call status or hangup callback (`CallSid`, `From`, `To`)
- A simple test body: `{ "call_id", "caller_phone", "caller_name", "summary" }`

## What you need

- An n8n account (cloud or self-hosted).
- Optional: a GoHighLevel Private Integration token with contacts write access, and the location ID for the sub-account.
- Optional: a Vapi assistant or a Twilio number to send the end-of-call webhook.

You can import the workflow and test parsing plus dedupe without any CRM credentials. Leave `ghl_location_id` as `REPLACE_ME` to skip the GHL write.

## Setup steps

1. In n8n, click **Import from File** and pick `workflow.json`.
2. Open the **Settings** node. Set your business name and, if you are writing to GHL, your location ID.
3. On **Upsert GHL Contact**, add a **Header Auth** credential. The name is `Authorization`. The value is `Bearer` plus your GHL token. Keep the token in n8n credentials, never in the workflow itself.
4. Turn the workflow on (Active). Copy the **Production URL** from the **End of Call** node.
5. In Vapi, set that URL as the assistant server URL and listen for `end-of-call-report`. Or in Twilio, paste it as the status or hangup callback.
6. Test with the sample payload in `TESTING.md`. Send it twice with the same `call_id`. The first run should be new. The second should be marked as a duplicate.

## Good to know

- The seen-call list lives in the workflow's own memory. It only saves when the workflow is on (Active), not during a one-off manual test run. If you delete the workflow, the list is gone too.
- This template has not yet been run against a live Vapi number or a live GHL sub-account. The JSON is written to import into n8n and to match the public Vapi and Twilio field names above.
- The public demo page is a browser sandbox. It does not write to GoHighLevel and it does not collect personal data.
- A sister demo of per-agency routing and duplicate blocking is at https://call-flow-demo.news-ce2.workers.dev. A free missed-call text-back template is at https://github.com/nobsai-hq/missed-call-text-back.

## Files

- `workflow.json`: the n8n workflow to import.
- `TESTING.md`: sample payloads and how to test.
- `src/index.js`: the Cloudflare Worker that serves the public demo page.
- `LICENSE`: MIT. Free to use and change.

## Built by

Built by Bobby Ve, No BS AI. I bridge the gap between enterprise AI and entrepreneurs.

---

Want it set up on your number? Text SETUP to [(470) 779-2869](sms:+14707792869)
