# PROGRESS — InvoiceNudge

> State file, not a diary. The agent reads this first in every session and updates it before handoff.
> Source of truth for *what* to build: `docs/BLUEPRINT.md`. For *why*: `docs/DECISIONS.md`. Review standard: `docs/REVIEW-CHECKLIST.md`.

## Current position
- Active epic: E00 — Foundation (NOT STARTED)
- Last green `check` commit: (none yet)
- Staging URL: (set in E00-S07)
- Production URL: (set in E08-S03)
- `NUDGES_LIVE`: staging `off` · production not created
- Blockers: none. Open questions affecting E00: OQ-05 (Render plan prices), OQ-06 (GitHub Pro), OQ-07 (domain for sending; needed by E01)

## Epic status
| Epic | Title | Status | Started | Done | Notes |
|---|---|---|---|---|---|
| E00 | Foundation | NOT STARTED | | | 7 stories |
| E01 | Onboarding and account | NOT STARTED | | | 6 stories |
| E02 | Clients and invoices | NOT STARTED | | | 7 stories |
| E03 | Owed overview and payments | NOT STARTED | | | 4 stories |
| E04 | Nudge scheduling and evening digest | NOT STARTED | | | 4 stories |
| E05 | Client email nudges with baseline safety | NOT STARTED | | | 6 stories |
| E06 | Operator tools and manual forwarding | NOT STARTED | | | 4 stories |
| E07 | Settings, export and deletion | NOT STARTED | | | 5 stories |
| E08 | Launch hardening | NOT STARTED | | | 4 stories |

<!-- Status values: NOT STARTED · IN PROGRESS · DONE · BLOCKED -->

## Story status and AC evidence
<!-- For each story: status + one evidence line per AC. Evidence = passing test name in `check` (with "seen red"), or `manual YYYY-MM-DD — observation`. -->

### E00
- E00-S01 — Bootstrap the repository, runtime pins and project docs — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E00-S02 — Build the `check` quality gate — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E00-S03 — Create the module skeleton and architecture fitness functions — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E00-S04 — Fail-fast configuration, structured logging, error tracking and health endpoint — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E00-S05 — Database access, migrations, drift check and test database harness — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E00-S06 — Telegram webhook skeleton with update de-duplication and bot test harness — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E00-S07 — CI pipeline, branch protection and staging deployment with rollback — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —

### E01
- E01-S01 — Field encryption and blind index for personal data — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E01-S02 — Conversation engine and system fallbacks — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E01-S03 — Start command and profile name steps — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E01-S04 — Reply-to email verification by code — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E01-S05 — Time-zone picker — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E01-S06 — Default currency, payment instructions and home menu — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —

### E02
- E02-S01 — Money parsing, formatting and totals — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E02-S02 — Client records: create, list, edit, archive — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E02-S03 — New-invoice wizard: client and amount steps — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E02-S04 — New-invoice wizard: due-date step — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E02-S05 — Reminder plan computation (presets, weekdays, send hour, DST) — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E02-S06 — New-invoice wizard: reference, link, reminder plan and save — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E02-S07 — Invoice card: edit, cancel and delete — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —

### E03
- E03-S01 — Invoice status and balance rules — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E03-S02 — "Who owes me" list with totals per currency — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E03-S03 — Mark paid in full, with undo and reopen — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E03-S04 — Record a part payment — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —

### E04
- E04-S01 — Nudge rows lifecycle and re-planning on invoice events — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E04-S02 — Notification outbox and dispatcher — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E04-S03 — Evening digest with hold and mark-paid — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E04-S04 — Pause, resume and change a reminder plan — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —

### E05
- E05-S01 — Compose reminder emails and signed action links — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E05-S02 — Send pipeline: claim, send once, record, retry — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E05-S03 — Client "I've already paid" page and freelancer confirmation — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E05-S04 — Stop reminders and one-click unsubscribe — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E05-S05 — Sending caps, send-now and sent notices — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E05-S06 — Email events webhook: bounces and complaints — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —

### E06
- E06-S01 — Complaint-based auto-suspension — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E06-S02 — Operator admin commands — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E06-S03 — Text to forward for chat apps — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E06-S04 — Product metrics in operator stats — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —

### E07
- E07-S01 — Settings screen and edits — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E07-S02 — Change reply-to email with re-verification — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E07-S03 — Export my data as CSV — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E07-S04 — Delete my account and retention jobs — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E07-S05 — Privacy page and bot command menu — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —

### E08
- E08-S01 — Backup restore drill and runbook — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E08-S02 — Load test at the scale ceiling and performance budgets — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E08-S03 — Production environment, domain authentication and go-live smoke — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —
- E08-S04 — Alerts on error rate, send failures and backlog — NOT STARTED
  - AC-1: —
  - AC-2: —
  - AC-3: —
  - AC-4: —
  - AC-5: —
  - AC-6: —
  - AC-7: —
  - AC-8: —

## VERIFY-AT-BUILD results
| Item (BLUEPRINT §18.10) | Needed by | Checked (date) | Result | Notes |
|---|---|---|---|---|
| V-01 Package versions in §13.1 (record newer versions, don't upgrade) | E00 | | | |
| V-02 Node 26 LTS status (LTS from 2026-10-28, S-32) | E00 (again E08) | | | |
| V-03 Bot API version and grammY support (§3.6, S-10, S-59) | E00 | | | |
| V-04 Webhook ports, TLS, source IPs (§3.6, S-13) | E00 (again E08) | | | |
| V-05 GitHub-hosted runners include Docker (§13.1) | E00 | | | |
| V-06 Sentry free-plan limits and SDK privacy option names (§12.7) | E00 | | | |
| V-07 Render rollback steps (§13.8) | E00 (again E08) | | | |
| V-08 `node:crypto` algorithms available (§13.1) | E01 | | | |
| V-09 Telegram message length limit 4,096 (§3.6) | E01 | | | |
| V-10 Resend free-tier limits and pricing (§12.7, S-26) | E01 (again E08) | | | |
| V-11 ISO 4217 list current (S-68) | E02 | | | |
| V-12 Temporal `disambiguation` option name (E02-S05 rule 6) | E02 | | | |
| V-13 Telegram flood limits and grammY auto-retry options (S-08, S-56) | E04 | | | |
| V-14 Resend idempotency semantics, webhook event names and message-ID field, SDK `webhooks.verify` (S-27, S-29, S-89) | E05 | | | |
| V-15 Resend DKIM signature covers `List-Unsubscribe` headers (S-47) | E05 | | | |
| V-16 Fastify `addContentTypeParser` with `parseAs: "string"` (E05-S06) | E05 | | | |
| V-17 Svix timestamp tolerance (E05-S06) | E05 | | | |
| V-18 Telegram document upload limit and BotFather `/setdescription` (S-08, S-12) | E07 | | | |
| V-19 CSV with BOM opens correctly (assumption 34) | E07 | | | |
| V-20 Render PITR windows, plans and prices (S-38, OQ-05) | E08 | | | |
| V-21 Render's Node 26.10.0 runtime has Temporal enabled (DR-04 build caveat, S-91) | E00 | | | |

## Deviations and amendments
| # | Date | Epic/Story | What changed | Why | Amendment ID (BLUEPRINT §18.7) |
|---|---|---|---|---|---|

## Accepted risks (skipped edge cases / tests) — each needs an owner and a revisit trigger
| # | Story | Edge case / test | Reason | Owner | Revisit when |
|---|---|---|---|---|---|

## Quarantined tests (a flaky test is a broken gate)
| Test | Date | Symptom | Follow-up story | Owner |
|---|---|---|---|---|

## Engineering metrics log (BLUEPRINT §14.6 — one row per epic)
| Epic | `check` runtime (M-23) | Median story diff (M-17) | Review cycles (M-16) | Escaped defects (M-19) | Mutation score (M-24) | What slowed this session / what had to be guessed |
|---|---|---|---|---|---|---|

## Parking lot (ideas out of scope — do NOT build)
- Working-days setting for non Monday–Friday weekends (OQ-14)
- Client time zone for send times (OQ-13)
- Payment history view per invoice
- Operator override of sending caps per account

## Preconditions for the next epic (E00)
- Private GitHub repo on a GitHub Pro account (OQ-06 default); official Node.js 26.10.0 build with Temporal (`node -e "console.log(typeof Temporal)"` prints `object`); Docker; gitleaks 8.30.1 installed locally
- Render account with Frankfurt access; staging bot token from @BotFather
- Start sending-domain DNS set-up now (OQ-07) so E01 is not blocked (R-13)

## Session log (one line per session)
| Date | Epic | Outcome | Final commit |
|---|---|---|---|
