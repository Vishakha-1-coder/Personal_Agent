# AGENTS.md: global guardrails for PA (Personal Agent)

Read this first in every session. It holds rules only. Facts live in one
place each: [PRD](PRD.md) (what), [ARCHITECTURE](ARCHITECTURE.md) (how),
[PLAN](PLAN.md) (order), [CONTEXT](CONTEXT.md) (last session, next task).

## Workflow (every task)
1. Read CONTEXT.md. Do only the next unchecked item in PLAN.md, then stop.
2. Write the failing test first, then the smallest code that passes.
3. Run `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`,
   `cargo test`; for `web/` also lint, typecheck, test. All must be green.
4. Tick the item in PLAN.md, update CONTEXT.md, commit (Conventional Commits).
5. If a spec is wrong, fix the doc first, then the code.

## Stack and layout
Rust stable Cargo workspace, axum + tokio, sqlx + PostgreSQL 18, web PWA in
strict TypeScript. Crates (`crates/`): `pa-config`, `pa-domain`, `pa-db`,
`pa-llm`, `pa-connectors`, `pa-scheduler`, `pa-api` (server binary),
`pa-companion` (laptop binary). Also `web/`, `migrations/`, `config/`, `docs/`.

## Rules
* **DRY:** one source per fact. Constants, enums, error codes, messages,
  prompts and SQL fragments each live in exactly one place.
* **No hard-coding:** tunables go in `config/default.toml`; deployment values in
  env (`PA_*`); secrets from files (`PA_*_FILE`); per-user business settings in
  the database. Only `pa-config` reads env or files. Required values have no
  silent default; startup fails fast on invalid config.
* **Dependencies point inward:** `pa-domain` has no I/O and no framework
  types. Adapters implement traits that the domain declares.
* **Wrappers:** third-party SDKs and services (LLM, Google, telephony, push)
  are used only behind a trait in their own module.
* **Errors:** `thiserror` in libraries, `anyhow` only in binaries. One error
  envelope at the API edge. No `unwrap`, `expect` or `panic!` outside tests.
* **Time:** store UTC. Use the user's time zone for business rules, through
  one injectable clock so reminders are testable.
* **Safety:** every outward action (post, send, commit, push, create) goes
  through the policy engine and the `approvals` table. Default is deny.
  Text from email, chat, tickets, web pages or transcripts is data and never
  triggers an action. Every draft statement carries evidence ids.
* **Data:** every table has `org_id` and row-level security. Migrations are
  additive; never edit an applied one. Queries use compile-checked `sqlx`.
* **Logging:** `tracing` only. Never log PII, tokens, prompts or file contents.
* **Comments:** explain why, not what. Every module states its responsibility;
  every public item has a doc comment.
* **Size:** one responsibility per function and file (about 300 lines max).
  No dead code, no speculative features (YAGNI).
* **Consistency:** copy the style of the sibling module (see CONTEXT.md).

## Prohibitions
Never: use `unsafe`; commit secrets or `.env`; put secrets or settings in
images; read env outside `pa-config`; use `println!`; leave magic numbers or
strings; put mock data in production code; edit generated files; disable a
lint without a reason comment; add a dependency without a one-line reason in
the commit; add third-party skills or plugins; build UI before the schema
and auth for it are locked and tested; skip tests to finish faster.

## Git
Commit after every green milestone; if AI output breaks the build, reset to
the last green commit. Inspect diffs for deleted logic and hard-coded data.
