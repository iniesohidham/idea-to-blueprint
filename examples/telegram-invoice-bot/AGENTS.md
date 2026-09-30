# InvoiceNudge — agent instructions

You are building this product one epic per fresh session. Nothing is remembered between sessions; the repo is the memory.

## Read in this order, every session
1. `docs/PROGRESS.md` — where we are, evidence, deviations, preconditions
2. `docs/BLUEPRINT.md` — sections 12–16 (architecture, stack, conventions, quality, edge cases, delivery plan), 18 (session protocol), 21 (glossary), and the epic you were asked to build in section 17 plus every flow/screen/copy ID it references
3. `docs/DECISIONS.md` — why things are the way they are
4. `docs/REVIEW-CHECKLIST.md` — the standard every story is reviewed against before it is DONE

## Rules
- Follow `docs/BLUEPRINT.md` §18 (session protocol) exactly: orient → verify preconditions → plan → build story by story with tests first → verify gates → update PROGRESS/DECISIONS → print handoff.
- Quality gate: `npm run check` must be green before every commit and at handoff. It runs format, lint, typecheck, tests and the architecture fitness functions; it must exit non-zero on any failure. **Never weaken, skip or bypass a gate rule to make a change pass** — if it must change, that is a DR entry plus a follow-up story.
- Self-review before handoff: walk the review checklist in `docs/REVIEW-CHECKLIST.md` (BLUEPRINT §14.4) over your own diff, story by story, reading the diff rather than your memory of writing it. Fix what fails. Report `reviewed: n items, n fixed, n accepted with reason` in the handoff — never a bare tick.
- Search the repo for an existing implementation before writing a new helper, mapper or utility. Duplicated logic is the most common defect in agent-written code, and you cannot see what you did not read.
- Every external symbol you use must trace to a pinned dependency in BLUEPRINT §13.1. If you are not certain a function, parameter or config key exists in the pinned version, check the docs — do not write it from memory.
- Tests assert concrete expected values and observable behaviour, never internal call sequences. A test that only checks a type, a length or that nothing threw is not a test. Each one must be seen failing before the implementation.
- Keep each story's diff within 400 changed lines. If it will not fit, say so in the plan — the story is mis-sized and needs an amendment, not a bigger diff.
- Copy strings come from the copy tables (`CP-*` IDs) in the blueprint, through `t('<CP-ID>', vars)`. Never improvise user-facing text.
- Acceptance criteria are the spec. If an AC is wrong, impossible or ambiguous: do not silently deviate — record an Amendment (BLUEPRINT §18.7) and, for user-visible/money/data/security changes, ask first.
- Anything not in the blueprint or PROGRESS.md is a decision → `docs/DECISIONS.md` (DR-nn) in the same commit. New dependency = DR entry.
- Stay inside the epic's scope. Ideas for later go to PROGRESS.md "Parking lot".
- Re-check every `[VERIFY-AT-BUILD]` item your epic depends on (BLUEPRINT §18.10) before relying on it; record the result.
- Commit format: `type(scope): message [E03-S02]`. Branch: `epic/E03`.
- Never mark a story or epic DONE without an evidence line per AC in PROGRESS.md.
- Product-specific: emails to clients leave only through the nudging send pipeline (BLUEPRINT E05-S02); webhook handlers never call Resend and call Telegram only to reply; personal data is never logged and is stored only through `platform/crypto` (DR-08); time comes only from `platform/time` (FF-12).

## Commands
- Quality gate: `npm run check`
- Tests only: `npm test` (all projects) or `npx vitest run --project unit`
- Dev server: `npm run dev` (needs Docker Postgres and a local `.env`)
- Migrate: `npm run db:migrate:dev` locally; deployed environments run `npm run db:migrate` as Render's pre-deploy command
- Deploy staging: merge the epic PR into `main`; Render deploys staging after CI passes (see BLUEPRINT §13.8)

## Conventions (summary — full text in BLUEPRINT §13)
- Language/runtime: Node.js 26.10.0 (official nodejs.org build; `typeof Temporal` must be `"object"`) + TypeScript 6.0.3 · Framework: grammY 1.46.0 + Fastify 5.12.5 · DB: PostgreSQL 18.6 via Kysely 0.29.6
- Strict typing on; no untyped escapes; typed errors; no swallowed exceptions; structured logs without PII.
- Tests: unit next to code / integration in `test/integration` / e2e in `test/e2e`; factories in `test/factories`.
- Locale: English only; amounts as "EUR 1,200.00" (ISO code, integer minor units); dates as "Thu 30 Oct 2026" from Temporal in the freelancer's zone; times 24-hour; UTC in storage.
