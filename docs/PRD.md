# PRD: PA (Personal Agent)

Status: draft 0.1, 2026-11-16. Full requirements (IDs `BR-`, `FR-`, `NFR-`) are
in the Google Doc "PA - Business Requirements Document and Project Plan";
every feature (`F-xxx`) is in the Google Sheet "PA - Feature Backlog v2".
This file is the short, binding summary for builders.

## 1. What PA is
A personal agent for office employees. It sits on top of the coding agents
they already use (Claude Code, Cursor, Antigravity) and handles the work
around the code: attendance reminders, three daily statuses, meetings, email,
minutes, a private activity record, domain news and alert calls. It relays an
agent's permission requests (git commit, git push) to the employee's phone.

Two spaces: **Desk** (office) and **Home** (personal). Home data is private;
administrators never see it. Names are UI labels only.

## 2. Users
* **Employee (developer):** wants fewer routine tasks, one view of what their
  agents did, and to approve agent actions from a phone.
* **Team lead:** wants accurate statuses, standup points, minutes.
* **Admin (IT):** wants SSO, isolation, audit, usage counts, no content.
* **CEO:** wants productivity, controlled cost, safe use of AI.

## 3. Principles (binding)
1. PA drafts, the person approves. No outward action without recorded approval.
2. Code and full agent chats stay on the laptop. Only redacted summaries leave,
   and only with opt-in.
3. Every factual statement in a draft links to evidence. No evidence, no line.
4. Everything personal is a setting, not code.
5. Works fully without the Propeak and HRMS connectors (own task list,
   attendance record and time ledger).

## 4. Core flows and acceptance criteria (release R0)
| Flow | Acceptance (all must hold) |
| --- | --- |
| Onboarding | New user signs in with Google, answers the interview, sees the consent screen, and a complete settings record exists. No summary uploads without opt-in. |
| Attendance | At shift start/end a push arrives with the configured buttons. Each reply is stored once. "Need more time" re-fires after the configured gap. Survives a server restart; no duplicates. |
| Status (3 per day) | At the user's times a draft appears, every statement links to evidence, the user edits and approves, only the approved text is posted to Google Chat as the user. With no evidence, PA asks the person what they did and that answer becomes the evidence. With the LLM off, a template draft is offered. |
| Agent events | A Claude Code task finishing or waiting shows on the phone within 2 s (p95). |
| Approval relay | A `git commit`/`git push` request shows the exact command, repo and branch; approve with a passkey allows it; deny, timeout or replay blocks it; force-push is never approvable remotely; every decision is in the audit log. |
| Privacy | Pause, export and delete work. Admin sees counts only. Home content reaches the LLM only when the person requests it for one item. |

## 5. Release scope
* **R0 (demo, 27 Nov):** the flows above on real infrastructure, plus
  security review and a seeded demo tenant.
* **R1 (about 11 Dec):** calendar, email, minutes, journal, news, alert calls,
  Cursor and Antigravity, content helpers, Hindi UI, load test.
* **R2 (after CEO approval):** Propeak, HRMS, voice conversation, rollout work.

## 6. Out of scope (do not build)
Propeak and HRMS connectors; two-way voice and inbound calls; native phone
apps; admin console beyond user and usage counts; third-party skills or
plugins; semantic cache or vector database; learned routers or fine-tuning;
manager views of employee content; autonomous actions without approval;
anything not listed in section 4 until R0 is green.

## 7. Success measures (targets, to calibrate after the pilot)
Reminder on time within 5 s: 99.5 %. Approval relay p95 under 2 s. Unapproved
outward actions: 0. Cross-user data exposure: 0. Draft statements with
evidence: 100 %. Drafts accepted with minor edits: 70 %. Pilot weekly active
users: 80 %. LLM cost per active user per day: within the approved budget.
