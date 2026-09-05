---
name: browserstack-triage-failed-build
description: Find the most recent BrowserStack Automate build for a project, list its failed
  sessions, and pull the logs needed to diagnose each failure.
api: BrowserStack Automate API
base_url: https://api.browserstack.com
operations:
  - listProjects
  - listBuilds
  - listBuildSessions
  - getSession
  - getSessionLogs
  - getSessionConsoleLogs
  - getSessionNetworkLogs
generated: '2026-09-04'
method: generated
source: openapi/browserstack-*-openapi.yml (operationIds verified against the specs),
  conventions/browserstack-conventions.yml, errors/browserstack-problem-types.yml,
  rate-limits/browserstack-rate-limits.yml
---

# Triage a failed BrowserStack Automate build

Read-only. No operation in this skill changes state.

## Before you start

Authenticate with HTTP Basic: username is the BrowserStack account username, password is the
account access key. Every request in this skill goes to `https://api.browserstack.com`.

## Steps

1. **Resolve the project.** `listProjects` — `GET /automate/projects.json`. Match on `name` and
   keep the numeric `id`.

2. **Find the build.** `listBuilds` — `GET /automate/builds.json`. This listing is
   account-scoped, not project-scoped: there is no path that lists one project's builds. Filter
   client-side on `automation_project_id == <project id>`, then take the most recent by
   `created_at`. Keep the build's `hashed_id` — not its `id`; every downstream path takes the
   hash.

3. **List the sessions.** `listBuildSessions` —
   `GET /automate/builds/{buildId}/sessions.json`, where `buildId` is the build's `hashed_id`.
   Select sessions whose `status` is `failed` or `error`.

4. **Read each failing session.** `getSession` —
   `GET /automate/sessions/{sessionId}.json` using the session's `hashed_id`. The `reason` field
   carries the failure string. The session also carries `video_url`, `browser_console_logs_url`,
   `har_logs_url`, `selenium_logs_url` and `appium_logs_url` — pre-signed artifact links, not
   sub-resources.

5. **Pull the logs you need.** In order of usefulness for a triage:
   - `getSessionConsoleLogs` — `GET /automate/sessions/{sessionId}/consolelogs` for JavaScript
     errors.
   - `getSessionLogs` — `GET /automate/sessions/{sessionId}/logs` for the Selenium command log.
   - `getSessionNetworkLogs` — `GET /automate/sessions/{sessionId}/networklogs` for failed
     requests.

6. **Report.** For each failed session give the session name, the `os`/`os_version`/`browser`/
   `browser_version`/`device` it ran on, the `reason`, and the specific log line that explains
   it. Group sessions that share a root cause; a whole-build failure usually has one.

## Rules

- **Rate limits.** 1,600 requests per 5 minutes per user and 160 per second per IP. A build with
  many sessions plus three log fetches each will hit this. Batch and pace. On a 429 there is no
  `Retry-After` header and no `RateLimit-*` header — back off on a fixed schedule.
- **Error bodies are not JSON.** A 401 from this host returns `text/html` with the body
  `HTTP Basic: Access denied.` Check the status code before parsing.
- **Do not walk upward by name.** A session carries `build_name` and `project_name` as strings,
  not IDs. Traverse downward from project to build to session; never try to resolve a session
  back to its build by matching names.
- **Stay read-only.** Do not call `updateSession`, `deleteSession`, `updateBuild` or
  `deleteBuild` while triaging. Deletion is irreversible — BrowserStack publishes no restore
  operation and no recovery window.
