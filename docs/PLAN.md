# PLAN: phased roadmap

Rule: do only the next unchecked item, then stop. Each phase ends with a
verifiable check ("Done when"). Feature ids refer to the Google Sheet
"PA - Feature Backlog v2". Dates are the R0 target (demo Fri 27 Nov 2026).

## Phase 0: Spike (half day, throwaway code, Mon 16 Nov)
- [ ] Tiny script: Claude Code `PreToolUse` hook on `git commit` waits for an
      answer from a local stub "phone" and returns allow or deny
- [ ] Record the real hook timeout, input fields and response format in
      ARCHITECTURE.md section 6
- [ ] Review the OpenMausBot and OpenClaw architecture for ideas; check licences
Done when: a commit is allowed, denied and timed out by the stub, and the
design in section 6 is confirmed or corrected.

## Phase 1: Foundation (D1, F-140 F-146 F-161 F-163)
- [ ] Cargo workspace with the crates listed in AGENTS.md (empty, compiling)
- [ ] `pa-config`: typed layered config, fail-fast validation, tests
- [ ] CI: fmt, clippy, test, `cargo audit`, `cargo deny`
- [ ] Threat model document and permission requests sent (Workspace OAuth,
      LLM approval, code-signing certificate)
Done when: CI is green and a config test proves a missing required value fails.

## Phase 2: Database, auth and settings (D2, F-001 F-002 F-003 F-005 F-010 F-011 F-143)
- [ ] Migration v1: tables in ARCHITECTURE.md section 3 with RLS
- [ ] Isolation tests: user A cannot read or write user B rows on any table
- [ ] Google OIDC sign-in, sessions, roles
- [ ] Settings store (versioned) and onboarding API
- [ ] API hardening: headers, CORS allowlist, rate limits, body limits
Done when: isolation tests pass and a new user can sign in and save settings.

## Phase 3: Scheduler and attendance (D3, F-020 F-021 F-022 F-024 F-025 F-111 F-145 F-164 F-165)
- [ ] Postgres job queue and injectable clock
- [ ] Reminder engine: time in, time out, snooze, custom, quiet hours
- [ ] Exactly-once delivery test across a restart and a daylight saving change
- [ ] Web push sender; own task list and attendance record
Done when: killing the server mid-run still delivers each reminder once.

## Phase 4: Web app shell (D4, F-007 F-012 F-110 F-112 F-118 F-158)
- [ ] PWA: login, onboarding interview, consent screen and privacy notice
- [ ] Home command box and today strip; reminder buttons; pause switch
- [ ] Accessibility check in CI
Done when: installed on a phone, a push arrives and a button reply is stored.

## Phase 5: LLM, policy and evidence (D5, F-120 to F-126 F-150 F-030 to F-032)
- [ ] `pa-llm`: provider trait, router from config, budgets, usage table
- [ ] Policy engine and untrusted-content wrapping with injection tests
- [ ] Evidence model, draft validator, intent classifier, evaluation set in CI
Done when: an unsupported statement is rejected and injected text triggers
no action.

## Phase 6: Status flow (D6, F-004 F-033 F-034 F-035 F-130 F-157)
- [ ] Draft, edit, approve, post to Google Chat as the user (text only)
- [ ] Template editor, audit log, template fallback when the LLM is off
Done when: an approved status appears in a test Chat space under the user's name.

## Phase 7: Companion and agent events (D7, F-040 F-041 F-042 F-045 F-160)
- [ ] Windows companion: pairing, outbound WebSocket, signed messages
- [ ] Claude Code hooks and MCP setup; events to phone; redaction; local pause
Done when: a real Claude Code task finishing shows on the phone, and planted
secrets never reach the server.

## Phase 8: Approval relay (D8, F-043 F-044 F-046 F-153, stretch F-152)
- [ ] Approval request, exact-command screen, passkey decision, expiry
- [ ] Deny, timeout and replay tests; force-push never approvable
- [ ] PA MCP tools
Done when: a commit is approved from a phone and denial and timeout both block.

## Phase 9: Hardening and demo (D9 to D10, F-141 F-142 F-162 F-166)
- [ ] Failure and load tests, security review, backup and restore rehearsal
- [ ] Dashboards, seeded demo tenant, recorded fallback clip, pilot users
Done when: the demo script runs end to end twice without errors.

## Release R1 (about 11 Dec): see the Sheet, release "R1"
- [ ] Calendar, email, minutes, journal, news, alert calls, Cursor and
      Antigravity, content helpers, Hindi UI, load test

## Release R2: after CEO approval: see the Sheet tab "After CEO approval"
- [ ] Propeak, HRMS, Google Sheets, trackers, voice, rollout
