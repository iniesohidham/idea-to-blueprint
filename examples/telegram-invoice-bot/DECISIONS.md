# DECISIONS — InvoiceNudge

> One entry per decision, newest last. Blueprint decision records DR-01 to DR-23 are copied here in E00 so this is the single place to look.
> Format for new entries: context → options → decision → consequences (including what the rejected option would have bought) → compliance → reversibility → evidence. Keep each under 15 lines. IDs are never reused; superseded entries stay with their status changed and a pointer to the new one.
> `AD-n` drivers and `FF-nn` fitness functions are defined in BLUEPRINT §12; `S-nn` sources in BLUEPRINT §22.

## DR-01 — Telegram chat bot only, no Mini App
**Status** Accepted — 2026-09-30 · Supersedes: none
**Drivers** AD-2 (operability), AD-8 (evolvability)
**Context** Every v1 job (log, list, mark paid, settings) fits in short messages with inline keyboards; Telegram's guidance favours specific commands and editing keyboards in place (S-12).
**Options** (a) chat bot only; (b) bot + Mini App for lists and forms; (c) bot + separate web dashboard.
**Decision** (a).
**Consequences** One interface technology and no front-end build. Cost: long lists are paginated text (8 invoices per page), and bulk editing is not possible. A Mini App would have bought richer tables and forms at the price of a web front end and Mini App authentication.
**Compliance** Manual — review checklist "Design" items; no front-end build tooling may be added without a superseding DR.
**Reversibility** Cheap — the modules' public interfaces are UI-agnostic (§12.2).

## DR-02 — Client reminders by email, sent from our domain, replies to the freelancer
**Status** Accepted — 2026-09-30
**Drivers** AD-1, AD-5
**Context** Bots cannot message people who have not started them (S-82). Clients are businesses with email. Resend supports `replyTo`, custom headers and idempotency keys [VERIFIED — S-72, S-89, 2026-09-30].
**Options** (a) email from our domain, `From: "<Name> via InvoiceNudge" <nudges@…>`, `Reply-To: <freelancer>`; (b) send from the freelancer's own mailbox via Gmail/Microsoft OAuth; (c) bot drafts only, freelancer sends.
**Decision** (a), plus (c) as a manual fallback (E06-S03).
**Consequences** Replies land in the freelancer's inbox. Our domain's reputation is shared across all users, which is why AD-5 and the caps exist. Option (b) would have bought "from my own address" credibility and no shared reputation, at the price of OAuth verification, provider-specific APIs and far more sensitive scopes.
**Compliance** Unit test `nudging/compose.test.ts::from_and_reply_to` asserts both headers; e2e CE-3 asserts the Reply-To value.
**Reversibility** Medium — templates and tokens are channel-agnostic; adding (b) later is additive.

## DR-03 — Modular monolith with an in-process scheduler
**Status** Accepted — 2026-09-30
**Drivers** AD-1, AD-2, AD-3, AD-6 — full scoring in §12.1
**Decision** One Node.js process serves the Telegram webhook, the email-events webhook and the client pages, and runs the scheduler and outbox loops. Postgres is the only state store.
**Consequences** One deploy unit and one log stream. Cost: scheduler work shares the event loop with webhook handling (mitigated by small batches; exit trigger in §12.1). A separate worker would have bought isolation of webhook latency from send bursts, at the price of a second service to deploy and monitor.
**Compliance** FF-01 to FF-06 (§12.10) enforce the module boundaries that keep extraction cheap.
**Reversibility** Cheap — `RUN_SCHEDULER=false` plus a second start command splits the process with no code change.

## DR-04 — Node.js 26 and TypeScript 6.0.3
**Status** Accepted — 2026-09-30
**Drivers** AD-3 (time correctness), AD-8
**Context** Node.js 26 ships the Temporal API enabled by default (released 2026-05-05, LTS from 2026-10-28, end of life 2029-04-30) [VERIFIED — S-60, S-32, 2026-09-30]. TypeScript 6.0 ships Temporal types via `lib: ["esnext"]` [VERIFIED — S-61, 2026-09-30]. TypeScript 7.0.2 is the latest release, but typescript-eslint 8.71.0 declares `typescript >=4.8.4 <6.1.0` as its peer range [VERIFIED — S-31, S-33, 2026-09-30].
**Options** (a) Node 26 + TS 6.0.3; (b) Node 24 LTS + TS 6.0.3 + a Temporal polyfill; (c) Node 26 + TS 7.0.2 with Biome instead of typescript-eslint; (d) Python 3.14 + aiogram (generic score winner, §13.1).
**Decision** (a).
**Consequences** `PlainDate` (due dates), `ZonedDateTime` (send times) and `Instant` (stored timestamps) are distinct compile-time types, so mixing them is a type error rather than a runtime bug. Cost: Node 26 is "Current" until 2026-10-28, and TS 7's roughly 10× faster compiler (S-33) is not used. **Build caveat:** Temporal needs a Node build compiled with Temporal support. A local Homebrew Node 26.3.0 build had `v8_enable_temporal_support: 0` and `typeof Temporal === "undefined"` [VERIFIED — S-91, 2026-09-30], so the project uses official nodejs.org binaries (nvm/fnm locally, `actions/setup-node` in CI), and the app refuses to start without Temporal (E00-S04). If Render's runtime lacks it (V-21), the fallback is `temporal-polyfill` 1.0.5 (S-31) behind `platform/time`, recorded as a DR. (b) would have bought "LTS today" and Render's default runtime (S-62), at the price of a polyfill dependency. (c) would have bought faster type checks, at the price of losing typescript-eslint's type-aware rules. (d) is recorded in §13.1.
**Compliance** `.node-version` = `26.10.0`, `engines.node` = `>=26.10.0 <27`; CI reads `.node-version`; `typescript` pinned exactly to `6.0.3`.
**Reversibility** Cheap — version bumps; revisit TS 7 when typescript-eslint supports it (R-06).

## DR-05 — PostgreSQL 18 with Kysely and hand-written migrations
**Status** Accepted — 2026-09-30
**Drivers** AD-1, AD-2, AD-4
**Context** PostgreSQL 18.6 is current (supported until 2030-11-14) [VERIFIED — S-37, 2026-09-30]; `uuidv7()` is built in [VERIFIED — S-84, 2026-09-30]; Render Postgres encrypts at rest with AES-256 and offers PG 13–18 [VERIFIED — S-83, 2026-09-30]. Kysely 0.29.6 supports `forUpdate()` and `skipLocked()` [VERIFIED — S-35, 2026-09-30] and ships a locking migrator [VERIFIED — S-34, 2026-09-30]. `kysely-codegen --verify` fails when generated types are stale [VERIFIED — S-36, 2026-09-30].
**Options** (a) Kysely + pg + kysely-codegen; (b) Drizzle ORM 0.45.3; (c) Prisma 7.10; (d) SQLite.
**Decision** (a).
**Consequences** SQL-shaped, typed queries; the schema lives in reviewed migration files; drift is caught mechanically (FF-11). Cost: no schema-in-TypeScript and more hand-written SQL. (b) would have bought TS-defined schemas and generated migrations, but its 1.0 release candidate is in flight (1.0.0-rc.5 alongside stable 0.45.3) [VERIFIED — S-31, 2026-09-30], which means docs/API churn during our build. (c) would have bought the most documentation and built-in drift detection, but the `prisma` CLI's `latest` tag already points to 8.0.0-rc.19 while `@prisma/client` is 7.10.0 [VERIFIED — S-31, 2026-09-30]. That is an install trap for agents. (d) would have bought zero database operations (invioTrack runs SQLite, S-01), at the price of no managed point-in-time recovery and no row-level `SKIP LOCKED` claiming.
**Compliance** FF-06, FF-07, FF-11 (§12.10).
**Reversibility** Medium — queries are isolated in module repositories.

## DR-06 — Resend as the email provider, with idempotency keys
**Status** Accepted — 2026-09-30
**Drivers** AD-1, AD-5
**Context** Resend honours an `Idempotency-Key` header for 24 hours. The same key and payload returns the original response; a different payload returns 409 [VERIFIED — S-27, 2026-09-30]. The Node SDK sends it via `emails.send(payload, { idempotencyKey })` [VERIFIED — S-89, 2026-09-30]. Webhooks are Svix-signed [VERIFIED — S-30, 2026-09-30]. Test addresses simulate delivered, bounced and complained outcomes [VERIFIED — S-28, 2026-09-30]. Free tier: 3,000 emails/month and 100/day; Pro US$20/month for 50,000 [VERIFIED — S-26, 2026-09-30] [VERIFY-AT-BUILD]. Postmark does not currently support idempotency keys [VERIFIED — S-25, 2026-09-30].
**Options** (a) Resend; (b) Postmark (separate transactional and broadcast streams, S-24); (c) Amazon SES.
**Decision** (a).
**Consequences** A retried send can never produce a second email within the 24 h window (the DR-07 state machine covers the rest). Cost: dependence on a younger vendor. Postmark would have bought a long deliverability track record and stream separation, at the price of building our own duplicate protection with no provider guarantee.
**Compliance** Integration test `nudging/send_pipeline.int.test.ts::retry_uses_same_idempotency_key`.
**Reversibility** Medium — all sending goes through the `EmailGateway` port (FF-13).

## DR-07 — Exactly-one-email-per-nudge state machine
**Status** Accepted — 2026-09-30
**Drivers** AD-1
**Context** A reminder sent twice, or after payment, is the most damaging failure the product can have.
**Options** (a) nudge rows with a state machine, claimed with `FOR UPDATE SKIP LOCKED`, re-checked right before sending, sent with `Idempotency-Key = nudge/<nudge_id>`; (b) a job-queue library with at-least-once jobs; (c) cron batch that sends everything due.
**Decision** (a) — details in §12.4 (ENT-Nudge) and E05-S02.
**Consequences** Every reminder's lifecycle is visible in one table. A nudge whose outcome is unknown for more than 23 hours becomes `unknown` and is never auto-resent, because the idempotency key expires after 24 h. (b) would have bought retries and backoff for free (pg-boss 12.35.0 exists [VERIFIED — S-31, 2026-09-30]), at the price of a second source of truth for "what is scheduled".
**Compliance** Unit tests on the transition table; integration test `send_pipeline.int.test.ts::two_workers_one_email`; e2e CE-3.
**Reversibility** One-way once production rows exist (schema of `nudge`).

## DR-08 — Field-level encryption for personal data, with a blind index
**Status** Accepted — 2026-09-30
**Drivers** AD-4
**Context** Telegram's developer terms require user data to be "encrypted at rest and stored separately from its encryption key" [VERIFIED — S-07, 2026-09-30]. Render encrypts disks with AES-256 (S-83), but logical dumps and exports would still be plaintext.
**Options** (a) provider disk encryption only; (b) application-level AES-256-GCM (Node `crypto`) for PII columns, key from the environment, plus an HMAC-SHA-256 blind index for email lookups; (c) per-user keys (crypto-shredding).
**Decision** (b) for: client name, client email, client contact name, freelancer display name, business name, freelancer email, payment instructions, invoice note, conversation data.
**Consequences** A database dump alone reveals no contacts. Cost: no SQL search or sort on encrypted columns. Client lists are decrypted and sorted in memory (≤ 200 per freelancer), and key rotation is a procedure (§12.8). (a) would have bought simplicity; (c) would have bought instant erasure from backups, at the price of key management per user.
**Compliance** FF-14: an integration test scans a `pg_dump`-style `SELECT *` of every table after the e2e suite and asserts no fixture email or name appears in plaintext. Whether this satisfies Telegram's §4.4 wording is confirmed with counsel (OQ-04).
**Reversibility** One-way once data exists (re-encryption migration needed to undo).

## DR-09 — Webhook delivery with secret token and update de-duplication
**Status** Accepted — 2026-09-30
**Drivers** AD-1, AD-6
**Context** Long polling cannot run while a webhook is set (S-08); zero-downtime deploys briefly run two instances (S-41); slow handling causes Telegram to re-send updates (S-54).
**Options** (a) webhook with `secret_token`, dedupe on `update_id`; (b) long polling (grammY's advice: fine for always-on servers, S-54).
**Decision** (a).
**Consequences** Clean overlap during deploys, and duplicate updates are harmless. Cost: a public HTTPS endpoint (we need one anyway for email events and client pages). (b) would have bought no inbound exposure, at the price of polling conflicts during deploy overlap.
**Compliance** Integration test `web/telegram_webhook.int.test.ts::rejects_missing_secret` and `::same_update_twice_processed_once`.
**Reversibility** Cheap.

## DR-10 — Money as integer minor units with an ISO 4217 table
**Status** Accepted — 2026-09-30
**Drivers** AD-1
**Context** ISO 4217 List One (published 2026-09-17) gives minor units per currency, e.g. 2 for USD, EUR, GBP and INR; 0 for JPY, KRW, VND and CLP [VERIFIED — S-68, 2026-09-30].
**Decision** `amount_minor bigint` + `currency char(3)`; a generated, committed `currencies.json` from List One; no floats anywhere; display "EUR 1,200.00" via `Intl.NumberFormat('en', { style: 'currency', currency, currencyDisplay: 'code' })`.
**Consequences** Exact arithmetic for balances and part payments. Cost: a parser that respects per-currency decimals (E02-S01). Floats would have bought nothing.
**Compliance** ESLint `no-restricted-syntax` forbids `parseFloat` and `Number(` inside `src/platform/money/**` except the parser's audited function; mutation testing on `platform/money` (break 70).
**Reversibility** One-way once data exists.

## DR-11 — Time model: UTC instants, local calendar dates, weekday send windows
**Status** Accepted — 2026-09-30
**Drivers** AD-3
**Decision** `timestamptz` in UTC for instants; `date` for due dates (the freelancer's calendar); the freelancer's IANA zone (validated against `Intl.supportedValuesOf('timeZone')` [VERIFIED — S-64, 2026-09-30]); send instants computed with Temporal in `platform/time`. All "now" comes from an injected `Clock`.
**Consequences** "Overdue" is computed against the freelancer's local date. Cost: every time computation goes through one module. Storing local wall-clock times would have bought readability at the price of DST bugs.
**Compliance** FF-12 (no `Date` or `Date.now()` outside `platform/time`); DST tests for Europe/Berlin, America/New_York, Australia/Sydney, Asia/Kolkata and America/Sao_Paulo in E02-S05.
**Reversibility** One-way.

## DR-12 — No reminder leaves without an evening-digest announcement
**Status** Accepted — 2026-09-30
**Drivers** AD-1
**Decision** A nudge can be sent automatically only if `announced_at` is set, i.e. it appeared in a digest at least 12 hours earlier. Exceptions: the freelancer taps "Send reminder now", or has turned the digest off (then `announced_at` is set at planning time). Unannounced nudges whose time has come are postponed to the next send window after the next digest.
**Consequences** The freelancer always has a chance to Hold. Cost: a newly logged overdue invoice gets its first reminder the day after next at the earliest, unless the freelancer taps Send now.
**Compliance** Unit test `nudging/domain/eligibility.test.ts::unannounced_is_not_sendable`; e2e CE-3.
**Reversibility** Cheap.

## DR-13 — Client opt-out: per-invoice stop link plus RFC 8058 one-click unsubscribe
**Status** Accepted — 2026-09-30
**Drivers** AD-5
**Context** RFC 8058 requires `List-Unsubscribe` (HTTPS) plus `List-Unsubscribe-Post: List-Unsubscribe=One-Click`, a POST, no redirects or cookies, and DKIM-signed headers [VERIFIED — S-47, 2026-09-30].
**Decision** Every reminder carries both headers (one-click stops all reminders from that freelancer to that address) and a body link to stop reminders for this invoice only. Opt-outs are permanent for that freelancer–address pair in-product.
**Consequences** Clients stay in control and complaint rates stay low. Cost: a freelancer cannot re-enable reminders to an opted-out address; they can still use "text to forward".
**Compliance** Integration tests on API-06 and API-07; e2e CE-5.
**Reversibility** Cheap (policy), but honouring past opt-outs is mandatory.

## DR-14 — No invoice issuing, no payment processing
**Status** Accepted — 2026-09-30 · Drivers AD-2, AD-4 · Context §2 non-goals (S-07, S-71) · Options (a) track only; (b) generate PDFs; (c) payment links via a processor · Decision (a) · Consequences: smaller legal surface. (b) would have bought parity with invioTrack; (c) would have bought faster payment, at the price of licensing and refunds · Compliance: no PDF or payment SDK may be added without a superseding DR (dependency policy §13.10) · Reversibility: cheap.

## DR-15 — Design direction: text-first chat UI and plain server-rendered client pages
**Status** Accepted — 2026-09-30 · Drivers AD-2, AD-6 · Decision: bot messages follow §9 rules; client pages are server-rendered HTML with inline CSS, the system font stack and no JavaScript, fonts or CDNs · Consequences: pages weigh under 15 KB and have nothing to break. A CSS framework would have bought faster styling at the price of a build step · Compliance: FF-15 (page size and no external URLs test) · Reversibility: cheap.

## DR-16 — Identity: Telegram user ID, verified reply-to email, admin allow-list
**Status** Accepted — 2026-09-30 · Drivers AD-4, AD-5 · Context: webhook requests are authenticated by the secret token (S-10); the Telegram user ID in an authenticated update identifies the freelancer · Options: (a) Telegram identity + emailed code to verify reply-to; (b) separate login · Decision (a); operators are listed in `ADMIN_TELEGRAM_IDS` · Consequences: no passwords. Email verification stops anyone setting a victim's address as Reply-To. A separate login would have bought portability off Telegram at the price of a whole auth system · Compliance: authorization tests per command (§12.6) · Reversibility: medium.

## DR-17 — Hosting on Render, Frankfurt
**Status** Accepted — 2026-09-30 · Drivers AD-2, AD-4, AD-7 · Context: Render offers Frankfurt [VERIFIED — S-39, 2026-09-30], pre-deploy commands for migrations, zero-downtime deploys with health checks, and auto-deploy "After CI Checks Pass" [VERIFIED — S-41, 2026-09-30]; PITR of 3 days (Hobby) or 7 days (Pro workspace) [VERIFIED — S-38, 2026-09-30]; Node version pinned via `.node-version` [VERIFIED — S-62, 2026-09-30]. Free web services sleep after 15 minutes, which would break webhooks (S-40), so a paid instance is required · Options: (a) Render; (b) Fly.io (shared-cpu-1x 512 MB about US$2.96/month [VERIFIED — S-43, 2026-09-30]); (c) a VPS · Decision (a) · Consequences: managed Postgres with PITR and a CI-gated deploy with no ops code. Fly.io would have bought lower compute prices and more regions, at the price of more platform concepts (Machines, separate Postgres product). Render's prices could not be read this session [UNKNOWN → OQ-05] · Compliance: `render.yaml` in the repo · Reversibility: medium.

## DR-18 — CI on GitHub Actions with protected `main`
**Status** Accepted — 2026-09-30 · Drivers AD-2 · Context: protected branches for private repos need GitHub Pro; Free includes 2,000 Actions minutes/month and Pro 3,000 [VERIFIED — S-51, 2026-09-30] · Decision: private repo on GitHub Pro with `main` protected, requiring the `check` job ([UNKNOWN → OQ-06] for the account) · Consequences: the gate cannot be bypassed by pushing to `main`. A public repo would have bought free protection at the price of exposing the code · Compliance: E00-S07 AC-3 · Reversibility: cheap.

## DR-19 — Lint with typescript-eslint `strictTypeChecked`, format with Prettier
**Status** Accepted — 2026-09-30 · Drivers AD-1 · Context: typed linting via `parserOptions.projectService: true` [VERIFIED — S-77, 2026-09-30]; `strict-type-checked` is not a "stable" config, so rules may change outside majors (S-77) and typescript-eslint is pinned exactly · Options: (a) ESLint 10.11.0 + typescript-eslint 8.71.0 + Prettier 3.9.9; (b) Biome 2.5.14 alone · Decision (a) · Consequences: floating promises, unsafe `any` flows and non-exhaustive switches are build errors. Biome would have bought one fast binary for format and lint, at the price of a narrower set of type-aware rules · Compliance: `npm run lint` in `check` with `--max-warnings=0` · Reversibility: cheap.

## DR-20 — Architecture rules enforced with dependency-cruiser
**Status** Accepted — 2026-09-30 · Drivers AD-2 · Context: dependency-cruiser supports forbidden rules with path regexes and group matching (`$1`) for peer folders [VERIFIED — S-76, 2026-09-30]; version 18.4.0 (S-31) · Options: (a) dependency-cruiser; (b) eslint-plugin-boundaries 7.2.0 (S-31) · Decision (a) · Consequences: rules live in one file and run as a separate `check` step with readable reports. (b) would have bought editor-time feedback · Compliance: FF-01 to FF-06 · Reversibility: cheap.

## DR-21 — Scheduler as a database-claiming poller, no job-queue library
**Status** Accepted — 2026-09-30 · Drivers AD-1, AD-2, AD-3 · Decision: a loop every 30 s claims due rows (nudges, notifications, digests) with `FOR UPDATE SKIP LOCKED` in batches of 50; missed runs are caught up. Nudges more than 24 h late after an outage are re-planned to the next send window, never blasted (CP-SCR15-15) · Consequences: the domain table is the queue. Two instances during deploy overlap cannot double-claim. pg-boss would have bought built-in retries, cron and monitoring, at the price of a parallel job store · Compliance: `scheduler.int.test.ts::overlapping_ticks_claim_disjoint_rows` · Reversibility: cheap.

## DR-22 — One copy catalogue keyed by CP IDs
**Status** Accepted — 2026-09-30 · Drivers AD-2 · Decision: every user-facing string lives in `src/platform/copy/catalog.ts` keyed by the CP IDs of §11; code calls `t('CP-SCR07-1', vars)`. A test asserts that the catalogue key set equals the CP IDs defined in §11 of `docs/BLUEPRINT.md` · Consequences: copy changes are table edits. Inline strings would have bought speed now and drift later · Compliance: FF-10 · Reversibility: cheap.

## DR-23 — English only; unambiguous formats
**Status** Accepted — 2026-09-30 · Drivers AD-2 · Decision: English UI; amounts as "EUR 1,200.00"; dates as "Thu 30 Oct 2026"; times 24-hour; `language_code` (S-10) is stored for future localisation but not used · Consequences: one catalogue. Localisation would have bought reach at the price of translation and review · Compliance: `platform/format.test.ts` · Reversibility: cheap (catalogue keyed by ID).

<!-- Next entry: DR-24 (first decision taken in a build session). -->
