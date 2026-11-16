# CONTEXT: session memory (update at the end of every session)

Last updated: 2026-11-16.

## Just done
* Discovery, competitor research, BRD (Google Doc) and feature Sheet.
* Created the five context files in `docs/` (draft 0.1) and ran the interview.
  No code yet.

## Decisions locked
* Rust, axum, sqlx, PostgreSQL 18, PWA in TypeScript. Spaces: Desk and Home.
* Own task list, attendance record and time ledger; no Propeak or HRMS in R0.
* Build the core ourselves; do not base it on MausBot, OpenClaw or Hermes.
* Interview answers: mixed laptops (Windows first, Mac and Linux best effort)
  and phones; redacted summaries may go to an external LLM; Google Workspace
  sign-in; demo is R0 on Fri 27 Nov; one developer with coding agents.
* LLM: the owner's GPT (OpenAI) access, behind the provider trait. Model ids
  live in config per tier, never in code.
* Status with no evidence: ask the person; the answer becomes the evidence.
* Approval timeout: deny after 5 minutes, fall back to the terminal prompt
  (value in config).
* Home content reaches the LLM only when the person asks, per item.
* "I will be late" is recorded only in R0; no message goes out.
* Hosting: free tier with fake data for development and demo only; a small
  paid always-on server before real employee data (free tiers sleep).

## Known bugs
* None (no code).

## Next task
PLAN.md Phase 0: the hook approval spike. Do not scaffold before it passes.

## Open questions
1. Is the GPT access an API key (billed per token) or a ChatGPT subscription
   (no API access)? Which monthly cap?
2. Who is the Workspace admin, and which 3 to 5 pilot users?
3. Is this folder its own git repository? (A shared parent repo was found.)
4. Who orders the code-signing certificate?

## Gold-standard files
None yet. Name the first good module per pattern here once it exists.
