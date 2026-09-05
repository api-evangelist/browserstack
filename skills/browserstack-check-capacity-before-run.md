---
name: browserstack-check-capacity-before-run
description: Check BrowserStack Automate parallel-session capacity and confirm a target
  browser/device combination is supported before launching a test run.
api: BrowserStack Automate API
base_url: https://api.browserstack.com
operations:
  - getPlan
  - getBrowsers
generated: '2026-09-04'
method: generated
source: openapi/browserstack-plan-api-openapi.yml,
  openapi/browserstack-browsers-api-openapi.yml,
  rate-limits/browserstack-rate-limits.yml
---

# Check capacity and platform support before a run

Read-only. Two calls, both cheap, both worth making before you queue work.

## Steps

1. **Check capacity.** `getPlan` — `GET /automate/plan.json`. Returns:
   - `parallel_sessions_running` — how many are live right now
   - `parallel_sessions_max_allowed` — the account ceiling
   - `team_parallel_sessions_max_allowed` — the team ceiling
   - `queued_sessions` and `queued_sessions_max_allowed`
   - `automate_plan` — the plan name

   Available headroom is `parallel_sessions_max_allowed - parallel_sessions_running`. If it is
   zero and `queued_sessions` is at `queued_sessions_max_allowed`, a new session will be
   rejected rather than queued. Say so rather than launching and failing.

2. **Confirm the platform exists.** `getBrowsers` — `GET /automate/browsers.json`. Returns every
   supported combination as `os`, `os_version`, `browser`, `browser_version`, `device`,
   `real_mobile`. Match the requested target exactly against this list. `Browser` is a value
   object with no ID — the combination itself is the key.

   The same field vocabulary appears on a `Session`, so a combination confirmed here can later
   be matched against what a session actually ran on.

3. **Report.** State the headroom, whether the requested combination is supported, and — if it
   is not — the nearest supported alternatives from the returned list. Never substitute a
   platform silently.

## Rules

- `getBrowsers` returns a large payload and changes rarely. Cache it; do not refetch per test.
- Rate limits are 1,600 requests per 5 minutes per user and 160 per second per IP, with no
  `Retry-After` header on a 429.
- A 401 from this host returns `text/html`, not JSON.
- Both operations are reads. Neither consumes test minutes.
