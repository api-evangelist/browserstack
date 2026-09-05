---
name: browserstack-rotate-access-key
description: Rotate the BrowserStack Automate access key. Irreversible and immediately
  breaking — requires explicit human confirmation before execution.
api: BrowserStack Automate API
base_url: https://api.browserstack.com
operations:
  - recycleAccessKey
generated: '2026-09-04'
method: generated
source: openapi/browserstack-accesskey-api-openapi.yml,
  conventions/browserstack-conventions.yml (reversibility),
  agentic-access/browserstack-agentic-access.yml
---

# Rotate the BrowserStack access key

**Stop and get explicit human approval before calling this.** It is the single most destructive
operation on BrowserStack's published API surface.

## What it does

`recycleAccessKey` — `PUT /automate/recycle_key.json` — issues a new account access key and
invalidates the current one immediately.

## Why it needs a human

- **Irreversible.** BrowserStack publishes no way to recover the previous key value and no
  operation to undo the rotation.
- **No grace period.** No overlap window is documented. The old key stops working at once.
- **Blast radius is the whole account.** The access key is account-wide, not scoped. Every CI
  pipeline, every SDK configuration, every running Local tunnel and every `BROWSERSTACK_ACCESS_KEY`
  environment variable anywhere in the organisation breaks until it is updated with the new value.
- **No idempotency.** There is no `Idempotency-Key` header on this or any BrowserStack
  operation. A retry after a timeout rotates the key a *second* time and invalidates the key the
  first call just issued. If a call times out, verify the current key before retrying — never
  retry blindly.

## Procedure

1. Confirm with the human, naming the account and the fact that the old key dies immediately.
2. Establish where the current key is stored — CI secrets, developer machines, tunnel configs —
   and confirm the human can update all of them.
3. Call `PUT /automate/recycle_key.json`.
4. Capture the new key from the response and hand it over through whatever secure channel the
   human specifies. Never write it to a log, a file in the repository, or a chat message.
5. Verify with a read — `getPlan` (`GET /automate/plan.json`) using the new credential.

## Rules

- Never call this as part of a broader automated flow.
- Never call it to "test" authentication.
- A 401 from this host returns `text/html` with `HTTP Basic: Access denied.`, not JSON.
