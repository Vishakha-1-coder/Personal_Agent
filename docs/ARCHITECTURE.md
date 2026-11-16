# ARCHITECTURE: PA (Personal Agent)

Status: draft 0.1, 2026-11-16. Locks the blueprint for release R0. Change this
file before changing code that contradicts it.

## 1. Components and dependency rule
```text
Laptop: coding agent --hooks/MCP--> pa-companion --WSS (outbound only)--+
Phone/Laptop browser (PWA) <------ HTTPS + WebSocket ------> pa-api <---+
pa-api -> pa-domain (rules, policy, evidence; no I/O)
pa-api -> pa-db (sqlx) | pa-llm | pa-connectors | pa-scheduler | pa-config
```
`pa-domain` depends on nothing. Every other crate depends inward on it and
implements its traits. Only binaries (`pa-api`, `pa-companion`) wire crates
together.

## 2. Configuration (three layers, one reader)
1. `config/default.toml`: non-secret tunables (model tier per task type,
   reminder defaults, retention days, budgets, rate limits, approval expiry).
2. Environment `PA_*` overrides deployment values; secrets via `PA_*_FILE`.
3. Database `settings`: per-user and per-org business settings.
Only `pa-config` reads layers 1 and 2 and returns one typed, validated `Config`.
Startup fails on a missing or invalid required value.

## 3. Data model (PostgreSQL 18, all tables: `id uuidv7`, `org_id`, RLS)
| Table | Key columns |
| --- | --- |
| `orgs`, `users` | email (encrypted + blind index), role (`user`,`org_admin`), time zone |
| `devices` | user, kind (`companion`,`phone`,`browser`), public key, revoked_at |
| `settings` | user, version, JSONB (shift, reminder gaps, status times/templates, channels, consent) |
| `reminders`, `reminder_events` | schedule rule, next_run_at, dedupe key (exactly-once delivery) |
| `attendance_records` | user, kind, reply, occurred_at |
| `tasks`, `time_entries` | PA's own task list and time ledger |
| `agent_events` | session id, type, redacted summary, occurred_at |
| `journal_entries`, `evidence` | entry text; evidence = typed reference (commit, event, reply) |
| `drafts`, `draft_statements` | kind, status, text; statement with evidence ids |
| `approvals` | action kind, exact payload, expires_at, decision, decided_by device, passkey result |
| `connectors`, `oauth_tokens` | provider, scopes, token (AES-256-GCM) |
| `jobs` | queue rows (`FOR UPDATE SKIP LOCKED`), attempts, run_at |
| `audit_log` | append-only: redacted input, proposal, approver, result |
| `llm_usage` | user, tier, tokens, cost |
| `consents` | user, purpose, version, granted_at |

## 4. API contracts (`/api/v1`, JSON `camelCase`, timestamps RFC 3339)
Error envelope for every failure: `{"error":{"code":"approval_expired","message":"...","requestId":"..."}}`.

| Endpoint | Request -> response |
| --- | --- |
| `GET /auth/google/login`, `GET /auth/google/callback` | OIDC redirect -> session cookie (HttpOnly, Secure, SameSite=Lax) |
| `GET /me` | -> `{id, email, role, timeZone, consents[]}` |
| `PUT /settings` | `{shift:{start,lengthMinutes}, reminders:{snoozeMinutes}, statuses:[{kind,time,template}], channels:{chatSpace}}` -> `{version}` |
| `POST /attendance/replies` | `{kind:"timeIn"\|"timeOut", reply:"done"\|"late"\|"leave"\|"moreTime", note?}` -> `{recordId, nextReminderAt?}` |
| `GET /drafts?kind=` | -> `[{id, kind, text, statements:[{text, evidenceIds[]}], status}]` |
| `PATCH /drafts/{id}` | `{text}` -> draft (statements re-validated) |
| `POST /drafts/{id}/approve` | `{}` -> `202 {approvalId}`; posting happens after policy check |
| `GET /approvals?status=pending` | -> `[{id, kind, command, repo, branch, expiresAt}]` |
| `POST /approvals/{id}/decision` | `{decision:"approve"\|"deny", passkeyAssertion?}` -> `{status}` |
| `POST /devices/pair` | `{pairingCode, publicKey}` -> `{deviceId}` |
| `GET /ws/companion` (WebSocket) | messages: `hello`, `event`, `approvalRequest`, `approvalDecision`, `taskDispatch`, `taskResult`; each signed with the device key |

## 5. Authentication and approval security
* Sign-in: Google OpenID Connect (Workspace), server-side session; MFA from Google.
* Companion pairing: browser shows a one-time code; the companion sends its
  public key with the code; the server stores the key. All companion messages
  are signed (Ed25519) and carry a nonce and expiry (replay rejected).
* Approvals: single-use, expire after the configured time (default deny),
  require a passkey (WebAuthn) assertion for `commit` and `push`. Force-push
  and delete are never approvable remotely.

## 6. Approval relay (core differentiator)
Agent hook (`PreToolUse` on a shell command matching `git commit`/`git push`)
-> companion creates a signed request -> server stores `approvals` row, pushes
to the phone -> person decides -> server returns the decision -> hook allows
or denies. No answer before expiry: deny and fall back to the terminal prompt.
Hook timeout and response format must be confirmed in the spike (PLAN phase 0).

## 7. Drafts, evidence and the LLM gateway
1. Scheduler collects evidence (journal, calendar, replies, git summaries).
2. `pa-llm` picks a tier from config (task type -> small/mid/frontier), calls the
   provider behind one trait, requests structured output (statement + evidence ids).
3. `pa-domain` validator rejects any statement without evidence or with a
   number or name not found in its evidence; one retry, then template fallback.
4. Budgets are checked before each call and recorded in `llm_usage`.
5. Untrusted text is wrapped and labelled as data inside prompts.

## 8. Integrations
`pa-connectors` exposes small capability traits (`ChatPoster`,
`CalendarReader`, `MailDrafter`, later `TaskSource`, `AttendanceSink`). R0
implements Google Chat (user OAuth, text only). Propeak and HRMS later
implement the same traits; the orchestrator does not change.

## 9. State management
The server is the source of truth. The web app keeps a server-state cache,
refreshed by WebSocket events; no global client store beyond the session.
The companion keeps only a small local queue of unsent events.

## 10. Operations
`tracing` with OpenTelemetry export; metrics for reminder lateness, approval
latency, LLM cost and errors. One-command deploy; Docker images; automatic
backups with a rehearsed restore. Secrets never in images or the repository.
