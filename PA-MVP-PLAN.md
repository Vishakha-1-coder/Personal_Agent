# PA (Personal Agent): MVP plan and checklist

Status: draft 1, 2026-11-15. This is the working plan. Tick items as they
land. Anything marked **(needs approval)** is blocked on the CEO, IT or an
external API and is tracked in section 11.

## 1. What PA is

PA is an orchestrator, not a replacement for Claude Code, Cursor or
Antigravity. Those tools write code. PA is the chief of staff around them:
it knows your day, tracks what you and your agents did, reminds you,
drafts the paperwork, and asks you before it does anything outward-facing.

Principles (every feature is checked against these):

1.  **Human in the loop.** PA drafts, you approve. Nothing is sent, posted,
    committed or submitted without an explicit approval.
2.  **Local-first.** Code and full agent chats stay on the laptop. Only
    redacted summaries reach the server, and only if the user opted in.
3.  **Narrow and excellent.** A few office jobs done reliably beat many done
    badly.
4.  **Configurable, not coded.** Times, formats and channels are per-user
    settings from onboarding.
5.  **Fail safe.** An approval that times out means **deny**.

## 2. The two spaces

| Space | Purpose | Default name | Other name ideas |
| --- | --- | --- | --- |
| Office | Work: attendance, statuses, meetings, agents, standup | **Desk** | Workbench, Studio, Shift |
| Personal | Life: reminders, notes, tracker, news | **Home** | Me, Sanctuary, Life |

Rules: the two spaces have separate data and separate access. An employer
admin can never see Home data. The name is a UI label only, so it is cheap
to change later: decide it by day 3.

- [ ] Pick the two space names

## 3. MVP feature checklist

Priority: **P0** = must be in the 2-week pilot, **P1** = only if P0 is
done and stable, **Later** = after the pilot (section 10).

### 3.1 Onboarding (P0)

The PA interviews the user once; answers become per-user settings.

- [ ] Role, domain and interests (drives the news digest)
- [ ] Shift start, shift length, working days, time zone, grace time
- [ ] Reminder times, snooze length, quiet hours
- [ ] Status times and format (template) for the three daily statuses
- [ ] Channels: Google Chat space for statuses, push, email
- [ ] What PA may send to the server (opt-in screen)
- [ ] All settings editable later, plus "pause PA" and "delete my data"

### 3.2 Reminders (P0)

- [ ] Time-in reminder with **[Done] [I'll be late] [On leave / other]**
- [ ] Time-out reminder at shift end with **[Done] [Need more time,
      remind in N min]** (N configurable, default 60)
- [ ] Custom reminders (one-off and recurring), both spaces
- [ ] Durable scheduling: survives restarts, no double sends, time zone
      and daylight saving safe
- [ ] Skip reminders on declared leave or holiday (manual at first)
- [ ] PA only **reminds**; it records nothing in the HR system until the
      HRMS integration is approved **(needs approval)**

### 3.3 Three daily statuses: create, review, send (P0)

- [ ] Morning: "what I will do today" (from tickets, calendar, yesterday)
- [ ] Before lunch (about 13:00): "what I did so far" (from git, agent
      activity, calendar)
- [ ] End of day: "what I did today" in the user's format
- [ ] Flow: PA drafts, the user edits on the phone or web, approves, PA
      posts to Google Chat as the user
- [ ] Per-user template, times and tone are configurable
- [ ] Propeak posting is **not** in the MVP **(needs approval)**

### 3.4 Coding-agent connection (P0)

- [ ] Local companion (Go binary, tray) with device pairing, outbound-only
      secure connection
- [ ] One-command install of hooks and MCP config for Claude Code
- [ ] Events to phone: task finished, agent waiting for you, error
- [ ] **Approval relay** for `git commit` and `git push` from the phone,
      signed and single-use, short expiry, timeout means deny
- [ ] Local redaction of secrets before any summary leaves the laptop
- [ ] PA's MCP server tools: `get_my_tasks`, `report_progress`,
      `ask_approval`, `notify_user`
- [ ] Cursor and Antigravity: MCP plus git watching (P1)
- [ ] PA to agent: a phone-approved task is run by the companion with the
      agent's headless mode in a limited permission mode (P1)

### 3.5 Meetings, email and MOM (P1, all with approval)

- [ ] Read the calendar: what meetings, when, join links, reminders
- [ ] Draft meeting invites and scheduling emails (create only after
      approval)
- [ ] Draft emails with Gmail drafts; **send only after approval**
- [ ] MOM: the user supplies notes or a transcript, PA drafts the minutes
      and action items, the user approves, PA emails them
- [ ] Standup points: yesterday, today, blockers, from real activity

### 3.6 Personal tracker and journal (P1)

- [ ] Automatic daily log: commits, agent sessions, meetings, statuses
- [ ] End-of-day and weekly "what I actually did" report for the user only
- [ ] Used as the source for statuses and (later) timesheets

### 3.7 Daily news digest (P1)

- [ ] News from RSS and a search API, filtered by the user's domain and
      interests
- [ ] Deduplicated, summarised, delivered once a day at a set time
- [ ] Every item links to its source

### 3.8 Phone calls (see section 7)

- [ ] P1: outbound escalation call with spoken alert and keypad approval
- [ ] Later: two-way voice conversation

### 3.9 Access control (P0, small)

- [ ] Google SSO (Workspace), MFA from Google
- [ ] Roles: **user** and **org admin** only. Admin manages users and
      integrations and sees usage counts, never content
- [ ] Postgres row-level security on every table
- [ ] Per-device revocable keys for the companion
- [ ] Audit log of every action PA took or proposed

> Update 2026-11-16: the language is now **Rust** (the owner's decision),
> not Go. See the Google Doc "PA (Personal Agent) - Project Plan" for the
> current plan; the table below is superseded where it says Go.

## 4. Tech stack (decided)

| Layer | Choice | Reason |
| --- | --- | --- |
| Backend | **Go** | Team skill, fast, one static binary |
| Database | **PostgreSQL 18** | Relational, row-level security, JSONB for settings |
| Jobs and scheduler | Postgres-backed queue (River) | Durable reminders without a second system |
| Realtime | WebSocket from companion and apps | Outbound only, no open ports on laptops |
| Companion | Go binary with tray | Same language, Windows, Linux and Mac |
| Web and phone | PWA (React) with Web Push | One codebase for laptop and phone; native apps later |
| Agent connection | MCP server (official Go SDK) plus Claude Code hooks | Supported extension points |
| Integrations | Google Chat, Calendar, Gmail via OAuth (user-delegated) | Posts and sends as the user |
| Telephony | Twilio, or an India provider (Exotel, Plivo) | Outbound calls and keypad input |
| Secrets | Env or Vault, envelope-encrypted OAuth tokens | Tokens never in plaintext |
| Observability | OpenTelemetry, Prometheus, Grafana, structured logs | No PII in logs |
| Deploy | Docker Compose behind Caddy (TLS) | Simple, self-hostable, later Kubernetes |

Add later, only on evidence: Redis, pgvector for memory search,
Kubernetes. Rust is not needed.

## 5. LLM design (the questions you asked)

Think of the LLM layer as a small, boring, well-guarded service, not a
chatbot.

### 5.1 Models and routing

- **Model router: yes, but a simple rules-based one.** Each task type is
  mapped to a tier in config. Do not build a learned router for the MVP.

| Tier | Used for | Model class |
| --- | --- | --- |
| Small and cheap | Intent classification, extracting fields, formatting statuses, summarising one commit list, news triage | Haiku-class (`claude-haiku-5-5`) |
| Mid | Status drafts from several sources, MOM, email drafts, news digest | Sonnet-class (`claude-sonnet-5-5`) |
| Frontier | Rare: ambiguous planning, long cross-project summaries, anything the mid tier fails validation on (escalate once) | Opus-class (`claude-opus-5-5`) |

- Put models behind **one internal interface** so the provider can change
  (a second provider is a config switch, not a rewrite).
- Check current prices and rate limits when you build; they change.
- Per-user daily token budget and a global monthly cap, with alerts.

### 5.2 Do we need "GenAI for classification"?

A small LLM with structured output (a JSON schema) is enough: classify
"what does this message want" into a fixed list of intents. No fine-tuning
and no custom ML model for the MVP. Add an eval set of about 50 labelled
examples so you can tell when a model change breaks it.

### 5.3 Semantic cache: not for the MVP

PA's answers are personal and time-dependent (your commits today, your
meetings), so cache hits would be rare and a wrong hit could leak or show
stale data. Use the provider's **prompt caching** for the long, repeated
system prompt and templates instead. Revisit a semantic cache only for
shared content such as the news digest (one summary per article, shared
across users).

### 5.4 Guardrails (P0)

- [ ] **Untrusted content rule:** anything read from emails, chats,
      tickets, web pages or agent transcripts is data, never instructions.
      It is wrapped and labelled as such in prompts.
- [ ] **Action policy engine** between the LLM and the world: the LLM can
      only *propose* actions from an allowlist. Each action has a risk
      tier: read (auto), draft (auto), send or post or commit (needs
      approval), delete or force-push (never remote)
- [ ] **Structured output validation:** every LLM result is parsed against
      a schema; invalid output is retried once, then escalated or failed
- [ ] **Secret and PII scanner** on anything leaving the laptop and on
      anything about to be posted
- [ ] **Budgets and rate limits** per user and per action
- [ ] **Audit trail** of prompt inputs (hashed or redacted), the proposed
      action, who approved it, and the result
- [ ] Prompt-injection test cases in the automated tests

### 5.5 Memory (keep it simple)

- [ ] Stored preferences and facts in Postgres (templates, tone, projects),
      visible and editable by the user
- [ ] No vector database in the MVP

## 6. Architecture in one picture

```text
 Laptop                                   Server (Go + Postgres)
┌──────────────────────────┐             ┌─────────────────────────────┐
│ Claude Code / Cursor /   │             │ API + WebSocket gateway     │
│ Antigravity              │             │  ├ Scheduler (reminders)    │
│   │ hooks + MCP          │  outbound   │  ├ Orchestrator + policy    │
│   ▼                      │ WSS / TLS   │  │   engine + LLM router    │
│ Companion (redaction,    ├────────────▶│  ├ Integrations: Google     │
│ git watcher, approvals)  │             │  │   Chat, Calendar, Gmail  │
└──────────────────────────┘             │  ├ Notifier: Push, Voice    │
                                         │  └ Postgres, audit log      │
 Phone (PWA) ◀── push / call ───────────▶│                             │
 Laptop browser (same PWA)               └─────────────────────────────┘
```

## 7. Calls

| Stage | What | How |
| --- | --- | --- |
| MVP (P1) | Outbound alert call: spoken message, keypad approval ("press 1 to approve the commit, 2 to deny") | Telephony provider plus text-to-speech. **No LLM on the call**, so it is cheap, fast and safe |
| Later | Two-way conversation about a task, a project, or the news | Streaming speech-to-text, the mid-tier LLM, text-to-speech, with a realtime voice API as an alternative. Choose the vendor at that point and test latency (target under about 1.5 s per turn) |
| Later | Inbound calls to PA | Same pipeline, with caller verification by phone number plus PIN |

Rules: calls can read out status and ask for approval; they can never do
risky actions (force-push, delete). Calls run only inside the user's quiet
hours and escalation settings. Recording and transcripts are off by
default.

- [ ] Get a phone number and a provider account **(needs approval)**
- [ ] Phone numbers are PII: encrypt at rest

## 8. Do we need Google Sheets or Google Docs?

**Not for the product to work.** Useful for the team only:

- [ ] One Google Doc for the CEO proposal (can be generated from this plan)
- [ ] One Google Sheet for the pilot: user list, bugs, feedback and
      permission-request status (optional; a GitHub project board also works)

PA itself uses Postgres, Google Chat, Calendar and Gmail.

## 9. Production-ready checklist (applies to the MVP)

**Security**
- [ ] Threat model written (data flows, who can see what)
- [ ] OAuth scopes minimal; app type **Internal** to your Workspace, which
      avoids Google's public verification (confirm with your admin)
- [ ] Tokens encrypted; keys from env or Vault; nothing secret in git
- [ ] HTTP security headers, rate limiting, body limits, CORS allowlist
- [ ] Dependency audit in CI

**Reliability**
- [ ] Reminder delivery tests: restart, duplicate event, clock change
- [ ] Idempotent actions, retries with limits, dead-letter list
- [ ] Health endpoint, backups, and a restore that has been tried

**Performance targets**
- [ ] Approval relay (agent to phone) under 2 seconds p95
- [ ] Reminder fired within 5 seconds of its scheduled time
- [ ] LLM calls streamed or queued, never blocking the request

**Privacy and trust**
- [ ] "What PA collects and never sees" page
- [ ] Per-user data export and delete
- [ ] Retention rules (for example, audit 1 year, drafts 30 days)

**Operations**
- [ ] CI: lint, tests, security scan; one-command deploy; rollback plan
- [ ] Dashboards: reminder success, LLM cost and error rate, approval latency

## 10. Two-week plan

**Week 1: foundation**

| Day | Deliverable |
| --- | --- |
| 1 | Spec, data model, threat model; new repo, CI, Postgres, base Go service |
| 2 | Google SSO, orgs and users, row-level security, settings store, onboarding API |
| 3 | Scheduler and reminder engine; space names decided |
| 4 | PWA shell: login, onboarding, today view, web push, reminder buttons |
| 5 | LLM gateway, router, guardrails and policy engine skeleton, eval set |
| 6 | Status drafts (3 per day) with templates and the approval screen |
| 7 | Google Chat posting as the user; audit log; buffer and review |

**Week 2: agents and pilot**

| Day | Deliverable |
| --- | --- |
| 8 | Companion: pairing, secure connection, hook and MCP install |
| 9 | Claude Code events to phone; local redaction |
| 10 | Approval relay for `git commit` and `git push` |
| 11 | Calendar read, meeting reminders, journal (P1 starts) |
| 12 | Hardening, security review, load and failure tests |
| 13 | Pilot with 3 to 5 colleagues, fixes |
| 14 | CEO and office demo |

**Demo script:** onboard, get the 9:00 reminder and tap a button, approve an
EOD status that posts to Google Chat, see a Claude Code task finish on the
phone, approve a `git commit` from the phone.

Honest scope note: P0 fits in two weeks for a pilot. P1 items (meetings,
MOM, journal, news, alert calls) land only if P0 is stable. Items in
section 12 do not fit.

## 11. Permissions and access to request from the CEO and IT

Ask for these **now**, because they have lead time.

| # | Request | Needed for | When |
| --- | --- | --- | --- |
| 1 | Approval to run a pilot with 3 to 5 volunteers, and an agreed "what PA collects" policy | Everything | Before week 2 |
| 2 | Google Workspace admin: allow an **Internal** OAuth app with Chat, Calendar, Gmail scopes | Statuses, meetings, email | Week 1 |
| 3 | Approval and budget for an LLM provider, plus confirmation that sending redacted summaries to it is allowed (data processing terms) | All AI features | Week 1 |
| 4 | A server or cloud project, a domain and TLS | Hosting | Week 1 |
| 5 | A phone number and telephony budget | Alert calls | Before week 2 |
| 6 | Propeak API access (read tasks, post updates) | Status to CRM | After MVP |
| 7 | HRMS API access: attendance, timesheet, leave, WFH, holidays, policy documents | Section 12 | After MVP |
| 8 | Security or legal review of the data flow | Company-wide rollout | After pilot |
| 9 | SSO policy and device policy (can the companion be installed?) | Rollout | After pilot |

## 12. After the MVP (needs the approvals above)

**Blocked on Propeak or HRMS approval**
- [ ] Propeak: push the task and status updates
- [ ] HRMS: read attendance and record time in/out where policy allows
- [ ] HRMS: timesheet reading and filling, with nudges when it is empty
- [ ] HRMS: leave balance, pending leave and WFH requests, creating them
- [ ] HRMS: holidays, so reminders skip them automatically
- [ ] HR policy questions answered from the policy documents with citations
- [ ] Adapters for both behind the same integration interface

**Engineering work**
- [ ] Two-way voice conversation and inbound calls
- [ ] Project start helper: generate `AGENTS.md`, `CLAUDE.md` and a project
      description, with coding-practice guides
- [ ] Learned preferences ("PA noticed you always edit X") and routines
- [ ] Cursor and Antigravity parity; a task board for agent work
- [ ] Native iOS and Android apps
- [ ] Admin console, audit export, retention controls
- [ ] Semantic cache for shared news content; vector memory if needed
- [ ] Load tests, an external penetration test, Kubernetes

## 13. Do now (next 48 hours)

- [ ] Create the new repo and name the project
- [ ] Decide the two space names (section 2)
- [ ] Send the permission requests 2, 3, 4 and 5 (section 11)
- [ ] Confirm Android or iPhone for the pilot users
- [ ] Choose 3 to 5 pilot colleagues
- [ ] Write the one-page "what PA collects" policy
- [ ] Verify the current Claude Code hook fields and the Google Chat
      "post as user" scope against the official docs before coding
- [ ] Start Day 1 of the plan

## 14. Open questions

1.  Android or iPhone?
2.  Is a Google Workspace admin reachable this week?
3.  Does your office allow sending redacted summaries to an external LLM
    provider, or must everything stay inside the company network?
4.  Which meeting tool (Google Meet, Zoom) and does it produce
    transcripts you can use for MOM?
5.  India phone numbers or global?
