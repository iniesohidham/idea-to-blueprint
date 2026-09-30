# Review checklist — InvoiceNudge

> Copied verbatim from BLUEPRINT §14.4. Both the agent's self-review and the human review use this list. Changes go through a blueprint Amendment (§18.7).

```
REVIEW CHECKLIST — apply to every story before it is marked DONE.
Stop at the first unanswered item and fix it; do not batch.

Gate
[ ] `npm run check` green on the whole repo, not just the changed files
[ ] No gate rule weakened, skipped or ignored to make this pass (if it was: DR entry, reason)
[ ] No new flaky test; any flake seen is quarantined with a follow-up story

Correctness
[ ] Every AC of this story is demonstrably met, verified against behaviour not description
[ ] Nothing implemented that no AC asked for
[ ] Every listed edge case is covered by a test or explicitly accepted with a reason

Design
[ ] Change sits in the module the architecture assigns it to (BLUEPRINT §12.2); no forbidden dependency direction
[ ] No logic duplicated from elsewhere in the repo (searched before writing)
[ ] Framework's documented path used (grammY, Fastify, Kysely docs), not an invention
[ ] One-way doors (schema on populated tables, public contracts, event formats, token format,
    email headers, new dependency) identified and recorded as a DR

Tests
[ ] Each AC maps to a named test; each was seen failing before implementation
[ ] Tests assert concrete expected values, not types/lengths/no-throw
[ ] Tests assert behaviour, not internal calls; they would survive a refactor
[ ] Deterministic: no wall clock, no network, no order dependence, seeded randomness
[ ] Mutation score on nudging/domain, invoices/domain, platform/money, platform/time not reduced

Failure behaviour
[ ] Empty / one / many / max / boundary all specified and covered
[ ] Dependency timeout, 5xx and partial response each have defined behaviour (Telegram, Resend, Postgres)
[ ] Repeat submission is idempotent where it must be; retries are safe
[ ] Nothing is left in a half-written state on failure

Security
[ ] Input validated at the trust boundary (webhook secret, Svix signature, token HMAC, zod schemas)
[ ] Authorization checked per object (freelancer_id scoping), not only authentication
[ ] Queries parameterised; output encoded for its sink (Telegram HTML, email HTML, page HTML)
[ ] No secret or PII in code, logs, error payloads or fixtures
[ ] Rate limits on anything that costs money, sends a message, or can be brute-forced
[ ] Every external symbol traces to a pinned dependency in the stack table; no invented APIs
[ ] Any new dependency: recognised, maintained, license-compatible, DR entry written

Scale
[ ] No query inside a loop; no unbounded result set; indexes exist for filtered/sorted columns
[ ] Behaviour at 100x current data is stated and acceptable (500 invoices per freelancer, 3,000 emails/day)

Observability
[ ] Decision points logged, structured, correlation ID present, no PII
[ ] Outcome metric emitted; errors reach error tracking with context
[ ] This change is diagnosable by someone who did not write it

Copy, locale, accessibility [UI stories: bot messages and client pages]
[ ] All strings from the copy table by ID; every screen state has its line
[ ] Amounts, dates and times follow BLUEPRINT §10.5 (ISO code amounts, "Thu 30 Oct 2026", 24-hour)
[ ] Buttons labelled in words; pages: focus visible, one h1, labels, contrast, targets >= 44 px

Shape
[ ] Diff is within 400 changed lines, or the story is split
[ ] A human can explain every non-obvious decision in this change
[ ] Handoff summary written: what changed, what was skipped, deviations, ACs satisfied
```
