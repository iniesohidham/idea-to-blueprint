# InvoiceNudge — Build Blueprint

> A Telegram bot that tracks freelancers' invoices and emails polite, escalating reminders to late clients on the freelancer's behalf.
> "InvoiceNudge" is a working name (see OQ-01).

| Field | Value |
|---|---|
| Version | 1.0 |
| Status | APPROVED (decision brief approved with "go" on 2026-09-30) |
| Date | 2026-09-30 |
| Author | idea-to-blueprint skill, run by Claude Opus 5.5 (1M context) for the product owner |
| Target build agents | Claude Code and Codex, one epic per fresh session |
| Research | 30+ web searches, 60+ page fetches, direct reads of the npm, PyPI, Node.js, GitHub and ISO 4217 registries; every external fact tagged and listed in §22 |
| Input idea | "I want a Telegram bot that helps freelancers track invoices and nudges late clients." |

## 0. How to use this document (How to use)

This is the only specification for InvoiceNudge. A build session that starts with an empty context has two sources of truth: the repository and this file. Anything missing here becomes a guess in code, so this file tries to leave nothing to guess.

**Who reads what**

| Reader | Read | Why |
|---|---|---|
| Product owner (human) | §1–§7, §19–§21 | What is being built, for whom, what we assumed, what is still open |
| Build session (coding agent) | §12–§18, §21, and the epic in play in §17, plus every flow (F-), screen (SCR-), copy (CP-), entity (ENT-), endpoint (API-), integration (INT-) and edge case (EC-) that epic references | Everything needed to build one epic without guessing |
| Reviewer of a story | §14.4 (review checklist) and the story's acceptance criteria | One standard for every review |
| Everyone | §21 Glossary | One word per concept |

**Language convention.** The whole document is in English because the product owner wrote in English. Machine-facing text (epics, stories, acceptance criteria, tests, architecture, conventions) is English by rule. Product copy (§11) is English because the product ships in English only in v1 (DR-23). The copy tables keep a "gloss / intent" column so the agent understands what each line is for, not only what it says.

**Evidence legend.** Every claim about the outside world carries one of these tags:

| Tag | Meaning |
|---|---|
| `[VERIFIED — S-nn, date]` | Read this session in the source `S-nn` listed in §22 |
| `[STATED]` | The product owner said it |
| `[ASSUMED — reason]` | A choice made where nothing was stated or found; the reason is given in one clause |
| `[UNKNOWN → OQ-07]` (example) | Could not be verified; routed to the numbered Open Question in §20, which has an owner and a proposed default |
| `[VERIFY-AT-BUILD]` | True today but volatile (versions, prices, platform limits); the build session re-checks it before relying on it (§18.10) |

No claim in this document carries the "unverified" tag the skill reserves for runs without web access: web research was available for the whole run.

**Status and change control.** This document is APPROVED. A build session never silently deviates from it. When reality disagrees with an acceptance criterion, the session records an entry in the Amendments log (§18.7), edits the text in place with `(amended A-nn)`, and asks the product owner first when the change touches user-visible behaviour, money, data retention or security. There is no placeholder text anywhere in this file; anything undecided is tagged UNKNOWN and routed to a numbered Open Question with a proposed default. <!-- lint-ignore -->

**ID schemes used throughout.** Sources `S-nn` · personas `P0–P4`, anti-personas `AP1–AP4` · goals `G1–G5`, metrics `M-n.n` (product) and `M-1n/M-2n` (engineering) · flows `F-nn` · screens `SCR-nn` · copy `CP-SCRnn-n`, `CP-EML-n`, `CP-CMD-n` · entities `ENT-Name` · endpoints `API-nn` · integrations `INT-name` · architecture drivers `AD-n` · fitness functions `FF-nn` · decision records `DR-nn` · edge cases `EC-CAT-nn` · epics `E00–E08` · stories `Enn-Snn` · acceptance criteria `AC-n` (scoped to a story) · tasks `Enn-Snn-Tn` · risks `R-nn` · open questions `OQ-nn` · amendments `A-nn`. IDs never change once assigned; a dropped item keeps its ID with a one-line retirement note.

**Table of contents**

- [0. How to use this document](#0-how-to-use-this-document-how-to-use)
- [1. Executive summary](#1-executive-summary)
- [2. Idea interpretation and scope](#2-idea-interpretation-and-scope-scope)
- [3. Research findings](#3-research-findings-research)
- [4. Assumptions and decision records](#4-assumptions-and-decision-records-assumptions--decisions)
- [5. Personas](#5-personas)
- [6. Goals, non-goals and success metrics](#6-goals-non-goals-and-success-metrics-goals--metrics)
- [7. User flows](#7-user-flows-flows)
- [8. Screen inventory and navigation](#8-screen-inventory-and-navigation-screens)
- [9. Design direction](#9-design-direction-design)
- [10. UX writing guide](#10-ux-writing-guide-ux-writing)
- [11. Copy tables](#11-copy-tables-copy)
- [12. Architecture](#12-architecture)
- [13. Tech stack and engineering conventions](#13-tech-stack-and-engineering-conventions-tech-stack)
- [14. Quality strategy, review and metrics](#14-quality-strategy-review-and-metrics-quality)
- [15. Edge-case register](#15-edge-case-register-edge-cases)
- [16. Delivery plan](#16-delivery-plan--sprints-and-epic-order-delivery-plan)
- [17. Epics and stories](#17-epics-and-stories-epics) — [E00](#epic-00--foundation) · [E01](#epic-01--onboarding-and-account) · [E02](#epic-02--clients-and-invoices) · [E03](#epic-03--owed-overview-and-payments) · [E04](#epic-04--nudge-scheduling-and-evening-digest) · [E05](#epic-05--client-email-nudges-with-baseline-safety) · [E06](#epic-06--operator-tools-and-manual-forwarding) · [E07](#epic-07--settings-export-and-deletion) · [E08](#epic-08--launch-hardening)
- [18. Session protocol](#18-session-protocol-for-claude-code--codex-session-protocol)
- [19. Risks and mitigations](#19-risks-and-mitigations-risks)
- [20. Open questions](#20-open-questions-open-questions)
- [21. Glossary](#21-glossary)
- [22. Sources](#22-sources)
- [Appendix A — Session prompts](#appendix-a--session-prompts-appendix-session-prompts)
- [Appendix B — Files to create in Epic 0](#appendix-b--files-to-create-in-epic-0-appendix-epic-0-files)

## 1. Executive summary

**What it is.** InvoiceNudge is a Telegram bot for freelancers. The freelancer logs each invoice they send (client, amount, due date, reference) in a short chat exchange. The bot shows who owes what. When an invoice goes overdue, it emails the client polite, escalating reminders on the freelancer's behalf: from our sending domain, with the freelancer's name on it and replies going straight to the freelancer. Reminders stop as soon as the invoice is marked paid, the client says it is paid, or the client opts out.

**For whom.** English-speaking freelancers anywhere in the world who already live in Telegram (P1, P2), and the people at their clients who receive the reminders (P3). The product is operated by one person (P4) and built by coding agents (P0).

**Outcome sentence.** After logging an invoice in under a minute, a freelancer (P1) gets paid without writing a single chasing email, and never has a client reminded about an invoice that is already paid, instead of keeping a spreadsheet and chasing by hand weeks after the due date.

**Why it matters.** Late payment is the norm, not the exception. 85% of freelancers are paid late at least some of the time, and just over 21% are paid late (or not at all) more than half the time [VERIFIED — S-03, 2026-09-30]. Across 100,000+ freelancers' invoices, 29% are paid after the due date. Over 75% of those late invoices are paid within 14 days of the due date, and 90% within a month [VERIFIED — S-02, 2026-09-30]. So a prompt, polite reminder in the first two weeks is where most of the money moves.

**How it is delivered.** A Telegram chat bot with inline keyboards, no Mini App (DR-01). Client reminders go out by email (DR-02), because a Telegram bot cannot start a conversation with someone who has never messaged it [VERIFIED — S-82, 2026-09-30]. Three tiny web pages let a client say "I've already paid" or stop reminders.

**What makes it different.** Full invoicing suites (FreshBooks, Zoho Invoice, Bonsai, Invoice Ninja) already automate reminders, but inside web apps that also want to issue your invoices. Accounts-receivable tools like Chaser start at US$259/month [VERIFIED — S-19, 2026-09-30]. The one Telegram-native competitor found, invioTrack, generates PDF invoices. Its site does not say clearly who receives its follow-ups [VERIFIED — S-16, 2026-09-30]. InvoiceNudge only tracks and chases. It works with whatever invoicing the freelancer already does, and it is built around trust: an evening preview of tomorrow's reminders with a one-tap Hold, a client-side "I've already paid" button, and one-click opt-out.

**v1 includes.** Onboarding with a verified reply-to email. Clients. Invoice logging in any ISO currency. A "who owes me" overview with totals per currency. Full and part payments. Reminder plans (Gentle, Standard, Firm, Off). An evening digest with Hold. Automatic reminder emails. Client "I've paid" and "stop" pages. One-click unsubscribe. Bounce and complaint handling with sending caps. Operator admin commands. Text to forward by hand. Settings, data export and account deletion.

**v1 excludes.** Issuing invoices or PDFs, taking payments, late fees and interest, accounting and tax, SMS/WhatsApp/Telegram messages to clients, teams, a Mini App or web dashboard, languages other than English, currency conversion, recurring invoices, reading client replies, and AI-written messages (details and reasons in §2).

**Build plan in one line.** 9 epics (E00–E08, 47 stories), about 9 half-day sessions in 5 sprints. The first releasable milestone is a tracker-only closed beta after E03, and reminders go live for allow-listed beta users after E05.

## 2. Idea interpretation and scope (Scope)

**The idea as written** [STATED]:

> "I want a Telegram bot that helps freelancers track invoices and nudges late clients."

**Interpretation.** "Track invoices" means recording invoices the freelancer has already issued elsewhere. It does not mean generating invoice documents, because invoice issuing is regulated country by country and increasingly means structured e-invoices: for example, mandatory Peppol e-invoicing for Belgian VAT-registered businesses since 2026-01-01 [VERIFIED — S-71, 2026-09-30]. "Nudges late clients" means the bot contacts the client, not only the freelancer. Telegram forbids bots from starting conversations [VERIFIED — S-82, 2026-09-30], so the nudge channel is email sent on the freelancer's behalf, with replies routed to the freelancer. The intake questions were answered "defaults"; every default is recorded as an assumption in §4.1.

**In scope for v1** (capability → persona → epic):

| # | Capability | Personas | Epic |
|---|---|---|---|
| C1 | Onboarding: display name, optional business name, verified reply-to email, time zone, default currency, payment instructions | P1, P2 | E01 |
| C2 | Clients: create, list, edit, archive; per-client reminder eligibility (opt-out, bounce) | P1, P2 | E02 |
| C3 | Invoices: guided logging (client, amount in any ISO 4217 currency, due date, reference, optional link), edit, cancel, delete | P1, P2 | E02 |
| C4 | "Who owes me": overdue / due within 7 days / later, with totals per currency, paginated | P1, P2 | E03 |
| C5 | Payments: mark paid in full (with undo and reopen), record part payments | P1, P2 | E03 |
| C6 | Reminder plans: Gentle / Standard / Firm / Off, per invoice; weekday sending at the freelancer's chosen hour in their time zone | P1, P2 | E04 |
| C7 | Evening digest at 18:00 local: tomorrow's reminders with Hold and Paid buttons; pause, resume, send-now controls | P1 | E04, E05 |
| C8 | Reminder emails: step-specific templates, reply-to the freelancer, signed "I've paid" and "stop" links, RFC 8058 one-click unsubscribe | P1, P3 | E05 |
| C9 | Client pages: "I've already paid" (pauses reminders, asks the freelancer to confirm), "Stop reminders" | P3, P1 | E05 |
| C10 | Deliverability and abuse safety: bounce and complaint suppression, sending caps, complaint-based auto-suspension | P1, P3, P4 | E05, E06 |
| C11 | Operator commands: stats, user lookup, suspend, unsuspend, audit | P4 | E06 |
| C12 | Text to forward: a ready-made reminder the freelancer can paste into WhatsApp or Telegram | P2 | E06 |
| C13 | Settings, data export (CSV), account deletion, privacy page, command menu | P1 | E07 |
| C14 | Launch hardening: restore drill, load test at the scale ceiling, production go-live, alerts | P0, P4 | E08 |

**Out of scope (non-goals), each with its reason**

| Non-goal | Reason |
|---|---|
| Generating invoice documents / PDFs | Country-specific legal content and e-invoicing mandates (S-71). Freelancers already have an invoicing tool; the client already has the invoice |
| Payments: card processing, payment links we create, holding money | Money transmission is out of reach for a one-person operator. Telegram's rules also route digital-goods payments through Stars [VERIFIED — S-07, 2026-09-30]. The freelancer's own payment instructions or link go in the reminder instead |
| Late fees, statutory interest, legal threats in copy | Legal regimes differ, e.g. EUR 40 compensation per late B2B invoice in the EU [VERIFIED — S-05, 2026-09-30] and New York State's Freelance Isn't Free Act, in force since 2024-08-28 [VERIFIED — S-04, 2026-09-30]; copy never mentions them (OQ-09) |
| SMS, WhatsApp or Telegram messages sent to clients | Bots cannot start Telegram conversations (S-82); WhatsApp Business API and SMS add cost, templates and compliance. Manual "text to forward" (C12) covers chat-first clients |
| Teams, multiple users per account, client portals | Single-freelancer product; anti-persona AP1 |
| Mini App or web dashboard | Chat covers v1 jobs; revisit via AD-8 exit trigger |
| Languages other than English; currency conversion | Global English-speaking audience; totals are shown per currency instead of converted |
| Recurring invoices, bank-feed reconciliation, accounting exports | Not needed for the chase outcome; parking lot |
| Reading or parsing client replies (inbound email) | Replies go to the freelancer's own inbox via Reply-To; we never see them |
| AI-generated reminder text | Deterministic, reviewed templates are safer for tone, legality and deliverability |
| Monetization / billing code | v1 is a free beta; pricing is OQ-08 |

**Scale ceiling of v1** [ASSUMED — intake default; sized for a first year of organic growth]:

| Dimension | Ceiling |
|---|---|
| Registered freelancers | 5,000 |
| Weekly active freelancers | 1,000 |
| Open invoices (all users) | 50,000; at most 500 open per freelancer |
| Clients per freelancer | 200 active |
| Reminder emails per day (all users) | 3,000 |
| Telegram updates at peak | 20 per second |
| Regions | 1 (EU, Frankfurt) |

Anything beyond these numbers is outside v1. The architecture exit triggers in §12.1 say what changes if we cross them.

## 3. Research findings (Research)

### 3.1 Domain overview and how the job is done today

**The job.** A freelancer issues an invoice (in an invoicing tool, a Google Doc, a template) and sends it to a client with a due date, often 14 or 30 days out. Then someone has to notice when the date passes and chase: a first polite email, a firmer one, sometimes a final notice. This is accounts receivable "dunning", done by hand by one person who would rather be working.

**The size of the problem.**
- 85% of freelancers have invoices paid late at least some of the time; just over 21% are paid late (or not at all) more than half the time [VERIFIED — S-03, 2026-09-30].
- 29% of freelance invoices are paid after the due date (3 years of data, 100,000+ freelancers). More than 75% of those late invoices are paid within 14 days of the due date and 90% within a month. Invoices over US$20,000 are 3 times more likely to be late than those under US$100 [VERIFIED — S-02, 2026-09-30].
- 49% of companies still manage contractor billing with manual systems such as spreadsheets [VERIFIED — S-03, 2026-09-30].

**How it is done today (the alternatives InvoiceNudge replaces).**
1. **Spreadsheet plus memory.** A sheet of invoices, a calendar reminder, then a hand-written email. Cheap, but the reminder is late, the tone is improvised, and chasing feels awkward, so it is postponed.
2. **The reminder feature of an invoicing suite** (FreshBooks, Zoho Invoice, Bonsai, Invoice Ninja). It works if every invoice is issued in that suite. It lives in a web app the freelancer opens rarely, and it costs a subscription for features they may not need.
3. **Accounts-receivable automation** (Chaser). Built for finance teams, priced for companies.
4. **A Telegram invoicing bot** (invioTrack). Chat-native, but centred on creating PDF invoices.

**Domain vocabulary** (seeds §21 Glossary): invoice, reference (invoice number), due date, overdue, days overdue, balance (amount still owed), part payment, reminder (the product calls it a "nudge" internally and "reminder" in copy), reminder plan, dunning, reply-to, suppression, bounce, complaint, opt-out / unsubscribe, one-click unsubscribe, transactional email.

**Analogous patterns.** Dunning sequences in subscription billing (staged tone over days), "reply-to the human" in transactional email from platforms, and the "digest before action" pattern from scheduling tools (preview tomorrow, veto tonight).

### 3.2 Benchmark

**Players studied (8):** invioTrack (direct, Telegram), FreshBooks, Zoho Invoice, Bonsai, Invoice Ninja (invoicing suites with reminders), Chaser (AR automation), Landolio (reminder templates/tooling for UK freelancers), and the substitute "spreadsheet + email".

**Feature matrix** (✔ = yes, ✘ = no, ◐ = partial, ? = not established this session; every non-? cell traces to the source in its row)

| Player | Chat-native (Telegram) | Tracks invoices issued elsewhere | Issues invoices | Automated reminders to client | Client can say "paid" | Free tier | Entry price | Sources |
|---|---|---|---|---|---|---|---|---|
| invioTrack | ✔ | ? | ✔ (PDF) | ◐ — follow-ups on days 1, 3, 7, 14, 30; recipient and channel not stated clearly | ? | ✔ (3 invoices, 2 clients) | Pro US$20/mo | S-16, S-01 |
| FreshBooks | ✘ | ? | ✔ | ✔ (all plans) + scheduled late fees | ? | ✘ (30-day trial) | Lite US$23/mo (5 billable clients) | S-17 |
| Zoho Invoice | ✘ | ? | ✔ | ✔ | ? | ✔ (500 invoices/yr) | US$0 | S-18 |
| Bonsai | ✘ | ? | ✔ | ? (not listed on pricing page) | ? | ✘ | Basic US$15/user/mo (monthly) | S-22 |
| Invoice Ninja | ✘ | ? | ✔ | ? | ? | ? | ? | S-23 |
| Chaser | ✘ | ◐ (via Xero/QuickBooks/Sage integrations) | ✘ | ✔ (email, SMS, automated calls) | ? | ✘ | Compact US$259/mo | S-19 |
| Landolio | ✘ | ✔ (templates used with any invoice) | ✘ | ? — conflicting sources | ✘ | ✔ (free tools) | Toolkit £19 one-time | S-20, S-21 |
| Spreadsheet + email | ✘ | ✔ | ✘ | ✘ (manual) | ✘ | ✔ | US$0 | — |
| **InvoiceNudge v1** | ✔ | ✔ | ✘ (by design) | ✔ (email, reply-to freelancer) | ✔ | ✔ (beta) | OQ-08 | this document |

**Competitor cards**

*invioTrack* [VERIFIED — S-16, S-01, 2026-09-30]
- What: Telegram bot. /new asks four questions (client, service, amount, due date) and returns a PDF invoice in about 30 seconds; payment tracking by manual marking; English, Spanish, Portuguese; WhatsApp "coming soon".
- Pricing: Free (3 invoices, 2 clients), Pro US$20/month, Business US$40/month (branding, CSV export).
- Built with Python, FastAPI, SQLite on Hetzner; soft-launched May 2026 with no paying customers at launch (founder's post).
- Does best: speed of first invoice; no app to install; deliberately avoids processing payments.
- Unclear: its homepage says follow-ups go out on days 1, 3, 7, 14 and 30 but does not say whether the client or the freelancer receives them, or by what channel. The launch post describes the freelancer sharing the PDF through their own channel.
- Adopt: four-question entry and the "no payment processing" stance. Avoid: forcing invoice creation inside the bot, and ambiguity about who gets reminded.

*FreshBooks* [VERIFIED — S-17, 2026-09-30]
- What: full invoicing and accounting suite. Plans Lite US$23, Plus US$43, Premium US$70/month (regular prices; promotions exist), Select custom. Billable-client limits 5 / 50 / unlimited. All plans include scheduled late fees and automated late-payment reminders; 30-day trial.
- Does best: reminders configured per client and per invoice.
- Adopt: per-invoice reminder control. Avoid: coupling reminders to issuing, and late fees (non-goal).

*Zoho Invoice* [VERIFIED — S-18, 2026-09-30]
- What: free invoicing (up to 500 invoices per year, 2 users) with automated payment reminders; free documents carry "Powered by Zoho Invoice" branding.
- Adopt: a free entry point. Avoid: product branding that competes with the freelancer's own name in client-facing mail. Our footer names the service once, factually (CP-EML-22).

*Bonsai* [VERIFIED — S-22, S-02, 2026-09-30]
- What: freelancer suite (proposals, contracts, invoices); US$15–59 per user per month billed monthly. Its pricing table does not list reminders. Its own dataset is the best public evidence on freelance late payment (S-02).
- Adopt: use its data to set the default reminder timing.

*Invoice Ninja* [VERIFIED — S-23, 2026-09-30]
- What: source-available invoicing app built with Laravel; actively maintained (last push 2026-09-29), about 10.1k GitHub stars. Its pricing page could not be read this session, so its reminder and plan details are not claimed here.

*Chaser* [VERIFIED — S-19, S-69, 2026-09-30]
- What: AR automation for finance teams. US$259 / 779 / 1,169 per month. Email and SMS reminders, automated calls, integrations with Xero, QuickBooks, Sage and 20+ more.
- Does best: a staged reminder playbook, published as a guide (S-69).
- Adopt: staged tone and subject-line rules. Avoid: its price point and team-oriented complexity (AP1).

*Landolio* [VERIFIED — S-20, S-21, 2026-09-30]
- Conflicting sources. A February 2026 blog post on its own site describes an automated email-escalation tool (free for 3 clients, Pro £9/month). Its current homepage sells a one-time £19 "Getting-Paid Toolkit" (37 email templates, an escalation system) plus free tools, with no subscription plan shown. We treat the homepage as current, because it is the newer, primary page.
- Adopt: the demand signal for polite escalation templates.

*Spreadsheet + email* (substitute)
- Zero cost and total control, but reminders happen late and the tone is improvised. This is the true baseline most P1s start from.

**Best-in-class per key flow**

| Our flow | Best-in-class reference | What they do | What we adopt / change |
|---|---|---|---|
| F-02 Log an invoice | invioTrack (S-16) | 4 questions, about 30 s to a finished record | 4-step wizard (client, amount, due date, reference) with smart defaults; target ≤ 60 s for a returning client (M-1.1) |
| F-05 Reminder cadence | Chaser guide (S-69), Zoho ZeptoMail guide (S-70), Bonsai data (S-02) | Pre-due courtesy; +1–3, +7–10, +20–25, +30 days; tone warm → formal | Standard plan +1 / +7 / +14 / +30 days, front-loaded because most late invoices clear within 14 days (S-02); the pre-due courtesy (−3 days) is part of the Firm plan only |
| F-05 Reminder content | S-69, S-70 | Invoice number in the subject; amount, due date, days overdue; one clear payment action; subject under 60 characters; "overdue" from about day 7 | Subject templates CP-EML-2 to CP-EML-6, each ≤ 60 characters at typical values; one "How to pay" block |
| F-05 Send time | S-69, S-70 | Weekday mid-morning in the recipient's time zone | 10:00 on Mon–Fri in the freelancer's time zone by default (client time zone is OQ-13) |
| F-07 Client already paid | none of the benchmarked products (not found) | — | Our differentiator: signed "I've already paid" link that pauses reminders and asks the freelancer to confirm |
| F-08 Opt-out | Gmail sender rules (S-46), RFC 8058 (S-47) | One-click unsubscribe | Per-invoice "stop" link plus RFC 8058 one-click header |

**Anti-patterns observed and what we do instead**
- *Collections-style tone and threats* ("legal action", fees): damages relationships and raises legal questions → never in copy (OQ-09).
- *Value only after you move your invoicing into the tool*: invoicing suites require issuing there → we track invoices issued anywhere.
- *Reminders that go out without the freelancer seeing them first*: no preview was found in the benchmark → evening digest invariant (DR-12).
- *Reminders after the client has paid*: the most embarrassing failure → one-tap "Paid" everywhere, client "I've paid" link, re-check immediately before sending (AD-1).
- *Open and click tracking pixels*: Resend offers `email.opened` and `email.clicked` events [VERIFIED — S-29, 2026-09-30] → we do not enable tracking (privacy, AD-4).

### 3.3 Gap and positioning

**The gap.** Freelancers who live in Telegram have no tool that chases late clients *for them* without first moving their invoicing into a suite. The Telegram-native option centres on making PDFs; the suites centre on issuing; AR tools are priced for finance teams. Nobody benchmarked handles the most anxious moment, a reminder landing after the client has already paid, as a first-class flow.

**Positioning statement.** For freelancers who send invoices however they like and hate chasing, InvoiceNudge is a Telegram bot that emails polite, escalating reminders to late clients on their behalf, with a preview the night before and an "I've already paid" button for the client. Invoicing suites only remind about invoices issued inside them, and a spreadsheet only reminds you.

### 3.4 Regulatory and platform-policy flags (routed to Open Questions; not legal advice)

| Flag | Why it matters | Where it goes |
|---|---|---|
| Telegram Bot Platform Developer Terms: privacy policy required (§4); delete user data on request (§4.2); user data encrypted at rest and stored separately from its key (§4.4); no unsolicited spam (§5.2); digital goods only via Stars [VERIFIED — S-07, 2026-09-30] | Hard requirements for operating the bot | Designed in: DR-08 (field encryption), E07-S04 (deletion), E07-S05 (privacy page); monetization later via Stars (OQ-08) |
| Personal data of third parties (clients' names and emails) processed on behalf of freelancers | GDPR role (controller vs processor), lawful basis, data processing terms, privacy text | [UNKNOWN → OQ-02] |
| Email law: CAN-SPAM exempts transactional or relationship messages from most rules, judged by primary purpose. For commercial mail, both sender and promoter can be liable, up to US$53,088 per email [VERIFIED — S-06, 2026-09-30] | Whether our reminders count as transactional in the US, and how equivalent EU/UK rules treat them | [UNKNOWN → OQ-03]; opt-out is built in regardless (DR-13) |
| Late-payment law (EU Directive 2011/7/EU: EUR 40 per late B2B invoice, member-state interest [VERIFIED — S-05, 2026-09-30]; New York FIFA effective 2024-08-28 [VERIFIED — S-04, 2026-09-30]) | Tempting to mention in copy; legal content | Never mentioned in v1 copy [UNKNOWN → OQ-09] |
| Gmail sender guidelines: SPF or DKIM, PTR, TLS and spam rate below 0.3% for all senders; bulk senders (5,000+/day) also need SPF and DKIM, DMARC and one-click unsubscribe for marketing mail [VERIFIED — S-46, 2026-09-30] | Deliverability of every reminder | AD-5, E08-S03, DR-13 |
| E-invoicing mandates (e.g. Belgium, S-71) | Reason we do not issue invoices | Non-goal (§2) |

### 3.5 UX benchmark observations

- **Speed to first value.** invioTrack gets a user to a finished invoice in four questions (S-16). Our onboarding is five short steps (name, email + code, time zone, currency, payment details), then the first invoice. Target: first invoice within 10 minutes of /start for 60% of new users (M-1.2).
- **Bot conversation norms** (Telegram's own guidance): commands should be specific (`/newlocation` beats `/new` with parameters), descriptions brief, and the bot should respond to every message; edit the keyboard in place when a setting toggles rather than posting new messages [VERIFIED — S-12, 2026-09-30]. We adopt all four (CP-SCR20-1 answers everything).
- **Reminder emails** (S-69, S-70): factual subjects with the invoice number, mobile-length subjects under 60 characters, assume good faith first, formal and brief at the final step, one clear payment action.
- **Opt-out mechanics** (S-47): mail receivers pre-fetch links, so any unsubscribe triggered by a plain GET is unsafe. RFC 8058 requires an HTTPS POST with `List-Unsubscribe=One-Click`, DKIM-signed headers, and no redirects or cookies. Our client pages therefore always require a button press (POST); a GET only shows a page (EC-MSG-11).
- **Accessibility.** The current web standard is WCAG 2.2, a W3C Recommendation of 12 December 2024. Its new criteria include 2.5.8 Target Size (Minimum) at level AA [VERIFIED — S-48, 2026-09-30]. It applies to the client pages. Inside Telegram, rendering and screen-reader behaviour belong to the Telegram apps, so the bot's contribution is text-first messages and labelled buttons (§9).

### 3.6 Telegram platform facts the design relies on

| Fact | Value | Evidence |
|---|---|---|
| Current Bot API | 10.3, released 2026-08-24 | [VERIFIED — S-10, S-11, 2026-09-30] [VERIFY-AT-BUILD] |
| grammY support | grammY 1.46.0 supports Bot API 10.3 | [VERIFIED — S-59, 2026-09-30] |
| Bots starting conversations | Not allowed; the user must message the bot first | [VERIFIED — S-82, 2026-09-30] |
| Undelivered update retention | ≤ 24 hours | [VERIFIED — S-10, 2026-09-30] |
| Webhook failure | Non-2xx responses are retried "a reasonable amount of attempts" | [VERIFIED — S-10, 2026-09-30] |
| Webhook authentication | `secret_token` (1–256 chars, `A-Z a-z 0-9 _ -`) echoed in header `X-Telegram-Bot-Api-Secret-Token` | [VERIFIED — S-10, 2026-09-30] |
| Webhook transport | Ports 443, 80, 88, 8443; TLS 1.2+; IPv4 only; from 149.154.160.0/20 and 91.108.4.0/22 | [VERIFIED — S-13, 2026-09-30] [VERIFY-AT-BUILD] |
| Long polling while a webhook is set | Not possible | [VERIFIED — S-08, 2026-09-30] |
| Slow webhook handling | Telegram re-sends the update, so a slow handler causes duplicate processing; grammY's `webhookCallback` times out after 10 s by default | [VERIFIED — S-54, S-58, 2026-09-30] |
| Flood limits | About 1 message/s per chat, 20/min per group, about 30/s broadcast; HTTP 429 carries `retry_after` | [VERIFIED — S-08, S-09, 2026-09-30] |
| Callback data | 1–64 bytes | [VERIFIED — S-14, 2026-09-30] |
| Copy-text button | `copy_text` 1–256 characters | [VERIFIED — S-53, 2026-09-30] |
| Deep-link start parameter | `A-Z a-z 0-9 _ -`, up to 64 characters | [VERIFIED — S-12, 2026-09-30] |
| Command names | Up to 32 characters; per-scope and per-language lists via `setMyCommands` | [VERIFIED — S-12, 2026-09-30] |
| Files | Bots download ≤ 20 MB, upload ≤ 50 MB; download link valid ≥ 1 hour | [VERIFIED — S-08, S-15, 2026-09-30] |
| Message text length | 4,096 characters after entity parsing | [ASSUMED — cited by grammY search results, but the Bot API reference page could not be fully read this session] [VERIFY-AT-BUILD]; the design keeps every message ≤ 3,500 characters, so the exact limit never matters |
| User language | `User.language_code` is an IETF language tag (optional) | [VERIFIED — S-10, 2026-09-30] |
| Old messages | Bots have limited cloud storage; older messages may be removed after processing, so Telegram is never our data store | [VERIFIED — S-82, 2026-09-30] |

## 4. Assumptions and decision records (Assumptions & Decisions)

### 4.1 Assumptions

Rows 1–15 are the intake defaults. The product owner answered the one-batch intake with "defaults", so each was assumed rather than stated. Rows 16–35 are choices made during research and design.

| # | Assumption | Why this was assumed | Impact if wrong |
|---|---|---|---|
| 1 | Users are English-speaking freelancers anywhere in the world; UI is English only | Intake default (market question) [ASSUMED — intake default] | A second language means extracting copy into per-locale catalogues (catalogue design in DR-22 already keys by ID) |
| 2 | B2C / prosumer: the freelancer is both user and future payer | Intake default [ASSUMED — intake default] | Agency or team buyers would need seats and roles (AP1) |
| 3 | v1 scope is C1–C14 in §2; PDFs, payments and direct chat nudges are excluded | Intake default: smallest set that delivers the chase outcome [ASSUMED — intake default] | Adding PDF issuing pulls in per-country invoice law (S-71) |
| 4 | Telegram chat bot only; no Mini App, no web dashboard | Intake default; chat covers every v1 job [ASSUMED — intake default] | A dashboard would add a front-end build; AD-8 keeps the door open |
| 5 | Late clients are nudged by email from our sending domain, with the freelancer's name in the From display and Reply-To set to the freelancer | Intake default; bots cannot start Telegram chats (S-82) [ASSUMED — intake default] | If freelancers insist on sending from their own mailbox, we need Gmail/Microsoft OAuth sending (large scope) |
| 6 | Reminders are sent automatically; the freelancer gets an evening digest at 18:00 local the day before, with Hold | Intake default: trust without per-email approval friction [ASSUMED — intake default] | If users want to approve each email, add an "approve each" mode (setting) |
| 7 | Personas P1–P4 as in §5 (plus P0 builder) | Intake default from benchmark [ASSUMED — intake default] | Missing persona → stories without an owner |
| 8 | Accessibility: text-first bot, every button labelled in words; client pages meet WCAG 2.2 AA | Intake default [ASSUMED — intake default] | A specific mandate (e.g. public sector) would need an audit |
| 9 | Free during the v1 beta; no billing code; paid plan later, likely via Telegram Stars | Intake default; Telegram requires Stars for digital goods (S-07) [ASSUMED — intake default] | Paying from day one adds a billing epic before E05 |
| 10 | The product never touches money; the reminder carries the freelancer's own payment instructions or link | Intake default [ASSUMED — intake default] | Collecting payments means licensing, PCI and refunds |
| 11 | No preferred or forbidden technologies | Intake default; stack chosen by agentic-engineering criteria (§13) [ASSUMED — intake default] | A mandated stack would reopen DR-04 to DR-06 |
| 12 | Managed hosting in an EU region; infrastructure ≤ US$60/month at the scale ceiling | Intake default [ASSUMED — intake default] | Pricing not verifiable this session (OQ-05) |
| 13 | Scale ceiling as in §2 (5,000 registered, 1,000 weekly active, 3,000 emails/day) | Intake default [ASSUMED — intake default] | 10× growth triggers the exit rules in §12.1 |
| 14 | Personal data: encrypted at rest, never in logs, deleted on request, exportable as CSV | Intake default; also required by Telegram's developer terms (S-07) [ASSUMED — intake default] | — (this is the floor, not a choice) |
| 15 | Solo builder with Claude Code / Codex; half-day sessions (3–7 stories per epic); unit + integration everywhere, e2e on 5 critical paths; staging + production; CI on every push | Intake default [ASSUMED — intake default] | Short sessions would split epics into E0xa / E0xb |
| 16 | Working days are Monday–Friday for every user; no reminder is sent on Saturday or Sunday | Common in target markets [ASSUMED — simplest rule; Friday–Saturday weekends exist in some countries] | Users with other weekends get reminders on their weekend; a "working days" setting is in the parking lot |
| 17 | Reminders go out at 10:00 in the freelancer's time zone by default; choosable 08:00–17:00 | Mid-morning weekday sending is the benchmark norm (S-69, S-70) [ASSUMED — benchmark norm] | Low effect; the setting exists |
| 18 | The freelancer's time zone is used, not the client's | Client time zone is unknown at entry [ASSUMED — see OQ-13] | Cross-continent clients may get reminders at night their time |
| 19 | Plans: Standard +1/+7/+14/+30 days; Gentle +3/+10/+21; Firm −3 (pre-due courtesy)/+1/+7/+14/+21; only Firm includes the pre-due courtesy | 75% of late invoices clear within 14 days (S-02); sequences in S-69/S-70 [ASSUMED — derived from benchmark] | Wrong cadence shows up in M-2.1 / M-2.2; presets are data, not code |
| 20 | Two reminders for the same invoice are never less than 2 calendar days apart | Avoids pestering after weekend shifts [ASSUMED — tone protection] | — |
| 21 | Sending caps: 30 reminder emails per freelancer per day (10 for accounts younger than 7 days); 10 new client email addresses per day; 200 active clients | Abuse and deliverability protection (AD-5) [ASSUMED — conservative starting values] | Heavy users hit the cap; the operator can raise it per account |
| 22 | Two spam complaints within 30 days auto-suspend a freelancer's sending until operator review | Gmail spam-rate threshold of 0.3% (S-46) is shared across all senders on our domain [ASSUMED — conservative] | False suspensions annoy good users; the operator reviews within 2 working days |
| 23 | Amounts are displayed with the ISO code prefix, e.g. "EUR 1,200.00" | "$" is ambiguous across USD, CAD, AUD [ASSUMED — clarity over familiarity] | Cosmetic |
| 24 | Dates are displayed as "Thu 30 Oct 2026"; times as 24-hour "10:00" | Unambiguous for a global English audience [ASSUMED — clarity] | Cosmetic |
| 25 | Maximum invoice amount 99,999,999.99 major units | Fits bigint minor units with margin; far above freelance invoices [ASSUMED — sanity bound] | Rare rejection; raise the bound |
| 26 | Email verification: 6-digit code, 15-minute expiry, 5 attempts, 60 s resend cooldown, 5 codes per hour | Common OTP norms [ASSUMED — conventional values] | — |
| 27 | Conversation (wizard) state expires after 24 h idle | Stale half-finished entries confuse [ASSUMED] | — |
| 28 | Client action links ("I've paid", "stop", unsubscribe) are valid for 60 days | Covers the longest plan (+30) with margin [ASSUMED] | Old emails show CP-SCR23-1 |
| 29 | "Paid in full" can be undone for 15 minutes from the message; after that the card offers "Reopen" | Mis-taps are common in chat [ASSUMED] | — |
| 30 | Recovery objectives: RTO ≤ 4 h, RPO ≤ 15 min | One maintainer, no on-call; PITR exists (S-38) [ASSUMED] | Longer outage delays reminders (catch-up rule in DR-21) |
| 31 | The repository is private on GitHub Pro, which includes protected branches for private repos | GitHub Free lacks protected branches in private repos (S-51) [ASSUMED — see OQ-06] | Without it, the gate is advisory only |
| 32 | The operator is the product owner; one admin Telegram ID at launch | Solo operation [ASSUMED] | — |
| 33 | Coverage floor 80% lines / 75% branches; mutation score break threshold 70 on critical modules | Floors against regression, not targets (§14.3) [ASSUMED — starting floors] | Adjusted by DR entry only |
| 34 | CSV exports are UTF-8 with a byte-order mark so spreadsheet apps open non-ASCII names correctly | Common spreadsheet behaviour [ASSUMED — VERIFY-AT-BUILD in E07-S03 manual check] | Garbled names in some apps |
| 35 | No email open or click tracking | Privacy and deliverability [ASSUMED — product principle] | Less insight into client behaviour; acceptable |

### 4.2 Decision records

Architectural decisions that need more room are expanded in §12. Stack rows are expanded in §13.1. Each record uses the format: status · drivers · context · options · decision · consequences (including what the rejected option would have bought) · compliance · reversibility. `AD-n` drivers are defined in §12.0.

#### DR-01 — Telegram chat bot only, no Mini App
**Status** Accepted — 2026-09-30 · Supersedes: none
**Drivers** AD-2 (operability), AD-8 (evolvability)
**Context** Every v1 job (log, list, mark paid, settings) fits in short messages with inline keyboards; Telegram's guidance favours specific commands and editing keyboards in place (S-12).
**Options** (a) chat bot only; (b) bot + Mini App for lists and forms; (c) bot + separate web dashboard.
**Decision** (a).
**Consequences** One interface technology and no front-end build. Cost: long lists are paginated text (8 invoices per page), and bulk editing is not possible. A Mini App would have bought richer tables and forms at the price of a web front end and Mini App authentication.
**Compliance** Manual — review checklist "Design" items; no front-end build tooling may be added without a superseding DR.
**Reversibility** Cheap — the modules' public interfaces are UI-agnostic (§12.2).

#### DR-02 — Client reminders by email, sent from our domain, replies to the freelancer
**Status** Accepted — 2026-09-30
**Drivers** AD-1, AD-5
**Context** Bots cannot message people who have not started them (S-82). Clients are businesses with email. Resend supports `replyTo`, custom headers and idempotency keys [VERIFIED — S-72, S-89, 2026-09-30].
**Options** (a) email from our domain, `From: "<Name> via InvoiceNudge" <nudges@…>`, `Reply-To: <freelancer>`; (b) send from the freelancer's own mailbox via Gmail/Microsoft OAuth; (c) bot drafts only, freelancer sends.
**Decision** (a), plus (c) as a manual fallback (E06-S03).
**Consequences** Replies land in the freelancer's inbox. Our domain's reputation is shared across all users, which is why AD-5 and the caps exist. Option (b) would have bought "from my own address" credibility and no shared reputation, at the price of OAuth verification, provider-specific APIs and far more sensitive scopes.
**Compliance** Unit test `nudging/compose.test.ts::from_and_reply_to` asserts both headers; e2e CE-3 asserts the Reply-To value.
**Reversibility** Medium — templates and tokens are channel-agnostic; adding (b) later is additive.

#### DR-03 — Modular monolith with an in-process scheduler
**Status** Accepted — 2026-09-30
**Drivers** AD-1, AD-2, AD-3, AD-6 — full scoring in §12.1
**Decision** One Node.js process serves the Telegram webhook, the email-events webhook and the client pages, and runs the scheduler and outbox loops. Postgres is the only state store.
**Consequences** One deploy unit and one log stream. Cost: scheduler work shares the event loop with webhook handling (mitigated by small batches; exit trigger in §12.1). A separate worker would have bought isolation of webhook latency from send bursts, at the price of a second service to deploy and monitor.
**Compliance** FF-01 to FF-06 (§12.10) enforce the module boundaries that keep extraction cheap.
**Reversibility** Cheap — `RUN_SCHEDULER=false` plus a second start command splits the process with no code change.

#### DR-04 — Node.js 26 and TypeScript 6.0.3
**Status** Accepted — 2026-09-30
**Drivers** AD-3 (time correctness), AD-8
**Context** Node.js 26 ships the Temporal API enabled by default (released 2026-05-05, LTS from 2026-10-28, end of life 2029-04-30) [VERIFIED — S-60, S-32, 2026-09-30]. TypeScript 6.0 ships Temporal types via `lib: ["esnext"]` [VERIFIED — S-61, 2026-09-30]. TypeScript 7.0.2 is the latest release, but typescript-eslint 8.71.0 declares `typescript >=4.8.4 <6.1.0` as its peer range [VERIFIED — S-31, S-33, 2026-09-30].
**Options** (a) Node 26 + TS 6.0.3; (b) Node 24 LTS + TS 6.0.3 + a Temporal polyfill; (c) Node 26 + TS 7.0.2 with Biome instead of typescript-eslint; (d) Python 3.14 + aiogram (generic score winner, §13.1).
**Decision** (a).
**Consequences** `PlainDate` (due dates), `ZonedDateTime` (send times) and `Instant` (stored timestamps) are distinct compile-time types, so mixing them is a type error rather than a runtime bug. Cost: Node 26 is "Current" until 2026-10-28, and TS 7's roughly 10× faster compiler (S-33) is not used. **Build caveat:** Temporal needs a Node build compiled with Temporal support. A local Homebrew Node 26.3.0 build had `v8_enable_temporal_support: 0` and `typeof Temporal === "undefined"` [VERIFIED — S-91, 2026-09-30], so the project uses official nodejs.org binaries (nvm/fnm locally, `actions/setup-node` in CI), and the app refuses to start without Temporal (E00-S04). If Render's runtime lacks it (V-21), the fallback is `temporal-polyfill` 1.0.5 (S-31) behind `platform/time`, recorded as a DR. (b) would have bought "LTS today" and Render's default runtime (S-62), at the price of a polyfill dependency. (c) would have bought faster type checks, at the price of losing typescript-eslint's type-aware rules. (d) is recorded in §13.1.
**Compliance** `.node-version` = `26.10.0`, `engines.node` = `>=26.10.0 <27`; CI reads `.node-version`; `typescript` pinned exactly to `6.0.3`.
**Reversibility** Cheap — version bumps; revisit TS 7 when typescript-eslint supports it (R-06).

#### DR-05 — PostgreSQL 18 with Kysely and hand-written migrations
**Status** Accepted — 2026-09-30
**Drivers** AD-1, AD-2, AD-4
**Context** PostgreSQL 18.6 is current (supported until 2030-11-14) [VERIFIED — S-37, 2026-09-30]; `uuidv7()` is built in [VERIFIED — S-84, 2026-09-30]; Render Postgres encrypts at rest with AES-256 and offers PG 13–18 [VERIFIED — S-83, 2026-09-30]. Kysely 0.29.6 supports `forUpdate()` and `skipLocked()` [VERIFIED — S-35, 2026-09-30] and ships a locking migrator [VERIFIED — S-34, 2026-09-30]. `kysely-codegen --verify` fails when generated types are stale [VERIFIED — S-36, 2026-09-30].
**Options** (a) Kysely + pg + kysely-codegen; (b) Drizzle ORM 0.45.3; (c) Prisma 7.10; (d) SQLite.
**Decision** (a).
**Consequences** SQL-shaped, typed queries; the schema lives in reviewed migration files; drift is caught mechanically (FF-11). Cost: no schema-in-TypeScript and more hand-written SQL. (b) would have bought TS-defined schemas and generated migrations, but its 1.0 release candidate is in flight (1.0.0-rc.5 alongside stable 0.45.3) [VERIFIED — S-31, 2026-09-30], which means docs/API churn during our build. (c) would have bought the most documentation and built-in drift detection, but the `prisma` CLI's `latest` tag already points to 8.0.0-rc.19 while `@prisma/client` is 7.10.0 [VERIFIED — S-31, 2026-09-30]. That is an install trap for agents. (d) would have bought zero database operations (invioTrack runs SQLite, S-01), at the price of no managed point-in-time recovery and no row-level `SKIP LOCKED` claiming.
**Compliance** FF-06, FF-07, FF-11 (§12.10).
**Reversibility** Medium — queries are isolated in module repositories.

#### DR-06 — Resend as the email provider, with idempotency keys
**Status** Accepted — 2026-09-30
**Drivers** AD-1, AD-5
**Context** Resend honours an `Idempotency-Key` header for 24 hours. The same key and payload returns the original response; a different payload returns 409 [VERIFIED — S-27, 2026-09-30]. The Node SDK sends it via `emails.send(payload, { idempotencyKey })` [VERIFIED — S-89, 2026-09-30]. Webhooks are Svix-signed [VERIFIED — S-30, 2026-09-30]. Test addresses simulate delivered, bounced and complained outcomes [VERIFIED — S-28, 2026-09-30]. Free tier: 3,000 emails/month and 100/day; Pro US$20/month for 50,000 [VERIFIED — S-26, 2026-09-30] [VERIFY-AT-BUILD]. Postmark does not currently support idempotency keys [VERIFIED — S-25, 2026-09-30].
**Options** (a) Resend; (b) Postmark (separate transactional and broadcast streams, S-24); (c) Amazon SES.
**Decision** (a).
**Consequences** A retried send can never produce a second email within the 24 h window (the DR-07 state machine covers the rest). Cost: dependence on a younger vendor. Postmark would have bought a long deliverability track record and stream separation, at the price of building our own duplicate protection with no provider guarantee.
**Compliance** Integration test `nudging/send_pipeline.int.test.ts::retry_uses_same_idempotency_key`.
**Reversibility** Medium — all sending goes through the `EmailGateway` port (FF-13).

#### DR-07 — Exactly-one-email-per-nudge state machine
**Status** Accepted — 2026-09-30
**Drivers** AD-1
**Context** A reminder sent twice, or after payment, is the most damaging failure the product can have.
**Options** (a) nudge rows with a state machine, claimed with `FOR UPDATE SKIP LOCKED`, re-checked right before sending, sent with `Idempotency-Key = nudge/<nudge_id>`; (b) a job-queue library with at-least-once jobs; (c) cron batch that sends everything due.
**Decision** (a) — details in §12.4 (ENT-Nudge) and E05-S02.
**Consequences** Every reminder's lifecycle is visible in one table. A nudge whose outcome is unknown for more than 23 hours becomes `unknown` and is never auto-resent, because the idempotency key expires after 24 h. (b) would have bought retries and backoff for free (pg-boss 12.35.0 exists [VERIFIED — S-31, 2026-09-30]), at the price of a second source of truth for "what is scheduled".
**Compliance** Unit tests on the transition table; integration test `send_pipeline.int.test.ts::two_workers_one_email`; e2e CE-3.
**Reversibility** One-way once production rows exist (schema of `nudge`).

#### DR-08 — Field-level encryption for personal data, with a blind index
**Status** Accepted — 2026-09-30
**Drivers** AD-4
**Context** Telegram's developer terms require user data to be "encrypted at rest and stored separately from its encryption key" [VERIFIED — S-07, 2026-09-30]. Render encrypts disks with AES-256 (S-83), but logical dumps and exports would still be plaintext.
**Options** (a) provider disk encryption only; (b) application-level AES-256-GCM (Node `crypto`) for PII columns, key from the environment, plus an HMAC-SHA-256 blind index for email lookups; (c) per-user keys (crypto-shredding).
**Decision** (b) for: client name, client email, client contact name, freelancer display name, business name, freelancer email, payment instructions, invoice note, conversation data.
**Consequences** A database dump alone reveals no contacts. Cost: no SQL search or sort on encrypted columns. Client lists are decrypted and sorted in memory (≤ 200 per freelancer), and key rotation is a procedure (§12.8). (a) would have bought simplicity; (c) would have bought instant erasure from backups, at the price of key management per user.
**Compliance** FF-14: an integration test scans a `pg_dump`-style `SELECT *` of every table after the e2e suite and asserts no fixture email or name appears in plaintext. Whether this satisfies Telegram's §4.4 wording is confirmed with counsel (OQ-04).
**Reversibility** One-way once data exists (re-encryption migration needed to undo).

#### DR-09 — Webhook delivery with secret token and update de-duplication
**Status** Accepted — 2026-09-30
**Drivers** AD-1, AD-6
**Context** Long polling cannot run while a webhook is set (S-08); zero-downtime deploys briefly run two instances (S-41); slow handling causes Telegram to re-send updates (S-54).
**Options** (a) webhook with `secret_token`, dedupe on `update_id`; (b) long polling (grammY's advice: fine for always-on servers, S-54).
**Decision** (a).
**Consequences** Clean overlap during deploys, and duplicate updates are harmless. Cost: a public HTTPS endpoint (we need one anyway for email events and client pages). (b) would have bought no inbound exposure, at the price of polling conflicts during deploy overlap.
**Compliance** Integration test `web/telegram_webhook.int.test.ts::rejects_missing_secret` and `::same_update_twice_processed_once`.
**Reversibility** Cheap.

#### DR-10 — Money as integer minor units with an ISO 4217 table
**Status** Accepted — 2026-09-30
**Drivers** AD-1
**Context** ISO 4217 List One (published 2026-09-17) gives minor units per currency, e.g. 2 for USD, EUR, GBP and INR; 0 for JPY, KRW, VND and CLP [VERIFIED — S-68, 2026-09-30].
**Decision** `amount_minor bigint` + `currency char(3)`; a generated, committed `currencies.json` from List One; no floats anywhere; display "EUR 1,200.00" via `Intl.NumberFormat('en', { style: 'currency', currency, currencyDisplay: 'code' })`.
**Consequences** Exact arithmetic for balances and part payments. Cost: a parser that respects per-currency decimals (E02-S01). Floats would have bought nothing.
**Compliance** ESLint `no-restricted-syntax` forbids `parseFloat` and `Number(` inside `src/platform/money/**` except the parser's audited function; mutation testing on `platform/money` (break 70).
**Reversibility** One-way once data exists.

#### DR-11 — Time model: UTC instants, local calendar dates, weekday send windows
**Status** Accepted — 2026-09-30
**Drivers** AD-3
**Decision** `timestamptz` in UTC for instants; `date` for due dates (the freelancer's calendar); the freelancer's IANA zone (validated against `Intl.supportedValuesOf('timeZone')` [VERIFIED — S-64, 2026-09-30]); send instants computed with Temporal in `platform/time`. All "now" comes from an injected `Clock`.
**Consequences** "Overdue" is computed against the freelancer's local date. Cost: every time computation goes through one module. Storing local wall-clock times would have bought readability at the price of DST bugs.
**Compliance** FF-12 (no `Date` or `Date.now()` outside `platform/time`); DST tests for Europe/Berlin, America/New_York, Australia/Sydney, Asia/Kolkata and America/Sao_Paulo in E02-S05.
**Reversibility** One-way.

#### DR-12 — No reminder leaves without an evening-digest announcement
**Status** Accepted — 2026-09-30
**Drivers** AD-1
**Decision** A nudge can be sent automatically only if `announced_at` is set, i.e. it appeared in a digest at least 12 hours earlier. Exceptions: the freelancer taps "Send reminder now", or has turned the digest off (then `announced_at` is set at planning time). Unannounced nudges whose time has come are postponed to the next send window after the next digest.
**Consequences** The freelancer always has a chance to Hold. Cost: a newly logged overdue invoice gets its first reminder the day after next at the earliest, unless the freelancer taps Send now.
**Compliance** Unit test `nudging/domain/eligibility.test.ts::unannounced_is_not_sendable`; e2e CE-3.
**Reversibility** Cheap.

#### DR-13 — Client opt-out: per-invoice stop link plus RFC 8058 one-click unsubscribe
**Status** Accepted — 2026-09-30
**Drivers** AD-5
**Context** RFC 8058 requires `List-Unsubscribe` (HTTPS) plus `List-Unsubscribe-Post: List-Unsubscribe=One-Click`, a POST, no redirects or cookies, and DKIM-signed headers [VERIFIED — S-47, 2026-09-30].
**Decision** Every reminder carries both headers (one-click stops all reminders from that freelancer to that address) and a body link to stop reminders for this invoice only. Opt-outs are permanent for that freelancer–address pair in-product.
**Consequences** Clients stay in control and complaint rates stay low. Cost: a freelancer cannot re-enable reminders to an opted-out address; they can still use "text to forward".
**Compliance** Integration tests on API-06 and API-07; e2e CE-5.
**Reversibility** Cheap (policy), but honouring past opt-outs is mandatory.

#### DR-14 — No invoice issuing, no payment processing
**Status** Accepted — 2026-09-30 · Drivers AD-2, AD-4 · Context §2 non-goals (S-07, S-71) · Options (a) track only; (b) generate PDFs; (c) payment links via a processor · Decision (a) · Consequences: smaller legal surface. (b) would have bought parity with invioTrack; (c) would have bought faster payment, at the price of licensing and refunds · Compliance: no PDF or payment SDK may be added without a superseding DR (dependency policy §13.10) · Reversibility: cheap.

#### DR-15 — Design direction: text-first chat UI and plain server-rendered client pages
**Status** Accepted — 2026-09-30 · Drivers AD-2, AD-6 · Decision: bot messages follow §9 rules; client pages are server-rendered HTML with inline CSS, the system font stack and no JavaScript, fonts or CDNs · Consequences: pages weigh under 15 KB and have nothing to break. A CSS framework would have bought faster styling at the price of a build step · Compliance: FF-15 (page size and no external URLs test) · Reversibility: cheap.

#### DR-16 — Identity: Telegram user ID, verified reply-to email, admin allow-list
**Status** Accepted — 2026-09-30 · Drivers AD-4, AD-5 · Context: webhook requests are authenticated by the secret token (S-10); the Telegram user ID in an authenticated update identifies the freelancer · Options: (a) Telegram identity + emailed code to verify reply-to; (b) separate login · Decision (a); operators are listed in `ADMIN_TELEGRAM_IDS` · Consequences: no passwords. Email verification stops anyone setting a victim's address as Reply-To. A separate login would have bought portability off Telegram at the price of a whole auth system · Compliance: authorization tests per command (§12.6) · Reversibility: medium.

#### DR-17 — Hosting on Render, Frankfurt
**Status** Accepted — 2026-09-30 · Drivers AD-2, AD-4, AD-7 · Context: Render offers Frankfurt [VERIFIED — S-39, 2026-09-30], pre-deploy commands for migrations, zero-downtime deploys with health checks, and auto-deploy "After CI Checks Pass" [VERIFIED — S-41, 2026-09-30]; PITR of 3 days (Hobby) or 7 days (Pro workspace) [VERIFIED — S-38, 2026-09-30]; Node version pinned via `.node-version` [VERIFIED — S-62, 2026-09-30]. Free web services sleep after 15 minutes, which would break webhooks (S-40), so a paid instance is required · Options: (a) Render; (b) Fly.io (shared-cpu-1x 512 MB about US$2.96/month [VERIFIED — S-43, 2026-09-30]); (c) a VPS · Decision (a) · Consequences: managed Postgres with PITR and a CI-gated deploy with no ops code. Fly.io would have bought lower compute prices and more regions, at the price of more platform concepts (Machines, separate Postgres product). Render's prices could not be read this session [UNKNOWN → OQ-05] · Compliance: `render.yaml` in the repo · Reversibility: medium.

#### DR-18 — CI on GitHub Actions with protected `main`
**Status** Accepted — 2026-09-30 · Drivers AD-2 · Context: protected branches for private repos need GitHub Pro; Free includes 2,000 Actions minutes/month and Pro 3,000 [VERIFIED — S-51, 2026-09-30] · Decision: private repo on GitHub Pro with `main` protected, requiring the `check` job ([UNKNOWN → OQ-06] for the account) · Consequences: the gate cannot be bypassed by pushing to `main`. A public repo would have bought free protection at the price of exposing the code · Compliance: E00-S07 AC-3 · Reversibility: cheap.

#### DR-19 — Lint with typescript-eslint `strictTypeChecked`, format with Prettier
**Status** Accepted — 2026-09-30 · Drivers AD-1 · Context: typed linting via `parserOptions.projectService: true` [VERIFIED — S-77, 2026-09-30]; `strict-type-checked` is not a "stable" config, so rules may change outside majors (S-77) and typescript-eslint is pinned exactly · Options: (a) ESLint 10.11.0 + typescript-eslint 8.71.0 + Prettier 3.9.9; (b) Biome 2.5.14 alone · Decision (a) · Consequences: floating promises, unsafe `any` flows and non-exhaustive switches are build errors. Biome would have bought one fast binary for format and lint, at the price of a narrower set of type-aware rules · Compliance: `npm run lint` in `check` with `--max-warnings=0` · Reversibility: cheap.

#### DR-20 — Architecture rules enforced with dependency-cruiser
**Status** Accepted — 2026-09-30 · Drivers AD-2 · Context: dependency-cruiser supports forbidden rules with path regexes and group matching (`$1`) for peer folders [VERIFIED — S-76, 2026-09-30]; version 18.4.0 (S-31) · Options: (a) dependency-cruiser; (b) eslint-plugin-boundaries 7.2.0 (S-31) · Decision (a) · Consequences: rules live in one file and run as a separate `check` step with readable reports. (b) would have bought editor-time feedback · Compliance: FF-01 to FF-06 · Reversibility: cheap.

#### DR-21 — Scheduler as a database-claiming poller, no job-queue library
**Status** Accepted — 2026-09-30 · Drivers AD-1, AD-2, AD-3 · Decision: a loop every 30 s claims due rows (nudges, notifications, digests) with `FOR UPDATE SKIP LOCKED` in batches of 50; missed runs are caught up. Nudges more than 24 h late after an outage are re-planned to the next send window, never blasted (CP-SCR15-15) · Consequences: the domain table is the queue. Two instances during deploy overlap cannot double-claim. pg-boss would have bought built-in retries, cron and monitoring, at the price of a parallel job store · Compliance: `scheduler.int.test.ts::overlapping_ticks_claim_disjoint_rows` · Reversibility: cheap.

#### DR-22 — One copy catalogue keyed by CP IDs
**Status** Accepted — 2026-09-30 · Drivers AD-2 · Decision: every user-facing string lives in `src/platform/copy/catalog.ts` keyed by the CP IDs of §11; code calls `t('CP-SCR07-1', vars)`. A test asserts that the catalogue key set equals the CP IDs defined in §11 of `docs/BLUEPRINT.md` · Consequences: copy changes are table edits. Inline strings would have bought speed now and drift later · Compliance: FF-10 · Reversibility: cheap.

#### DR-23 — English only; unambiguous formats
**Status** Accepted — 2026-09-30 · Drivers AD-2 · Decision: English UI; amounts as "EUR 1,200.00"; dates as "Thu 30 Oct 2026"; times 24-hour; `language_code` (S-10) is stored for future localisation but not used · Consequences: one catalogue. Localisation would have bought reach at the price of translation and review · Compliance: `platform/format.test.ts` · Reversibility: cheap (catalogue keyed by ID).

## 5. Personas

Five personas: four humans plus the builder. Personas are referenced by ID in every story. `P0` exists so technical stories keep the same format.

### P0 — Builder (the coding agent and the maintainer reviewing its work)
- **Role**: implements one epic per fresh session from this document; the human maintainer reviews and merges.
- **Context**: starts with an empty context; the repo and this document are the only memory.
- **Goals**: a green `check`, unambiguous acceptance criteria, a handoff the next session can resume from.
- **Pains**: invented APIs, unclear ownership of code, standards that change between sessions.
- **Success looks like**: "I built E04 without asking a single question the document should have answered."

### P1 — Maya, solo freelancer (primary)
- **Role**: independent brand and web designer, 34, based in Lisbon; 5–15 active clients across Portugal, the UK and the US; invoices in EUR, GBP and USD from a Google Docs template or her bank's invoicing tool.
- **Context**: uses Telegram all day on an iPhone and on desktop for friends and two client groups; opens her laptop's invoicing tool once a month.
- **Goals**: know at a glance who owes her what; get paid within a week of the due date without writing chasing emails.
- **Jobs-to-be-done**: "When an invoice passes its due date, help me remind the client without it feeling awkward." "When money arrives, let me tick it off in one tap."
- **Pains and workarounds today**: a spreadsheet she forgets to update; calendar reminders she snoozes; rewrites the same polite email every month; worries a reminder will land after a client has already paid.
- **Tech comfort and reading tolerance**: high comfort with apps, low tolerance for long messages; reads the first line and taps.
- **Language and register**: plain, warm, professional English; no jargon ("dunning", "AR"); no exclamation marks.
- **Accessibility**: uses iOS larger text; relies on button labels, not emoji.
- **Trust concerns**: "Will it email my client something rude, twice, or after they've paid?" "Can a client see it's automated?" "Where does my client list go?"
- **Key scenarios**: (1) sends an invoice to a London agency, logs it in the bot while waiting for the tram (F-02); (2) the evening before a reminder, sees it in the digest and holds it because the client mentioned a delay (F-05, F-06); (3) the client clicks "I've already paid", she checks her bank and confirms (F-07).
- **Success looks like**: "I haven't written a chasing email in three months, and nobody's been annoyed."

### P2 — Arjun, high-volume contractor (secondary)
- **Role**: full-stack contractor running a two-person studio in Bengaluru; 25–40 invoices a month to US and EU clients in USD, EUR and INR; invoices issued from an accounting tool.
- **Context**: Telegram on Android and desktop; wants overviews and totals, not hand-holding; some clients prefer WhatsApp.
- **Goals**: one view of what is outstanding per currency; reminders that run without him; a quick way to nudge chat-first clients himself.
- **Jobs-to-be-done**: "Every Monday, show me what's overdue and how much, per currency." "For clients who ignore email, give me a message I can paste into WhatsApp."
- **Pains**: currency mixing in spreadsheets; part payments that make balances wrong; chasing across email and chat.
- **Tech comfort and reading tolerance**: expert; prefers terse labels and fewer confirmations; tolerates dense lists.
- **Language and register**: English as a working language; wants short, unambiguous wording; ISO currency codes, not symbols.
- **Accessibility**: none specific.
- **Trust concerns**: reminders about part-paid invoices must show the correct balance; data export for his accountant.
- **Key scenarios**: (1) records a part payment of USD 1,500 against a USD 4,000 invoice (F-04); (2) reviews /owed with totals "USD 12,400.00 + EUR 3,150.00 + INR 180,000.00" (F-03); (3) copies a ready-made reminder into WhatsApp for a client who never reads email (F-15).
- **Success looks like**: "My overdue total is down and I don't track any of it by hand."

### P3 — Sam, the client-side payer (receives reminders, never uses the bot)
- **Role**: operations manager at a 30-person startup in London who approves and pays freelancer invoices; sometimes forwards them to an accounts-payable inbox.
- **Context**: reads email on a laptop and phone, dozens of vendor emails a day.
- **Goals**: understand in one glance what is owed, to whom, and how to pay; stop getting reminders once paid.
- **Pains**: vague reminders without an invoice number; reminders that keep coming after payment; no easy way to say "already paid".
- **Tech comfort and reading tolerance**: moderate; skims subject lines; clicks one link at most.
- **Language and register**: neutral business English; respects politeness; dislikes threats and automation that pretends to be a person.
- **Accessibility**: may use a screen reader or high zoom; pages must work at 200% zoom.
- **Trust concerns**: "Is this phishing?" The email must name the freelancer, show the reference and amount, and not ask for credentials or payment on our site.
- **Key scenarios**: (1) receives a friendly reminder, pays that afternoon and clicks "I've already paid" (F-07); (2) the payment is stuck in approval and he replies to the email, which reaches the freelancer (F-05); (3) he no longer works with the freelancer and uses "stop reminders" (F-08).
- **Success looks like**: "It told me exactly what was owed, and when I said it was paid, it stopped."

### P4 — Operator (the product owner running the service)
- **Role**: runs InvoiceNudge alone: deliverability, abuse, support email, deploy approvals.
- **Context**: Telegram on phone; checks /admin once a day; no on-call rota.
- **Goals**: keep the shared sending domain's reputation clean (spam rate far below 0.3%, S-46); spot abuse early; answer support within 2 working days.
- **Pains**: one abusive user can hurt every user's deliverability.
- **Tech comfort**: technical; wants terse, exact output.
- **Language and register**: terse, factual.
- **Trust concerns**: suspending the wrong account; missing a complaint spike.
- **Key scenarios**: (1) receives an auto-suspension alert, reviews the user's stats, keeps the suspension (F-13); (2) reads daily stats (F-13).
- **Success looks like**: "Complaints stay near zero and I spend ten minutes a day on operations."

### Anti-personas (we do not design for them)
- **AP1 — Agencies and finance teams** with AR staff, approvals and multiple users: Chaser serves them (S-19); they need roles and integrations we will not build.
- **AP2 — Bulk senders** who want to email lists or "chase" people who never agreed to pay them. The caps, templates-only copy and verification exist to make the product useless for them.
- **AP3 — Businesses that must issue compliant e-invoices** (e.g. Belgian VAT-registered companies, S-71): they need an invoicing system.
- **AP4 — Users who want to collect card payments inside Telegram**: out of scope (DR-14).

### Persona × capability matrix

| Capability | P0 | P1 | P2 | P3 | P4 |
|---|---|---|---|---|---|
| C1 Onboarding | — | ● | ● | — | — |
| C2 Clients | — | ● | ● | — | — |
| C3 Invoices | — | ● | ● | — | — |
| C4 Who owes me | — | ● | ●● (totals per currency) | — | — |
| C5 Payments | — | ● | ●● (part payments) | — | — |
| C6 Reminder plans | — | ● | ● | ◐ (receives the result) | — |
| C7 Digest and controls | — | ●● | ● | — | — |
| C8 Reminder emails | — | ● | ● | ●● | ◐ |
| C9 Client pages | — | ◐ (notified) | ◐ | ●● | — |
| C10 Deliverability safety | ● | ◐ | ◐ | ● | ●● |
| C11 Operator commands | — | — | — | — | ●● |
| C12 Text to forward | — | ◐ | ●● | — | — |
| C13 Settings, export, deletion | — | ● | ● | — | — |
| C14 Launch hardening | ●● | — | — | — | ● |

●● = primary need · ● = needs · ◐ = indirectly affected · — = not involved

## 6. Goals, non-goals and success metrics (Goals & Metrics)

**Goals**
- **G1 — Logging is effortless** (P1, P2): an invoice is recorded in under a minute, so the tracker is complete.
- **G2 — Late invoices get paid sooner** (P1, P2): reminders go out on time and shorten the days overdue.
- **G3 — Reminders never embarrass the freelancer** (P1, P3): no duplicates, none after payment or opt-out, no complaints.
- **G4 — One person can run it** (P4, P0): deliverability and operations stay healthy without on-call.
- **G5 — Freelancers keep using it** (P1, P2).

**Metrics** (product; engineering metrics are in §14.6)

| ID | Metric | Definition | Measurement | Target | Type | Instrumented by |
|---|---|---|---|---|---|---|
| M-1.1 | Invoice entry time | Median seconds from wizard start (SCR-07 shown) to invoice saved, returning clients only | `invoice.entry_duration_ms` column | ≤ 60 s [ASSUMED — benchmark: about 30 s in invioTrack, S-16] | Leading | E02-S06 |
| M-1.2 | Activation | Share of new freelancers who save a first invoice within 10 minutes of first /start | `freelancer.created_at` vs first `invoice.created_at` | ≥ 60% [ASSUMED] | Leading | E06-S04 |
| M-2.1 | Paid within 14 days of first reminder | Share of invoices with ≥ 1 sent reminder that are marked paid within 14 days of the first reminder | `nudge.sent_at`, `invoice.paid_at` | ≥ 80% [ASSUMED — baseline: 75% of late invoices paid within 14 days of the due date without our reminders, S-02] | Lagging | E06-S04 |
| M-2.2 | Days overdue at payment | Median (paid date − due date) for invoices paid after the due date | `invoice` rows | Trend down month over month [ASSUMED] | Lagging | E06-S04 |
| M-3.1 | Duplicate reminders | Count of nudges with more than one provider message ID, or two `sent` rows for the same invoice and step | `nudge`, `email_event` | 0 (hard invariant) | Guardrail | E05-S02 |
| M-3.2 | Reminders after "already paid" | Share of sent reminders followed by a client "I've paid" whose payment date is before the send | `audit_event` | ≤ 2% [ASSUMED] | Guardrail | E05-S03 |
| M-3.3 | Complaint rate | Spam complaints ÷ delivered reminders, rolling 30 days | Resend `email.complained` (S-29) | < 0.1% (Gmail limit 0.3%, S-46) | Guardrail | E05-S06 |
| M-3.4 | Hard-bounce rate | Bounces ÷ sent, rolling 30 days | `email.bounced` (S-29) | < 2% [ASSUMED] | Guardrail | E05-S06 |
| M-4.1 | On-time sending | Share of sent reminders whose send instant is within 15 minutes of `scheduled_for` | `nudge` | ≥ 99% | Guardrail | E05-S02 |
| M-4.2 | Operator time | Minutes per week the operator spends on operations (self-reported per week) | Weekly note in `docs/OPS-LOG.md` | ≤ 70 [ASSUMED] | Lagging | E08-S04 |
| M-5.1 | Week-4 retention | Share of activated freelancers who log or update an invoice in week 4 after activation | `invoice`, `payment` rows | ≥ 40% [ASSUMED] | Lagging | E06-S04 |

Product metrics are computed from our own database by operator stats queries (E06-S04). No third-party analytics SDK is used (AD-4).

**Non-goals** are listed with reasons in §2. In short: no invoice issuing, no payments, no fees or interest, no direct chat messages to clients, no teams, no Mini App, English only, no conversion, no recurring invoices, no inbound email, no AI-written copy, no billing in v1.

## 7. User flows (Flows)

Flows are the minimum steps that respect each persona's decisions. "Tap" means pressing an inline-keyboard button; "send" means typing a message. Screen IDs are bot message states (SCR-01–SCR-20), client web pages (SCR-21–SCR-24) and email (§11 CP-EML).

### F-01 — First run and onboarding (P1, P2)
**Job** — Get set up so the bot can remind clients in my name.
**Trigger** — The user opens the bot from a link or search and taps Start (`/start`, optionally with a deep-link parameter, which is ignored in v1).
**Preconditions** — Private chat. No `ENT-Freelancer` row for this Telegram user, or onboarding not finished.
**Steps**

| # | Screen | User does | System responds | Copy | Notes |
|---|---|---|---|---|---|
| 1 | SCR-01 | sends /start | creates ENT-Freelancer (onboarding_step = `name`), shows welcome | CP-SCR01-1, CP-SCR01-2 | ≤ 1 s |
| 2 | SCR-02 | taps Set up | asks for the name clients will see; offers the Telegram name | CP-SCR02-1, CP-SCR02-2, CP-SCR02-8 | step 1 of 5 |
| 3 | SCR-02 | taps "Use {telegram_name}" or sends a name | saves the name; asks for a business name | CP-SCR02-5, CP-SCR02-6 | |
| 4 | SCR-02 | sends a business name or taps No business name | saves; moves on | CP-SCR03-1 | step 2 of 5 |
| 5 | SCR-03 | sends an email address | validates it, sends a 6-digit code | CP-SCR03-3, CP-SCR03-4 | code expires in 15 min |
| 6 | SCR-03 | sends the code | verifies it; marks email verified | CP-SCR03-12 | |
| 7 | SCR-04 | sends a city name or taps a region | matches IANA zones; asks to confirm, showing local time | CP-SCR04-1, CP-SCR04-3 | step 3 of 5 |
| 8 | SCR-04 | taps "Yes, that's right" | saves the zone | CP-SCR04-11 | |
| 9 | SCR-05 | taps a currency (or Other + code) | saves the default currency | CP-SCR05-1, CP-SCR05-2 | step 4 of 5 |
| 10 | SCR-05 | sends payment instructions or taps Skip for now | saves; onboarding done | CP-SCR05-5, CP-SCR05-8, CP-SCR05-9 | step 5 of 5 |

Total: 10 messages or taps at minimum, plus reading one email.

**Decision points** — Step 3: Telegram name accepted → skip typing. Step 7: exactly one match → confirm; several → pick list (CP-SCR04-6); none → CP-SCR04-7 and region buttons. Step 10: skip → CP-SCR05-10.
**Failure branches** — Invalid name (CP-SCR02-3, CP-SCR02-4). Invalid email (CP-SCR03-2). Email service down (CP-SCR03-11; state kept at step 5). Wrong code (CP-SCR03-7), expired (CP-SCR03-8), locked (CP-SCR03-9), resend too soon (CP-SCR03-10), hourly cap (CP-SCR03-13). Address previously bounced (CP-SCR03-14). User leaves mid-way: state persists up to 24 h; `/start` resumes with CP-SCR01-6; after 24 h CP-SCR20-5. Unknown input at any step → the step's own error copy; media → CP-SCR20-3.
**Exit** — ENT-Freelancer complete: name, verified email, zone, currency, (optional) payment instructions; `onboarding_step = null`; home keyboard shown.
**Success metric** — M-1.2; onboarding completion time ≤ 3 minutes median [ASSUMED].
**Benchmark note** — invioTrack reaches first value in 4 questions (S-16). We need 5 short steps first, because we email clients in the user's name; the verified reply-to is non-negotiable (DR-16).

### F-02 — Log an invoice (P1, P2)
**Job** — Record an invoice I just sent so it is tracked and chased.
**Trigger** — `/new`, the "New invoice" home button, or the CP-SCR05-9 button after onboarding.
**Preconditions** — Onboarded. Fewer than 500 open invoices.
**Steps**

| # | Screen | User does | System responds | Copy | Notes |
|---|---|---|---|---|---|
| 1 | SCR-07 | sends /new | shows up to 6 recent clients plus New client and Show all clients | CP-SCR07-1, CP-SCR07-2, CP-SCR07-3, CP-SCR07-15 | wizard timer starts (M-1.1) |
| 2 | SCR-07 | taps a client | selects the client | CP-SCR08-1 | new client: steps 2a–2c |
| 2a | SCR-07 | taps New client, sends a name | asks for the email | CP-SCR07-4, CP-SCR07-5 | |
| 2b | SCR-07 | sends the email | checks format, duplicates, suppression, daily new-address cap | CP-SCR07-6 | |
| 2c | SCR-07 | sends a first name or taps Skip | creates ENT-Client | CP-SCR08-1 | |
| 3 | SCR-08 | sends "1200" or "1,200.50 USD" | parses using the default or typed currency | CP-SCR09-1 to CP-SCR09-6 | |
| 4 | SCR-09 | taps "In 14 days" or sends a date | resolves the date in the freelancer's calendar | CP-SCR10-1, CP-SCR10-2 | |
| 5 | SCR-10 | taps "Use INV-007" or sends a reference | checks for a duplicate reference; shows the summary with the reminder plan | CP-SCR10-7, CP-SCR10-10, CP-SCR10-11 | |
| 6 | SCR-10 | taps Save invoice | saves ENT-Invoice, plans nudges | CP-SCR10-20 | |

Returning client: 5 taps and 1 typed amount; target ≤ 60 s (M-1.1).

**Decision points** — Step 2b: an existing client with the same email → CP-SCR07-10 (use existing / create new). Address opted out or bounced → CP-SCR07-14, and reminders are shown as not possible in the summary (CP-SCR10-9). Step 4: past date → CP-SCR09-11. Step 5: duplicate reference → CP-SCR10-4. Step 5: "Reminders: {plan}" → plan picker (CP-SCR10-12 to CP-SCR10-16); "Add invoice link" → CP-SCR10-18.
**Failure branches** — Invalid email (CP-SCR07-8); name too long (CP-SCR07-9); daily new-address cap (CP-SCR07-13); client cap (CP-SCR07-16). Amount errors CP-SCR08-2 to CP-SCR08-6. Date errors CP-SCR09-7 to CP-SCR09-10. Reference too long CP-SCR10-3. Invalid link CP-SCR10-19. Save fails → CP-SCR10-22, wizard data kept, Save can be retried. Wizard idle 24 h → CP-SCR20-5.
**Exit** — ENT-Invoice `open`; nudges planned per plan (E04); summary message shows the first reminder date.
**Success metric** — M-1.1.
**Benchmark note** — We adopt invioTrack's four questions (S-16) but keep the invoice the user already issued elsewhere, with no PDF.

### F-03 — See who owes me (P2, P1)
**Job** — Know what is outstanding, per currency, and what is late.
**Trigger** — `/owed` or the "Who owes me" home button; Monday habit for P2.
**Preconditions** — Onboarded.
**Steps**

| # | Screen | User does | System responds | Copy | Notes |
|---|---|---|---|---|---|
| 1 | SCR-11 | sends /owed | lists open invoices grouped Overdue / Due in the next 7 days / Later, 8 per page, with totals per currency | CP-SCR11-1 to CP-SCR11-10, CP-SCR11-14 | server time ≤ 300 ms at 500 open invoices |
| 2 | SCR-11 | taps Next / Previous | edits the same message to show the next page | CP-SCR11-12, CP-SCR11-13 | edit in place (S-12) |
| 3 | SCR-12 | taps an invoice button | shows the invoice card with actions | CP-SCR12-1 | |

**Decision points** — No open invoices → CP-SCR11-11.
**Failure branches** — Database error → CP-SCR11-16. Stale page button after data changed → re-render with CP-SCR20-2.
**Exit** — The user has seen balances; may continue to F-04, F-06 or F-15.
**Success metric** — M-5.1 (repeat use).
**Benchmark note** — Totals per currency with ISO codes; no conversion (non-goal).

### F-04 — Record a payment (P1, P2)
**Job** — Tick off money that arrived, in full or in part, so reminders stop or show the right balance.
**Trigger** — "Paid in full" or "Part payment" on the invoice card (SCR-12), "Paid {ref}" in the digest (SCR-14), or "Yes, mark as paid" after a client says paid (SCR-15).
**Preconditions** — Invoice `open` and owned by this freelancer.
**Steps**

| # | Screen | User does | System responds | Copy | Notes |
|---|---|---|---|---|---|
| 1 | SCR-12 | taps Paid in full | records ENT-Payment for the balance, sets `paid`, cancels remaining nudges | CP-SCR13-1, CP-SCR13-2 | Undo valid 15 min |
| 1a | SCR-13 | taps Undo (≤ 15 min) | deletes that payment, reopens, re-plans future nudges | CP-SCR13-3 | |
| 2 | SCR-13 | taps Part payment, sends "1500" | records ENT-Payment; balance updated; future reminders show the new balance | CP-SCR13-5, CP-SCR13-8 | amount = balance → treated as full |

**Decision points** — Part amount equal to the balance → full payment path (CP-SCR13-1). Amount greater than the balance → CP-SCR13-7.
**Failure branches** — Unreadable amount CP-SCR13-6; currency code typed CP-SCR13-10; Undo after 15 min CP-SCR13-4; already paid (stale button) CP-SCR13-13; a reminder already claimed for sending when Paid is tapped → that email may still go out (it was handed to the provider); CP-SCR13-1 is still shown and the audit records the race (EC-CONC-07).
**Exit** — Invoice `paid` (or `open` with a lower balance); nudge rows `cancelled` or re-rendered with the new balance at send time.
**Success metric** — M-3.2.
**Benchmark note** — One-tap paid everywhere; the embarrassment case (reminder after payment) is designed out as far as a timing race allows.

### F-05 — Evening digest and the automatic reminder (P1, P3)
**Job** — Let reminders go out on time, with a chance to veto the night before.
**Trigger** — Scheduler: at 18:00 in the freelancer's zone, if any nudge is due before the next digest.
**Preconditions** — The freelancer has the digest on; nudges `planned` for the next day.
**Steps**

| # | Screen | User does | System responds | Copy | Notes |
|---|---|---|---|---|---|
| 1 | SCR-14 | — | sends the digest: one line per nudge due before the next digest; marks them `announced` | CP-SCR14-1, CP-SCR14-2, CP-SCR14-3 | up to 10 lines, then CP-SCR14-11 |
| 2 | SCR-14 | optionally taps Hold {ref} or Paid {ref} | Hold → nudge `skipped`, line edited; Paid → F-04 | CP-SCR14-9, CP-SCR14-10 | |
| 3 | email | — | at the send hour (e.g. 10:00 local, Mon–Fri): re-checks eligibility, sends the email once | CP-EML-1 to CP-EML-22 | Idempotency-Key `nudge/<id>` |
| 4 | SCR-15 | — | tells the freelancer the reminder went out | CP-SCR15-1 | |
| 5 | inbox | P3 reads the email, replies or clicks a link | a reply goes to the freelancer; links → F-07 / F-08 | — | |

**Decision points** — At step 3 the nudge is not sent (and is `cancelled` or postponed) if: the invoice is paid, cancelled or deleted; reminders are paused; the address is suppressed; the freelancer is suspended or deleted; the email is unverified; the nudge was not announced (DR-12); the daily cap is reached (postponed, CP-SCR15-11).
**Failure branches** — Provider 5xx or timeout → retry at +1 min, +5 min, +30 min, +2 h with the same idempotency key; after 4 failed attempts → `failed` + CP-SCR15-2. Outcome unknown for more than 23 h → `unknown` + CP-SCR15-13. Freelancer blocked the bot → the digest and notices are dropped and `bot_blocked_at` is set; reminders still go out, since the freelancer opted in and can unblock (EC-BOT-08). Outage longer than 24 h → CP-SCR15-15.
**Exit** — Nudge `sent` with the provider message ID; next nudge scheduled.
**Success metric** — M-4.1, M-3.1, M-2.1.
**Benchmark note** — Cadence from S-02, S-69, S-70; the digest-before-send invariant is our own (not found in the benchmark).

### F-06 — Pause, resume, send now, change plan (P1)
**Job** — Take control of reminders for one invoice.
**Trigger** — Buttons on the invoice card (SCR-12).
**Preconditions** — Invoice `open`, owned.
**Steps**

| # | Screen | User does | System responds | Copy | Notes |
|---|---|---|---|---|---|
| 1 | SCR-12 | taps Pause reminders | sets `nudges_paused_reason = freelancer` | CP-SCR12-34, CP-SCR12-38 | |
| 2 | SCR-12 | taps Resume reminders | clears the pause; re-plans future steps from today | CP-SCR12-35, CP-SCR12-39 | |
| 3 | SCR-12 | taps Send reminder now → Send now | sends the next step immediately (bypasses the digest) | CP-SCR12-24, CP-SCR12-25, CP-SCR12-26, CP-SCR15-1 | not within 48 h of the last send |
| 4 | SCR-12 | taps More → Change reminder plan → a plan | re-plans the remaining steps | CP-SCR12-41, CP-SCR12-42, CP-SCR12-43 | |

**Decision points** — Resume after a client opt-out → CP-SCR12-36 (refused; toast CP-SCR12-40). Send now within 48 h of the last reminder → CP-SCR12-27. Send now with no remaining step → the button is hidden.
**Failure branches** — Stale button → CP-SCR20-2. Cap reached → CP-SCR15-11.
**Exit** — Nudge rows reflect the new state.
**Success metric** — M-3.2 (holds and pauses prevent after-payment reminders).
**Benchmark note** — FreshBooks-style per-invoice control (S-17), in chat.

### F-07 — Client says "I've already paid" (P3 → P1)
**Job** — (P3) Stop reminders for something already paid; (P1) confirm it.
**Trigger** — P3 clicks "Already paid? Let {name} know" (CP-EML-20) in a reminder.
**Preconditions** — Valid signed token (≤ 60 days old); invoice exists.
**Steps**

| # | Screen | User does | System responds | Copy | Notes |
|---|---|---|---|---|---|
| 1 | SCR-21 | P3 opens the link (GET) | shows the reference, balance and due date, and a confirm button; changes nothing | CP-SCR21-1, CP-SCR21-2, CP-SCR21-3, CP-SCR21-8, CP-SCR21-9 | GET is safe from link scanners |
| 2 | SCR-21 | P3 clicks "I've paid this invoice" (POST) | pauses reminders (`client_says_paid`), notifies the freelancer | CP-SCR21-4, CP-SCR21-5 | idempotent |
| 3 | SCR-15 | P1 taps "Yes, mark as paid" | F-04 full payment | CP-SCR15-5, CP-SCR15-6 | |
| 3a | SCR-15 | P1 taps "Not received yet" | clears the pause; next reminder no earlier than 3 days later | CP-SCR15-7, CP-SCR15-8 | |

**Decision points** — Already reported → CP-SCR21-6. Invoice already paid → CP-SCR21-7. No free-text note from the client in v1 (OQ-15). P1 does not answer → reminders stay paused; the digest shows "Waiting for you" (CP-SCR14-6) every evening until answered.
**Failure branches** — Token invalid or expired → SCR-23 CP-SCR23-1, CP-SCR23-2. Invoice deleted → CP-SCR23-3. Server error → CP-SCR23-4. Too many requests → CP-SCR23-5.
**Exit** — Invoice paused (awaiting the freelancer) or paid.
**Success metric** — M-3.2.
**Benchmark note** — Not found in any benchmarked product; our differentiator.

### F-08 — Client stops reminders or unsubscribes (P3)
**Job** — Stop getting reminders about one invoice, or from this freelancer altogether.
**Trigger** — (a) "Don't want reminders about this invoice?" link (CP-EML-21); (b) the mail client's one-click unsubscribe (RFC 8058 POST to API-07); (c) opening the List-Unsubscribe URL in a browser (GET API-08).
**Preconditions** — Valid token.
**Steps**

| # | Screen | User does | System responds | Copy | Notes |
|---|---|---|---|---|---|
| 1a | SCR-22 | opens the stop link (GET), clicks the button (POST) | pauses reminders for this invoice (`client_opt_out`), cancels its nudges, notifies the freelancer | CP-SCR22-1 to CP-SCR22-5, CP-SCR15-9 | |
| 1b | — | mail client POSTs `List-Unsubscribe=One-Click` | adds ENT-Suppression (freelancer, address, `unsubscribe`); cancels all that freelancer's nudges to the address; notifies the freelancer | CP-SCR15-10 | 200 with an empty body, no redirect (S-47) |
| 1c | SCR-22 | opens the unsubscribe URL, clicks Unsubscribe | same as 1b | CP-SCR22-6, CP-SCR22-7, CP-SCR22-8 | |

**Decision points** — Already stopped → show the done state again (idempotent).
**Failure branches** — As F-07 (SCR-23).
**Exit** — No further email from this freelancer to that address (1b/1c) or about this invoice (1a).
**Success metric** — M-3.3 (opt-out is easier than a complaint).
**Benchmark note** — RFC 8058 (S-47); Gmail bulk-sender norms (S-46).

### F-09 — Bounce and complaint handling (system → P1, P4)
**Job** — Protect deliverability and tell the freelancer what to fix.
**Trigger** — Resend webhook `email.bounced` or `email.complained` (S-29) to API-02.
**Preconditions** — Valid Svix signature (S-30); event not seen before.
**Steps**

| # | Screen | User does | System responds | Copy | Notes |
|---|---|---|---|---|---|
| 1 | — | — | stores ENT-EmailEvent (dedupe on the webhook ID) | — | |
| 2 | SCR-15 | — | bounce → global suppression of the address (reason `bounce`); cancels nudges to it; notifies the freelancer | CP-SCR15-3 | |
| 3 | SCR-15 | — | complaint → suppression for this freelancer; notifies them; counts complaints; 2 in 30 days → suspend sending + operator alert | CP-SCR15-4, CP-SCR15-12, CP-SCR19-7 | |

**Failure branches** — Invalid signature → 400, nothing stored. Unknown message ID → stored, no action. Duplicate delivery → 200, no second action.
**Exit** — Suppression rows and notifications created.
**Success metric** — M-3.3, M-3.4.
**Benchmark note** — Gmail spam-rate threshold 0.3% (S-46).

### F-10 — Manage clients (P1)
**Job** — Fix a client's email or greeting, or archive an old client.
**Trigger** — `/clients` or the home button.
**Steps**

| # | Screen | User does | System responds | Copy | Notes |
|---|---|---|---|---|---|
| 1 | SCR-16 | sends /clients | lists active clients (10 per page) with balances | CP-SCR16-1, CP-SCR16-2, CP-SCR16-17 | |
| 2 | SCR-16 | taps a client | shows the client card with reminder eligibility | CP-SCR16-4 to CP-SCR16-7 | |
| 3 | SCR-16 | taps Edit email, sends a new address | validates, updates, re-plans blocked nudges | CP-SCR16-15 | |
| 4 | SCR-16 | taps Archive → Archive | hides the client from lists; open invoices keep going | CP-SCR16-12, CP-SCR16-13, CP-SCR16-18 | |

**Decision points** — No clients → CP-SCR16-3. New email suppressed → CP-SCR07-14.
**Failure branches** — Invalid email CP-SCR07-8; stale button CP-SCR20-2.
**Exit** — Client updated or archived.
**Success metric** — M-3.4 (bounces fixed).
**Benchmark note** — List → detail, the minimal pattern.

### F-11 — Settings and reply-to change (P1)
**Job** — Adjust time zone, reminder time, default plan, digest, currency, payment details or reply-to email.
**Trigger** — `/settings`.
**Steps** — SCR-17 shows the summary (CP-SCR17-1) and buttons (CP-SCR17-2 to CP-SCR17-11). Each button reuses the onboarding step's screen and copy (name SCR-02, email SCR-03 with CP-SCR17-14, time zone SCR-04 then CP-SCR17-17, currency and payment SCR-05) or its own (reminder time CP-SCR17-12, CP-SCR17-13; digest CP-SCR17-15, CP-SCR17-16; default plan uses CP-SCR10-12 to CP-SCR10-16). Changing the zone or reminder time re-plans all future nudges.
**Failure branches** — As in each reused step. Email change: until the new code is confirmed, the old address stays in use (CP-SCR17-14).
**Exit** — Freelancer updated; nudges re-planned where affected.
**Success metric** — M-4.1 (correct send times).
**Benchmark note** — Edit in place (S-12).

### F-12 — Export and delete my account (P1)
**Job** — Take my data, or remove it all.
**Trigger** — Settings → "Export my data" or "Delete my account".
**Steps**

| # | Screen | User does | System responds | Copy | Notes |
|---|---|---|---|---|---|
| 1 | SCR-18 | taps Export my data | sends 4 CSV files | CP-SCR18-1, CP-SCR18-2, CP-SCR18-11 | 1 export per hour |
| 2 | SCR-18 | taps Delete my account | shows counts and consequences | CP-SCR18-5, CP-SCR18-6, CP-SCR18-7 | |
| 3 | SCR-18 | taps Delete everything | cancels nudges, hard-deletes all rows, anonymises the audit trail | CP-SCR18-9 | irreversible |

**Failure branches** — Export fails CP-SCR18-3; export too soon CP-SCR18-4; delete fails CP-SCR18-10 (transaction rolled back, nothing removed).
**Exit** — CSVs delivered, or the account is gone and /start begins a fresh onboarding.
**Success metric** — Deletion completes within 60 s for the largest account (500 invoices) [ASSUMED].
**Benchmark note** — Telegram developer terms: delete on request (S-07).

### F-13 — Operator moderation (P4)
**Job** — Check health and act on abuse.
**Trigger** — `/admin` from an allow-listed Telegram ID, or an auto-suspension alert.
**Steps** — `/admin stats` → CP-SCR19-1; `/admin metrics` → CP-SCR19-9; `/admin user <telegram_id>` → CP-SCR19-2; `/admin suspend <telegram_id> <reason>` → CP-SCR19-4; `/admin unsuspend <telegram_id>` → CP-SCR19-5. Alerts: CP-SCR19-7.
**Failure branches** — Unknown subcommand CP-SCR19-8; missing arguments CP-SCR19-3; unknown user CP-SCR19-6. A non-admin sending `/admin` gets CP-SCR20-1, exactly as for any unknown command, so the command's existence is not revealed.
**Exit** — Account status changed and audited.
**Success metric** — M-4.2, M-3.3.

### F-14 — Recovery: stale buttons, timeouts, unknown input, blocked bot (all personas)
**Job** — Never leave the user stuck.
**Trigger / behaviour** — Unknown text or command → CP-SCR20-1. Stale inline button → CP-SCR20-2 and a re-render of the current state. Sticker, voice, photo or document → CP-SCR20-3. Unexpected error → CP-SCR20-4 (the transaction is rolled back). Wizard older than 24 h → CP-SCR20-5. Added to a group → CP-SCR20-6, then leave the group. Flooding (> 30 messages per minute) → CP-SCR20-7. Not onboarded → CP-SCR20-8 and resume. Edited message → CP-SCR20-9. `/cancel` → CP-SCR06-7 or CP-SCR06-8. Bot blocked by the user (Telegram 403) → mark `bot_blocked_at`, stop notifications; on the next message from the user, clear it.
**Success metric** — Zero unanswered updates in logs (`bot.update.unhandled` count = 0).

### F-15 — Text to forward (P2)
**Job** — Remind a chat-first client myself, with the right words and numbers.
**Trigger** — Invoice card → More → "Get text to forward".
**Steps** — The bot sends CP-SCR12-45 followed by a separate plain-text message CP-SCR12-46 (so a long press copies or forwards only the draft); the draft uses the current balance and the freelancer's payment instructions. Nothing is sent to the client by us, and no nudge row is created.
**Failure branches** — Stale button → CP-SCR20-2.
**Exit** — The freelancer pastes the text into WhatsApp or Telegram.
**Success metric** — Count of `forward_text.generated` log events (usage signal).
**Benchmark note** — `copy_text` buttons are limited to 256 characters (S-53), too short for a full draft, so the draft is its own message.

## 8. Screen inventory and navigation (Screens)

**Navigation model.** The bot has no persistent screens, only messages. Navigation works three ways:
1. **Commands** in Telegram's menu: `/new`, `/owed`, `/clients`, `/settings`, `/help`, `/cancel` (CP-CMD-1 to CP-CMD-6). `/admin` is not listed in the menu and only answers allow-listed operators.
2. **The home keyboard**, an inline keyboard with four buttons (CP-SCR06-2 to CP-SCR06-5), shown after onboarding, after `/start` for returning users, and after `/cancel`.
3. **Inline buttons** under each message, which edit that message in place (lists, cards, pickers) or start a step.

A "screen" here is a message state: its text, its inline keyboard, and the conversation step it puts the user in. Wizards (onboarding, new invoice, edits) keep their progress in ENT-ConversationState. Any command interrupts the wizard, and `/cancel` clears it. Client pages are four server-rendered HTML pages. There are 24 screens in total, well under the 40-screen scope alarm.

| ID | Screen | Purpose | Personas | Entry points | Primary action | States covered | Copy IDs | Edge cases |
|---|---|---|---|---|---|---|---|---|
| SCR-01 | Welcome | Greet and start onboarding, or greet a returning user | P1, P2 | /start | Set up | new, resumed, returning (0 / 1 / many overdue) | CP-SCR01-1 to CP-SCR01-6 | EC-BOT-03, EC-BOT-10 |
| SCR-02 | Profile name | Name and optional business name for the From line | P1, P2 | onboarding, settings | Send name / Use Telegram name | prompt, too short, too long, business prompt, business too long | CP-SCR02-1 to CP-SCR02-8 | EC-INPUT-01, EC-INPUT-02, EC-INPUT-04 |
| SCR-03 | Reply-to email and code | Verify the email clients reply to | P1, P2 | onboarding, settings | Send code | prompt, invalid, sending, sent, wrong, expired, locked, cooldown, cap, provider down, bounced address, verified | CP-SCR03-1 to CP-SCR03-14 | EC-AUTH-01, EC-AUTH-02, EC-AUTH-03, EC-AUTH-04, EC-AUTH-11 |
| SCR-04 | Time zone | Pick an IANA zone by city or region | P1, P2 | onboarding, settings | Confirm zone | prompt, one match, many, none, region list, paged list, saved | CP-SCR04-1 to CP-SCR04-11 | EC-TIME-10, EC-SEARCH-01 |
| SCR-05 | Currency and payment details | Default currency and how clients pay | P1, P2 | onboarding, settings | Pick currency | prompt, other code, invalid code, payment prompt, too long, skipped, done | CP-SCR05-1 to CP-SCR05-10 | EC-INPUT-02, EC-PAY-06 |
| SCR-06 | Home and help | Main menu, help text, cancel | P1, P2 | after onboarding, /help, /cancel | New invoice | home, help, cancelled, nothing to cancel | CP-SCR06-1 to CP-SCR06-8 | EC-BOT-01 |
| SCR-07 | New invoice: client | Choose or create the client | P1, P2 | /new | Tap a client | recent list, all clients, new name, new email, greeting, invalid, duplicate, cap, suppressed, client limit, open-invoice limit | CP-SCR07-1 to CP-SCR07-17 | EC-INPUT-10, EC-DATA-11, EC-BIZ-01 |
| SCR-08 | New invoice: amount | Amount and currency | P1, P2 | after client | Send amount | prompt, unreadable, ≤ 0, decimals, too large, unknown code | CP-SCR08-1 to CP-SCR08-7 | EC-INPUT-09, EC-INPUT-12, EC-PAY-06 |
| SCR-09 | New invoice: due date | Due date | P1, P2 | after amount | Tap "In 14 days" | prompt, unreadable, ambiguous, too far past, too far future, already overdue | CP-SCR09-1 to CP-SCR09-12 | EC-TIME-09, EC-TIME-11 |
| SCR-10 | New invoice: reference and save | Reference, link, plan, summary, save | P1, P2 | after due date | Save invoice | prompt, too long, duplicate, summary (plan / off / not possible), plan picker, link prompt, invalid link, saved (plan / off), save failed | CP-SCR10-1 to CP-SCR10-23 | EC-INPUT-10, EC-CONC-02 |
| SCR-11 | Who owes me | Grouped open invoices with totals | P1, P2 | /owed, home | Open an invoice | loaded, empty, paged, error, paused and client-says-paid markers | CP-SCR11-1 to CP-SCR11-18 | EC-DATA-01, EC-DATA-03, EC-DATA-05 |
| SCR-12 | Invoice card | Everything about one invoice, plus actions | P1, P2 | /owed, digest, notices | Paid in full | open (each reminder status), paused, opted out, bounced, cancel confirm, delete confirm, send-now confirm and blocked, edit menu, edit saved, conflict, not found, forward text | CP-SCR12-1 to CP-SCR12-46 | EC-CONC-01, EC-BOT-07, EC-PERM-02 |
| SCR-13 | Record payment | Full or part payment, undo, reopen | P1, P2 | card, digest, notices | Paid in full | full, undo, undo expired, part prompt, unreadable, over balance, part recorded, currency typed, reopen, already paid | CP-SCR13-1 to CP-SCR13-13 | EC-CONC-07, EC-INPUT-09 |
| SCR-14 | Evening digest | Tomorrow's reminders, with veto | P1 | scheduler 18:00 local | Hold | list, overflow, waiting-for-you, held, held last step, too late | CP-SCR14-1 to CP-SCR14-12 | EC-TIME-12, EC-CONC-05 |
| SCR-15 | Notices | What happened to reminders and clients | P1, P2 | scheduler, webhooks, client pages | Contextual (Mark as paid, etc.) | sent, failed, bounced, complaint, client says paid, not received, opted out (invoice / all), cap, suspended, unknown outcome, outage catch-up | CP-SCR15-1 to CP-SCR15-15 | EC-MSG-01, EC-MSG-10, EC-INT-01 |
| SCR-16 | Clients | List and edit clients | P1, P2 | /clients, home | Open a client | list, empty, card (allowed / unsubscribed / bounced), edit, archive confirm, archived, paged | CP-SCR16-1 to CP-SCR16-18 | EC-DATA-11, EC-DATA-06 |
| SCR-17 | Settings | Account settings | P1, P2 | /settings, home | Pick a setting | summary, reminder-time picker, saved, email change note, digest on/off, zone changed | CP-SCR17-1 to CP-SCR17-17 | EC-TIME-04, EC-AUTH-12 |
| SCR-18 | Export and delete | Data rights | P1, P2 | settings | Export my data | preparing, done, failed, too soon, delete confirm, deleted, delete failed | CP-SCR18-1 to CP-SCR18-11 | EC-PERM-05, EC-DATA-10 |
| SCR-19 | Operator | Stats, metrics, user lookup, suspend, alerts | P4 | /admin (allow-listed), scheduler alerts | Run a subcommand | stats, metrics, user, usage, suspended, unsuspended, not found, auto-suspension alert, operational alerts, help | CP-SCR19-1 to CP-SCR19-14 | EC-ADMIN-02, EC-ADMIN-03, EC-OPS-06 |
| SCR-20 | System fallbacks | Answer every update | all | any | Contextual | unknown input, stale button, media, error, expired, group, flood, not onboarded, edited, toasts | CP-SCR20-1 to CP-SCR20-12 | EC-BOT-01 to EC-BOT-09 |
| SCR-21 | Client page: I've paid | Client tells the freelancer they paid | P3 | CP-EML-20 link | I've paid this invoice | ask, done, already reported, already paid | CP-SCR21-1 to CP-SCR21-9 | EC-MSG-11, EC-SEC-11 |
| SCR-22 | Client page: stop and unsubscribe | Stop reminders (one invoice) or unsubscribe (all from this freelancer) | P3 | CP-EML-21 link, List-Unsubscribe URL | Stop reminders | ask, done (invoice), ask, done (all) | CP-SCR22-1 to CP-SCR22-9 | EC-MSG-06, EC-MSG-11 |
| SCR-23 | Client page: errors | Explain bad or expired links and failures | P3 | any client link | Reply to the email | expired / invalid, invoice gone, server error, rate limited | CP-SCR23-1 to CP-SCR23-5 | EC-SEC-11, EC-NET-06 |
| SCR-24 | Privacy page | Plain-language privacy summary | P1, P3 | BotFather privacy link, email footer | Read | single state | CP-SCR24-1 to CP-SCR24-6 | EC-PERM-05 |

**Screen notes** (what is above the fold, meaning the first lines of the message plus the keyboard, and the single primary action)

- **SCR-01 Welcome.** Two-sentence value statement and a "2 minutes" expectation; one button, Set up. Returning users get the overdue count (three variants for 0 / 1 / many) and the home keyboard.
- **SCR-02 Profile name.** One question per message. The Telegram first + last name is offered as a one-tap default, so most users type nothing. A step indicator ("Step 1 of 5") ends each onboarding prompt.
- **SCR-03 Reply-to email.** The prompt says why the email matters (replies go there). After sending, the message shows the address and expiry, with two buttons: Send a new code (disabled by copy, not hidden, during the 60 s cooldown) and Change email.
- **SCR-04 Time zone.** A free-text city search against the city part of IANA zone names (`Europe/Berlin` → "Berlin"), plus six region buttons. A match is confirmed by showing the current local time there, which catches wrong picks. Lists show 8 zones per page.
- **SCR-05 Currency and payment details.** Six common currency buttons plus Other. The payment-details prompt gives an example and the 500-character limit; Skip is always available.
- **SCR-06 Home.** A four-button grid in two rows: New invoice (primary, first), Who owes me, Clients, Settings.
- **SCR-07 to SCR-10 New-invoice wizard.** One question per message with a step indicator. The keyboard carries the likely answers (recent clients, date shortcuts, suggested reference). Each step's message is edited to show the answer given, so the chat reads as a record. The final summary (SCR-10) repeats all values and the first reminder date before Save.
- **SCR-11 Who owes me.** First line: totals per currency. Then sections, one line per invoice, with invoice buttons (reference labels) below and Previous/Next paging. 8 invoices per page keeps each message under 1,500 characters.
- **SCR-12 Invoice card.** A 4-line summary, with Paid in full as the first button. Secondary actions: Part payment, Pause/Resume, Send reminder now, Edit, More. Destructive actions (Cancel invoice, Delete) sit behind More and always get a consequence-stating confirmation.
- **SCR-13 Record payment.** An acknowledgement edits the card text and carries a time-limited Undo button.
- **SCR-14 Evening digest.** A header with the count, one line per reminder, and two buttons per line (Hold, Paid). Invoices waiting for the freelancer's confirmation come first.
- **SCR-15 Notices.** A single message per event, with a contextual button when an action is expected.
- **SCR-16 Clients.** List → card; eligibility status in words ("Reminders: stopped — they unsubscribed").
- **SCR-17 Settings.** A summary block, then one button per setting (11 buttons in rows of 2).
- **SCR-18 Export and delete.** Export sends files. Delete shows counts of what will be erased, and "Keep my account" is on the right so it is not the first target.
- **SCR-19 Operator.** Plain text, no keyboards; fixed-width numbers.
- **SCR-20 Fallbacks.** One sentence each, always with a way forward.
- **SCR-21 to SCR-23 Client pages.** Mobile-first single column: heading, a key-value block (reference, amount, due date), one primary button, and the footer. No navigation, no scripts, no cookies.
- **SCR-24 Privacy.** Headings and short paragraphs.

## 9. Design direction (Design)

**Direction: chosen, not reference-based.** No design reference was provided (intake default). The product surface is mostly Telegram, where the client app draws the UI. Our design choices are therefore message structure, keyboard layout and copy. The only pixels we control are four client pages and the reminder email. Direction: text-first, calm, factual, one primary action per message; plain, fast, accessible HTML for the pages (DR-15).

### 9.1 Bot message design rules
- **One message, one purpose.** A message asks one question or reports one outcome. Maximum 1,500 characters for lists; 600 for everything else (well under Telegram's 4,096 limit, §3.6).
- **Formatting.** `parse_mode: "HTML"`. Only `<b>` for key facts (amounts, dates, references) and `<code>` for codes. Every interpolated user value is HTML-escaped by the copy renderer (EC-INPUT-05).
- **Keyboards.** Inline keyboards only (no reply keyboards). Up to 2 buttons per row for actions; up to 3 for short choices (currencies, date shortcuts). The primary action is first (top-left). Destructive actions are never first and always lead to a confirmation.
- **Edit in place.** Paging, toggles, holds and acknowledgements edit the existing message (S-12). New messages are for new events.
- **Emoji.** None in v1 copy. Meaning never depends on emoji or colour.
- **Button labels.** Verbs that name the result ("Save invoice", "Paid in full", "Hold INV-007"), ≤ 24 characters where possible; Telegram truncates long labels.
- **Callback data.** `v1:<action>:<id>` ≤ 64 bytes (S-14), validated server-side (§12.5).
- **Typing indicator.** `sendChatAction("typing")` for any reply expected to take more than 1 s (exports).

### 9.2 Tokens (client pages and email HTML)

| Token | Value | Contrast / note |
|---|---|---|
| `bg` | `#FFFFFF` | — |
| `surface` | `#F6F8FA` | key-value block background |
| `text-primary` | `#1F2328` | 15.80:1 on `bg`; 14.84:1 on `surface` |
| `text-secondary` | `#57606A` | 6.39:1 on `bg`; 6.00:1 on `surface` |
| `accent` (primary button background) | `#0B57D0` | white text on it 6.39:1 |
| `accent-text` | `#FFFFFF` | — |
| `success` | `#1A7F37` | 5.08:1 on `bg` (done-state heading icon-free text) |
| `warning` | `#9A6700` | 4.87:1 on `bg` |
| `danger` | `#CF222E` | 5.36:1 on `bg` |
| `border` | `#D0D7DE` | decorative only (1.45:1); never the only boundary of a control |
| `focus-ring` | `#0B57D0`, 2 px solid, 2 px offset | 6.39:1 on `bg` |

Ratios computed with the WCAG relative-luminance formula for this document.

- **Type.** The system font stack: `system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif`. No web fonts, so no font CDN (DR-15). Scale: h1 24/32 px weight 600; body 16/24 px weight 400; small 14/20 px; key-value labels 14 px `text-secondary`. Two weights (400, 600).
- **Spacing.** Base 4 px; scale 4 / 8 / 12 / 16 / 24 / 32 / 48. Page padding 16 px mobile, 24 px ≥ 600 px. Content max width 560 px, centred.
- **Radius and elevation.** Buttons 8 px radius; key-value block 8 px; no shadows.
- **Breakpoints.** Mobile ≤ 600 px, then desktop (one-column layout at every width).
- **Components (hand-written, no library).** Page heading (h1); paragraph; key-value list (`<dl>`); primary button (`<button type="submit">` full-width on mobile, min height 48 px, width ≥ 44 px, above the WCAG 2.2 AA 2.5.8 minimum target [VERIFIED — S-48, 2026-09-30]); text link; footer.
- **Iconography.** None.
- **Motion.** None.
- **Dark mode.** Not in v1. Pages declare `<meta name="color-scheme" content="light">` so browsers do not auto-invert. Telegram's own dark theme applies to bot messages automatically.
- **Email HTML.** A single 600 px column, the same tokens, inline CSS only, no images, and a plain-text part identical in content (CP-EML lines). Every link is a full visible URL (P3 trust: "is this phishing?").

### 9.3 Accessibility baseline
- **Client pages: WCAG 2.2 level AA** (S-48). `<html lang="en">`; one `<h1>`; a `<title>` from the copy table; `<dl>` for facts; submit buttons with visible text; a visible focus ring; no information by colour alone; text resizable to 200% without loss (single column, no fixed heights); `prefers-reduced-motion` irrelevant (no motion); the confirmation result is announced by rendering a new page with its own `<h1>` (no live regions needed, no JS).
- **Bot.** Every button has a text label. Lists include the facts in the message text, not only in button labels, because screen readers read message text first. No emoji. How Telegram's apps expose inline keyboards to screen readers could not be verified this session [UNKNOWN → OQ-12]; the text-first rule is the proposed default.

### 9.4 Persona-driven design consequences
- P1 reads the first line and taps → the key fact goes first in every message (amount, reference, date); primary action first; step indicators in wizards.
- P2 is an expert with high volume → the list shows 8 lines per page with totals first; confirmations only for destructive actions; the plan choice is remembered as the default.
- P3 is wary of phishing → the email shows reference, amount, due date and freelancer name before any link; links are visible full URLs on our domain; pages ask for nothing and take no payment.
- P4 wants terse output → operator messages are plain text with fixed field order.

### 9.5 Do / don't for this product
- **Do**: one question per message; edit in place; confirm destructive actions with consequences; show the first reminder date at save time; say what did *not* happen on failure ("Nothing was saved").
- **Don't**: emoji or exclamation marks; reply keyboards; multi-message tutorials; sending anything to clients without a digest announcement (DR-12); hover-only or colour-only meaning on pages; tracking pixels; images in email.

## 10. UX writing guide (UX Writing)

### 10.1 Voice
InvoiceNudge sounds like a calm, competent assistant who handles an awkward task quietly. It is plain, specific and kind, and never cute. It says what happened, what it means for your money or your client, and what to do next.

- **Say the outcome, not the mechanism.** "Saved. I'll remind Acme on Thu 30 Oct if it's still unpaid", not "Invoice created successfully".
- **One idea per line.** Titles ≤ 6 words; bodies ≤ 2 short sentences; buttons 1–4 words.
- **Verb-first buttons that name the result.** "Save invoice", "Paid in full", "Hold INV-007", "Send now"; never "OK", "Confirm" or "Continue" alone.
- **Errors = what happened + what to do.** Never blame; never expose internals; always offer an action. On failure, say what did *not* happen ("Nothing was saved", "Nothing was removed").
- **Confirmations describe consequences.** "Delete INV-007 for good? Its payments and reminder history go too. This can't be undone." Buttons: "Delete for good" / "Keep it".
- **Reversible beats confirm.** Paid in full shows Undo for 15 minutes instead of asking "Are you sure?".
- **No filler.** No "Oops", "Please note", "Successfully", "Awesome"; no exclamation marks; no emoji.
- **The bot speaks as "I"** to the freelancer. Reminder emails speak as the freelancer ("I") and are signed with their name, and a factual footer names the service (CP-EML-22). P3 must never be misled about automation: the From line says "via InvoiceNudge".

### 10.2 Tone matrix

| Moment | P1 Maya (solo) | P2 Arjun (volume) | P3 Sam (client) | P4 Operator |
|---|---|---|---|---|
| First run | Warm, sets expectations ("about 2 minutes") | Same copy; he skips the defaults by tapping | — | — |
| Core task success | Outcome + next date ("I'll remind Acme on …") | Same, first line carries the number | Done page confirms what happens next | Terse ack ("Sending suspended for …") |
| Validation error | Specific fix with an example ("like 30 Oct") | Same | — | Usage line |
| System failure | Calm; states nothing was lost; retry | Same | "Nothing was changed"; reply to the email | Raw fact + count |
| Money moments | Amount and reference in bold; balance after part payment | ISO codes, per-currency totals | Amount, reference, due date before any link | — |
| Destructive action | Consequence + two verb buttons | Same | Stop/unsubscribe: consequence, one button | Suspend requires a reason |
| Notifications | One sentence, with the client and reference first | Same | Reminder emails: friendly → neutral → firm → final | Alerts with a next command |

### 10.3 Reminder email tone ladder (P3)
| Step kind | Tone | Rules |
|---|---|---|
| `pre_due` (Firm plan only) | Courtesy | "Just a heads-up"; no "overdue"; "no need to reply" |
| `friendly` | Assume good faith | "may have slipped through"; ask to take a look |
| `neutral` | Factual | States days past due; asks when payment is scheduled |
| `firm` | Direct | States days overdue; gives a "pay by" date (send date + 7 days); invites a reply about blockers |
| `final` | Formal, brief | "final automated reminder"; "pay by" date; no pleasantries; **no threats, fees, interest or legal words** (OQ-09) |

The word "overdue" first appears in the `neutral` step's subject, around day 7 on the Standard plan, following S-69.

### 10.4 Terminology (one word per concept; the full list is in §21)
- **invoice** (not "bill", "request")
- **reminder** in all user-facing copy; **nudge** only in code and internal docs
- **reference** for the invoice number (button label "Reference"; in copy "Ref")
- **owed** / **balance** for what is left to pay; **paid** for fully settled
- **part payment** (not "partial", "instalment")
- **reminder plan** with plan names **Gentle / Standard / Firm / Off**
- **evening plan** for the digest in user copy ("tomorrow's reminders")
- **hold** for skipping one reminder; **pause** for stopping all reminders on an invoice until resumed
- **client** for the business or person who owes money; **contact name** for the person greeted
- **stop reminders** (per invoice) vs **unsubscribe** (all reminders from this freelancer)

### 10.5 Formatting rules
- **Money.** `{amount}`, `{balance}`, `{paid}`, `{totals}` are formatted by `platform/money.format(minor, currency)` → `Intl.NumberFormat('en', { style: 'currency', currency, currencyDisplay: 'code' })`. Output example: "EUR 1,200.00"; JPY with no decimals, "JPY 150,000" (S-68). The separator between code and number is the U+00A0 no-break space that `Intl` emits [VERIFIED — S-91, 2026-09-30]; it is kept (it stops the code and number splitting across lines), and tests assert it explicitly. `{totals}` joins per-currency amounts with " + ", ordered by amount descending, ties by currency code.
- **Dates.** `{due_date}`, `{date}`, `{first_nudge_date}`, `{pay_by_date}` → "Thu 30 Oct 2026" (weekday short, day, month short, year), from a Temporal `PlainDate` in the freelancer's calendar. Relative wording is used only for days overdue ("3 days overdue", "1 day overdue", "due today", "due tomorrow", "due in 5 days").
- **Times.** `{time}`, `{local_time}`, `{send_hour}` → 24-hour "10:00".
- **Counts and plurals.** English singular/plural chosen by `count === 1`. Separate copy IDs exist where the sentence changes shape (CP-SCR01-3/4/5, CP-SCR11-6/7).
- **Emails and URLs.** Shown verbatim. In bot messages they are wrapped in `<code>` to prevent Telegram auto-formatting.
- **Names.** Client and freelancer names are shown exactly as entered (HTML-escaped). `{contact_name}` falls back to the greeting "Hello," (CP-EML-9).
- **Variables** in braces use the names above. Every template is rendered through one function that fails the test suite if a variable is missing (EC-MSG-09).

### 10.6 Message formulas
- **Validation error.** What is wrong + an example of right: "I couldn't read an amount in "{input}". Send a number like 1200 or 1,200.50."
- **Operation failure.** What didn't happen + state of data + action: "I couldn't save that invoice. Nothing was stored — tap Save invoice to try again."
- **Empty state.** What lives here + first action: "No clients yet. They're added when you log an invoice with /new."
- **Success.** Outcome + next: "Saved. I'll remind {client} on {first_nudge_date} if it's still unpaid."
- **Destructive confirm.** Consequence (including who else is affected) + question, with verb buttons.
- **Limit reached.** Limit + when it resets or what to do: "You've reached today's limit of 30 reminders. 4 will go out tomorrow instead."
- **Session expired.** What happened + nothing lost or what was cleared + action: "That conversation timed out after 24 hours, so I cleared it. Start again with /new."

### 10.7 Channel rules
- **Bot messages.** See §9.1. The one-line toasts from `answerCallbackQuery` are ≤ 60 characters.
- **Email.** Subject ≤ 60 characters at typical values (reference ≤ 12 chars, amount ≤ 16 chars), per S-69. Preheader ≤ 90 characters. Plain-text and HTML parts carry identical text. The single primary action is the payment instructions block; the links are secondary and appear after the sign-off.
- **Client pages.** h1 ≤ 8 words; body ≤ 2 sentences; one button.

### 10.8 Copy review checklist (applied to every table in §11)
Every state has a row · no self-praise adjectives · every error names an action · every destructive confirm names the consequence and uses verb buttons · money, dates and times follow §10.5 · register matches the tone matrix · max length set · terms match §10.4 · gloss column states intent.

## 11. Copy tables (Copy)

Rules: code never contains user-facing strings; it calls `t('<ID>', vars)` (DR-22). `\n` marks a line break inside one message. Variables follow §10.5. "Max length" is characters after variable substitution at typical values; the test suite renders every line with the longest fixture values and asserts the limit (EC-L10N-03). The product language is English, so the gloss column states the intent the agent must preserve rather than a translation.

### 11.1 SCR-01 Welcome

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR01-1 | Message | Hi {first_name}. I track who owes you money and send polite reminders to late clients, so you don't have to chase. Setup takes about 2 minutes. | Value + time expectation | Decide whether to invest 2 minutes | New user | 200 | `{first_name}` from Telegram; if absent use "Hi." |
| CP-SCR01-2 | Button | Set up | Start onboarding | Commit | New user | 24 | Primary |
| CP-SCR01-3 | Message | Welcome back, {first_name}. Nothing is overdue. | Reassure | Quick status | Returning, 0 overdue | 80 | Followed by the home keyboard |
| CP-SCR01-4 | Message | Welcome back, {first_name}. 1 invoice is overdue. | Status, singular | Quick status | Returning, 1 overdue | 80 | |
| CP-SCR01-5 | Message | Welcome back, {first_name}. {overdue_count} invoices are overdue. | Status, plural | Quick status | Returning, ≥ 2 overdue | 80 | |
| CP-SCR01-6 | Message | Let's pick up where you left off. | Resume onboarding | Continue | Onboarding unfinished, < 24 h | 60 | Followed by the pending step's prompt |

### 11.2 SCR-02 Profile name

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR02-1 | Prompt | What name should clients see on reminders? Usually your full name. | Ask for the From display name | Look professional | Prompt | 120 | |
| CP-SCR02-2 | Button | Use {telegram_name} | One-tap default | Save typing | Prompt | 40 | Hidden if the Telegram name is empty; truncated at 30 chars + "…" |
| CP-SCR02-3 | Inline error | Please send a name — at least 1 character. | Empty or only invisible characters | Fix input | Invalid: empty | 80 | EC-INPUT-01 |
| CP-SCR02-4 | Inline error | That's {length} characters. Keep it to 60 or fewer. | Too long | Fix input | Invalid: > 60 | 80 | Length counted in graphemes (EC-INPUT-04) |
| CP-SCR02-5 | Prompt | Do you invoice under a business name? It appears next to your name in reminders. | Optional business name | Decide | Business prompt | 120 | |
| CP-SCR02-6 | Button | No business name | Skip | Move on | Business prompt | 24 | |
| CP-SCR02-7 | Inline error | Keep the business name to 80 characters or fewer. | Too long | Fix input | Invalid business | 80 | |
| CP-SCR02-8 | Step indicator | Step {n} of 5 | Progress | Know how much is left | All onboarding prompts | 16 | Appended on a new line to SCR-02 to SCR-05 prompts |

### 11.3 SCR-03 Reply-to email and code

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR03-1 | Prompt | Which email should client replies go to? I'll send reminders from our address, with replies going straight to you. | Explain Reply-To | Trust | Prompt | 160 | |
| CP-SCR03-2 | Inline error | That doesn't look like an email address. Check it and send it again. | Syntax fail | Fix | Invalid | 90 | Also used in edit flows |
| CP-SCR03-3 | Status | Sending a 6-digit code to {email}… | Progress | Wait | Sending | 100 | Replaced by 3-4 or 3-11 |
| CP-SCR03-4 | Message | I've sent a 6-digit code to {email}. It expires in 15 minutes. Send it here. | Next step + expiry | Act | Sent | 140 | |
| CP-SCR03-5 | Button | Send a new code | Resend | Didn't get it | Sent / expired | 24 | |
| CP-SCR03-6 | Button | Change email | Go back | Typo | Sent | 24 | |
| CP-SCR03-7 | Inline error | That code doesn't match. Attempts left: {attempts_left}. | Wrong code | Retry | Wrong | 60 | Label form avoids a plural rule; `{attempts_left}` is 1–4 |
| CP-SCR03-8 | Inline error | That code has expired. Tap Send a new code. | Expired | Recover | Expired | 60 | EC-AUTH-04 |
| CP-SCR03-9 | Inline error | Too many wrong codes. Request a new code in {minutes} minutes. | Locked after 5 wrong | Wait | Locked | 80 | Lock 15 min |
| CP-SCR03-10 | Inline error | You can request a new code in {seconds} seconds. | Cooldown | Wait | Cooldown | 60 | EC-AUTH-03 |
| CP-SCR03-11 | Error | I couldn't send the code right now. Nothing is lost — try again in a few minutes. | Provider down | Retry later | Provider error | 100 | Nothing stored as sent |
| CP-SCR03-12 | Success | Email confirmed. Client replies will go to {email}. | Outcome | Confidence | Verified | 100 | |
| CP-SCR03-13 | Inline error | That's 5 codes in an hour. Try again after {time}. | Hourly cap | Wait | Cap | 70 | EC-AUTH-11 |
| CP-SCR03-14 | Inline error | Emails to {email} bounced before. Use a different address. | Globally suppressed (bounce) | Pick another | Suppressed | 100 | |

### 11.4 SCR-04 Time zone

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR04-1 | Prompt | Which city are you in? I use your time zone to send reminders during your working hours. Type a city, or pick a region. | Why + how | Set once | Prompt | 160 | |
| CP-SCR04-2 | Buttons | Europe · Americas · Asia · Africa · Australia & Pacific · UTC | Region shortcuts | Browse | Prompt | 24 each | "UTC" saves `Etc/UTC` directly |
| CP-SCR04-3 | Confirm | {zone_label} — it's {local_time} there now. Is that right? | Confirm with local time | Catch mistakes | One match | 120 | `{zone_label}` = IANA ID with "_" → " " |
| CP-SCR04-4 | Button | Yes, that's right | Save | Confirm | One match | 24 | |
| CP-SCR04-5 | Button | No, pick again | Back to prompt | Correct | One match | 24 | |
| CP-SCR04-6 | Prompt | I found {count} matches. Which one? | Disambiguate | Pick | Many matches (2–8) | 60 | Buttons show zone labels |
| CP-SCR04-7 | Inline error | I couldn't find "{input}". Try a nearby larger city, or pick a region. | No match | Recover | None | 120 | |
| CP-SCR04-8 | Prompt | Pick your time zone: | Region list header | Pick | Region list | 30 | |
| CP-SCR04-9 | Button | More | Next page | Browse | Region list | 12 | 8 zones per page |
| CP-SCR04-10 | Button | Back | Previous page / regions | Browse | Region list | 12 | |
| CP-SCR04-11 | Success | Time zone set to {zone_label}. | Outcome | Confidence | Saved | 80 | |

### 11.5 SCR-05 Currency and payment details

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR05-1 | Prompt | Which currency do you invoice in most? You can use others per invoice. | Default currency | Set once | Prompt | 100 | |
| CP-SCR05-2 | Buttons | EUR · USD · GBP · CAD · AUD · INR · Other | Common codes | Pick | Prompt | 8 each | |
| CP-SCR05-3 | Prompt | Send the 3-letter currency code, like CHF or BRL. | Other currency | Type code | Other | 70 | Case-insensitive |
| CP-SCR05-4 | Inline error | {input} isn't a currency code I know. Use a 3-letter ISO code like CHF. | Unknown code | Fix | Invalid | 90 | Validated against `currencies.json` (S-68) |
| CP-SCR05-5 | Prompt | How should clients pay you? I'll include this in every reminder. For example, bank details or a payment link. Up to 500 characters. | Payment instructions | Make paying easy | Payment prompt | 180 | Stored encrypted |
| CP-SCR05-6 | Button | Skip for now | Skip | Later | Payment prompt | 24 | |
| CP-SCR05-7 | Inline error | That's {length} characters. Please keep it to 500 or fewer. | Too long | Shorten | Invalid | 80 | |
| CP-SCR05-8 | Success | You're set up. Log your first invoice and I'll keep an eye on it. | Done + next | Start | Done | 100 | |
| CP-SCR05-9 | Button | Log an invoice | Start F-02 | Act | Done | 24 | Primary |
| CP-SCR05-10 | Note | No problem. Reminders will ask clients to use the payment details on your invoice. | Consequence of skip | Understand | Skipped | 110 | Precedes CP-SCR05-8 |

### 11.6 SCR-06 Home, help, cancel

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR06-1 | Message | What would you like to do? | Home prompt | Choose | Home | 40 | |
| CP-SCR06-2 | Button | New invoice | F-02 | Act | Home | 24 | Primary, first |
| CP-SCR06-3 | Button | Who owes me | F-03 | Check | Home | 24 | |
| CP-SCR06-4 | Button | Clients | F-10 | Manage | Home | 24 | |
| CP-SCR06-5 | Button | Settings | F-11 | Adjust | Home | 24 | |
| CP-SCR06-6 | Help | I track invoices and remind late clients by email.\n/new — log an invoice\n/owed — see who owes you\n/clients — your clients\n/settings — time zone, reminders, account\n/cancel — stop what you're doing | Command overview | Learn | /help | 300 | |
| CP-SCR06-7 | Ack | Cancelled. Nothing was saved. | Wizard cleared | Reassure | /cancel with active step | 40 | |
| CP-SCR06-8 | Ack | There's nothing to cancel. | No active step | Reassure | /cancel idle | 40 | |

### 11.7 SCR-07 New invoice: client

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR07-1 | Prompt | Who is this invoice for? | Pick client | Start fast | Recent clients exist | 40 | Up to 6 recent client buttons (by last invoice), labels truncated at 24 chars |
| CP-SCR07-2 | Button | New client | Create client | Add | Always | 24 | |
| CP-SCR07-3 | Button | Show all clients | Paged list, 8 per page | Find | > 6 clients | 24 | Reuses CP-SCR04-9 / CP-SCR04-10 for paging |
| CP-SCR07-4 | Prompt | Client name? (company or person) | Name | Enter | New client | 50 | 1–80 graphemes |
| CP-SCR07-5 | Prompt | Email address that should get reminders for {client}? | Recipient | Enter | New client | 110 | |
| CP-SCR07-6 | Prompt | Who should reminders greet? Send a first name, or skip. | Contact name | Personalise | New client | 80 | 1–40 graphemes |
| CP-SCR07-7 | Button | Skip | Greeting "Hello," | Move on | Contact prompt | 12 | |
| CP-SCR07-8 | Inline error | That doesn't look like an email address. Check it and send it again. | Syntax fail | Fix | Invalid email | 90 | Same text as CP-SCR03-2 |
| CP-SCR07-9 | Inline error | Keep the client name to 80 characters or fewer. | Too long | Fix | Invalid name | 60 | Empty → CP-SCR02-3 |
| CP-SCR07-10 | Prompt | You already have {existing_client} with {email}. Use that client? | Duplicate email | Avoid duplicates | Duplicate | 140 | EC-DATA-11 |
| CP-SCR07-11 | Button | Use {existing_client} | Use existing | Save time | Duplicate | 30 | |
| CP-SCR07-12 | Button | Create new anyway | Allow a duplicate | Different contact at same address | Duplicate | 24 | |
| CP-SCR07-13 | Inline error | You've added 10 new client emails today. You can add more tomorrow, or pick an existing client. | New-address cap | Understand limit | Cap | 120 | Resets at 00:00 freelancer-local |
| CP-SCR07-14 | Warning | {email} can't receive reminders from you (they opted out or the address bounced). You can still track this invoice. | Suppressed address | Understand | Suppressed | 150 | Client is still created |
| CP-SCR07-15 | Step indicator | Step 1 of 4 · Client | Progress | Orientation | Wizard | 24 | Wizard steps use "of 4" |
| CP-SCR07-16 | Inline error | You've reached 200 clients. Archive some in /clients to add more. | Client cap | Recover | Cap | 80 | |
| CP-SCR07-17 | Message | You have 500 open invoices, the maximum. Mark some as paid or cancel them in /owed to add more. | Open-invoice cap | Recover | Invoice cap | 110 | Sent instead of starting the wizard |

### 11.8 SCR-08 New invoice: amount

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR08-1 | Prompt | How much is the invoice for {client}? For example 1200 or 1,200.50 in {currency}. Add a code like USD to use another currency. | Amount entry | Enter | Prompt | 170 | `{currency}` = default |
| CP-SCR08-2 | Inline error | I couldn't read an amount in "{input}". Send a number like 1200 or 1,200.50. | Unparseable | Fix | Invalid | 110 | `{input}` truncated to 20 chars |
| CP-SCR08-3 | Inline error | The amount must be more than 0. | Zero or negative | Fix | Invalid | 40 | |
| CP-SCR08-4 | Inline error | {currency} uses {minor_digits} decimal places. Try {example}. | Too many decimals | Fix | Invalid | 70 | e.g. "JPY uses 0 decimal places. Try 150000." |
| CP-SCR08-5 | Inline error | That's above the 99,999,999.99 limit. Check the amount. | Too large | Fix | Invalid | 60 | |
| CP-SCR08-6 | Inline error | {code} isn't a currency code I know. Use a 3-letter ISO code like EUR. | Unknown code | Fix | Invalid | 80 | |
| CP-SCR08-7 | Step indicator | Step 2 of 4 · Amount | Progress | Orientation | Wizard | 24 | |

### 11.9 SCR-09 New invoice: due date

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR09-1 | Prompt | When is it due? | Due date | Pick | Prompt | 20 | |
| CP-SCR09-2 | Button | In 7 days | today + 7 | Pick | Prompt | 16 | "today" = freelancer-local date |
| CP-SCR09-3 | Button | In 14 days | today + 14 | Pick | Prompt | 16 | |
| CP-SCR09-4 | Button | In 30 days | today + 30 | Pick | Prompt | 16 | |
| CP-SCR09-5 | Button | End of month | last day of the current month | Pick | Prompt | 16 | If today is the last day → last day of next month |
| CP-SCR09-6 | Hint | Or type a date like 30 Oct or 2026-10-30. | Accepted formats | Type | Prompt | 50 | Same message as CP-SCR09-1, second line |
| CP-SCR09-7 | Inline error | I couldn't read "{input}" as a date. Try 30 Oct or 2026-10-30. | Unparseable | Fix | Invalid | 90 | |
| CP-SCR09-8 | Inline error | Is {input} day/month or month/day? Please type it like 30 Oct or 2026-10-30. | Ambiguous numeric date | Fix | Ambiguous | 100 | Any `nn/nn[/nnnn]` or `nn.nn` form |
| CP-SCR09-9 | Inline error | That's more than a year ago. For older debts, talk to the client directly. | > 365 days past | Understand | Too old | 90 | |
| CP-SCR09-10 | Inline error | That's more than 2 years away. Check the date. | > 730 days ahead | Fix | Too far | 60 | |
| CP-SCR09-11 | Note | That date has passed — this invoice is already {days} days overdue. The first reminder will be in tomorrow's plan, not sooner. | Already overdue | Understand timing | Past date | 150 | `{days}` ≥ 1; "1 days" → rendered via CP-SCR11-7 phrasing rule "1 day" |
| CP-SCR09-12 | Step indicator | Step 3 of 4 · Due date | Progress | Orientation | Wizard | 24 | |

### 11.10 SCR-10 New invoice: reference, link, plan, save

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR10-1 | Prompt | Invoice number or reference? Clients see this in reminders. | Reference | Match own records | Prompt | 70 | 1–40 chars |
| CP-SCR10-2 | Button | Use {suggested_ref} | Suggested next reference | Save typing | Prompt | 30 | `INV-` + 3-digit sequence per freelancer (INV-001…) |
| CP-SCR10-3 | Inline error | Keep the reference to 40 characters or fewer. | Too long | Fix | Invalid | 50 | |
| CP-SCR10-4 | Prompt | You already have {ref} for {client}. Save this one too? | Duplicate reference for same client | Avoid mistakes | Duplicate | 100 | Case-insensitive match on open or paid invoices |
| CP-SCR10-5 | Button | Save anyway | Accept duplicate | Proceed | Duplicate | 16 | |
| CP-SCR10-6 | Button | Change reference | Re-enter | Fix | Duplicate | 20 | |
| CP-SCR10-7 | Summary | {client} · {amount}\nRef {ref} · due {due_date}\nReminders: {preset_name} — first on {first_nudge_date} | Final check | Verify | Summary (plan on) | 220 | Bold on amount and date |
| CP-SCR10-8 | Summary line | Reminders: off | Plan off | Verify | Summary (off) | 20 | Replaces line 3 of CP-SCR10-7 |
| CP-SCR10-9 | Summary line | Reminders: not possible — {client} can't receive email from you | Suppressed | Understand | Summary (suppressed) | 110 | Replaces line 3 |
| CP-SCR10-10 | Button | Save invoice | Commit | Finish | Summary | 20 | Primary |
| CP-SCR10-11 | Button | Reminders: {preset_name} | Open plan picker | Adjust | Summary | 24 | |
| CP-SCR10-12 | Prompt | How firm should reminders be? | Plan picker | Choose | Picker | 40 | Also used by CP-SCR12-42 and settings |
| CP-SCR10-13 | Button | Gentle — 3 reminders | +3, +10, +21 days | Choose | Picker | 24 | |
| CP-SCR10-14 | Button | Standard — 4 reminders | +1, +7, +14, +30 days | Choose | Picker | 24 | Default |
| CP-SCR10-15 | Button | Firm — 5 reminders | −3 (courtesy), +1, +7, +14, +21 days | Choose | Picker | 24 | Only plan with the pre-due courtesy |
| CP-SCR10-16 | Button | Off — just track it | No reminders | Choose | Picker | 24 | |
| CP-SCR10-17 | Button | Add invoice link | Optional URL | Enrich | Summary | 20 | |
| CP-SCR10-18 | Prompt | Send the link to the invoice or payment page (https://…). | Link entry | Enter | Link prompt | 70 | |
| CP-SCR10-19 | Inline error | Links must start with https:// and be under 500 characters. | Invalid link | Fix | Invalid | 70 | |
| CP-SCR10-20 | Success | Saved. I'll remind {client} on {first_nudge_date} if it's still unpaid. You'll see each reminder in your evening plan first. | Outcome + next | Confidence | Saved (plan on) | 180 | Home keyboard follows |
| CP-SCR10-21 | Success | Saved. I'll track it; no reminders will be sent. | Outcome | Confidence | Saved (off / suppressed) | 60 | |
| CP-SCR10-22 | Error | I couldn't save that invoice. Nothing was stored — tap Save invoice to try again. | DB failure | Retry | Save failed | 90 | Wizard data kept |
| CP-SCR10-23 | Step indicator | Step 4 of 4 · Check and save | Progress | Orientation | Wizard | 30 | |

### 11.11 SCR-11 Who owes me

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR11-1 | Header | You're owed {totals} across {count} open invoices. | Totals per currency | Big picture | Loaded (count ≥ 2) | 200 | For count = 1: "…across 1 open invoice." (same ID, plural rule) |
| CP-SCR11-2 | Section | Overdue | Group heading | Scan | Loaded | 12 | Bold |
| CP-SCR11-3 | Section | Due in the next 7 days | Group heading | Scan | Loaded | 24 | |
| CP-SCR11-4 | Section | Later | Group heading | Scan | Loaded | 8 | |
| CP-SCR11-5 | Line | {client} · {balance} · {ref} · {status_text} | One invoice | Scan | Loaded | 120 | Client truncated at 24 chars |
| CP-SCR11-6 | Status text | {days} days overdue | ≥ 2 days late | Urgency | Overdue | 20 | |
| CP-SCR11-7 | Status text | 1 day overdue | 1 day late | Urgency | Overdue | 16 | |
| CP-SCR11-8 | Status text | due today | Due date = local today | Timing | Due today | 12 | |
| CP-SCR11-9 | Status text | due in {days} days | Future | Timing | Upcoming | 16 | |
| CP-SCR11-10 | Status text | due tomorrow | Tomorrow | Timing | Upcoming | 12 | |
| CP-SCR11-11 | Empty | Nobody owes you anything right now. Log an invoice with /new when you send one. | Empty state | Next step | Empty | 90 | |
| CP-SCR11-12 | Button | Next | Next page | Browse | > 8 invoices | 12 | |
| CP-SCR11-13 | Button | Previous | Previous page | Browse | page > 1 | 12 | |
| CP-SCR11-14 | Footer | Page {page} of {pages} | Position | Orientation | Paged | 20 | |
| CP-SCR11-15 | Footer | Tap a reference to open it. | How to act | Discover | Loaded | 30 | |
| CP-SCR11-16 | Error | I couldn't load your invoices. Try /owed again in a moment. | DB failure | Retry | Error | 70 | |
| CP-SCR11-17 | Marker | · paused | Reminders paused by freelancer | Awareness | Paused | 10 | Appended to CP-SCR11-5 |
| CP-SCR11-18 | Marker | · client says paid | Awaiting confirmation | Awareness | Client says paid | 20 | Appended to CP-SCR11-5 |

### 11.12 SCR-12 Invoice card

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR12-1 | Card | {client} · {ref}\nAmount {amount} · paid {paid} · owed {balance}\nDue {due_date} ({status_text})\nReminders: {nudge_status_line} | Full status | Understand | Open | 300 | `{status_text}` from CP-SCR11-6 to CP-SCR11-10 |
| CP-SCR12-2 | Status line | next on {date} ({step_name}) | Next reminder | Timing | Planned | 60 | `{step_name}`: courtesy / friendly / neutral / firm / final |
| CP-SCR12-3 | Status line | paused by you | Paused | Awareness | Paused | 20 | |
| CP-SCR12-4 | Status line | paused — client says they've paid | Awaiting confirmation | Awareness | Client says paid | 40 | |
| CP-SCR12-5 | Status line | stopped — client opted out | Opt-out | Awareness | Opted out | 30 | |
| CP-SCR12-6 | Status line | done — all {n} sent | Plan finished | Awareness | Complete | 24 | |
| CP-SCR12-7 | Status line | off | Plan off | Awareness | Off | 4 | |
| CP-SCR12-8 | Status line | blocked — emails to {email} bounce | Bounce | Fix email | Bounced | 80 | |
| CP-SCR12-9 | Button | Paid in full | F-04 | Tick off | Open | 16 | Primary, first |
| CP-SCR12-10 | Button | Part payment | F-04 part | Record | Open | 16 | |
| CP-SCR12-11 | Button | Pause reminders | F-06 | Control | Open, not paused | 20 | |
| CP-SCR12-12 | Button | Resume reminders | F-06 | Control | Paused by freelancer | 20 | |
| CP-SCR12-13 | Button | Send reminder now | F-06 | Act | Open, step remaining | 20 | |
| CP-SCR12-14 | Button | Edit | Edit menu | Fix | Open | 8 | |
| CP-SCR12-15 | Button | More | Secondary actions | Explore | Open | 8 | Opens: Change reminder plan, Get text to forward, Cancel invoice, Delete |
| CP-SCR12-16 | Button | Cancel invoice | Void | Stop chasing | More menu | 16 | |
| CP-SCR12-17 | Confirm | Cancel {ref}? It stays in your history as cancelled and no more reminders go to {client}. | Consequence | Decide | Cancel confirm | 130 | |
| CP-SCR12-18 | Button | Cancel invoice | Confirm cancel | Commit | Cancel confirm | 16 | Danger style n/a in Telegram; placed second |
| CP-SCR12-19 | Button | Keep it | Abort | Safety | Cancel / delete confirm | 12 | Placed first |
| CP-SCR12-20 | Button | Delete | Hard delete | Remove mistake | More menu | 8 | |
| CP-SCR12-21 | Confirm | Delete {ref} for good? Its payments and reminder history go too. This can't be undone. | Consequence | Decide | Delete confirm | 120 | |
| CP-SCR12-22 | Button | Delete for good | Confirm delete | Commit | Delete confirm | 16 | |
| CP-SCR12-23 | Ack | {ref} deleted. | Outcome | Confidence | Deleted | 40 | |
| CP-SCR12-24 | Confirm | Email {client} a reminder about {ref} now? It will be reminder {step_number} of {total}. | Consequence | Decide | Send-now confirm | 120 | |
| CP-SCR12-25 | Button | Send now | Confirm send | Commit | Send-now confirm | 12 | |
| CP-SCR12-26 | Button | Not now | Abort | Safety | Send-now confirm | 12 | |
| CP-SCR12-27 | Inline error | {client} was reminded about {ref} on {date}. The next one can go from {date_next}. | 48 h spacing | Understand | Send-now blocked | 130 | `{date_next}` = last send + 2 days |
| CP-SCR12-28 | Prompt | What do you want to change? | Edit menu | Choose | Edit | 30 | |
| CP-SCR12-29 | Buttons | Amount · Due date · Reference · Link | Editable fields | Choose | Edit | 12 each | Client cannot be changed; cancel and re-log instead |
| CP-SCR12-30 | Ack | Updated. Reminders now follow the new details. | Outcome | Confidence | Edit saved | 60 | Amount edit below payments total → CP-SCR13-7 wording |
| CP-SCR12-31 | Inline error | This invoice changed on another device. Here's the latest version — make your change again. | Optimistic-lock conflict | Recover | Conflict | 110 | EC-CONC-01 |
| CP-SCR12-32 | Inline error | That invoice doesn't exist anymore. | Deleted / not owned | Understand | Not found | 40 | Same text for "not yours" (EC-PERM-02) |
| CP-SCR12-33 | Button | Get text to forward | F-15 | Remind by chat | More menu | 20 | |
| CP-SCR12-34 | Ack | Reminders for {ref} paused. Tap Resume reminders when you want them back. | Outcome | Confidence | Paused | 90 | |
| CP-SCR12-35 | Ack | Reminders for {ref} resumed. Next: {date}. | Outcome | Confidence | Resumed | 60 | |
| CP-SCR12-36 | Inline error | {client} asked not to get reminders about {ref}, so I can't resume them. | Opt-out respected | Understand | Resume refused | 100 | |
| CP-SCR12-37 | Ack | {ref} cancelled. No more reminders will be sent. | Outcome | Confidence | Cancelled | 60 | |
| CP-SCR12-38 | Ack | Reminders for {ref} paused. | Short toast variant | Confidence | Paused (toast) | 40 | answerCallbackQuery text |
| CP-SCR12-39 | Ack | Reminders for {ref} resumed. | Short toast variant | Confidence | Resumed (toast) | 40 | answerCallbackQuery text |
| CP-SCR12-40 | Inline error | Reminders for {ref} can't be resumed: {client} opted out. | Short toast variant | Understand | Resume refused (toast) | 60 | |
| CP-SCR12-41 | Button | Change reminder plan | Plan picker | Adjust | More menu | 20 | Opens CP-SCR10-12 |
| CP-SCR12-42 | Prompt | Pick a new reminder plan for {ref}. | Plan picker header | Choose | Change plan | 50 | Followed by CP-SCR10-13 to CP-SCR10-16 |
| CP-SCR12-43 | Ack | Reminder plan for {ref}: {preset_name}. | Outcome | Confidence | Plan changed | 50 | |
| CP-SCR12-44 | Status line | waiting — {client} can't receive email from you | Suppressed | Awareness | Suppressed | 80 | Unsubscribed or complained (not bounced) |
| CP-SCR12-45 | Intro | Here's a reminder you can forward or paste into any chat: | Explains next message | Use it | Forward text | 60 | |
| CP-SCR12-46 | Draft | Hi {contact_name_or_there}, a quick reminder that invoice {ref} for {balance} was due on {due_date}. {payment_line}Thanks, {name} | Paste-ready text | Send by chat | Forward text | 700 | `{contact_name_or_there}` = contact name or "there"; `{payment_line}` = "You can pay here: {payment_instructions} " or empty |

### 11.13 SCR-13 Record payment

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR13-1 | Ack | Marked {ref} as paid — {amount}. Remaining reminders cancelled. | Outcome | Relief | Paid in full | 100 | |
| CP-SCR13-2 | Button | Undo | Revert within 15 min | Fix mis-tap | Paid in full | 8 | Expires 15 min after the ack |
| CP-SCR13-3 | Ack | Undone. {ref} is open again; reminders resume from {date}. | Outcome | Confidence | Undone | 80 | |
| CP-SCR13-4 | Inline error | Undo is no longer available. Open the invoice and tap Reopen invoice. | Undo expired | Recover | Undo expired | 80 | |
| CP-SCR13-5 | Prompt | How much did {client} pay? Owed now: {balance}. | Part amount | Enter | Part prompt | 90 | |
| CP-SCR13-6 | Inline error | I couldn't read an amount in "{input}". Send a number like 400. | Unparseable | Fix | Invalid | 80 | |
| CP-SCR13-7 | Inline error | That's more than the {balance} owed. Send a smaller amount, or tap Paid in full. | Over balance | Fix | Over | 100 | |
| CP-SCR13-8 | Ack | Recorded {paid} from {client}. Still owed: {balance}. Reminders will mention the new balance. | Outcome | Confidence | Part recorded | 130 | |
| CP-SCR13-9 | Note | That covers the full balance, so {ref} is now paid. | Part = balance | Understand | Part = full | 70 | Followed by CP-SCR13-1 |
| CP-SCR13-10 | Inline error | Payments are recorded in the invoice currency ({currency}). Send the amount without a currency code. | Different currency typed | Fix | Currency typed | 110 | Same code as invoice is accepted silently |
| CP-SCR13-11 | Button | Reopen invoice | Paid → open | Correct | Paid card | 16 | Removes the last full-payment record |
| CP-SCR13-12 | Ack | {ref} reopened. Owed: {balance}. | Outcome | Confidence | Reopened | 50 | |
| CP-SCR13-13 | Inline error | {ref} is already marked as paid. | Stale button | Understand | Already paid | 50 | |

### 11.14 SCR-14 Evening digest

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR14-1 | Header | Tomorrow's reminders ({count}): | Digest header | Review | ≥ 1 reminder | 40 | "Tomorrow" = the next working day; Friday's digest lists Monday's |
| CP-SCR14-2 | Line | {time} · {client} · {ref} · {balance} · {status_text} · {step_name} | One reminder | Scan | Line | 140 | |
| CP-SCR14-3 | Footer | They'll go out unless you hold them. Mark anything already paid first. | Consequence of inaction | Decide | ≥ 1 reminder | 80 | |
| CP-SCR14-4 | Button | Hold {ref} | Skip this reminder | Veto | Per line | 24 | |
| CP-SCR14-5 | Button | Paid {ref} | Mark paid (F-04) | Tick off | Per line | 24 | |
| CP-SCR14-6 | Section line | Waiting for you: {client} says {ref} is paid. | Pending confirmation | Resolve | Client said paid | 100 | Listed first; also sent when no reminders are due |
| CP-SCR14-7 | Button | Mark as paid | Confirm | Resolve | Waiting | 16 | |
| CP-SCR14-8 | Button | Not received yet | Reject claim | Resolve | Waiting | 20 | → CP-SCR15-8 |
| CP-SCR14-9 | Line edit | Held — {client} won't get this reminder. The next one is on {date}. | Outcome | Confidence | Held | 100 | Replaces the line |
| CP-SCR14-10 | Line edit | Held — that was the last planned reminder for {ref}. | Outcome | Understand | Held last step | 70 | |
| CP-SCR14-11 | Overflow | …and {more} more. See /owed for all. | > 10 reminders | Know there's more | Overflow | 50 | Only the first 10 get buttons |
| CP-SCR14-12 | Toast | Too late to hold — that reminder was sent at {time}. | Already sent | Understand | Stale hold | 60 | answerCallbackQuery |

### 11.15 SCR-15 Notices

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR15-1 | Notice | Reminded {client} about {ref} ({balance}, {status_text}). Replies go to {email}. | Sent | Awareness | Sent | 150 | One message per send; batched into one message if > 3 in the same minute (list form, same line format) |
| CP-SCR15-2 | Notice | I couldn't email {client} about {ref} after 4 tries. I'll try the next reminder as planned. If this keeps happening, check the address in /clients. | Permanent failure | Act | Failed | 170 | |
| CP-SCR15-3 | Notice | Email to {email} ({client}) bounced — the address may be wrong. Reminders to it are stopped. Update the email in /clients to restart them. | Bounce | Fix | Bounced | 170 | |
| CP-SCR15-4 | Notice | {client} marked a reminder as spam. I've stopped reminders to {email}. You can still send them the text yourself. | Complaint | Understand | Complaint | 150 | |
| CP-SCR15-5 | Notice | {client} says they've paid {ref} ({balance}). Reminders are paused. Did the money arrive? | Client claim | Confirm | Client says paid | 130 | |
| CP-SCR15-6 | Button | Yes, mark as paid | F-04 full | Confirm | Client says paid | 20 | |
| CP-SCR15-7 | Button | Not received yet | Reject claim | Continue chasing | Client says paid | 20 | |
| CP-SCR15-8 | Ack | Okay. Reminders resume from {date}. | Outcome | Confidence | Not received | 50 | `{date}` ≥ today + 3 days |
| CP-SCR15-9 | Notice | {client} asked to stop reminders about {ref}. I won't email them about it again. | Invoice opt-out | Understand | Stopped | 110 | |
| CP-SCR15-10 | Notice | {email} ({client}) unsubscribed from your reminders. I won't email that address for you again. You can still track their invoices. | Unsubscribe | Understand | Unsubscribed | 160 | |
| CP-SCR15-11 | Notice | You've reached today's limit of {cap} reminders. {count} will go out tomorrow instead. | Daily cap | Understand | Cap | 100 | Once per day at most |
| CP-SCR15-12 | Notice | Reminder sending is paused on your account after spam complaints. Tracking still works. Contact {support_email} if you think this is a mistake. | Suspended | Understand | Suspended | 160 | |
| CP-SCR15-13 | Notice | I'm not sure the reminder about {ref} reached {client} — the email service didn't confirm. I won't resend it automatically. | Unknown outcome | Decide | Unknown | 150 | Send now remains available after 48 h |
| CP-SCR15-14 | Notice | Reminder sending is back on for your account. | Unsuspended | Relief | Restored | 50 | |
| CP-SCR15-15 | Notice | Reminders due during a service outage were moved to {date}. | Catch-up rule | Understand | Outage | 80 | |

### 11.16 SCR-16 Clients

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR16-1 | Header | Your clients ({count}): | List header | Scan | Loaded | 30 | |
| CP-SCR16-2 | Line | {client} · {email} · {owed_text} | One client | Scan | Loaded | 120 | `{owed_text}` = "owes {totals}" or "nothing owed" |
| CP-SCR16-3 | Empty | No clients yet. They're added when you log an invoice with /new. | Empty | Next step | Empty | 80 | |
| CP-SCR16-4 | Card | {client}\nEmail: {email}\nGreeting: {greeting}\nOpen invoices: {count} · owed {totals} | Client detail | Check | Card | 250 | `{greeting}` = contact name or "Hello," |
| CP-SCR16-5 | Status line | Reminders: allowed | Eligible | Awareness | Allowed | 24 | |
| CP-SCR16-6 | Status line | Reminders: stopped — they unsubscribed | Unsubscribed or complained | Awareness | Suppressed | 45 | |
| CP-SCR16-7 | Status line | Reminders: stopped — emails bounced | Bounce | Fix | Bounced | 40 | |
| CP-SCR16-8 | Button | Edit name | Edit | Fix | Card | 16 | |
| CP-SCR16-9 | Button | Edit email | Edit | Fix | Card | 16 | |
| CP-SCR16-10 | Button | Edit greeting | Edit | Fix | Card | 16 | |
| CP-SCR16-11 | Button | Archive | Archive | Tidy | Card | 12 | |
| CP-SCR16-12 | Confirm | Archive {client}? They leave your client list. Their {count} open invoices stay and keep their reminders. | Consequence | Decide | Archive confirm | 130 | |
| CP-SCR16-13 | Button | Archive | Confirm | Commit | Archive confirm | 12 | Second |
| CP-SCR16-14 | Button | Keep | Abort | Safety | Archive confirm | 8 | First |
| CP-SCR16-15 | Ack | Email updated. Reminders for {client} go to {email} from now on. | Outcome | Confidence | Email edited | 100 | |
| CP-SCR16-16 | Ack | Saved. | Name or greeting edited | Confidence | Edited | 8 | |
| CP-SCR16-17 | Button | More clients | Next page | Browse | > 10 clients | 16 | |
| CP-SCR16-18 | Ack | {client} archived. | Outcome | Confidence | Archived | 40 | |

### 11.17 SCR-17 Settings

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR17-1 | Summary | Settings\nName: {name}\nBusiness: {business}\nReplies go to: {email}\nTime zone: {zone_label} (now {local_time})\nReminder time: {send_hour} on weekdays\nDefault plan: {preset_name}\nEvening plan: {digest_state}\nCurrency: {currency}\nPayment details: {payment_state} | Overview | Check | Summary | 450 | `{business}` "none"; `{digest_state}` "on at 18:00" / "off"; `{payment_state}` "set" / "not set" |
| CP-SCR17-2 | Button | Name & business | Edit | Choose | Summary | 20 | → SCR-02 |
| CP-SCR17-3 | Button | Reply-to email | Edit | Choose | Summary | 20 | → SCR-03 via E07-S02 |
| CP-SCR17-4 | Button | Time zone | Edit | Choose | Summary | 16 | → SCR-04 |
| CP-SCR17-5 | Button | Reminder time | Edit | Choose | Summary | 16 | |
| CP-SCR17-6 | Button | Default plan | Edit | Choose | Summary | 16 | → CP-SCR10-12 |
| CP-SCR17-7 | Button | Evening plan on/off | Toggle | Choose | Summary | 20 | Edits in place |
| CP-SCR17-8 | Button | Currency | Edit | Choose | Summary | 12 | → SCR-05 |
| CP-SCR17-9 | Button | Payment details | Edit | Choose | Summary | 16 | → CP-SCR05-5 |
| CP-SCR17-10 | Button | Export my data | F-12 | Take data | Summary | 16 | |
| CP-SCR17-11 | Button | Delete my account | F-12 | Leave | Summary | 20 | Last row |
| CP-SCR17-12 | Prompt | What time should reminders go out on weekdays? Your time zone: {zone_label}. | Send hour | Choose | Reminder time | 100 | Buttons 08:00 … 17:00 (10 buttons, 3 per row) |
| CP-SCR17-13 | Ack | Saved. Planned reminders now go out at {send_hour} your time. | Outcome | Confidence | Saved | 70 | |
| CP-SCR17-14 | Note | Changing this sends a new code to the new address. Until it's confirmed, replies still go to {old_email}. | Safety of change | Understand | Email change | 130 | |
| CP-SCR17-15 | Ack | Evening plan turned off. Reminders will go out without a heads-up the day before. Turn it back on here anytime. | Consequence | Understand | Digest off | 120 | |
| CP-SCR17-16 | Ack | Evening plan on. You'll get tomorrow's reminders at 18:00 your time. | Outcome | Confidence | Digest on | 80 | |
| CP-SCR17-17 | Ack | Time zone set to {zone_label}. Planned reminders moved to {send_hour} in the new zone. | Outcome | Confidence | Zone changed | 110 | |

### 11.18 SCR-18 Export and delete

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR18-1 | Status | Preparing your data… | Progress | Wait | Preparing | 30 | With typing action |
| CP-SCR18-2 | Success | Here's everything I store about your clients, invoices, payments and reminders, as 4 CSV files. | Outcome | Confidence | Done | 110 | |
| CP-SCR18-3 | Error | I couldn't build your export. Nothing changed — try again in a few minutes. | Failure | Retry | Failed | 80 | |
| CP-SCR18-4 | Inline error | You exported your data {minutes} minutes ago. You can export again in {wait} minutes. | 1 per hour | Wait | Too soon | 90 | |
| CP-SCR18-5 | Confirm | Delete your account? This erases your profile, {clients} clients and {invoices} invoices, and cancels {nudges} planned reminders. Clients won't be emailed again. This can't be undone. | Consequence | Decide | Delete confirm | 220 | |
| CP-SCR18-6 | Button | Delete everything | Confirm | Commit | Delete confirm | 20 | Second |
| CP-SCR18-7 | Button | Keep my account | Abort | Safety | Delete confirm | 20 | First |
| CP-SCR18-8 | Ack | Kept. Nothing was deleted. | Abort outcome | Reassure | Kept | 30 | |
| CP-SCR18-9 | Success | Your account and data are deleted. Backups that still contain them expire within 7 days. Send /start if you ever want to begin again. | Outcome | Closure | Deleted | 140 | 7 days = maximum PITR window (S-38) |
| CP-SCR18-10 | Error | I couldn't delete your account just now. Nothing was removed — try again in a few minutes. | Failure | Retry | Failed | 90 | |
| CP-SCR18-11 | File names | clients.csv · invoices.csv · payments.csv · reminders.csv | Export files | Recognise | Done | 20 each | |

### 11.19 SCR-19 Operator

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR19-1 | Stats | Stats (last 24 h)\nFreelancers: {total} · active 7d: {wau}\nOpen invoices: {open}\nReminders sent: {sent} · failed: {failed}\nBounces: {bounces} · complaints: {complaints}\nSuspended: {suspended}\nActivation 30d: {activation_pct}% · paid ≤14d after first reminder: {paid14_pct}% | Health | Scan | /admin stats | 500 | Metrics M-1.2, M-2.1 |
| CP-SCR19-2 | User card | User {telegram_id} · joined {date}\nInvoices open: {open} · reminders 30d: {sent30}\nComplaints 30d: {complaints30} · status: {status} | One account | Investigate | /admin user | 250 | No client PII shown |
| CP-SCR19-3 | Usage | Usage: /admin suspend <telegram_id> <reason> | Missing args | Fix | Usage | 60 | |
| CP-SCR19-4 | Ack | Sending suspended for {telegram_id}. Reason logged. | Outcome | Confidence | Suspended | 60 | |
| CP-SCR19-5 | Ack | Sending restored for {telegram_id}. | Outcome | Confidence | Unsuspended | 50 | |
| CP-SCR19-6 | Error | No user with Telegram ID {telegram_id}. | Not found | Fix | Not found | 50 | |
| CP-SCR19-7 | Alert | Auto-suspended sending for {telegram_id}: {complaints} complaints in 30 days. Review: /admin user {telegram_id} | Auto-suspension | Review | Alert | 130 | |
| CP-SCR19-8 | Help | Admin commands: stats, metrics, user <id>, suspend <id> <reason>, unsuspend <id>. | Command list | Recall | Unknown subcommand | 100 | |
| CP-SCR19-10 | Alert | Alert: {pct}% of updates failed in the last 15 min (threshold 5%). Check Sentry and the logs. | Error-rate alert | Investigate | Alert | 110 | Once per hour at most |
| CP-SCR19-11 | Alert | Alert: {count} reminders failed in the last hour (threshold 10). Run /admin stats. | Send-failure alert | Investigate | Alert | 90 | |
| CP-SCR19-12 | Alert | Alert: {count} reminders are more than 30 min late. The scheduler may be stalled; check /healthz and the logs. | Stall alert | Investigate | Alert | 120 | |
| CP-SCR19-13 | Alert | Alert: {count} notifications are waiting to be sent (threshold 200). | Backlog alert | Investigate | Alert | 80 | |
| CP-SCR19-14 | Alert | Complaint received for user {telegram_id} ({complaints} in 30 days). Review: /admin user {telegram_id} | Complaint alert | Review | Alert | 110 | Every complaint |
| CP-SCR19-9 | Metrics | Metrics (last 30 days)\nActivation: {activation_pct}\nPaid ≤ 14 days after first reminder: {paid14_pct}\nMedian days overdue at payment: {median_days}\nWeek-4 retention: {retention_pct}\nMedian entry time, returning clients: {entry_s} | Product metrics M-1.1, M-1.2, M-2.1, M-2.2, M-5.1 | Judge the product | /admin metrics | 300 | Each value is "n/a" without data, else a number with % or s |

### 11.20 SCR-20 System fallbacks

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR20-1 | Message | I didn't catch that. Use /new to log an invoice or /owed to see who owes you. | Unknown input | Find a path | Unknown | 90 | Also for /admin from non-admins |
| CP-SCR20-2 | Message | That button is out of date. Here's the latest. | Stale callback | Continue | Stale | 50 | Followed by a fresh render |
| CP-SCR20-3 | Message | I can only read text. Type your answer, or use /help. | Media received | Recover | Media | 60 | EC-BOT-02 |
| CP-SCR20-4 | Message | Something went wrong on my side. Nothing was saved. Try again, or send /cancel to start over. | Unexpected error | Recover | Error | 100 | Error ID logged, not shown |
| CP-SCR20-5 | Message | That conversation timed out after 24 hours, so I cleared it. Start again with /new. | Expired wizard | Restart | Expired | 90 | Onboarding variant ends "Send /start." |
| CP-SCR20-6 | Message | I only work in private chats. Message me directly: @{bot_username} | Group add | Redirect | Group | 70 | Then leaves the group |
| CP-SCR20-7 | Message | You're sending messages faster than I can handle. Wait {seconds} seconds. | Flood | Slow down | Flood | 80 | > 30 updates/min per user |
| CP-SCR20-8 | Message | Let's finish setting you up first — it takes about 2 minutes. | Not onboarded | Continue | Not onboarded | 70 | Then the pending onboarding prompt |
| CP-SCR20-9 | Message | I use your latest message, not edits. Send the corrected value as a new message. | Edited message | Recover | Edited | 90 | Only during a wizard; otherwise ignored |
| CP-SCR20-10 | Message | This account can't use InvoiceNudge. Contact {support_email} if you think this is a mistake. | Fully blocked account (operator) | Understand | Blocked | 100 | Reserved; v1 suspension only blocks sending (CP-SCR15-12) |
| CP-SCR20-11 | Toast | Done | Callback ack | Feedback | Toast | 8 | |
| CP-SCR20-12 | Toast | Working on it… | Slow callback | Feedback | Toast | 16 | |

### 11.21 SCR-21 Client page: "I've already paid"

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR21-1 | h1 | Tell {name} you've paid | Page purpose | Resolve | Ask | 60 | `{name}` = freelancer display name |
| CP-SCR21-2 | Body | Invoice {ref} · {balance} · due {due_date}. If you've already sent the payment, confirm below and the reminders stop while {name} checks. | Facts + consequence | Decide | Ask | 200 | Facts also shown as a `<dl>` |
| CP-SCR21-3 | Button | I've paid this invoice | POST confirm | Act | Ask | 30 | |
| CP-SCR21-4 | h1 | Thanks — {name} has been told | Outcome | Closure | Done | 60 | |
| CP-SCR21-5 | Body | Reminders about {ref} are paused while they check their account. If they can't find the payment, they'll reply to you directly. | Next | Closure | Done | 160 | |
| CP-SCR21-6 | Body | You already told {name} this invoice is paid. Reminders stay paused while they check. | Idempotent repeat | Reassure | Already reported | 110 | h1 CP-SCR21-4 |
| CP-SCR21-7 | Body | {name} has already marked {ref} as paid. No more reminders will come. | Already paid | Reassure | Already paid | 90 | h1 CP-SCR21-4 |
| CP-SCR21-8 | Title | Payment confirmation — {ref} | `<title>` | Orientation | All states | 60 | |
| CP-SCR21-9 | Footer | Sent on behalf of {name} by InvoiceNudge. Questions about this invoice? Reply to the reminder email. | Provenance | Trust | All states | 130 | Link to SCR-24 "Privacy" |

### 11.22 SCR-22 Client page: stop and unsubscribe

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR22-1 | h1 | Stop reminders about {ref}? | Page purpose | Decide | Ask (invoice) | 50 | |
| CP-SCR22-2 | Body | {name} will be told you asked to stop. It doesn't change what's owed. | Consequence | Decide | Ask (invoice) | 100 | |
| CP-SCR22-3 | Button | Stop reminders about this invoice | POST | Act | Ask (invoice) | 40 | |
| CP-SCR22-4 | h1 | Reminders stopped | Outcome | Closure | Done (invoice) | 30 | |
| CP-SCR22-5 | Body | You won't get more reminders about {ref} from {name} through InvoiceNudge. | Scope of outcome | Closure | Done (invoice) | 100 | |
| CP-SCR22-6 | h1 | Unsubscribe from all reminders sent for {name}? | Page purpose | Decide | Ask (all) | 70 | GET of the List-Unsubscribe URL |
| CP-SCR22-7 | Button | Unsubscribe | POST | Act | Ask (all) | 16 | |
| CP-SCR22-8 | Body | You're unsubscribed. {name} won't be able to send you reminders through InvoiceNudge. | Outcome | Closure | Done (all) | 100 | h1 CP-SCR22-4 |
| CP-SCR22-9 | Title | Stop reminders — {ref} | `<title>` | Orientation | All states | 50 | "Unsubscribe — {name}" for the all-variant uses the same ID with `{ref}` → name |

### 11.23 SCR-23 Client page: errors

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR23-1 | h1 | This link has expired | Bad or old token | Understand | Expired / invalid | 30 | Same page for invalid signatures (EC-SEC-11) |
| CP-SCR23-2 | Body | Links in reminder emails work for 60 days. To reach the sender, reply to the reminder email. | Recovery | Act | Expired / invalid | 100 | |
| CP-SCR23-3 | Body | This invoice is no longer tracked, so there's nothing to update. To reach the sender, reply to the reminder email. | Invoice deleted | Understand | Gone | 120 | h1 "Nothing to update" = CP-SCR23-6 |
| CP-SCR23-4 | Body | Something went wrong on our side and nothing was changed. Try the link again in a few minutes. | 5xx | Retry | Error | 100 | h1 CP-SCR23-6 |
| CP-SCR23-5 | Body | Too many requests from your network. Try again in a minute. | 429 | Wait | Rate limited | 70 | h1 CP-SCR23-6 |
| CP-SCR23-6 | h1 | Nothing to update | Generic heading | Orientation | Gone / error / limited | 30 | |

### 11.24 SCR-24 Privacy page

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-SCR24-1 | h1 | Privacy at InvoiceNudge | Page title | Orientation | Single | 30 | |
| CP-SCR24-2 | Section | What we store: your name, business name, reply-to email, time zone and payment details; your clients' names and emails; your invoices, payments and reminders. Names, emails and payment details are encrypted. | Data inventory | Trust | Single | 300 | Must match §12.4 |
| CP-SCR24-3 | Section | Why: to remind your clients about unpaid invoices on your behalf and show you who owes what. We don't sell data, show ads, or track email opens. | Purpose | Trust | Single | 220 | |
| CP-SCR24-4 | Section | Who processes it: Telegram (messages), Resend (sending email), Render (hosting, EU region), Sentry (error reports without personal data). | Processors | Trust | Single | 220 | Update when a processor changes |
| CP-SCR24-5 | Section | Deleting it: send /settings to the bot and choose Delete my account. Everything is erased at once; backups expire within 7 days. Clients can stop reminders using the links in any reminder. | Rights | Act | Single | 260 | |
| CP-SCR24-6 | Section | Full policy: the complete legal text, published by the operator. | Legal text slot | Trust | Single | 80 | Replaced by counsel's text before public launch (OQ-02); build ships this line until then |

### 11.25 Email channel (reminders and verification)

Sender: `From: "{from_display}" <{EMAIL_FROM_ADDRESS}>`, `Reply-To: {freelancer_email}`. Headers on every reminder: `List-Unsubscribe: <{unsubscribe_url}>`, `List-Unsubscribe-Post: List-Unsubscribe=One-Click` (S-47). Tags: `kind=reminder`, `step=<step_kind>` (S-72).

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-EML-1 | From display | {name} via InvoiceNudge | Honest provenance | Recognise sender | All reminders | 60 | With business: "{name} ({business}) via InvoiceNudge", truncated to 60 |
| CP-EML-2 | Subject | Invoice {ref} is due on {due_date} | Courtesy | Plan payment | pre_due | 60 | |
| CP-EML-3 | Subject | Reminder: invoice {ref} ({balance}) | Friendly | Notice | friendly | 60 | |
| CP-EML-4 | Subject | Invoice {ref} is now {days} days overdue | Neutral | Prioritise | neutral | 60 | `{days}` ≥ 2 by construction |
| CP-EML-5 | Subject | Overdue: invoice {ref} ({balance}), {days} days | Firm | Prioritise | firm | 60 | |
| CP-EML-6 | Subject | Final reminder: invoice {ref} ({balance}) | Final | Act | final | 60 | |
| CP-EML-7 | Preheader | {balance} · due {due_date} · from {name} | Inbox preview | Recognise | All reminders | 90 | Hidden span |
| CP-EML-8 | Greeting | Hi {contact_name}, | Personal | — | Contact name set | 50 | |
| CP-EML-9 | Greeting | Hello, | Neutral | — | No contact name | 8 | |
| CP-EML-10 | Body | Just a heads-up that invoice {ref} for {balance} is due on {due_date}. If it's already scheduled, thank you — no need to reply. | Courtesy | Plan | pre_due | 200 | |
| CP-EML-11 | Body | Invoice {ref} for {balance} was due on {due_date}. It may have slipped through — could you take a look when you have a moment? | Assume good faith | Act | friendly | 200 | |
| CP-EML-12 | Body | Invoice {ref} for {balance} is now {days} days past its due date of {due_date}. Please let me know when payment is scheduled. | Factual | Act | neutral | 200 | |
| CP-EML-13 | Body | Invoice {ref} for {balance} is {days} days overdue. Please arrange payment by {pay_by_date}, or reply to let me know what's holding it up. | Direct | Act | firm | 220 | `{pay_by_date}` = send date + 7 days |
| CP-EML-14 | Body | This is my final automated reminder about invoice {ref} for {balance}, {days} days overdue. Please pay by {pay_by_date} or reply to discuss. | Formal, brief | Act | final | 220 | No fees, interest or legal words (OQ-09) |
| CP-EML-15 | Body line | Thanks for the {paid} received so far; {balance} is still open. | Part payment acknowledged | Accuracy | paid > 0 | 110 | Inserted after the body |
| CP-EML-16 | Block | How to pay:\n{payment_instructions} | Payment action | Pay | Instructions set | 540 | |
| CP-EML-17 | Block | Payment details are on the invoice. | Fallback | Pay | No instructions | 40 | |
| CP-EML-18 | Link line | View invoice: {invoice_url} | Optional link | See invoice | Link set | 540 | Omitted if no link |
| CP-EML-19 | Sign-off | Thanks,\n{name}{business_line} | Signature | Recognise | All | 140 | `{business_line}` = "\n{business}" or empty |
| CP-EML-20 | Action link | Already paid? Let {name} know: {paid_url} | F-07 | Resolve | All reminders | 200 | Full URL visible |
| CP-EML-21 | Action link | Don't want reminders about this invoice? {stop_url} | F-08 | Opt out | All reminders | 200 | |
| CP-EML-22 | Footer | This reminder was sent by InvoiceNudge on behalf of {name}. Replies go to {name} directly. | Provenance | Trust | All reminders | 140 | Plus a link to SCR-24 |
| CP-EML-23 | Subject | Your InvoiceNudge code: {code} | Verification | Find code | Verification | 40 | Code first in body too |
| CP-EML-24 | Body | Your code is {code}. It expires in 15 minutes. If you didn't ask for it, ignore this email — nothing happens without the code. | Verification | Enter code | Verification | 160 | No links |

### 11.26 Bot command menu and profile texts

| ID | Element | Copy (en) | Gloss / intent | Persona intent | State / variant | Max length | Notes |
|---|---|---|---|---|---|---|---|
| CP-CMD-1 | Command | new — Log an invoice | Menu entry | Discover | setMyCommands | 32 + 256 | Command name ≤ 32 chars (S-12) |
| CP-CMD-2 | Command | owed — See who owes you | Menu entry | Discover | setMyCommands | 32 + 256 | |
| CP-CMD-3 | Command | clients — Your clients | Menu entry | Discover | setMyCommands | 32 + 256 | |
| CP-CMD-4 | Command | settings — Time zone, reminders, account | Menu entry | Discover | setMyCommands | 32 + 256 | |
| CP-CMD-5 | Command | help — How this works | Menu entry | Discover | setMyCommands | 32 + 256 | |
| CP-CMD-6 | Command | cancel — Stop the current step | Menu entry | Discover | setMyCommands | 32 + 256 | |
| CP-CMD-7 | Short description | Tracks your invoices and sends polite reminders to late clients by email. | Bot profile | Decide to try | BotFather | 120 | |
| CP-CMD-8 | Description | Log invoices in seconds, see who owes you, and let polite reminders go to late clients by email, with a preview the evening before. Replies go straight to you. | Pre-start screen | Decide to start | BotFather | 512 | |

## 12. Architecture

The order is drivers, then shape, then technology. The drivers below were written before the stack was chosen. §13.1 shows that AD-3 changed the stack decision against the generic scoring.

### 12.0 Architecture drivers

| ID | Driver | Measurable target | Rank | Why this rank | Source (persona / scale ceiling / regulation / hosting) |
|---|---|---|---|---|---|
| AD-1 | Reminder correctness | 0 duplicate emails per nudge (M-3.1); 0 emails for invoices that were paid, cancelled, deleted, paused, opted out or suppressed at claim time; eligibility re-checked in the same transaction ≤ 1 s before the provider call; ≤ 2% of reminders precede a client "already paid" whose payment predates the send (M-3.2) | 1 | A wrong email to a client damages the freelancer's relationship: the one failure users will not forgive | P1 trust concerns, P3 |
| AD-2 | Operability by one maintainer | 1 deploy unit, 1 database, 1 `check` command; full local run with 2 commands (`docker` + `npm run dev`); every reminder's lifecycle reconstructable from DB rows plus logs by correlation ID; restore drill ≤ 4 h (RTO) with RPO ≤ 15 min | 2 | Solo builder with agent sessions; nobody is on call (P4) | P0, P4, scale ceiling |
| AD-3 | Time correctness | Reminders fire at the freelancer's send hour (default 10:00) on Mon–Fri in their IANA zone; ≥ 99% sent within 15 min of `scheduled_for` (M-4.1); DST transitions tested for 5 zones for 2026–2028 | 3 | Reminders at 3am or on Sunday look like spam and hurt trust; "overdue" must mean the freelancer's local date | P1, P3, scale ceiling (global users) |
| AD-4 | Privacy and data protection | PII columns encrypted with a key stored outside the DB (FF-14); 0 PII strings in logs (FF-08); account deletion completes in ≤ 60 s; backups age out ≤ 7 days after deletion (S-38) | 4 | Required by Telegram developer terms §4.2 and §4.4 (S-07); third-party (client) data | Regulation, P3 |
| AD-5 | Deliverability and abuse resistance | Complaint rate < 0.1% rolling 30 days (Gmail limit 0.3%, S-46); hard bounces < 2%; SPF + DKIM + DMARC aligned on the sending subdomain; opt-outs honoured within 60 s; per-freelancer caps (assumption 21) | 5 | One abusive user can damage every user's delivery on the shared domain | P3, P4, hosting reality |
| AD-6 | Chat responsiveness | Webhook handler returns 200 in < 1 s at p95 and < 3 s at p99 at 20 updates/s; grammY timeout 10 s never reached (S-54); no handler performs network I/O except the Telegram reply and the database | 6 | Slow handlers cause re-delivered updates and duplicate processing (S-54, S-58) | P1 low patience, scale ceiling |
| AD-7 | Cost ceiling | ≤ US$60/month infrastructure at the scale ceiling (compute + Postgres + email + error tracking) | 7 | Free beta; one person pays | Hosting reality, assumption 12 |
| AD-8 | Evolvability to a second surface | Adding a Mini App or web dashboard, or Stars billing, requires 0 changes to feature-module internals, only new adapters and one new module | 8 | Likely next 12 months: monetization (OQ-08) and richer views | Business next steps |

**Explicit non-drivers (what we are not optimising for)**
- **Not highly available / multi-region.** One instance, one region; a restore takes up to 4 h (AD-2). Reminders are not time-critical to the minute; a missed hour is recovered by the catch-up rule.
- **Not horizontally scalable beyond the ceiling.** 20 updates/s and 3,000 emails/day fit one process. No sharding, no read replicas, no cache layer.
- **Not real-time.** Minute-level scheduling granularity (30 s ticks) is enough.
- **Not multi-tenant teams or roles** beyond freelancer and operator.
- **Not offline-capable.** Telegram handles connectivity on the client side.
- **Not sub-100 ms.** Telegram's own latency dominates; < 1 s acknowledgement is the target.
- **Not multi-language.** English only (DR-23).
- **Not zero-dependency on vendors.** Telegram, Resend and Render are accepted single points of failure with documented fallbacks (§12.7).

### 12.1 Architecture style decision

Candidates realistic for one builder at this ceiling:
- **A. Modular monolith, single process.** Fastify HTTP server (Telegram webhook, email-events webhook, client pages, health) + in-process scheduler and outbox loops that claim work from Postgres with `FOR UPDATE SKIP LOCKED`.
- **B. Monolith + separate worker service.** Same code base; the web service handles HTTP; a Render background worker runs the scheduler and outbox.
- **C. Serverless functions + platform cron.** A function per endpoint; a cron trigger runs a sending function every minute; managed Postgres.

Scores: 0–3 per driver, weighted by rank (AD-1 weight 8 … AD-8 weight 1).

| Driver (rank, weight) | A. Modular monolith | B. Monolith + worker | C. Serverless + cron |
|---|---|---|---|
| AD-1 reminder correctness (1, ×8) | 3 — one transaction boundary, one process | 3 — same DB claiming | 2 — at-least-once cron invocations and function retries add duplicate paths |
| AD-2 operability (2, ×7) | 3 — one service, one log stream | 2 — two services to deploy and watch | 1 — many functions, distributed traces, vendor-specific tooling |
| AD-3 time correctness (3, ×6) | 3 — 30 s ticks under our control | 3 | 2 — cron granularity and platform schedule limits |
| AD-4 privacy (4, ×5) | 3 — secrets in one environment | 3 | 2 — secrets spread across functions |
| AD-5 deliverability (5, ×4) | 3 | 3 | 3 |
| AD-6 responsiveness (6, ×3) | 2 — scheduler shares the event loop (mitigated by small batches) | 3 — isolated | 1 — cold starts on the webhook path |
| AD-7 cost (7, ×2) | 3 — one instance | 2 — two instances | 3 — scale to zero |
| AD-8 evolvability (8, ×1) | 3 | 3 | 2 |
| **Weighted total (max 108)** | **105** | **99** | **68** |

**Decision: A, a modular monolith with the scheduler in-process** (DR-03).
- **What B would have bought:** webhook latency isolated from send bursts, and independent restarts. **Price:** a second service, a second deploy target and doubled compute cost. At 3,000 emails/day (about 2 per minute on average) the burst risk is small; A handles it with batches of 50 and yields between rows.
- **What C would have bought:** scale-to-zero cost. **Price:** duplicate-execution paths on the most important operation (AD-1) and much harder debugging for one person (AD-2).
- **Could any driver have changed this?** Yes. If AD-6 were rank 1, B would win. That is why the exit triggers below watch webhook latency.

**Exit triggers.** This decision holds until one of these happens:
- webhook p95 acknowledgement > 1 s for 3 consecutive days with scheduler ticks correlated in logs (→ move to B: set `RUN_SCHEDULER=false` on the web service and add a worker with `RUN_HTTP=false`; no code change);
- reminder volume > 20,000 emails/day or registered freelancers > 50,000 (→ re-score A/B and consider a queue);
- a second surface (Mini App or dashboard) needs a separately scaled frontend (→ add a web client, keep the core monolith);
- more than 2 people commit weekly and merge conflicts are measured (not felt) in more than 10% of PRs.

Until then, extraction is out of scope. The boundaries in §12.2, enforced by §12.10, keep it cheap.

### 12.2 Module map and boundaries

Modules follow business capabilities (bounded contexts), not technical layers. Each module lives in `src/modules/<name>/` and exposes only `index.ts`.

| Module | Business capability | Owns entities (tables) | Public interface (exported from `index.ts`) | Must never know about |
|---|---|---|---|---|
| `accounts` | Freelancer profile, onboarding state, reply-to verification, sending status, deletion and export orchestration | ENT-Freelancer, ENT-EmailVerification | `getFreelancer`, `getByTelegramId`, `createFromTelegram`, `updateProfile`, `startEmailVerification`, `confirmEmailCode`, `setSendingStatus`, `deleteAccount`, `exportAccount` | Telegram or grammY types; HTML |
| `clients` | Client records and reminder eligibility display | ENT-Client | `createClient`, `listClients`, `getClient`, `updateClient`, `archiveClient`, `findByEmail` | Invoices; email sending |
| `invoices` | Invoice and payment lifecycle, balances, "owed" views | ENT-Invoice, ENT-Payment | `createInvoice`, `getInvoiceCard`, `listOwed`, `editInvoice`, `cancelInvoice`, `deleteInvoice`, `recordPayment`, `undoPayment`, `reopenInvoice`, `pauseReminders`, `resumeReminders`; emits `InvoiceChanged` domain events | Nudges; email; Telegram |
| `nudging` | Reminder plans, nudge lifecycle, digest, scheduler, send pipeline, client actions (paid, stop) | ENT-Nudge | `planForInvoice` (event handler), `buildDigest`, `holdNudge`, `sendNow`, `runSchedulerTick`, `handleClientPaid`, `handleClientStop`, `handleUnsubscribe`, `forwardText` | Telegram or grammY types; HTTP framework types |
| `deliverability` | Suppression list, email events, caps, complaint policy | ENT-Suppression, ENT-EmailEvent | `isSuppressed`, `suppress`, `recordEmailEvent`, `capStatus`, `complaintsInWindow` | Nudge internals |
| `notifications` | Reliable Telegram messages to freelancers (outbox) | ENT-Notification | `enqueue(kind, freelancerId, refs, dedupeKey)`, `runDispatchTick` | Business rules of other modules |
| `admin` | Operator commands and stats | none (reads through other modules' interfaces; stats via read-only views, see below) | `stats`, `userCard`, `suspend`, `unsuspend` | Raw client PII |
| `bot` (adapter) | grammY bot, command routing, conversation engine, keyboards, rendering copy | ENT-ConversationState, ENT-ProcessedUpdate | `createBot`, `webhookHandler` | SQL; email |
| `web` (adapter) | Fastify server: Telegram webhook route, email-events route, client pages, privacy, health | none | `createServer` | SQL; grammY internals beyond the webhook handler |
| `platform` (shared kernel) | config, db, time, money, crypto, copy, logging, errors, email gateway, telegram gateway, domain events, audit | ENT-AuditEvent | per sub-folder `index.ts` | Any module, adapter or business rule |

**Dependency rule (direction).** Adapters (`bot`, `web`) → feature modules' `index.ts` → `platform`. Allowed module-to-module edges (DAG; nothing else):

```
platform        → (nothing internal)
notifications   → platform
accounts        → platform, notifications
clients         → platform, accounts
invoices        → platform, accounts, clients, notifications
deliverability  → platform, accounts, notifications
nudging         → platform, accounts, clients, invoices, deliverability, notifications
admin           → platform, accounts, deliverability, invoices, nudging
bot             → platform, accounts, clients, invoices, nudging, deliverability, admin, notifications
web             → platform, bot (webhookHandler only), nudging, deliverability
src/main/*      → everything (composition root)
```

`invoices` must not import `nudging`. When an invoice changes (created, edited, paid, cancelled, deleted, paused, resumed), `invoices` publishes an `InvoiceChanged` event on the in-process bus in `platform/events`. The bus is synchronous and runs **inside the same database transaction**. `nudging` subscribes in the composition root (`src/main/wire.ts`). This keeps the graph acyclic and the re-planning atomic with the change.

**Data ownership.** One module owns each table (column "Owns entities"). Other modules read through the owner's interface, never by joining into its tables. **Explicit relaxation:** the operator stats in `admin` read from SQL views `v_admin_stats` and `v_metrics_*`, which are created by migrations and contain only counts and dates (no PII). This is the only cross-table read path, and FF-07 whitelists those view names.

**Plausible extraction seams.**
1. `nudging` scheduler + send pipeline → a worker process (exit trigger 1). It is cheap because the loops are started from `src/main/start.ts` behind `RUN_SCHEDULER`, and they talk only to Postgres, Resend and the `notifications` interface.
2. `notifications` dispatcher → the same worker.
3. `web` client pages → a separate edge app. Cheap because they call only `nudging.handleClient*` through signed tokens.

**Where new code goes by default**
- A new bot command or button → `src/bot/handlers/<feature>.ts`, calling one feature module function; no business logic in handlers.
- A new rule about invoices, balances or payments → `src/modules/invoices/domain/`.
- A new rule about when or whether a reminder is sent → `src/modules/nudging/domain/`.
- A new SQL query → the owning module's `repo/` folder (for `bot`'s own tables, `src/bot/repo/`); nowhere else (FF-06).
- A new user-facing string → a new CP ID row in §11 (via an Amendment) and `src/platform/copy/catalog.ts`.
- A helper used by two modules → `src/platform/<area>/` only after the second use, never on the first.
- A new external service → a port in `platform` plus an adapter; a DR entry first (§13.10).
- A new HTTP route → `src/web/routes/`, calling one module function.

### 12.3 Component diagram

```
                    Telegram Bot API (INT-telegram)                     Resend (INT-email)
                       |  webhook POST            ^ sendMessage etc.       ^ POST /emails      | webhook POST
                       v  (secret header)         |                        | (Idempotency-Key) v  (Svix-signed)
+----------------------------------------------------------------------------------------------------------+
|  Render web service (Frankfurt) — one Node.js 26 process                                                  |
|                                                                                                           |
|  web (Fastify 5)                                                                                          |
|   ├─ POST /tg/webhook ──────────────► bot (grammY 1.46) ──► feature modules (index.ts only)               |
|   ├─ POST /webhooks/resend ─────────► deliverability.recordEmailEvent                                     |
|   ├─ GET/POST /c/:token/paid|stop ──► nudging.handleClientPaid / handleClientStop                         |
|   ├─ GET/POST /u/:token ────────────► nudging.handleUnsubscribe                                           |
|   ├─ GET /privacy, GET /healthz                                                                           |
|                                                                                                           |
|  feature modules: accounts · clients · invoices · nudging · deliverability · notifications · admin       |
|        │ invoices ──InvoiceChanged (sync, same tx)──► nudging                                             |
|                                                                                                           |
|  loops (RUN_SCHEDULER=true): every 30 s                                                                   |
|   ├─ nudging.runSchedulerTick   (plan digests, claim due nudges, send via platform/email)                 |
|   └─ notifications.runDispatchTick (claim pending notifications, send via platform/telegram)             |
|                                                                                                           |
|  platform: config · db (Kysely) · time (Temporal) · money · crypto · copy · log (pino) · errors ·         |
|            email gateway · telegram gateway · events · audit                                              |
+----------------------------------------------------------------------------------------------------------+
            |                                                   |
            v                                                   v
   Render Postgres 18 (Frankfurt, AES-256 at rest, PITR)    Sentry (INT-sentry, errors without PII)
            ^
            | client browsers (P3) reach /c/* and /u/* over HTTPS; mail clients POST one-click unsubscribe
```

- **web.** HTTP entry point. Validates input at the trust boundary (secret header, Svix signature, HMAC tokens), sets security headers, renders the client pages from the copy catalogue, and never runs SQL. Fastify's `inject()` makes every route testable without a network [VERIFIED — S-75, 2026-09-30].
- **bot.** grammY bot created with a preset `botInfo` (no `getMe` at start) [VERIFIED — S-90, 2026-09-30]. Mounted via `webhookCallback(bot, "fastify", { secretToken, timeoutMilliseconds: 9000, onTimeout: "throw" })` [VERIFIED — S-55, S-57, 2026-09-30]. Middleware order: update de-duplication → flood control → load freelancer → conversation engine → command/callback routers → fallback (CP-SCR20-1). Handlers call one module function each and render copy.
- **Feature modules.** Business rules in `domain/` (pure functions, no I/O), persistence in `repo/`, orchestration in `service.ts`, public surface in `index.ts`.
- **Loops.** A single `setInterval`-free loop per concern: `await tick(); await sleep(30_000)`, so ticks never overlap within a process. Each tick claims at most 50 rows with `FOR UPDATE SKIP LOCKED` inside short transactions, so two processes during a deploy overlap cannot claim the same row (DR-21).
- **Platform.** Adapters to the outside world behind ports (`EmailGateway`, `TelegramGateway`, `Clock`, `Crypto`), so tests use fakes and production uses real clients.

### 12.4 Data model

Conventions: table names are singular snake_case; IDs are `uuid` defaulting to `uuidv7()` [VERIFIED — S-84, 2026-09-30]; all timestamps are `timestamptz` stored in UTC; `due_date` is a `date` in the freelancer's calendar; money is `bigint` minor units + `char(3)` ISO 4217 code (DR-10); columns ending `_enc` hold AES-256-GCM ciphertext (`key_id(1) ‖ iv(12) ‖ tag(16) ‖ ciphertext`); columns ending `_bidx` hold HMAC-SHA-256 blind indexes of normalised values (DR-08). IDs are never shown to users; the bot shows references, and client links carry signed tokens.

**ENT-Freelancer** (`freelancer`, owner `accounts`)

| Field | Type | Constraints | Null | Default | Notes |
|---|---|---|---|---|---|
| id | uuid | PK | no | `uuidv7()` | |
| telegram_user_id | bigint | UNIQUE | no | — | Identity (DR-16) |
| telegram_chat_id | bigint | — | no | — | Private chat ID |
| display_name_enc | bytea | — | yes | — | Set at onboarding step 1 |
| business_name_enc | bytea | — | yes | — | |
| email_enc | bytea | — | yes | — | Verified reply-to |
| email_bidx | bytea | — | yes | — | For suppression checks |
| email_verified_at | timestamptz | — | yes | — | Sending requires non-null |
| time_zone | text | valid IANA ID (app-validated via S-64) | no | `'Etc/UTC'` | |
| send_hour | smallint | CHECK 8–17 | no | 10 | |
| default_currency | char(3) | in `currencies.json` | yes | — | |
| default_preset | text | CHECK in (`gentle`,`standard`,`firm`,`off`) | no | `'standard'` | |
| digest_enabled | boolean | — | no | true | |
| payment_instructions_enc | bytea | plaintext ≤ 500 chars | yes | — | |
| next_invoice_seq | integer | ≥ 1 | no | 1 | Suggested reference INV-001… |
| onboarding_step | text | CHECK in (`name`,`business`,`email`,`email_code`,`time_zone`,`currency`,`payment`) | yes | `'name'` | null = onboarded |
| sending_status | text | CHECK in (`active`,`suspended`) | no | `'active'` | |
| suspended_reason | text | ≤ 200 chars, operator-written, no PII | yes | — | |
| bot_blocked_at | timestamptz | — | yes | — | Set on Telegram 403 |
| language_code | text | — | yes | — | From Telegram, unused in v1 |
| created_at / updated_at / last_seen_at | timestamptz | — | no/no/yes | `now()` | |

**ENT-EmailVerification** (`email_verification`, owner `accounts`): `id` uuid PK · `freelancer_id` FK → freelancer ON DELETE CASCADE · `email_enc` bytea · `email_bidx` bytea · `code_hash` bytea (HMAC-SHA-256 of the 6-digit code with `CODE_PEPPER`) · `attempts` smallint default 0 (max 5) · `expires_at` timestamptz (created + 15 min) · `consumed_at` timestamptz null · `created_at`. Index (`freelancer_id`, `created_at`). Only the latest unconsumed row is valid; issuing a new code consumes older ones (EC-AUTH-04).

**ENT-Client** (`client`, owner `clients`)

| Field | Type | Constraints | Null | Default | Notes |
|---|---|---|---|---|---|
| id | uuid | PK | no | `uuidv7()` | |
| freelancer_id | uuid | FK → freelancer ON DELETE CASCADE | no | — | Ownership scope |
| name_enc | bytea | plaintext 1–80 graphemes | no | — | |
| email_enc | bytea | plaintext ≤ 254, syntactically valid | no | — | |
| email_bidx | bytea | HMAC of lowercased, trimmed email | no | — | |
| contact_name_enc | bytea | plaintext 1–40 graphemes | yes | — | Greeting |
| archived_at | timestamptz | — | yes | — | |
| created_at / updated_at | timestamptz | — | no | `now()` | |

Indexes: (`freelancer_id`, `archived_at`); (`freelancer_id`, `email_bidx`). Duplicates are allowed after explicit confirmation (CP-SCR07-12), so there is no unique constraint.

**ENT-Invoice** (`invoice`, owner `invoices`)

| Field | Type | Constraints | Null | Default | Notes |
|---|---|---|---|---|---|
| id | uuid | PK | no | `uuidv7()` | |
| freelancer_id | uuid | FK → freelancer ON DELETE CASCADE | no | — | |
| client_id | uuid | FK → client ON DELETE RESTRICT | no | — | Clients are archived, never deleted alone |
| reference | text | 1–40 chars | no | — | Not encrypted: business identifier, shown in digest lines |
| amount_minor | bigint | CHECK > 0 AND ≤ 9999999999 | no | — | |
| currency | char(3) | in `currencies.json` | no | — | |
| due_date | date | — | no | — | Freelancer's calendar |
| status | text | CHECK in (`open`,`paid`,`cancelled`) | no | `'open'` | |
| paid_at / cancelled_at | timestamptz | — | yes | — | |
| invoice_url | text | `https://` prefix, ≤ 500 chars | yes | — | |
| nudge_preset | text | CHECK in (`gentle`,`standard`,`firm`,`off`) | no | — | Copied from default at creation |
| nudges_paused_reason | text | CHECK in (`freelancer`,`client_says_paid`,`client_opt_out`) | yes | — | |
| nudges_paused_at | timestamptz | — | yes | — | |
| plan_version | integer | ≥ 1 | no | 1 | Bumped on every re-plan |
| version | integer | ≥ 1 | no | 1 | Optimistic lock (EC-CONC-01) |
| entry_duration_ms | integer | ≥ 0 | yes | — | M-1.1 |
| created_at / updated_at | timestamptz | — | no | `now()` | |

Indexes: (`freelancer_id`, `status`, `due_date`); (`client_id`). Derived values (never stored): `paid_minor` = Σ payments; `balance_minor` = `amount_minor − paid_minor`; `overdue` = status `open` ∧ local today > `due_date`.

**ENT-Payment** (`payment`, owner `invoices`): `id` uuid PK · `invoice_id` FK → invoice ON DELETE CASCADE · `freelancer_id` uuid (denormalised for ownership checks) · `amount_minor` bigint CHECK > 0 · `kind` text CHECK in (`full`,`part`) · `recorded_at` timestamptz default `now()`. Index (`invoice_id`). Invariant (service-enforced, tested): Σ `amount_minor` ≤ invoice `amount_minor`.

**ENT-Nudge** (`nudge`, owner `nudging`)

| Field | Type | Constraints | Null | Default | Notes |
|---|---|---|---|---|---|
| id | uuid | PK | no | `uuidv7()` | Idempotency key `nudge/<id>` |
| invoice_id | uuid | FK → invoice ON DELETE CASCADE | no | — | |
| freelancer_id | uuid | FK → freelancer ON DELETE CASCADE | no | — | For claims and caps |
| plan_version | integer | — | no | — | Matches invoice.plan_version at creation |
| step_index | smallint | 0-based | no | — | |
| step_kind | text | CHECK in (`pre_due`,`friendly`,`neutral`,`firm`,`final`) | no | — | Selects CP-EML templates |
| scheduled_for | timestamptz | — | no | — | Computed by `platform/time` (DR-11) |
| status | text | CHECK in (`planned`,`announced`,`sending`,`sent`,`skipped`,`cancelled`,`failed`,`unknown`) | no | `'planned'` | Transitions below |
| announced_at | timestamptz | — | yes | — | DR-12 |
| send_now | boolean | — | no | false | Freelancer-initiated |
| claimed_at | timestamptz | — | yes | — | Set at the first claim only; drives the 23 h `unknown` rule |
| recipient_bidx | bytea | — | yes | — | Blind index of the address the email went to; set when sent; used by unsubscribe and bounce handling |
| attempt_count | smallint | 0–4 | no | 0 | |
| next_attempt_at | timestamptz | — | yes | — | Backoff |
| provider_message_id | text | UNIQUE | yes | — | Resend email ID |
| sent_at | timestamptz | — | yes | — | |
| last_error_code | text | ≤ 64 chars, no PII | yes | — | |
| cancel_reason | text | CHECK in (`paid`,`cancelled`,`deleted`,`paused`,`opted_out`,`suppressed`,`replanned`,`suspended`,`account_deleted`,`superseded`) | yes | — | |
| created_at / updated_at | timestamptz | — | no | `now()` | |

Constraints and indexes: UNIQUE (`invoice_id`, `plan_version`, `step_index`); partial index on (`scheduled_for`) WHERE status IN (`planned`,`announced`); partial index on (`next_attempt_at`) WHERE status = `sending`.

Nudge state transitions (the only allowed ones; `nudging/domain/transitions.ts` is the single source, unit-tested):

```
planned   → announced (digest delivered) | cancelled | skipped (hold) | sending (send_now only)
announced → sending (claim at scheduled_for) | cancelled | skipped (hold) | planned (re-plan: superseded)
sending   → sent (provider 2xx) | sending (retry scheduled) | failed (4th failure) | unknown (> 23 h unresolved) | cancelled (ineligible at re-check before provider call)
sent, skipped, cancelled, failed, unknown → (terminal)
```

**ENT-Notification** (`notification`, owner `notifications`): `id` uuid PK · `freelancer_id` FK ON DELETE CASCADE · `kind` text (e.g. `nudge_sent`, `nudge_failed`, `bounce`, `complaint`, `client_says_paid`, `client_stopped`, `unsubscribed`, `cap_reached`, `suspended`, `restored`, `unknown_outcome`, `outage_moved`, `digest`, `operator_alert`) · `refs` jsonb (IDs only, never PII) · `dedupe_key` text UNIQUE · `status` CHECK in (`pending`,`sent`,`failed`,`dropped`) · `attempt_count` smallint · `next_attempt_at` timestamptz · `telegram_message_id` bigint null · `created_at`, `sent_at`. Index (`status`, `next_attempt_at`). Text is rendered at dispatch time from current data, so no PII is stored in the outbox.

**ENT-Suppression** (`suppression`, owner `deliverability`): `id` uuid PK · `freelancer_id` uuid null FK ON DELETE CASCADE (null = global, used for bounces) · `email_bidx` bytea · `reason` CHECK in (`unsubscribe`,`complaint`,`bounce`) · `source_nudge_id` uuid null · `created_at`. Two partial unique indexes: (`freelancer_id`, `email_bidx`) WHERE `freelancer_id` IS NOT NULL; (`email_bidx`) WHERE `freelancer_id` IS NULL. A bounce suppression is lifted automatically only when a freelancer changes a client to a *different* address; the bounced address stays suppressed for 90 days (EC-MSG-02).

**ENT-EmailEvent** (`email_event`, owner `deliverability`): `id` uuid PK · `provider_event_id` text UNIQUE (the `svix-id` header) · `provider_message_id` text · `type` text (Resend event names, S-29) · `nudge_id` uuid null FK → nudge ON DELETE SET NULL · `bounce_type` text null · `occurred_at`, `received_at` timestamptz. No addresses and no payload bodies are stored.

**ENT-ConversationState** (`conversation_state`, owner `bot`): `telegram_user_id` bigint PK · `flow` text · `step` text · `data_enc` bytea (encrypted JSON of values entered so far) · `message_id` bigint null (message to edit) · `started_at`, `updated_at`, `expires_at` (updated_at + 24 h).

**ENT-ProcessedUpdate** (`processed_update`, owner `bot`): `update_id` bigint PK · `received_at` timestamptz. Rows older than 48 h are deleted by the retention job; Telegram never holds updates longer than 24 h (S-10).

**ENT-AuditEvent** (`audit_event`, owner `platform/audit`): `id` uuid PK · `freelancer_id` uuid null FK ON DELETE SET NULL · `actor` CHECK in (`freelancer`,`client`,`operator`,`system`) · `action` text (e.g. `invoice.paid`, `nudge.sent`, `client.says_paid`, `client.stopped`, `client.unsubscribed`, `account.deleted`, `sending.suspended`) · `subject_type` text · `subject_id` uuid null · `detail` jsonb (no PII: IDs, counts, codes) · `created_at`. Indexes (`freelancer_id`, `created_at`), (`action`, `created_at`).

**Views** (read-only, owner `admin`): `v_admin_stats`, `v_metrics_activation`, `v_metrics_paid_after_first_nudge`, `v_metrics_days_overdue`, `v_metrics_retention`. They contain counts and dates only.

**Retention and deletion.**

| Data | Retention | Enforced by |
|---|---|---|
| processed_update | 48 h | retention job (E07-S04) |
| email_verification | 24 h after expiry | retention job |
| conversation_state | until `expires_at` | retention job + read-time check |
| notification (sent / dropped / failed) | 30 days | retention job |
| email_event | 90 days | retention job |
| bounce suppression (global) | 90 days | retention job |
| unsubscribe / complaint suppression | while the freelancer account exists | cascade on deletion |
| audit_event | 400 days; `freelancer_id` set null on account deletion | retention job + FK |
| everything else of an account | until the freelancer deletes invoices or the account | user action |
| backups | Render PITR window: 3 days (Hobby) or 7 days (Pro workspace) (S-38) | platform |

**Migration policy.** Kysely migrations in `src/platform/db/migrations/` named `YYYYMMDDHHMM_<snake_name>.ts`, each exporting `up` and `down` (S-34). `down` exists for local development only; production is forward-only. Changes to populated tables follow expand → backfill → contract across at least two deploys, and every migration must be compatible with the previous release's code (rollback safety, EC-OPS-03). Migrations run in Render's pre-deploy command (S-41); a failed migration fails the deploy and the old release keeps running. After migrating, `kysely-codegen --verify` must pass (FF-11). Seed data: none in production; `npm run db:seed:dev` loads fixtures locally and refuses to run when `APP_ENV` is `staging` or `production` (EC-OPS-07).

### 12.5 API contracts

HTTP endpoints (Fastify; JSON schemas for bodies live in `src/web/schemas/*.ts` and are the source of truth; responses documented here):

| ID | Method path | Purpose | Auth | Request | Response | Errors |
|---|---|---|---|---|---|---|
| API-01 | POST `/tg/webhook` | Telegram updates | Header `X-Telegram-Bot-Api-Secret-Token` equals `TELEGRAM_WEBHOOK_SECRET` (S-10) | Telegram `Update` JSON | 200 empty (after handling, ≤ 1 s p95) | 401 wrong or missing secret; 200 for duplicates (no reprocessing); 500 only on unexpected errors (Telegram retries) |
| API-02 | POST `/webhooks/resend` | Email events | Svix signature verified with `resend.webhooks.verify` on the raw body (S-30, S-89) | Resend event JSON | 200 `{"ok":true}` | 400 `{"error":{"code":"invalid_signature"}}`; 200 for duplicates |
| API-03 | GET `/c/:token/paid` | Show the "I've paid" page | Token HMAC valid, action = `paid`, not expired | — | 200 HTML SCR-21 (ask, already reported or already paid state) | 404 HTML SCR-23 (invalid / expired / gone) |
| API-04 | POST `/c/:token/paid` | Confirm "I've paid" | Same token | form `confirm=1` | 200 HTML SCR-21 done state | 404 SCR-23; 429 SCR-23 CP-SCR23-5; 500 SCR-23 CP-SCR23-4 |
| API-05 | GET `/c/:token/stop` | Show the stop page | Token, action = `stop` | — | 200 HTML SCR-22 | 404 SCR-23 |
| API-06 | POST `/c/:token/stop` | Stop reminders for the invoice | Same token | form `confirm=1` | 200 HTML SCR-22 done state | as API-04 |
| API-07 | POST `/u/:token` | RFC 8058 one-click unsubscribe (also the form target of API-08) | Token, action = `unsub` | body `List-Unsubscribe=One-Click` (form-encoded or multipart, S-47) | 200 (empty body for one-click; HTML CP-SCR22-8 when `Accept: text/html`) — never a redirect, no cookies (S-47) | 404 empty / SCR-23 |
| API-08 | GET `/u/:token` | Unsubscribe confirmation page | Token, action = `unsub` | — | 200 HTML SCR-22 (ask all) | 404 SCR-23 |
| API-09 | GET `/healthz` | Liveness + DB check | none | — | 200 `{"status":"ok","db":"ok","version":"<git sha>"}` | 503 `{"status":"degraded","db":"down"}` |
| API-10 | GET `/privacy` | Privacy page | none | — | 200 HTML SCR-24 | — |

**Client action tokens.** `base64url( version(1) ‖ action(1) ‖ nudge_id(16) ‖ expires_epoch_s(4) ‖ HMAC-SHA-256(key=LINK_SIGNING_KEY, first 22 bytes)[0..16] )`, 54 characters. Expiry is 60 days after the email is sent (assumption 28). The token carries no PII. Verification is constant-time (`crypto.timingSafeEqual`). An unknown, altered or expired token and a deleted invoice all return the same 404 page family (EC-SEC-11).

**Error envelope** (JSON endpoints): `{ "error": { "code": "<snake_case>", "message": "<English, no PII>" } }`. HTML endpoints render SCR-23.

**Versioning.** HTTP paths are unversioned (public URLs in sent emails must keep working for 60 days). Token `version` byte = 1; a new token format increments it, and old versions stay accepted until expiry.

**Bot contract (callback data).** Format `v1:<action>:<uuid>` or `v1:<action>:<uuid>:<arg>`, ≤ 64 bytes (S-14), parsed by one zod schema in `src/bot/callbacks.ts`. Actions: `inv` (open card), `pay` (paid in full), `part`, `undo`, `reopen`, `pause`, `resume`, `now` (send now), `now_ok`, `edit`, `more`, `plan`, `plan_set:<preset>`, `fwd`, `cancel`, `cancel_ok`, `del`, `del_ok`, `hold`, `claim_ok` (client-says-paid confirm), `claim_no`, `cli` (client card), `cli_edit:<field>`, `cli_arch`, `cli_arch_ok`, `page:<n>`, `tz:<index>`, `set:<setting>`, `exp`, `acct_del`, `acct_del_ok`. Every ID in callback data is re-authorised against the sender's `freelancer_id` (§12.6). An unknown action or an ID that fails authorisation → CP-SCR20-2 or CP-SCR12-32.

**Pagination.** Offset paging with a stable sort: /owed sorts by (group, `due_date`, `reference`, `id`); /clients by (decrypted name case-insensitive, `id`). Page size 8 (owed) and 10 (clients). Page numbers are in callback data; a page beyond the end renders the last page.

### 12.6 Auth and authorisation model

**Principals.** Telegram (webhook caller), Resend (webhook caller), freelancer (Telegram user with an ENT-Freelancer row), operator (Telegram user ID in `ADMIN_TELEGRAM_IDS`), client (holder of a valid action token), anonymous (anyone else).

| Action | Anonymous Telegram user | Freelancer (onboarding) | Freelancer (active) | Freelancer (sending suspended) | Operator | Client (token) |
|---|---|---|---|---|---|---|
| /start, onboarding steps | ✔ (creates account) | ✔ | ✔ (returning) | ✔ | ✔ (as freelancer) | — |
| /new, /owed, /clients, /settings, cards, payments | ✘ → CP-SCR20-8 | ✘ → CP-SCR20-8 | ✔ own data only | ✔ own data only | ✔ own data only | — |
| Send reminders (automatic or send-now) | — | ✘ (email unverified) | ✔ | ✘ (CP-SCR15-12) | — | — |
| Export / delete account | — | ✔ (delete) | ✔ | ✔ | ✔ own | — |
| /admin commands | ✘ → CP-SCR20-1 | ✘ → CP-SCR20-1 | ✘ → CP-SCR20-1 | ✘ → CP-SCR20-1 | ✔ | — |
| "I've paid" / stop / unsubscribe for one nudge's invoice | — | — | — | — | — | ✔ only the invoice / freelancer–address pair encoded in the token |

**Object-level rule.** Every repository function takes `freelancerId` as its first argument, and every query on an owned table filters by it. A callback or command that names an ID belonging to another freelancer behaves exactly like a missing ID (CP-SCR12-32) and changes nothing (EC-PERM-02). Tested by `test/integration/authz.int.test.ts`, which replays every callback action with another freelancer's IDs.

**Sessions.** None to manage: each update is authenticated by the webhook secret, and the Telegram user ID in it identifies the principal. Account deletion ends everything; a suspended freelancer can still use tracking. Operator rights change when `ADMIN_TELEGRAM_IDS` changes (restart).

### 12.7 Integrations

**INT-telegram — Telegram Bot API**
- **Purpose.** All freelancer and operator interaction; webhook in, messages out.
- **Docs.** S-10, S-12, S-13, S-08, S-82; library grammY 1.46.0 (Bot API 10.3) [VERIFIED — S-59, 2026-09-30] [VERIFY-AT-BUILD].
- **Environments.** Separate bots (tokens from @BotFather) for staging and production. Local development uses the staging bot with a tunnel, or tests only.
- **Sandbox.** None relied on. Automated tests replace the network with a grammY API transformer that records calls and returns canned results [VERIFIED — S-58, S-90, 2026-09-30].
- **Timeouts and retries.** Interactive replies: the auto-retry plugin with `maxRetryAttempts: 3`, `maxDelaySeconds: 5`, so a handler never waits long enough to breach AD-6 [VERIFIED — S-56, 2026-09-30]. Outbox dispatcher: `maxRetryAttempts: 1` per tick, and our own backoff of 1 min, 5 min, 30 min, 2 h, 6 h, then `dropped` (logged) after 24 h.
- **Idempotency.** Telegram sends are not idempotent; our outbox `dedupe_key` (e.g. `digest:<freelancer>:<local-date>`, `nudge_sent:<nudge_id>`) prevents duplicate enqueues. A retry after a network timeout can rarely duplicate a notice; this is accepted for notices. The digest's `announced_at` is set only after Telegram confirms delivery (DR-12).
- **Errors.** 429 → wait `retry_after` (S-09). 403 "bot was blocked" → set `bot_blocked_at`, drop pending notices for that freelancer. 400 "message is not modified" on edits → treat as success.
- **Fallback.** If Telegram is down: incoming updates queue on Telegram's side for up to 24 h (S-10). Digests fail to deliver, so their nudges stay unannounced and are postponed (never sent unannounced). Reminder emails already announced still go out.
- **Credentials owner.** Operator (P4). Bot usernames are OQ-01.

**INT-email — Resend**
- **Purpose.** Reminder emails and verification codes.
- **Docs.** S-26, S-27, S-28, S-29, S-30, S-72, S-73, S-74, S-89; SDK `resend` 6.31.0 [VERIFIED — S-31, 2026-09-30].
- **Status.** Active; pricing Free 3,000/month, 100/day; Pro US$20/month for 50,000 [VERIFIED — S-26, 2026-09-30] [VERIFY-AT-BUILD]. At the ceiling (3,000/day, about 90,000/month), the plan is Pro US$35/month for 100,000 [VERIFIED — S-26, 2026-09-30] [VERIFY-AT-BUILD].
- **Sending domain.** A subdomain, as Resend recommends (S-74), e.g. `send.<product-domain>`, with SPF, DKIM and DMARC [UNKNOWN → OQ-07]. Sending region: EU if offered (OQ-11).
- **Sandbox.** Test recipients `delivered@resend.dev`, `bounced@resend.dev`, `complained@resend.dev` with `+label` (S-28). Staging rewrites every *reminder* recipient to `delivered+<nudge_id>@resend.dev`, or `bounced+…`/`complained+…` when the client's name starts with "Bounce Test" or "Complaint Test" (staging-only fixtures).
- **Timeout.** 10 s per call.
- **Retries.** Attempts at +1 min, +5 min, +30 min, +2 h (4 in total), always with `idempotencyKey: "nudge/<nudge_id>"` [VERIFIED — S-27, S-89, 2026-09-30]. Because keys expire after 24 h, a nudge still `sending` 23 h after its first attempt becomes `unknown` and is never auto-resent (DR-07).
- **Rate limit.** 10 requests/s per team (S-73). We send sequentially, at most 5 requests/s, and honour `retry-after` on 429.
- **Webhooks.** Events `email.delivered`, `email.bounced`, `email.complained`, `email.delivery_delayed`, `email.failed`, `email.suppressed` subscribed (S-29); `email.opened` / `email.clicked` not subscribed and tracking not enabled. Verified with Svix via `resend.webhooks.verify` (S-30, S-89). Dedupe on `svix-id`.
- **Fallback.** Provider down → nudges stay in retry, then `failed` with CP-SCR15-2; the verification email shows CP-SCR03-11.
- **Credentials owner.** Operator (API key, webhook secret).

**INT-sentry — Sentry error tracking**
- **Purpose.** Unhandled errors and failed jobs with stack traces.
- **Docs.** S-63; SDK `@sentry/node` 11.1.0 [VERIFIED — S-31, 2026-09-30].
- **Status.** Developer plan free: 1 user, 5,000 errors/month, 30-day lookback [VERIFIED — S-63, 2026-09-30] [VERIFY-AT-BUILD].
- **Privacy.** The SDK is configured to attach no request bodies, headers, IPs or user objects. The exact option names are checked against the Sentry Node docs at build [VERIFY-AT-BUILD]. A test transport asserts that no fixture PII appears in captured events (FF-08).
- **Sandbox.** Not needed; disabled when `SENTRY_DSN` is empty (local, test).
- **Fallback.** If Sentry is unreachable, errors are still logged (level `error`) with the correlation ID.
- **Credentials owner.** Operator.

**Hosting (Render)** is covered in §12.8 and §13.8.

### 12.8 Environments, configuration and secrets

| Environment | Purpose | Bot | Database | Email | Notes |
|---|---|---|---|---|---|
| local | Development | none, or the staging bot via tunnel | Docker `postgres:18` | fake gateway | `npm run dev` |
| test | `check` and CI | fake (API transformer) | Testcontainers Postgres 18 (@testcontainers/postgresql 12.2.0, S-31) | fake gateway | deterministic clock |
| staging | Pre-production on Render | staging bot | Render Postgres (paid, smallest) | Resend; reminder recipients overridden to `@resend.dev` (verification codes go to the tester's own address) | refuses to start without the override |
| production | Live | production bot | Render Postgres (paid) | Resend real | refuses to start with the override set |

**Environment variables** (validated by one zod schema in `src/platform/config/schema.ts` at startup; any missing or invalid value stops the process with a message naming the variable, EC-OPS-01):

| Variable | Meaning | Example / format | Required in |
|---|---|---|---|
| `APP_ENV` | `local` · `test` · `staging` · `production` | `staging` | all |
| `PORT` | HTTP port | `10000` | staging, production |
| `PUBLIC_BASE_URL` | Base for client links | `https://app.<domain>` (https only) | staging, production |
| `DATABASE_URL` | Postgres connection string | `postgres://…` | all except test (Testcontainers provides it) |
| `TELEGRAM_BOT_TOKEN` | Bot token | from @BotFather | staging, production |
| `TELEGRAM_BOT_USERNAME` | For CP-SCR20-6 and botInfo | `InvoiceNudgeBot` | staging, production |
| `TELEGRAM_WEBHOOK_SECRET` | `secret_token` value | 64 chars `[A-Za-z0-9_-]` (S-10) | staging, production |
| `RESEND_API_KEY` | Email API key | `re_…` | staging, production |
| `RESEND_WEBHOOK_SECRET` | Svix signing secret | `whsec_…` | staging, production |
| `EMAIL_FROM_ADDRESS` | Sender address | `reminders@send.<domain>` | staging, production |
| `EMAIL_RECIPIENT_OVERRIDE` | Staging rewrite of reminder recipients (verification emails are not rewritten) | `resend-test` | staging only; must be unset in production |
| `SUPPORT_EMAIL` | Shown in CP-SCR15-12 and CP-SCR20-10 (address decided in OQ-10) | `support@<domain>` | staging, production |
| `DATA_ENCRYPTION_KEYS` | Comma list `id:base64key` (32-byte keys) | `1:…` | all |
| `DATA_ENCRYPTION_ACTIVE_KEY_ID` | Key ID used for new writes | `1` | all |
| `BLIND_INDEX_KEY` | 32-byte base64 HMAC key | — | all |
| `LINK_SIGNING_KEY` | 32-byte base64 HMAC key for tokens | — | all |
| `CODE_PEPPER` | 32-byte base64 key for verification-code hashes | — | all |
| `ADMIN_TELEGRAM_IDS` | Comma list of operator user IDs | `123456789` | staging, production |
| `SENTRY_DSN` | Error tracking | URL, or empty to disable | optional |
| `LOG_LEVEL` | pino level | `info` | optional (default `info`) |
| `RUN_SCHEDULER` | Run loops in this process | `true` | optional (default `true`) |
| `SCHEDULER_TICK_SECONDS` | Loop period | `30` | optional (default 30, allowed 5–120) |
| `NUDGES_LIVE` | `off` · `allowlist` · `on` | `allowlist` | all (default `off`) |
| `NUDGES_ALLOWLIST` | Telegram user IDs allowed to send when `allowlist` | `123,456` | when `allowlist` |
| `EMAIL_GATEWAY_MODE` | `resend` · `sink` (log instead of send; load tests only) | `resend` | optional (default `resend`); `sink` refused unless `APP_ENV=staging` |
| `TELEGRAM_GATEWAY_MODE` | `api` · `sink` (record instead of calling Telegram; load tests only) | `api` | optional (default `api`); `sink` refused unless `APP_ENV=staging` |

**Secrets.** Stored only in Render environment groups and the operator's password manager; never in the repo (gitleaks in `check`), logs or error payloads. Encryption, blind-index, link and pepper keys are generated once per environment with `node -e "console.log(crypto.randomBytes(32).toString('base64'))"`, and each production key is recorded in two places (password manager + offline copy). Losing `DATA_ENCRYPTION_KEYS` makes the data unreadable (R-10).

**Key rotation** (documented in `docs/RUNBOOK.md`, E08-S01): add a new key ID to `DATA_ENCRYPTION_KEYS`, switch `DATA_ENCRYPTION_ACTIVE_KEY_ID`, deploy, run `npm run crypto:reencrypt` (batched, idempotent), then remove the old key after verification. Rotating `BLIND_INDEX_KEY` requires recomputing all `_bidx` columns with the same script. Rotating `LINK_SIGNING_KEY` invalidates outstanding email links, so it is done only after a leak.

### 12.9 Observability, security, performance and backups

**Logging.** pino (Fastify's logger) JSON to stdout; Render collects stdout. Every log line has `level`, `time`, `msg`, `event` (dotted name, e.g. `nudge.sent`), `correlation_id` (`upd:<update_id>` for bot, Fastify request ID for HTTP, `tick:<uuid>` for loops) and IDs (`freelancer_id`, `invoice_id`, `nudge_id`). **Never** logged: names, emails, payment instructions, message text, tokens, codes, secrets. pino `redact` paths cover `req.headers.authorization`, `req.headers["x-telegram-bot-api-secret-token"]`, `*.email`, `*.name`, `*.text`, `*.token`. FF-08 enforces the rule by capturing all log output of the e2e suite and searching it for fixture PII.

**Key log events.** `bot.update.received` / `.duplicate` / `.unhandled`; `invoice.created` (`entry_duration_ms`); `payment.recorded`; `nudge.planned` / `.announced` / `.claimed` / `.sent` (`latency_ms` from `scheduled_for`) / `.retry` / `.failed` / `.unknown` / `.cancelled` (`reason`); `email_event.received` (`type`); `suppression.added` (`reason`); `sending.suspended`; `notification.sent` / `.dropped`; `scheduler.tick` (`claimed`, `duration_ms`); `config.invalid`.

**Metrics.** No metrics backend in v1 (AD-7). Operational counts come from SQL views (§12.4) shown by `/admin stats`, and from log events. Health: API-09 checks the DB with `select 1` (timeout 2 s).

**Error tracking.** Sentry for unhandled errors and for `failed` / `unknown` nudges (INT-sentry).

**Alerts** (E08-S04), sent to operators via Telegram by the app itself, at most once per hour per kind:
- error rate > 5% of updates in 15 min;
- nudge send failures > 10 in 1 h;
- due-but-unsent nudges older than 30 min (> 20 rows, a scheduler stall);
- notification backlog > 200 pending;
- complaint received (always);
- `check` of `/healthz` from outside [UNKNOWN → OQ-16].

**Security baseline** (mapped to OWASP Top 10:2025 [VERIFIED — S-49, 2026-09-30]):

| OWASP 2025 | Our control |
|---|---|
| A01 Broken Access Control | Object-level `freelancer_id` scoping in every repo function; callback IDs re-authorised; operator allow-list; signed client tokens scoped to one nudge (§12.6) |
| A02 Security Misconfiguration | Fail-fast config; staging/production guards (EC-OPS-10); security headers on HTML: `Content-Security-Policy: default-src 'none'; style-src 'unsafe-inline'; form-action 'self'; base-uri 'none'; frame-ancestors 'none'`, `Referrer-Policy: no-referrer`, `X-Content-Type-Options: nosniff`, `Cache-Control: no-store` |
| A03 Software Supply Chain Failures | Lockfile committed, `npm ci`; exact version pins; `npm audit --audit-level=high` in `check` (S-86); new dependency requires a DR; gitleaks secret scan (S-81) |
| A04 Cryptographic Failures | AES-256-GCM field encryption (Node `crypto`), HMAC-SHA-256 tokens and blind indexes, keys outside the DB; TLS enforced by Render and Telegram (S-13, S-83) |
| A05 Injection | Kysely parameterised queries only (no `sql.raw` with user input, lint-enforced); HTML escaping in the copy renderer for Telegram HTML and pages; email HTML built from escaped templates |
| A06 Insecure Design | Abuse caps, verified reply-to, templates-only reminder text, digest before send (DR-12), opt-out (DR-13) |
| A07 Authentication Failures | Webhook secret compared in constant time; verification codes hashed, 5 attempts, cooldowns and hourly cap |
| A08 Software or Data Integrity Failures | Svix-verified email webhooks; HMAC tokens; migrations reviewed and forward-only |
| A09 Security Logging and Alerting Failures | Structured logs with correlation IDs; audit events for sensitive actions; alerts above |
| A10 Mishandling of Exceptional Conditions | Typed errors (§13.9); every handler has a fallback reply (CP-SCR20-4); transactions roll back on error; no swallowed exceptions (lint rule) |

**Rate limits.** Per Telegram user: 30 updates/min (in-memory token bucket, CP-SCR20-7). Client pages: 30 requests/min per IP via `@fastify/rate-limit` 11.2.0 (S-31). Email webhook: none (signature required). Verification codes: 60 s cooldown, 5/hour. Sending caps: assumption 21.

**Performance budgets** (asserted in E08-S02 load test on staging, and where possible in integration tests):

| Budget | Target | Where asserted |
|---|---|---|
| Webhook acknowledgement | p95 < 1 s, p99 < 3 s at 20 updates/s | E08-S02 |
| DB queries per update | ≤ 12 | integration test query counter (`db.queryCount`) |
| /owed server time at 500 open invoices | ≤ 300 ms | `owed.int.test.ts` |
| Scheduler tick with 50 due nudges (fake email) | ≤ 5 s | `scheduler.int.test.ts` |
| Client page | ≤ 15 KB HTML, TTFB ≤ 300 ms on staging | FF-15, E08-S02 |
| Memory | RSS ≤ 350 MB on the 512 MB instance | E08-S02 |

**Backups and restore.**
- Render Postgres PITR: 3 days on a Hobby workspace, 7 days on Pro. A restore creates a new database instance (S-38). Before public launch the workspace moves to Pro for 7-day PITR [UNKNOWN → OQ-05].
- **Restore procedure** (`docs/RUNBOOK.md`, rehearsed in E08-S01 and then monthly):
  1. In Render, restore the production DB to a new instance at time T (≥ 10 minutes ago, S-38).
  2. Set `RUN_SCHEDULER=false` on the web service, so no reminders go out while data is inconsistent.
  3. Point `DATABASE_URL` at the restored instance and deploy.
  4. Run `npm run ops:verify-restore`: row counts, latest `updated_at`, decrypt a sample with the current keys.
  5. Review nudges that were `sent` after T in the old DB (from Resend's dashboard or logs) and mark them `sent` in the restored DB, so no duplicates go out.
  6. Set `RUN_SCHEDULER=true`.
  7. Record timings in `docs/OPS-LOG.md`. Target RTO ≤ 4 h, RPO ≤ 15 min.
- An untested backup is not a backup: E08-S01 includes the drill and records its result.

### 12.10 Fitness functions

| ID | Architectural rule it enforces | Automated check | Where it runs | Fails the build? |
|---|---|---|---|---|
| FF-01 | No circular dependencies anywhere in `src/` | dependency-cruiser rule `no-circular` (S-76) | `npm run arch` in `check` | Yes |
| FF-02 | Outside a module, only its `index.ts` may be imported | two dependency-cruiser forbidden rules (S-76): (a) from `path: "^src/modules/([^/]+)/"` to `path: "^src/modules/[^/]+/.+"`, `pathNot: "^src/modules/$1/\|^src/modules/[^/]+/index[.]ts$"`; (b) from `path: "^src/(bot\|web\|main)/"` to `path: "^src/modules/[^/]+/.+"`, `pathNot: "^src/modules/[^/]+/index[.]ts$"` | `check` | Yes |
| FF-03 | Module dependencies follow the DAG in §12.2 | one dependency-cruiser forbidden rule per module listing disallowed targets | `check` | Yes |
| FF-04 | `platform` imports no module or adapter | dependency-cruiser: from `^src/platform/` to `^src/(modules\|bot\|web\|main)/` forbidden | `check` | Yes |
| FF-05 | grammY only in `src/bot/**` and `src/platform/telegram/**`; Fastify only in `src/web/**` and `src/main/**` | dependency-cruiser rules on the `grammy` and `fastify` packages | `check` | Yes |
| FF-06 | SQL (`kysely`, `pg`) only in `src/modules/*/repo/**`, `src/bot/repo/**` (conversation state, processed updates), `src/platform/db/**`, `src/platform/audit/**` and migrations | dependency-cruiser rule on the `kysely` and `pg` packages | `check` | Yes |
| FF-07 | Each module (and `bot`) queries only its own tables (plus whitelisted `v_*` views in `admin`) | Vitest architecture test `test/arch/table-ownership.test.ts` parses `selectFrom`/`insertInto`/`updateTable`/`deleteFrom` string literals in each `repo/` file and compares them with the ownership map in §12.2 | `check` (unit project) | Yes |
| FF-08 | No PII in logs or error-tracking events | `test/e2e/no-pii.e2e.test.ts` runs the critical e2e scenarios with capturing logger and Sentry test transport, then searches for every fixture name and email | `check` (e2e project) | Yes |
| FF-09 | No inline user-facing strings in Telegram calls | ESLint `no-restricted-syntax`: a string or template literal as the text argument of `reply`, `editMessageText`, `sendMessage` or `answerCallbackQuery` in `src/bot/**`, `src/modules/**` and `src/platform/telegram/**` | `npm run lint` in `check` | Yes |
| FF-10 | Copy catalogue equals the §11 copy tables | `test/arch/copy-catalog.test.ts` parses `docs/BLUEPRINT.md` §11 for `CP-…` IDs and compares with `catalog.ts` keys (both directions) | `check` | Yes |
| FF-11 | Migrations and generated DB types agree | `npm run db:verify`: start Testcontainers PG 18, run migrations, `kysely-codegen --dialect postgres --out-file src/platform/db/types.gen.ts --verify` (S-36) | `check` | Yes |
| FF-12 | No wall-clock access outside `platform/time` | ESLint `no-restricted-globals` / `no-restricted-syntax` for `Date`, `Date.now`, `Temporal.Now` outside `src/platform/time/**` | `check` | Yes |
| FF-13 | Email provider SDK only behind the gateway | dependency-cruiser: `resend` importable only from `src/platform/email/**` | `check` | Yes |
| FF-14 | No plaintext PII in the database | `test/e2e/no-plaintext-pii.e2e.test.ts` dumps every table (`select * `) after the e2e scenarios and searches for fixture names, emails and payment text | `check` | Yes |
| FF-15 | Client pages stay small and self-contained | `test/integration/pages.int.test.ts`: each SCR-21–SCR-24 state < 15 KB, no `http(s)://` resource URLs other than `PUBLIC_BASE_URL` links, `<html lang="en">`, exactly one `<h1>`, a `<title>`, every `<button>` has text | `check` | Yes |

Rules that cannot be enforced mechanically, and so are checked in review: "business logic in `domain/` is pure" (review D2), and "no module reads another's data by any side channel" (review D2 + FF-07 heuristic).

### 12.11 Cross-cutting decision list

| Area | Item | Decision |
|---|---|---|
| Data | Relational vs other | PostgreSQL 18, DR-05 |
| Data | Ownership model | One module per table, §12.2, FF-07 |
| Data | Identifiers | `uuidv7()`; never shown to users; tokens instead (§12.5) |
| Data | Soft vs hard delete | Invoices: "Cancel" is a status, "Delete" is hard; clients: archive only; accounts: hard delete (DR-08, E07-S04); audit rows anonymised |
| Data | Timestamps and zones | DR-11 |
| Data | Money | DR-10 |
| Data | Migrations | §12.4 migration policy |
| Data | Seeds and fixtures | Factories in `test/factories/`; dev seed guarded (EC-OPS-07) |
| Data | Retention and deletion | §12.4 retention table |
| Time | Sync vs queued | Webhook handlers do DB work + one Telegram reply only; emails and notices are queued rows (DR-21) |
| Time | Idempotency | `update_id` dedupe (DR-09); `nudge/<id>` idempotency key (DR-06); `svix-id` dedupe; outbox `dedupe_key`; client actions are idempotent state changes |
| Time | Transaction boundaries | One transaction per command or callback, including `InvoiceChanged` handlers; claims in short transactions; the provider call happens outside any open transaction, with state written before and after (E05-S02) |
| Time | Eventual consistency | Only notifications (seconds) and digest announcements; money and nudge state are strongly consistent |
| Time | Concurrency control | `invoice.version` optimistic lock; `FOR UPDATE SKIP LOCKED` for claims; unique constraints for dedupe |
| Time | Scheduled jobs and missed runs | Loops every 30 s; catch-up on start; > 24 h late → re-plan (DR-21) |
| Interfaces | API style and contract | §12.5; zod schemas are the source of truth |
| Interfaces | Versioning | Token version byte; callback prefix `v1:` |
| Interfaces | Error shape | Single envelope (§12.5); typed errors (§13.9) |
| Interfaces | Pagination | Offset + stable sort (§12.5) |
| Interfaces | Validation location | At the trust boundary (bot handlers, web routes) with zod, and again in domain functions for invariants |
| Interfaces | Thin or stateful client | Telegram is a thin client; conversation state is server-side (ENT-ConversationState) |
| Identity | Authentication | DR-16 |
| Identity | Role × permission | §12.6 |
| Identity | Object-level rule | §12.6 |
| Identity | Admin separation | `/admin` hidden, allow-listed, audited |
| Identity | Ending sessions everywhere | Not applicable: no sessions; deletion ends access |
| Failure | Timeouts / retries | §12.7 per integration |
| Failure | Circuit breaking | Explicitly absent: volumes are low, and backoff + unknown-state handling suffice |
| Failure | Degrade vs block | Telegram down → no digests, so announced-only sending continues; Resend down → retries then `failed`; DB down → webhook 500 (Telegram retries) and `/healthz` 503 |
| Failure | Partial failure of multi-step operations | Account deletion in one transaction; invoice + nudge planning in one transaction; send pipeline states make each step resumable |
| Operations | Environments | §12.8 |
| Operations | Config and secrets | §12.8 fail-fast |
| Operations | Logging and PII | §12.9 |
| Operations | Health and metrics | API-09; SQL views |
| Operations | Backup and restore | §12.9 procedure; E08-S01 drill |
| Operations | Deploy and rollback | §13.8 |
| Operations | State outside the DB | None (no uploads, no cache); exports are streamed to Telegram and not stored |
| Frontend | Rendering strategy | Server-rendered HTML for SCR-21–SCR-24, no JS (DR-15) |
| Frontend | State management / data fetching | Not applicable (no client-side app) |
| Frontend | Forms and validation | One POST button per page; server-side validation of token and body |
| Frontend | Asset and font loading | None: system fonts, inline CSS (FF-15) |
| Frontend | Offline behaviour | Not applicable |

## 13. Tech stack and engineering conventions (Tech Stack)

### 13.1 Stack decision and stack table

**Stack table.** Every row says what is used (pinned), why (the deciding driver or agent criterion), what the runner-up would have bought, and the evidence. The language choice itself is scored after the table.

| Layer | Technology | Pinned version | Deciding reason (driver / agent criterion) | Runner-up — why it lost, what it would have bought | Evidence |
|---|---|---|---|---|---|
| Runtime | Node.js | 26.10.0 (`.node-version`); move to the first 26.x LTS patch after 2026-10-28 | AD-3: Temporal enabled by default; EOL 2029-04-30 | Node 24.21.0 LTS: "LTS today" and Render's default (S-62); lost because it needs a Temporal polyfill dependency and an upgrade within 19 months | [VERIFIED — S-32, S-60, 2026-09-30] [VERIFY-AT-BUILD] |
| Package manager | npm (bundled) | 11.19.1 (ships with Node 26.10.0) | Agent default; built-in `npm audit`; one fewer tool | pnpm: faster installs and strict dependency isolation; lost on "one fewer tool" | [VERIFIED — S-32, 2026-09-30] |
| Language | TypeScript | 6.0.3 (exact) | Strict by default; Temporal types; supported by typescript-eslint | TypeScript 7.0.2: about 10× faster type checks (S-33); lost because typescript-eslint 8.71.0 requires `typescript <6.1.0` | [VERIFIED — S-31, S-33, S-61, 2026-09-30] |
| Bot framework | grammY (+ `@grammyjs/auto-retry`) | 1.46.0 (+ 2.0.2) | Typed Bot API 10.3 surface; documented webhook, flood, deployment and testing guidance (S-54, S-56, S-58) | aiogram 3.31.0 (Python): built-in FSM; lost with the language decision | [VERIFIED — S-31, S-59, S-65, 2026-09-30] |
| HTTP server | Fastify (+ `@fastify/rate-limit`, `@fastify/formbody`) | 5.12.5 (+ 11.2.0, 9.0.0) | grammY adapter (`"fastify"`), `inject()` testing, JSON schemas, pino built in | Node `http` + grammY `"http"` adapter: fewer dependencies; lost because routing, form parsing, rate limiting and test injection would be hand-written | [VERIFIED — S-31, S-57, S-75, 2026-09-30] |
| Database | PostgreSQL (Render managed, Frankfurt) | 18.6 | AD-1 row claiming with `SKIP LOCKED`; AD-2 managed PITR; AD-4 AES-256 at rest; `uuidv7()` | SQLite: zero DB operations (invioTrack's choice, S-01); lost on PITR and concurrent claiming | [VERIFIED — S-37, S-38, S-83, S-84, 2026-09-30] |
| Query builder and migrations | Kysely + `pg` driver; `kysely-codegen` (dev) | 0.29.6 + 8.23.0; 0.20.0 | SQL-shaped typed queries; locking migrator; `--verify` drift check (FF-11) | Drizzle ORM 0.45.3: TS-defined schema and generated migrations; lost because 1.0 is in release candidate (churn during our build) | [VERIFIED — S-31, S-34, S-35, S-36, 2026-09-30] |
| Validation and config | zod | 4.6.5 | One schema language for config, callback data and HTTP bodies | Fastify JSON schema only: no extra dependency; lost because config and callback parsing also need it | [VERIFIED — S-31, 2026-09-30] |
| Logging | pino | 10.3.1 | Fastify's native logger; JSON; `redact` | Console: nothing to install; lost on structure and redaction | [VERIFIED — S-31, 2026-09-30] |
| Date and time | Temporal (built in) + TS `lib: ["esnext"]` | Node 26 built-in | AD-3 (see scoring) | `temporal-polyfill` 1.0.5 on Node 24: LTS today; lost as above | [VERIFIED — S-60, S-61, S-31, 2026-09-30] |
| Cryptography | `node:crypto` (AES-256-GCM, HMAC-SHA-256, `timingSafeEqual`, `randomInt`) | built in | AD-4 without a dependency | libsodium bindings: misuse-resistant APIs; lost on dependency count | [ASSUMED — Node built-in; algorithm availability is asserted by unit tests at E01-S01] [VERIFY-AT-BUILD] |
| Email API | Resend + `resend` SDK | API current; SDK 6.31.0 | AD-1 idempotency keys; AD-5 webhooks and test addresses | Postmark 5.1.0 SDK: stream separation, long reputation; lost on missing idempotency (S-25) | [VERIFIED — S-26, S-27, S-28, S-31, S-89, 2026-09-30] [VERIFY-AT-BUILD] |
| Error tracking | Sentry `@sentry/node` | 11.1.0 | Free tier fits (5k errors/month); Node 26 in engines | Logs only: no vendor; lost on alerting and grouping | [VERIFIED — S-31, S-63, 2026-09-30] |
| Unit / integration / e2e tests | Vitest (+ `@vitest/coverage-v8`) | 5.0.2 (+ 5.0.2) | Fast TS-native runner; projects for unit, integration and e2e; v8 coverage thresholds (S-79) | Node's built-in test runner: no dependency; lost on coverage thresholds and projects | [VERIFIED — S-31, S-79, 2026-09-30] |
| Test database | `@testcontainers/postgresql` | 12.2.0 | Real Postgres 18 per test run (SKIP LOCKED, constraints) | PGlite: no Docker; lost because it is single-connection, so claiming races can't be tested | [VERIFIED — S-31, 2026-09-30] |
| Mutation testing | StrykerJS (`@stryker-mutator/core` + `vitest-runner`) | 10.0.0 + 10.0.0 | Maintained mutation tool for this stack; `thresholds.break` fails CI (S-78) | None (red-before-green only): cheaper CI; lost because money, time and eligibility rules need a stronger signal | [VERIFIED — S-31, S-78, 2026-09-30] |
| Lint | ESLint + `@eslint/js` + typescript-eslint (`strictTypeChecked`) + `@vitest/eslint-plugin` + `eslint-config-prettier` | 10.11.0 + 10.0.1 + 8.71.0 + 1.6.27 + 10.1.8 | AD-1: floating promises, unsafe `any`, non-exhaustive switches become errors | Biome 2.5.14: one fast binary; lost on type-aware rule depth (DR-19) | [VERIFIED — S-31, S-77, 2026-09-30] |
| Format | Prettier | 3.9.9 | Zero-decision formatting | Biome formatter: faster; lost with the lint decision | [VERIFIED — S-31, 2026-09-30] |
| Architecture rules | dependency-cruiser | 18.4.0 | FF-01 to FF-06, FF-13 with path regex and group matching | eslint-plugin-boundaries 7.2.0: editor feedback; lost on report clarity (DR-20) | [VERIFIED — S-31, S-76, 2026-09-30] |
| Dead code | knip | 6.38.0 | Unused files, exports and dependencies fail `check` | ts-prune-style scripts: lighter; lost on dependency detection | [VERIFIED — S-31, 2026-09-30] |
| Duplication | jscpd | 5.3.3 | Duplicated code is the default failure of agent code; `--threshold` fails the build (S-80) | Review only: no tool; lost because reviewers can't see what they didn't read | [VERIFIED — S-31, S-80, 2026-09-30] |
| Secret scanning | gitleaks (binary) | 8.30.1 | `gitleaks dir` / `gitleaks git` with non-zero exit on findings (S-81) | GitHub secret scanning: no binary; lost because it doesn't run locally in `check` | [VERIFIED — S-44, S-81, 2026-09-30] |
| Dependency audit | `npm audit --audit-level=high` | npm 11.19.1 | Built in; exits non-zero at the threshold (S-86) | osv-scanner 2.6.0 (`osv-scanner scan source -r .`, S-85): broader database; lost on "no extra binary" | [VERIFIED — S-45, S-85, S-86, 2026-09-30] |
| Script runner (dev only) | tsx | 4.23.15 | Run TS scripts and the dev server without a build step | Node's own TypeScript stripping: no dependency; lost because its stability on 26.x was not verified this session | [VERIFIED — S-31, 2026-09-30] |
| Hosting | Render web service (plan `0.5c-512mb`, formerly "Starter") + Render Postgres, Frankfurt | platform (no version) | AD-2 managed deploys, pre-deploy migrations, CI-gated auto-deploy, PITR | Fly.io: cheaper compute and more regions (S-43); lost on platform concepts for one maintainer | [VERIFIED — S-38, S-39, S-41, S-42, 2026-09-30]; prices [UNKNOWN → OQ-05] |
| CI | GitHub Actions on a GitHub Pro account | platform | Required status check protects `main` for private repos | Public repo on Free: free protection; lost on code confidentiality | [VERIFIED — S-51, 2026-09-30]; account [UNKNOWN → OQ-06] |
| Container runtime (tests) | Docker Engine (local) / GitHub-hosted `ubuntu-latest` runner | platform | Required by Testcontainers | Local Postgres install: no Docker; lost on reproducibility | [ASSUMED — GitHub-hosted Ubuntu runners include Docker; not re-read this session] [VERIFY-AT-BUILD] |

**How the language and ecosystem were chosen.** Candidates were scored against the research-protocol criteria, weighted for a coding agent (0–3 per row), and then against the architecture drivers that separate them. Candidate A: TypeScript on Node.js 26 + grammY + Fastify + Kysely/PostgreSQL. B: Python 3.14 + aiogram 3.31 + SQLAlchemy 2.1/Alembic + pytest/mypy/ruff. C: Go 1.27 + go-telegram/bot + pgx/sqlc + go test.

| Criterion (weight) | A. TypeScript | B. Python | C. Go | Basis |
|---|---|---|---|---|
| Mainstream adoption + docs (5) | 3 | 3 | 2 — bot library docs thinner | [ASSUMED — judgement from library docs read this session: S-54, S-56, S-58 (grammY), S-65 (aiogram), S-67 (go-telegram)] |
| Strict static typing enforceable (5) | 3 — `tsc` strict + type-aware lint | 2 — mypy strict is gradual; third-party typing uneven | 3 | [ASSUMED — judgement; S-61, S-77] |
| One-command deterministic gate, fast (5) | 3 | 3 — ruff is very fast | 3 | [ASSUMED — judgement; S-31, S-66] |
| Mature testing tooling (4) | 3 — Vitest, Testcontainers, Stryker | 3 — pytest | 3 — go test | [VERIFIED — S-31, S-66, 2026-09-30] |
| Convention-heavy framework (4) | 2 — grammY + Fastify, conventions ours | 3 — aiogram routers + built-in FSM | 1 — minimal conventions | [ASSUMED — judgement] |
| Stable / LTS, low churn (4) | 2 — Node 26 LTS on 2026-10-28; TS 6→7 transition | 3 — Python 3.14 until 2030-10 | 3 — Go 1.27 | [VERIFIED — S-32, S-33, S-66, 2026-09-30] |
| Small dependency surface (3) | 1 — npm trees | 2 | 3 | [ASSUMED — judgement] |
| Hosting fit (4) | 3 | 3 | 3 | [ASSUMED — Render's Node support verified (S-62); its Python and Go support not re-read this session] |
| i18n / locale (3) | 2 — `Intl` built in | 2 | 2 | [ASSUMED — English-only v1 makes this row nearly neutral] |
| License (3) | 3 | 3 | 3 | [ASSUMED — npm package licenses verified (S-31); Python and Go library licenses not re-read this session] |
| Team familiarity (2) | 2 | 2 | 2 | [ASSUMED — no team; neutral] |
| **Generic total (max 126)** | **107** | **113** | **108** | [ASSUMED — weighted sum of the rows above] |
| AD-3 time correctness, rank 3 (×6) | 3 — `Temporal.PlainDate`, `ZonedDateTime`, `Instant` are distinct compile-time types [VERIFIED — S-88, S-60, S-61, 2026-09-30] | 1 — aware and naive datetimes are the same class; mixing them fails only at runtime [VERIFIED — S-87, 2026-09-30] | 1 — one `time.Time` type for dates and instants | [ASSUMED — Go's standard library time model not re-read this session] |
| AD-8 evolvability, rank 8 (×1) | 3 — a Mini App or dashboard front end would share the language and types | 1 | 1 | [ASSUMED — judgement] |
| Drivers AD-1, AD-2, AD-4, AD-5, AD-6, AD-7 | equal (3 each) | equal | equal | [ASSUMED — all three can implement DB claiming, one-process deploys, field encryption and caps equally] |
| **Driver-adjusted total** | **128** | **120** | **115** | [ASSUMED — generic total + AD-3 ×6 + AD-8 ×1] |

**Decision: A (TypeScript).** Python wins the generic agent criteria by 6 points and would have bought a faster, simpler gate (ruff), aiogram's built-in FSM conventions, a more stable runtime line and fewer dependencies. It loses on AD-3, the rank-3 driver: the product's most important computations mix calendar dates (due dates), wall-clock times (send hour) and instants (storage), and only Temporal makes mixing them a type error. That is the "a driver changed the answer" case, stated openly rather than hidden in the table.

### 13.2 Forbidden choices

| Forbidden | Reason | Evidence |
|---|---|---|
| TypeScript 7.x as the `typescript` dependency | typescript-eslint 8.71.0 peer range is `<6.1.0`; revisit when typescript-eslint supports TS 7 (R-06) | [VERIFIED — S-31, S-33, 2026-09-30] |
| `prisma` / `@prisma/client` | Not our ORM; the CLI's `latest` tag is 8.0.0-rc.19, an install trap | [VERIFIED — S-31, 2026-09-30] |
| `drizzle-orm` (any version) | Not our ORM; 1.0 is in release candidate | [VERIFIED — S-31, 2026-09-30] |
| Long polling (`bot.start()`) outside local experiments | Cannot coexist with a webhook; conflicts during deploy overlap (DR-09) | [VERIFIED — S-08, 2026-09-30] |
| grammY session / conversations plugins | Conversation state must live encrypted in ENT-ConversationState under our transaction rules; a second storage model would split it | [ASSUMED — design consistency, DR-08] |
| `Date`, `Date.now()`, moment, luxon, date-fns, dayjs | Temporal via `platform/time` only (FF-12) | [ASSUMED — DR-11 policy] |
| Floats for money, `parseFloat`, `Number()` on amounts outside the audited parser | Exactness (DR-10) | [ASSUMED — DR-10 policy] |
| Job-queue libraries (pg-boss, graphile-worker, BullMQ + Redis) | The nudge table is the queue (DR-21) | [VERIFIED — S-31, 2026-09-30] (versions checked; excluded by decision) |
| Render free web services for the bot | They sleep after 15 minutes idle, which breaks webhooks and loops | [VERIFIED — S-40, 2026-09-30] |
| Email open or click tracking; `email.opened` / `email.clicked` webhooks | Privacy (AD-4), assumption 35 | [VERIFIED — S-29, 2026-09-30] (events exist; not enabled) |
| Resend `react` email rendering | Pulls React into the server for four templates | [VERIFIED — S-89, 2026-09-30] (option exists; not used) |
| A second email provider SDK (postmark, SES) | One gateway (FF-13) | [VERIFIED — S-25, 2026-09-30] |
| Web fonts, CDNs, client-side JS on client pages | DR-15, FF-15 | [ASSUMED — DR-15 policy] |
| Analytics or tracking SDKs | AD-4; metrics come from our DB | [ASSUMED — AD-4 policy] |
| `sql.raw()` or string-built SQL with any user-derived value | Injection (A05) | [ASSUMED — security policy; lint rule in E00-S02] |
| Mini App tooling, front-end frameworks | DR-01 | [ASSUMED — DR-01 policy] |
| PDF or payment SDKs | DR-14 | [ASSUMED — DR-14 policy] |

### 13.3 Repository layout

```
.
├── AGENTS.md                      # Codex entry point (Appendix B)
├── CLAUDE.md                      # Claude Code entry point (identical content)
├── .node-version                  # 26.10.0
├── .env.example                   # every variable in §12.8 with a one-line meaning
├── package.json / package-lock.json
├── tsconfig.json / tsconfig.build.json
├── eslint.config.mjs / .prettierrc.json / .prettierignore
├── .dependency-cruiser.cjs        # FF-01..FF-06, FF-13
├── knip.json / .jscpd.json / stryker.config.json / vitest.config.ts
├── render.yaml                    # staging + production services
├── .github/workflows/ci.yml
├── docs/
│   ├── BLUEPRINT.md  PROGRESS.md  DECISIONS.md  REVIEW-CHECKLIST.md  RUNBOOK.md  OPS-LOG.md
├── scripts/                       # db-verify.ts, gen-currencies.ts, crypto-reencrypt.ts, ops-verify-restore.ts, load-test.ts
├── data/iso4217-list-one.xml      # source for currencies.json (S-68)
├── src/
│   ├── main/                      # start.ts (composition root), wire.ts (event subscriptions), loops.ts
│   ├── web/                       # server.ts, routes/{telegram,resend,client-pages,privacy,health}.ts, views/*.ts, schemas/*.ts
│   ├── bot/                       # bot.ts, middleware/*, conversation/*, handlers/*, keyboards/*, callbacks.ts, render.ts
│   ├── modules/
│   │   ├── accounts/              # index.ts, service.ts, domain/*, repo/*
│   │   ├── clients/
│   │   ├── invoices/
│   │   ├── nudging/               # domain/{plans,schedule,eligibility,transitions,compose,tokens}.ts, repo/*, scheduler.ts, sender.ts
│   │   ├── deliverability/
│   │   ├── notifications/
│   │   └── admin/
│   └── platform/
│       ├── config/  db/{index.ts,migrations/,types.gen.ts}  time/  money/{currencies.json,parse.ts,format.ts}
│       ├── crypto/  copy/{catalog.ts,render.ts}  log/  errors/  email/  telegram/  events/  audit/
└── test/
    ├── factories/                 # freelancer, client, invoice, nudge builders
    ├── harness/                   # testApp(), fakeTelegram(), fakeEmail(), fakeClock(), updates.ts
    ├── arch/                      # FF-07, FF-10 tests
    ├── integration/               # *.int.test.ts (real Postgres via Testcontainers)
    └── e2e/                       # *.e2e.test.ts (critical paths CE-1..CE-5, FF-08, FF-14)
```

Unit tests sit next to the code as `*.test.ts`.

### 13.4 The `check` command

`npm run check` is the definition of green. It runs identically locally and in CI, is non-interactive, and exits non-zero on any failure. Docker must be running (Testcontainers).

```json
{
  "scripts": {
    "format:check": "prettier --check .",
    "lint": "eslint . --max-warnings=0",
    "typecheck": "tsc -p tsconfig.json --noEmit",
    "arch": "depcruise src --config .dependency-cruiser.cjs",
    "deadcode": "knip",
    "dup": "jscpd src --threshold 3",
    "test": "vitest run",
    "test:cov": "vitest run --coverage",
    "db:verify": "tsx scripts/db-verify.ts",
    "audit": "npm audit --audit-level=high",
    "secrets": "gitleaks dir . --redact --exit-code 1",
    "check": "npm run format:check && npm run lint && npm run typecheck && npm run arch && npm run deadcode && npm run dup && npm run test:cov && npm run db:verify && npm run audit && npm run secrets",
    "mutate": "stryker run",
    "check:full": "npm run check && npm run mutate",
    "dev": "tsx watch src/main/start.ts",
    "build": "tsc -p tsconfig.build.json",
    "start": "node dist/main/start.js",
    "db:migrate": "node dist/platform/db/migrate.js",
    "db:migrate:dev": "tsx src/platform/db/migrate.ts",
    "db:seed:dev": "tsx scripts/seed-dev.ts",
    "gen:currencies": "tsx scripts/gen-currencies.ts",
    "test:contract": "vitest run --project contract",
    "ops:set-webhook": "node dist/scripts/ops-set-webhook.js",
    "ops:set-commands": "node dist/scripts/ops-set-commands.js",
    "ops:verify-restore": "node dist/scripts/ops-verify-restore.js",
    "crypto:reencrypt": "node dist/scripts/crypto-reencrypt.js",
    "load-test": "tsx scripts/load-test.ts"
  }
}
```

Expected runtime on a 4-core laptop: format + lint + typecheck ≤ 60 s; arch + knip + jscpd ≤ 20 s; tests including the coverage-floor check ≤ 90 s (unit ≤ 20 s); db:verify ≤ 30 s; audit + secrets ≤ 15 s. **Total ≤ 3.5 minutes** [ASSUMED — to be measured in E00-S02 and recorded as M-23]. `npm run mutate` (Stryker on critical modules) takes ≤ 10 minutes and runs in CI (§13.8), not in `check`.

### 13.5 Formatting, lint and type strictness

- **Prettier.** Defaults plus `printWidth: 100`, `singleQuote: true`, `trailingComma: "all"`. No other options.
- **tsconfig.json** (TypeScript 6 defaults include `strict: true`, `module: esnext` and `types: []`, S-61; set explicitly anyway):
  `"strict": true, "noUncheckedIndexedAccess": true, "exactOptionalPropertyTypes": true, "noImplicitOverride": true, "noFallthroughCasesInSwitch": true, "noImplicitReturns": true, "useUnknownInCatchVariables": true, "verbatimModuleSyntax": true, "module": "nodenext", "moduleResolution": "nodenext", "target": "es2025", "lib": ["esnext"], "types": ["node"], "skipLibCheck": false, "isolatedModules": true`.
- **ESLint** (`eslint.config.mjs`, flat config per S-77): `js.configs.recommended`, `tseslint.configs.strictTypeChecked`, `tseslint.configs.stylisticTypeChecked`, `languageOptions.parserOptions.projectService: true`, `@vitest/eslint-plugin` recommended for tests, `eslint-config-prettier` last. Additional rules set to `error`: `@typescript-eslint/switch-exhaustiveness-check`, `@typescript-eslint/no-explicit-any`, `@typescript-eslint/ban-ts-comment` (only `ts-expect-error` with a description of ≥ 10 chars), `no-console`, `eqeqeq`, `@typescript-eslint/explicit-module-boundary-types`, `no-restricted-syntax` / `no-restricted-globals` for FF-09, FF-12 and `sql.raw`, `no-restricted-imports` for the forbidden packages in §13.2.
- **Escape hatches.** `any`, `as unknown as`, non-null `!`, `@ts-ignore` and `eslint-disable` are banned except on a line with a trailing justification: `// eslint-disable-next-line <rule> -- <reason> (DR-nn)`. The review checklist counts them; every one needs a DR or an Amendment.

### 13.6 Testing conventions

- **Layout.** Unit tests next to code (`src/**/x.test.ts`); integration in `test/integration/*.int.test.ts`; e2e in `test/e2e/*.e2e.test.ts`; architecture tests in `test/arch/`. Vitest `projects` for `unit`, `integration`, `e2e` (S-79).
- **Naming.** `describe('<unit or behaviour>')` + `it('<expected behaviour in plain English>')`, e.g. `it('does not send an unannounced nudge')`. Test files and test names cited in stories are the real names to create.
- **Test database.** One Postgres 18 container per test run (Vitest `globalSetup`); each test file gets a fresh schema (migrations applied once, then `TRUNCATE … RESTART IDENTITY CASCADE` between tests).
- **Factories.** `test/factories/*.ts` builders with overridable defaults (e.g. `makeInvoice({ dueDate: '2026-10-01' })`); PII fixtures use recognisable strings (`Fixture Client 1`, `fixture.client1@example.test`) so FF-08 and FF-14 can search for them.
- **Clock.** `fakeClock('2026-10-05T08:00:00Z')` injected everywhere; tests never read the wall clock (FF-12).
- **Randomness.** `platform/crypto.randomCode()` injectable; tests seed it.
- **Telegram.** `test/harness/fakeTelegram.ts` installs a grammY transformer (S-90) that records `{ method, payload }` and returns canned results; `sendUpdate(app, update)` posts JSON through `app.inject()` with the webhook secret; `updates.ts` builds message, command and callback updates.
- **Email.** `FakeEmailGateway` records `{ to, from, replyTo, subject, text, html, headers, idempotencyKey }`; it can be told to fail with 5xx, time out or return 409. A separate `npm run test:contract` (not in `check`, needs network and a key) sends to Resend test addresses (S-28) and is run manually at E05 and on staging; the `contract` Vitest project is only defined when `RESEND_CONTRACT_API_KEY` is set, so `vitest run` inside `check` never includes it.
- **E2E.** "End-to-end" here means the full server (Fastify + bot + modules + loops) against real Postgres, with fake Telegram, fake email and fake clock, driven by Telegram update JSON and HTTP requests to client pages. The 5 critical paths are named in §14.3.
- **Red before green.** Every story's new tests are run and seen failing before implementation; the session records it in PROGRESS.md.

### 13.7 Git conventions

- **Branches.** `main` (protected; required check `ci / check`); one branch per epic `epic/E03`; rebase-merge via PR, so each story's commit stays on `main`.
- **Commits.** `type(scope): message [E03-S02]`. Types: `feat`, `fix`, `test`, `refactor`, `chore`, `docs`, `build`, `ci`. Scope = module or area (`nudging`, `bot`, `web`, `platform`, `ci`).
- **PRs.** One per epic; the body is the handoff summary (§18.5); merge only with `check` green and the review checklist applied.
- **Production.** The `production` branch is fast-forwarded from `main` by the operator after staging smoke checks.

### 13.8 CI/CD and deployment

- **CI** (`.github/workflows/ci.yml`):
  - Job `check`: `ubuntu-latest`; checkout; setup Node from `.node-version`; `npm ci`; install gitleaks 8.30.1 from its GitHub release; `npm run check`.
  - Job `mutation`: runs on PRs touching `src/modules/nudging/domain/**`, `src/modules/invoices/domain/**`, `src/platform/money/**` or `src/platform/time/**`, and nightly on `main`; runs `npm run mutate`.
  - Actions are pinned to full commit SHAs, recorded in DECISIONS.md at E00-S07.
- **Branch protection.** `main` requires `ci / check` (and `ci / mutation` when it runs); no direct pushes; no force pushes (DR-18).
- **Render** (`render.yaml`, two web services + two Postgres instances):
  - Build: `npm ci && npm run build`.
  - Pre-deploy: `npm run db:migrate` (S-41).
  - Start: `npm start`.
  - Health check path: `/healthz`.
  - Staging auto-deploys from `main` "After CI Checks Pass"; production auto-deploys from `production` "After CI Checks Pass" (S-41).
  - Node version from `.node-version` (S-62).
  - After the first deploy of each environment, run `npm run ops:set-webhook`, which calls `setWebhook` with `secret_token` and `allowed_updates: ["message", "callback_query", "my_chat_member"]`, and `npm run ops:set-commands`, which calls `setMyCommands` with CP-CMD-1 to CP-CMD-6.
- **Zero downtime.** Render starts the new instance, routes traffic after the health check passes, and sends SIGTERM to the old one (S-41). On SIGTERM: stop accepting loop work, finish the current tick, close Fastify, exit within 25 s.
- **Rollback.** Use Render's rollback to the previous deploy [VERIFY-AT-BUILD — S-41 mentions rollbacks; the exact UI steps were not read], or revert the commit on `production` and push. Migrations are backward compatible for one release, so the previous code runs on the new schema (EC-OPS-03).

### 13.9 Error handling, logging and configuration conventions

- **Errors.** `AppError` subclasses in `platform/errors` carry a `code` (snake_case) and an optional `copyId`: `ValidationError`, `NotFoundError`, `ConflictError`, `LimitError`, `ProviderError` (retryable flag), `ConfigError`. Domain functions return `Result<T, DomainError>` for expected failures and throw only for bugs. Adapters map errors to copy IDs; unknown errors → CP-SCR20-4 + Sentry. `catch` blocks must re-throw, map or log with context, never swallow (review item + ESLint `no-empty`).
- **Logging.** Only through `platform/log` (pino child loggers with `correlation_id`); `console` is banned (lint).
- **Configuration.** Only through `platform/config.get()`, which returns a frozen, typed object built from the zod schema at startup; code never reads `process.env` elsewhere (lint rule `no-process-env` via `no-restricted-properties`).

### 13.10 Dependency policy and version drift

- **Adding a runtime or dev dependency requires a DR entry** in DECISIONS.md, stating what it replaces, its maintenance status (last release date, not archived), its license, and why fewer than ~50 lines of our own code would not do. Typosquat check: the exact package name is copied from the official docs page.
- **Pins are exact** (`"grammy": "1.46.0"`, no `^`/`~`); `package-lock.json` is committed; installs use `npm ci`.
- **Version drift rule.** The versions in §13.1 are the ones this document was verified against on 2026-09-30. A session that finds a newer version does not upgrade silently. It records the finding in PROGRESS.md, and upgrades only in a dedicated `chore(deps)` story with its own DR, changelog review and a green `check`. Security patches (`npm audit` high or critical) are the exception: patch within the session, and record it.

## 14. Quality strategy, review and metrics (Quality)

This section makes "done" mean the same thing in session 1 and session 9. Every story is written by an agent and reviewed once by one human, so the standard is written down before any code exists.

### 14.1 The two layers

| | Layer 1 — the gate | Layer 2 — the review |
|---|---|---|
| Who | `npm run check` locally and in CI | The agent (self-review) and then the human, with the same checklist (§14.4) |
| What | Format, lint (types-aware), type check, architecture rules, dead code, duplication, tests with a coverage floor, schema drift, dependency audit, secret scan, plus mutation on critical modules in CI | Design fit, boundaries, reversibility, failure behaviour, test meaning, security reasoning, copy and accessibility, naming |
| Rule | If a reviewer argues about something a machine could decide, it moves into the gate in the next epic (a story in the Parking lot, then an Amendment) | If a machine cannot decide it, no lint rule "fixes" it; it stays on the checklist |

Two rules about the gate: **it is never weakened to make a change pass** (weakening it needs a DR entry with a reason and a follow-up story to restore it), and **a flaky test is a broken gate**: quarantine it in the same session with a follow-up story; never re-run until green.

### 14.2 The gate

| # | Component | Command (inside `npm run check`) | Threshold / rule | Tool (verified) |
|---|---|---|---|---|
| 1 | Format | `prettier --check .` | zero diffs | Prettier 3.9.9 (S-31) |
| 2 | Lint | `eslint . --max-warnings=0` | zero errors, zero warnings; includes FF-09, FF-12 and the banned imports in §13.2 | ESLint 10.11.0 + typescript-eslint 8.71.0 `strictTypeChecked` (S-31, S-77) |
| 3 | Types | `tsc -p tsconfig.json --noEmit` | zero errors, strict flags of §13.5 | TypeScript 6.0.3 (S-31) |
| 4 | Architecture | `depcruise src --config .dependency-cruiser.cjs` | zero violations of FF-01 to FF-06, FF-13 | dependency-cruiser 18.4.0 (S-31, S-76) |
| 5 | Dead code | `knip` | zero unused files, exports or dependencies | knip 6.38.0 (S-31) |
| 6 | Duplication | `jscpd src --threshold 3` | duplicated share ≤ 3% of `src/` | jscpd 5.3.3 (S-31, S-80) |
| 7 | Tests + coverage floor | `vitest run --coverage` | all unit, integration, e2e and arch tests pass (FF-07, FF-08, FF-10, FF-14, FF-15 included); coverage floor lines ≥ 80%, branches ≥ 75% (a floor against regression, never a goal) | Vitest 5.0.2 + coverage-v8 (S-31, S-79) |
| 8 | Schema drift | `tsx scripts/db-verify.ts` | migrations apply cleanly to an empty Postgres 18; `kysely-codegen --verify` passes (FF-11) | kysely-codegen 0.20.0 (S-36) |
| 9 | Dependency audit | `npm audit --audit-level=high` | no high or critical advisories | npm 11 (S-86) |
| 10 | Secret scan | `gitleaks dir . --redact --exit-code 1` | zero findings | gitleaks 8.30.1 (S-44, S-81) |
| 11 | Mutation (CI job, not local `check`) | `stryker run` | mutation score ≥ 70 (`thresholds.break: 70`, low 70, high 85) on `src/modules/nudging/domain/**`, `src/modules/invoices/domain/**`, `src/platform/money/**`, `src/platform/time/**` | StrykerJS 10.0.0 vitest-runner (S-31, S-78) |
| 12 | Budgets | inside the integration tests | DB queries per update ≤ 12; /owed ≤ 300 ms at 500 invoices; scheduler tick ≤ 5 s at 50 claims; pages < 15 KB (§12.9) | Vitest (query counter in `test/harness`) |
| 13 | Copy and accessibility | inside the tests | FF-10 (catalogue equals §11); FF-15 (page structure); every copy line renders within its max length with the longest fixtures | Vitest |

Expected runtime of `npm run check`: ≤ 3.5 minutes (measured in E00-S02 and tracked as M-23). Unit tests alone: ≤ 20 s (`vitest run --project unit`).

### 14.3 Test strategy

**Shape.** Many fast unit tests on pure domain functions (plans, schedule, eligibility, transitions, money parsing and formatting, date parsing, tokens, copy rendering). Integration tests for every repository, handler and route against real Postgres. Five end-to-end critical paths. No browser tests (client pages have no JavaScript; FF-15 checks structure).

**Critical e2e paths** (kept green in every epic from the one that introduces them):

| ID | Path | Spec file | Introduced |
|---|---|---|---|
| CE-1 | Onboarding: /start → name → verified email (code from fake email) → time zone → currency → home | `test/e2e/onboarding.e2e.test.ts` | E01 |
| CE-2 | Log an invoice for a new client → appears in /owed with the correct totals | `test/e2e/log-invoice.e2e.test.ts` | E02 (list assertion added in E03) |
| CE-3 | Overdue invoice → digest at 18:00 local → exactly one email at 10:00 local next weekday with correct headers and content → "Paid in full" cancels the remaining nudges → no further email when the clock advances 40 days | `test/e2e/nudge-cycle.e2e.test.ts` | E05 |
| CE-4 | Client opens the "I've paid" link (GET changes nothing) → confirms (POST) → reminders paused → freelancer gets CP-SCR15-5 → "Yes, mark as paid" → invoice paid | `test/e2e/client-paid.e2e.test.ts` | E05 |
| CE-5 | One-click unsubscribe POST → suppression row → all that freelancer's nudges to the address cancelled → freelancer notified → no email when the clock advances | `test/e2e/unsubscribe.e2e.test.ts` | E05 |

**Test-quality rules**
- **Behaviour, not implementation.** Tests assert observable outcomes: rows stored, emails recorded by the fake gateway (to, subject, headers), Telegram calls recorded by the fake API (method, text from a CP ID), HTTP status and body. They never assert that an internal function was called.
- **Concrete assertions.** Expected values are spelled out (`'EUR 1,200.00'`, `'2026-10-13T08:00:00Z'`), never "is a string" or "has length".
- **Red before green.** Each AC's test is run against the unimplemented behaviour and seen failing; PROGRESS.md records "seen red" per AC.
- **Determinism.** No wall clock (fake clock), no network (fakes), seeded randomness, no order dependence (fresh data per test), no sleeps (the scheduler tick is called directly in tests).
- **Fixtures and factories** for anything reused; PII fixtures use searchable strings.
- **Mutation testing** on the four critical areas with StrykerJS (gate row 11). Surviving mutants in those areas are either killed by a new test or recorded in PROGRESS.md as accepted, with a reason.
- **Coverage is a floor** that stops regressions; raising the number is never a story goal.

### 14.4 Review checklist

Committed verbatim as `docs/REVIEW-CHECKLIST.md` in E00-S01 and referenced from AGENTS.md and CLAUDE.md. Both the agent's self-review and the human review use exactly this list.

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

### 14.5 Reviewing agent-written code

Most code in this repo is machine-authored, which changes the defect profile rather than removing it. The latest evidence found this session is Veracode's Spring 2026 update. Across 150+ models, syntax pass rates are 95%+ while the security pass rate is 55%, on 80 tasks covering SQL injection, XSS, log injection and insecure cryptography [VERIFIED — S-52, 2026-09-30]. No other current figures (on duplication, churn or review bottlenecks) were verified this session, so none are stated. Practical consequences for this repo:

1. **Plausible but wrong APIs.** grammY, Kysely, Fastify, Resend and Temporal all have near-miss APIs across versions. Every external symbol must exist in the pinned version (§13.1). When unsure, the session reads the docs page or the package's type definitions in `node_modules` before writing the call.
2. **Security first.** Object-level `freelancer_id` scoping, token checks and HTML escaping are reviewed on every diff (D6), because tooling cannot see authorisation logic.
3. **Duplication is the default failure.** The agent cannot see the helper it did not read. Before writing a formatter, parser or query, search `src/platform` and the module's `domain/`. jscpd catches the obvious cases.
4. **Churn is the honesty metric** (M-18). Code rewritten within two weeks of merge means the story or the blueprint was wrong, and both get fixed.
5. **A human is the author of record.** Whoever merges must be able to explain every non-obvious decision in the diff; if they cannot, it is not ready.
6. **Annotate before review.** The handoff summary lists what changed, which AC each part satisfies, what was skipped and every deviation.
7. **Review load is the constraint.** Stories are sized for ≤ 400 changed lines. A larger diff is reported in the handoff as a sizing defect.

### 14.6 Metrics

System- and team-level only; each metric raises a question, never a verdict.

| ID | Metric | Definition | Data source | Instrumented by | Starting target | What a move in it would prompt |
|---|---|---|---|---|---|---|
| M-10 | Deployment frequency | Production deploys per week | Render deploy history for the production service | E00-S07 (process), read monthly | ≥ 1 per week during build [ASSUMED] | Falling: sessions are stuck or releases batch up; check PROGRESS blockers |
| M-11 | Change lead time | Median time from commit on `main` to production deploy (S-50) | Git timestamps + Render deploy log | E00-S07 | ≤ 3 days [ASSUMED] | Rising: staging smoke is slow or releases are held; simplify promotion |
| M-12 | Change fail rate | Share of production deploys needing immediate intervention (rollback, hotfix) (S-50) | `docs/OPS-LOG.md` entries | E08-S04 | ≤ 15% [ASSUMED] | Rising: gaps in tests for the failing area; add tests to the gate |
| M-13 | Failed deployment recovery time | Median time from a failed deploy to service restored (S-50) | OPS-LOG | E08-S04 | ≤ 1 h [ASSUMED] | Rising: rollback path unclear; rehearse |
| M-14 | Deployment rework rate | Share of deploys that are unplanned, caused by a production incident (S-50) | OPS-LOG | E08-S04 | ≤ 10% [ASSUMED] | Rising: escaped defects; see M-19 |
| M-15 | Time to first review | Median time from PR opened to first human review comment or approval | GitHub PR timestamps | E00-S07 | Same working day [ASSUMED] | Rising: the reviewer is the bottleneck; shrink stories |
| M-16 | Review cycles to merge | Median number of review rounds per PR | GitHub PR events | E00-S07 | 1–2 | Rising: requirements unclear; amend the blueprint |
| M-17 | Median story diff size | Median changed lines per story commit | `git log --numstat` per `[Enn-Snn]` commit | E00-S07 | ≤ 400 | Rising: stories mis-sized; split in the next epic |
| M-18 | Rework (churn) rate | Share of lines changed in a 14-day window that modify lines added in the previous 14 days | `git log` analysis script `scripts/churn.ts` (E08-S04) | E08-S04 | ≤ 20% [ASSUMED] | Rising: output isn't sticking; review ACs and blueprint accuracy |
| M-19 | Escaped defects ratio | Defects found in staging or production ÷ defects found by gate + review, per epic | PROGRESS.md defect lines | E00-S07 (format) | ≤ 0.25 [ASSUMED] | Rising: add the missing test type or checklist item |
| M-20 | Duplication trend | jscpd duplicated share over time | jscpd JSON report in CI artifacts | E00-S02 | ≤ 3% and not rising | Rising: search-before-write is failing; review D2 |
| M-21 | Flaky tests | Count of quarantined tests | PROGRESS.md quarantine list | E00-S02 | 0 | Any: fix before new features |
| M-22 | Open vulnerable dependencies | High or critical advisories open | `npm audit --json` in CI | E00-S02 | 0 | Any: patch in the same session |
| M-23 | `check` runtime | Wall time of `npm run check` in CI | CI job duration | E00-S02 | ≤ 3.5 min | Over 5 min: people stop running it locally; split projects or parallelise |
| M-24 | Mutation score (critical modules) | Stryker score on the four critical areas | Stryker report in CI | E02-S05 | ≥ 70 (break), trend up | Falling: tests assert too little; strengthen assertions |

Pair every number with one qualitative note in the handoff: what slowed this session down, and what had to be guessed.

### 14.7 Anti-metrics (what this product will not measure)

- Code volume produced (counts of lines, files or characters added), per person or in total.
- Commit counts or pull-request counts per person.
- Story points or "stories per session" as productivity.
- The coverage percentage as a goal (it is only a regression floor in the gate).
- Anything that ranks or grades an individual, including the solo builder's "speed". The metrics above diagnose the process.
- Self-reported time savings from AI assistance, unless measured against a baseline; otherwise they are `[ASSUMED]` at best.

If someone wants a single number, show change fail rate (M-12) next to deployment frequency (M-10), and refuse to reduce it further.

### 14.8 Definition of green, definition of done, manual verification

**Green** = `npm run check` exits 0 on the merged commit, and CI's `check` job (and `mutation`, when triggered) is green.

**Default Story DoD** (every story's "Story DoD" line starts from this list; additions are listed in the story):
1. Every AC has a passing automated test, or a manual check recorded in PROGRESS.md with date and observation.
2. Each of those tests was seen failing before the implementation; PROGRESS.md records it.
3. `npm run check` is green on the whole repo; no gate rule was weakened.
4. The review checklist (§14.4) was applied end to end, and the result recorded as `reviewed: n items, n fixed, n accepted with reason`.
5. Copy used matches §11 exactly (by CP ID; FF-10 green).
6. Each listed edge case is covered by a test or marked "accepted risk" in PROGRESS.md with a reason and owner.
7. Every external symbol traces to a pinned dependency; no new dependency without a DR.
8. No logic duplicated (searched before writing).
9. The commit message contains the story ID.
10. PROGRESS.md updated: story DONE, one evidence line per AC, deviations.

**Epic DoD** is in each epic (§17). **Manual verification policy.** What cannot be automated is done by the product owner at the end of the epic, on staging, and recorded in PROGRESS.md as `manual YYYY-MM-DD — <observation>`: real Telegram rendering on iOS and Android (lists, buttons, bold), receiving a real reminder email from staging at a Resend test address viewed in the Resend dashboard, and client pages viewed at 200% zoom and with a screen reader (VoiceOver or TalkBack) once per epic that changes them.

## 15. Edge-case register (Edge Cases)

Global, cross-cutting edge cases. Stories reference these IDs and add story-specific behaviour. IDs follow the skill's catalogue numbering; IDs marked "(new)" are product-specific additions. Category IR (Iran-specific) does not apply: users are not Iranian (assumption 1).

### INPUT — input and validation

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-INPUT-01 | Empty, whitespace-only or invisible-only input | Normalise (trim, strip zero-width and control characters), then treat as empty; the step's "empty" error; no state change | E01-S02, E01-S03, E02-S02 | CP-SCR02-3 |
| EC-INPUT-02 | Text over the limit (including pasted text) | Reject with the length copy; limits: display name 60, business 80, client name 80, contact 40, reference 40, payment details 500, link 500 | E01-S03, E01-S06, E02-S02, E02-S06 | CP-SCR02-4, CP-SCR02-7, CP-SCR05-7, CP-SCR07-9, CP-SCR10-3 |
| EC-INPUT-03 | Invisible or control characters pasted (U+200B–U+200F, U+2060, U+FEFF, tabs, newlines in one-line fields) | Stripped by `platform/text.normaliseLine` before validation | E01-S02, E02-S02 | — |
| EC-INPUT-04 | Emoji, combining marks, non-Latin scripts in names | Stored and rendered unchanged; length counted in graphemes (`Intl.Segmenter`) | E01-S03, E02-S02 | — |
| EC-INPUT-05 | HTML-, script- or SQL-looking input (`<b>`, `&`, `'; drop table`) | Stored verbatim; escaped for Telegram HTML, email HTML and pages; parameterised SQL | E01-S02, E05-S01 | — |
| EC-INPUT-06 | Email with capitals or surrounding spaces | Trimmed; blind index uses lowercase; display keeps the trimmed original | E01-S04, E02-S02 | — |
| EC-INPUT-09 | Amount boundaries: 0, 0.01, 99,999,999.99, 100,000,000.00, 1.005 EUR, 150000.5 JPY | 0 → CP-SCR08-3; 0.01 accepted; max accepted; max + 0.01 → CP-SCR08-5; 3 decimals for EUR → CP-SCR08-4; decimals for JPY → CP-SCR08-4 | E02-S01 | CP-SCR08-3, CP-SCR08-4, CP-SCR08-5 |
| EC-INPUT-10 | Duplicate content: a client with the same email; an invoice with the same reference for the same client | Ask to confirm (CP-SCR07-10 / CP-SCR10-4); both paths tested | E02-S02, E02-S06 | CP-SCR07-10, CP-SCR10-4 |
| EC-INPUT-12 | Separators: "1,200.50", "1.200,50", "1200,50", "1 200", "1'200", "€1,200", "1200 usd" | Last separator followed by 1–2 digits is decimal; "," or "." followed by exactly 3 digits and no other separator is thousands; spaces and apostrophes are thousands; symbols ignored; trailing or leading ISO code overrides the default currency; the summary shows the parsed value for confirmation | E02-S01 | CP-SCR08-2 |
| EC-INPUT-13 (new) | Date formats: "30 Oct", "Oct 30", "30 October 2026", "2026-10-30", "today", "tomorrow", "in 10 days", "10/30/2026", "30.10.2026" | Named-month and ISO formats accepted; missing year → the next occurrence on or after today minus 365 days (so "5 Jan" typed in December means next January); numeric day/month forms → CP-SCR09-8 | E02-S04 | CP-SCR09-7, CP-SCR09-8 |
| EC-INPUT-14 (new) | Links: `http://`, `javascript:`, `mailto:`, spaces, > 500 chars | Only `https://` URLs that parse with `new URL()` and are ≤ 500 chars | E02-S06 | CP-SCR10-19 |

### AUTH — identity and verification

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-AUTH-01 | Wrong code 5 times | Locked for 15 min; attempts shown | E01-S04 | CP-SCR03-7, CP-SCR03-9 |
| EC-AUTH-02 | "Send a new code" double-tapped | One email (cooldown check in the same transaction as the insert) | E01-S04 | CP-SCR03-10 |
| EC-AUTH-03 | Resend within 60 s | Blocked with remaining seconds | E01-S04 | CP-SCR03-10 |
| EC-AUTH-04 | Expired code entered; old code after a new one was issued | "Expired" copy; older codes invalid once a new one exists | E01-S04 | CP-SCR03-8 |
| EC-AUTH-11 | More than 5 codes in an hour | Blocked until the hour window passes | E01-S04 | CP-SCR03-13 |
| EC-AUTH-12 | Reply-to changed | Old address stays active until the new one is verified | E07-S02 | CP-SCR17-14 |
| EC-AUTH-13 (new) | Webhook call without or with a wrong secret header | 401; nothing processed; logged at `warn` without the header value | E00-S06 | — |
| EC-AUTH-14 (new) | `/admin` from a non-operator | Same reply as any unknown command; nothing reveals the command exists | E06-S02 | CP-SCR20-1 |

### DATA — data and state

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-DATA-01 | First use: no invoices, no clients | Empty states with one next action | E02-S02, E03-S02 | CP-SCR11-11, CP-SCR16-3 |
| EC-DATA-02 | Exactly one item | Singular copy variants | E01-S03, E03-S02 | CP-SCR01-4, CP-SCR11-1 |
| EC-DATA-03 | Many items: 500 open invoices, 200 clients | Paged (8 / 10 per page); budgets hold (§12.9) | E02-S02, E03-S02 | CP-SCR11-12, CP-SCR16-17 |
| EC-DATA-04 | Paging boundaries: exactly 8 items; an invoice paid while paging; page past the end | No Next button at exactly 8; re-query on every page; past-the-end → last page | E03-S02 | CP-SCR11-14 |
| EC-DATA-05 | Sort ties | Secondary keys: `reference`, then `id` (owed); `id` (clients) | E03-S02 | — |
| EC-DATA-06 | Archived client with open invoices | Invoices stay in /owed and keep reminders; the client is hidden from the /new picker and the /clients list | E02-S02 | CP-SCR16-12 |
| EC-DATA-07 | Deleting an invoice or an account | Cascades: invoice → payments, nudges; account → everything owned; audit anonymised | E02-S07, E07-S04 | CP-SCR12-21, CP-SCR18-5 |
| EC-DATA-10 | Export of large accounts; commas, quotes and newlines in fields | CSV with RFC 4180 quoting; UTF-8 with BOM; ≤ 50 MB (Telegram upload limit, S-08) | E07-S03 | CP-SCR18-2 |
| EC-DATA-11 | Duplicate client by email | Detected by blind index; confirm dialog | E02-S02 | CP-SCR07-10 |
| EC-DATA-12 | Card shown, then the invoice changed on another device | Optimistic lock → conflict copy + fresh card | E02-S07 | CP-SCR12-31 |
| EC-DATA-13 (new) | CSV formula injection (`=`, `+`, `-`, `@` at field start) | Prefix with `'` in exports | E07-S03 | — |

### TIME — dates, time zones, schedules

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-TIME-02 | Local day boundaries | "Overdue" and "due today" use the freelancer's local date; 23:59 vs 00:00 tested in Pacific/Auckland and America/Los_Angeles | E03-S01 | CP-SCR11-6 to CP-SCR11-10 |
| EC-TIME-04 | DST transitions | Reminders keep 10:00 local across transitions; tests on 2026-03-29 and 2026-10-25 (Europe/Berlin), 2026-03-08 and 2026-11-01 (America/New_York), 2026-04-05 and 2026-10-04 (Australia/Sydney); Asia/Kolkata as a no-DST control | E02-S05 | — |
| EC-TIME-06 | Device clocks | Irrelevant: all times come from the server's `Clock` | E01-S04, E02-S05 | — |
| EC-TIME-09 | Exact boundaries | Code: `expires_at <= now` → expired; token: `exp <= now` → expired; `due_date == today` → "due today", not overdue | E01-S04, E03-S01, E05-S01 | CP-SCR03-8, CP-SCR11-8 |
| EC-TIME-10 (new) | Time-zone or send-hour change | Future `planned`/`announced` nudges re-planned in the new zone; sent history unchanged | E01-S05, E07-S01 | CP-SCR17-17 |
| EC-TIME-11 (new) | Due date in the past at entry; > 1 year past; > 2 years ahead | Past allowed (first reminder no sooner than the next digest cycle); limits enforced | E02-S04 | CP-SCR09-9 to CP-SCR09-11 |
| EC-TIME-12 (new) | Weekend and spacing rules | Mon–Fri only; weekend dates move to Monday; two reminders ≥ 2 days apart; Friday's digest lists Monday's reminders | E02-S05, E04-S03 | CP-SCR14-1 |
| EC-TIME-13 (new) | Outage: nudges more than 24 h past `scheduled_for` | Re-planned to the next send window after the next digest; one CP-SCR15-15 notice per freelancer | E05-S02 | CP-SCR15-15 |
| EC-TIME-14 (new) | "End of month" on the last day of a month; 29 Feb 2028 | Last day of the next month; leap years via Temporal | E02-S04 | CP-SCR09-5 |

### NET and INT — network and integrations

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-NET-03 | Timeout after the server completed the work | Resend: same idempotency key on retry (S-27); Telegram webhook re-delivery: `update_id` dedupe | E00-S06, E05-S02 | — |
| EC-NET-06 | Our 5xx on client pages | SCR-23 error page; error tracked | E05-S03 | CP-SCR23-4 |
| EC-NET-09 | Retry storms | Exponential backoff with ±20% jitter; attempt caps (§12.7) | E04-S02, E05-S02 | — |
| EC-INT-01 | Telegram, Resend or Sentry down | Fallbacks per §12.7 | E00-S04, E04-S02, E05-S02 | CP-SCR15-2 |
| EC-INT-03 | Provider rate limits (Telegram 429, Resend 429) | Wait `retry_after` / `retry-after`, then retry | E04-S02, E05-S02 | — |
| EC-INT-04 | Sandbox vs production | Staging requires the recipient override; production refuses it | E00-S04, E05-S02 | — |
| EC-INT-05 | Secret or key rotation | Documented procedure; re-encrypt script idempotent | E08-S01 | — |
| EC-INT-07 | Email webhook with an invalid signature or replayed | 400 / dedupe on `svix-id` | E05-S06 | — |
| EC-INT-09 (new) | Resend 409 `invalid_idempotent_request` | Treated as a bug: nudge `failed`, error tracked, never retried with a new key | E05-S02 | CP-SCR15-2 |

### CONC — concurrency and idempotency

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-CONC-01 | Two devices edit one invoice | `version` check → conflict copy | E02-S07 | CP-SCR12-31 |
| EC-CONC-02 | Double tap Save invoice, Paid in full, Send now | One invoice (wizard state consumed in the save transaction), one payment (callback idempotency on message ID + action), one send | E02-S06, E03-S03, E05-S05 | — |
| EC-CONC-04 | Email event arrives for a message we have not yet recorded | Stored with `nudge_id = null`; linked when the nudge records `provider_message_id` | E05-S06 | — |
| EC-CONC-05 | Scheduler overlap (two processes during deploy) | `SKIP LOCKED` disjoint claims; digest dedupe key per freelancer and local date | E04-S03, E05-S02 | — |
| EC-CONC-06 | Out-of-order email events (delivered after bounced) | Suppression is monotonic: bounce and complaint are never undone by later events | E05-S06 | — |
| EC-CONC-07 | "Paid in full" while a nudge is `sending` | The in-flight email may go out; the ack shows; audit event `nudge.raced_payment`; M-3.2 counts it | E03-S03, E05-S02 | CP-SCR13-1 |
| EC-CONC-08 (new) | Hold tapped after the reminder was sent | Toast "Too late to hold" | E04-S03 | CP-SCR14-12 |

### PAY — money representation (no payments are processed)

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-PAY-06 | Currency minor units and rounding | Integer minor units; exponent from ISO 4217 (S-68); no rounding anywhere (inputs with too many decimals are rejected) | E02-S01 | CP-SCR08-4 |
| EC-PAY-13 (new) | Part payments exceeding the balance; editing the amount below what was paid | Rejected with copy | E03-S04, E02-S07 | CP-SCR13-7 |
| EC-PAY-14 (new) | Several currencies in totals | Per-currency totals, never summed across currencies | E03-S02 | CP-SCR11-1 |

### MSG — email and notifications

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-MSG-01 | Accepted but never delivered (`delivery_delayed`, no `delivered`) | No automatic resend; the nudge stays `sent`; only bounces and complaints notify | E05-S06 | — |
| EC-MSG-02 | Wrong or dead address | Hard bounce → global suppression for 90 days; freelancer told to fix the email | E05-S06 | CP-SCR15-3 |
| EC-MSG-04 | Duplicate send risk (job retried) | Unique nudge row + idempotency key + state machine | E05-S02 | — |
| EC-MSG-05 | Quiet hours | Sends only Mon–Fri at the send hour (08:00–17:00 local) | E02-S05 | — |
| EC-MSG-06 | Opt-out and unsubscribe | Honoured within 60 s; permanent for the pair | E05-S04 | CP-SCR15-9, CP-SCR15-10 |
| EC-MSG-09 | Template variables missing | Tests render every template with fixtures; at runtime a missing variable throws before sending → nudge `failed`, error tracked | E01-S02, E05-S01 | — |
| EC-MSG-10 | Provider rate limit or daily cap | Queue with backoff; cap rolls over to the next day with one notice | E05-S02, E05-S05 | CP-SCR15-11 |
| EC-MSG-11 (new) | Mail security scanners prefetch links | GET never changes state; only POST does (S-47) | E05-S03, E05-S04 | — |
| EC-MSG-12 (new) | Long references or amounts in subjects | Subjects ≤ 60 chars at typical values; ≤ 100 with maximum fixtures; never truncated mid-reference | E05-S01 | CP-EML-2 to CP-EML-6 |
| EC-MSG-13 (new) | Client replies to the From address instead of Reply-To | The sending subdomain has no inbox; such mail bounces back to the client. Accepted: Reply-To handles normal replies; the footer tells clients replies go to the freelancer | E05-S01 | CP-EML-22 |

### PERM — permissions and privacy

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-PERM-02 | Another freelancer's IDs in callbacks | Same as not found; no change | E02-S07, E03-S03 | CP-SCR12-32 |
| EC-PERM-03 | Operator actions | Audited with actor `operator` and reason | E06-S02 | — |
| EC-PERM-04 | PII in logs or error events | Forbidden; FF-08 | E00-S04 | — |
| EC-PERM-05 | Export and deletion | Complete; deletion within 60 s; backups age out ≤ 7 days | E07-S03, E07-S04 | CP-SCR18-9 |
| EC-PERM-07 | Retention | Jobs enforce §12.4 | E07-S04 | — |

### BIZ — business rules

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-BIZ-01 | Limits: 200 clients, 500 open invoices, 10 new addresses/day, 30 reminders/day (10 in week one) | Told before the action where predictable; reminders roll to the next day | E02-S02, E02-S03, E05-S05 | CP-SCR07-13, CP-SCR07-16, CP-SCR07-17, CP-SCR15-11 |
| EC-BIZ-02 | Sending suspended while nudges are planned | Due nudges `cancelled` (`suspended`); on unsuspend, open invoices are re-planned from that day | E06-S01 | CP-SCR15-12, CP-SCR15-14 |
| EC-BIZ-09 (new) | Invoice cancelled or deleted with nudges pending | Nudges `cancelled` in the same transaction | E02-S07 | CP-SCR12-37 |
| EC-BIZ-10 (new) | Plan finished and still unpaid | Stays in /owed; card shows "done — all n sent"; Send now hidden; text to forward available | E04-S01 | CP-SCR12-6 |

### BOT — messenger bot behaviour

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-BOT-01 | Unknown command or free text | Helpful fallback | E01-S02 | CP-SCR20-1 |
| EC-BOT-02 | Stickers, voice, photos, documents; forwarded messages | Media → CP-SCR20-3; forwarded text is read as text | E01-S02 | CP-SCR20-3 |
| EC-BOT-03 | Added to a group | Reply once, leave the chat (`my_chat_member` update) | E01-S02 | CP-SCR20-6 |
| EC-BOT-04 | Flooding and Telegram 429 | 30 updates/min per user; auto-retry on 429 | E01-S02, E04-S02 | CP-SCR20-7 |
| EC-BOT-05 | Webhook downtime | Telegram keeps updates ≤ 24 h (S-10); re-delivery deduped | E00-S06 | — |
| EC-BOT-06 | Wizard left for 24 h | Cleared with CP-SCR20-5 on the next interaction | E01-S02 | CP-SCR20-5 |
| EC-BOT-07 | Stale button | Toast + fresh render | E01-S02, E03-S03, E04-S03 | CP-SCR20-2 |
| EC-BOT-08 | Bot blocked by the freelancer | `bot_blocked_at` set; notifications dropped; reminder emails continue; cleared on the next update | E04-S02 | — |
| EC-BOT-09 | Edited messages | During a wizard → CP-SCR20-9; otherwise ignored | E01-S02 | CP-SCR20-9 |
| EC-BOT-10 (new) | /start with a payload, or by an onboarded user | Payload ignored; returning variant | E01-S03 | CP-SCR01-3 to CP-SCR01-5 |
| EC-BOT-11 (new) | Lists that would exceed message limits | Paging keeps messages < 1,500 chars | E03-S02 | — |

### ADMIN — operator

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-ADMIN-02 | Suspend (destructive) | Requires a reason; audited; reversible by unsuspend | E06-S02 | CP-SCR19-3, CP-SCR19-4 |
| EC-ADMIN-03 | Operator viewing user data | Counts only, no client PII | E06-S02 | CP-SCR19-2 |
| EC-ADMIN-06 (new) | Auto-suspension of a good user | Operator alerted; review within 2 working days; unsuspend notifies the user | E06-S01 | CP-SCR19-7, CP-SCR15-14 |

### OPS — operations

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-OPS-01 | Missing or invalid environment variable | Refuse to start; message names the variable | E00-S04 | — |
| EC-OPS-02 | Migration fails during deploy | Deploy aborted; previous release keeps running (S-41) | E00-S05, E00-S07 | — |
| EC-OPS-03 | Rollback onto a newer schema | Migrations backward compatible for one release | E00-S07 | — |
| EC-OPS-04 | Backups untested | Restore drill (E08-S01), then monthly | E08-S01 | — |
| EC-OPS-06 | Alert thresholds | §12.9 | E08-S04 | CP-SCR19-7 |
| EC-OPS-07 | Seed script against staging or production | Refuses to run | E00-S05 | — |
| EC-OPS-09 | Health endpoint | Reports DB state; used by Render | E00-S04 | — |
| EC-OPS-10 (new) | Email override misconfiguration | Staging without override or production with override → refuse to start | E00-S04, E05-S02 | — |
| EC-OPS-11 (new) | SIGTERM during a tick | Finish or release the current row; `sending` rows resume via `next_attempt_at` | E00-S07, E05-S02 | — |
| EC-OPS-12 (new) | Encryption key missing, malformed or rotated | Startup validation; unknown `key_id` on read → error with the key ID (no data); re-encrypt script idempotent | E01-S01, E08-S01 | — |

### SEC — security

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-SEC-01 | Injection | Kysely parameters only; lint ban on `sql.raw` | E00-S05 | — |
| EC-SEC-02 | XSS / HTML injection in Telegram HTML, email HTML, pages | Escaping in one renderer; CSP on pages | E01-S02, E05-S01, E05-S03 | — |
| EC-SEC-05 | Secrets in repo or logs | gitleaks in `check`; pino redaction | E00-S02, E00-S04 | — |
| EC-SEC-06 | Vulnerable dependencies | `npm audit --audit-level=high` in `check` | E00-S02 | — |
| EC-SEC-07 | Enumeration (token guessing, "does this invoice exist") | 128-bit HMAC; identical 404 page for every failure | E05-S03 | CP-SCR23-1 |
| EC-SEC-08 | Security headers | Set on every HTML response (§12.9) | E05-S03 | — |
| EC-SEC-11 (new) | Tampered, expired or wrong-action tokens | Same 404 page; constant-time comparison | E05-S01, E05-S03 | CP-SCR23-1, CP-SCR23-2 |
| EC-SEC-12 (new) | Reply-to hijack (entering someone else's email) | Sending needs a verified email | E01-S04 | CP-SCR03-4 |
| EC-SEC-13 (new) | Using the bot to email arbitrary people | Verified sender, templates-only text, caps, one-click opt-out, complaint auto-suspension | E05-S05, E06-S01 | CP-SCR15-11, CP-SCR15-12 |

### A11Y, L10N, SEARCH, FILE, DEV

| ID | Case | Expected behaviour | Covered by | Copy |
|---|---|---|---|---|
| EC-A11Y-01 | Screen readers on client pages | `lang`, one `h1`, `<title>`, button text, `<dl>` facts | E05-S03, E05-S04 | CP-SCR21-8, CP-SCR22-9 |
| EC-A11Y-03 | Colour-only meaning | None anywhere | E05-S03 | — |
| EC-A11Y-04 | 200% zoom | Single column, no fixed heights; manual check | E05-S03 | — |
| EC-A11Y-06 | Timeouts (15-min code) | Expiry stated; new code one tap away | E01-S04 | CP-SCR03-4 |
| EC-A11Y-08 (new) | Emoji-only or icon-only buttons | Forbidden; every button label is words | E01-S02 | — |
| EC-L10N-03 | Long strings | Every copy line rendered with the longest fixtures within its max length | E01-S02 | — |
| EC-L10N-07 | Formatting | Only `platform/money.format` and `platform/time.formatDate` | E02-S01 | — |
| EC-L10N-09 (new) | Non-Latin names (Cyrillic, CJK, Arabic) | Rendered correctly in Telegram, email and CSV (UTF-8) | E02-S02, E07-S03 | — |
| EC-SEARCH-01 | City search normalisation ("sao paulo" vs `America/Sao_Paulo`, "new york") | Case-, accent- and underscore-insensitive prefix match on the city part | E01-S05 | CP-SCR04-6 |
| EC-SEARCH-02 | No city match | CP-SCR04-7 + region buttons | E01-S05 | CP-SCR04-7 |
| EC-FILE-09 (new) | Export size | Asserted < 50 MB at the ceiling (500 invoices ≈ well under 1 MB) | E07-S03 | — |
| EC-DEV-11 (new) | Telegram clients render keyboards differently (iOS, Android, Desktop, Web) | Labels ≤ 24 chars; manual check on 2 clients per UI epic (§14.8) | E01-S02 | — |

## 16. Delivery plan — sprints and epic order (Delivery Plan)

One epic = one fresh half-day session (assumption 15). Sizes: S ≈ 2–3 h, M ≈ 3–4 h, L ≈ 4–5 h of agent work including tests and self-review.

### 16.1 Epic order

| Order | Epic | Goal | Depends on | Stories | Size | Why now, not earlier/later |
|---|---|---|---|---|---|---|
| 1 | E00 Foundation | Repo, gate, boundaries, DB, webhook skeleton, CI and staging deploy | — | 7 | L | Every later session needs the gate and the fitness functions before any feature code exists |
| 2 | E01 Onboarding and account | A freelancer can set up: name, verified reply-to, time zone, currency, payment details | E00 | 6 | L | Every feature needs a freelancer with a zone and currency; encryption lands before any PII is stored |
| 3 | E02 Clients and invoices | Log invoices for new or existing clients; compute reminder plans; edit, cancel, delete | E01 | 7 | L | The core record; reminders need invoices, and the save summary shows the first reminder date, so the pure plan computation lands here |
| 4 | E03 Owed overview and payments | See who owes what; record full and part payments | E02 | 4 | M | Completes the tracker, which is releasable without reminders (closed beta 1) |
| 5 | E04 Nudge scheduling and evening digest | Persist and re-plan nudges, notify freelancers reliably, send the digest, pause/resume/change plans | E03 | 4 | M | The nudge lifecycle and the announcement invariant must exist before any email goes out |
| 6 | E05 Client email nudges with baseline safety | Send reminders exactly once; client "I've paid", stop and unsubscribe; caps; bounces and complaints | E04 | 6 | L | Core loop plus the safety that must ship with it (guardrail: messages to strangers) |
| 7 | E06 Operator tools and manual forwarding | Auto-suspension, admin commands, text to forward, product metrics | E05 | 4 | M | Needs send and complaint data from E05 |
| 8 | E07 Settings, export and deletion | Edit every setting; data export; account deletion and retention; privacy page and menu | E06 | 5 | M | Required before public beta (Telegram developer terms, S-07) |
| 9 | E08 Launch hardening | Restore drill, load test, production go-live, alerts | E07 | 4 | M | Last: needs the whole system to exercise |

Total: 47 stories in 9 sessions.

### 16.2 Sprints and milestones

| Sprint | Epics | Milestone | Exit criteria |
|---|---|---|---|
| 1 | E00, E01 | Staging bot answers /start and completes onboarding | CE-1 green; staging deploy via CI; product owner onboards on a real phone |
| 2 | E02, E03 | **Closed beta 1 — tracker only** (5–10 invited freelancers, `NUDGES_LIVE=off`) | CE-2 green; product owner logs 10 real invoices and marks payments on iOS and Android |
| 3 | E04, E05 | **Closed beta 2 — reminders for allow-listed users** (`NUDGES_LIVE=allowlist`) | CE-3, CE-4, CE-5 green; SPF/DKIM/DMARC pass on the sending subdomain (OQ-07); 20 staging reminders to Resend test addresses with correct content |
| 4 | E06, E07 | Public beta ready | Operator can suspend; export and deletion verified; privacy page live; OQ-02 answered or explicitly accepted by the product owner |
| 5 | E08 | **Public launch** (`NUDGES_LIVE=on`) | Restore drill ≤ 4 h recorded; load test within budgets; alerts firing in a test; production smoke passed |

### 16.3 Critical path and safe reordering

- **Critical path:** E00 → E01 → E02 → E04 → E05. E03 depends on E02 and is needed by E04 (payments cancel nudges), so it stays on the path.
- **Safe to reorder:** E06 and E07 can swap, since they touch different modules (`admin`/`nudging` forward text vs `accounts`/settings), provided E07-S05's command menu is written after all commands exist. E08-S04 (alerts) can move into sprint 4 if the operator wants alerts during public beta.
- **Not safe to reorder:** E05 before E04 (no announcement invariant, DR-12); any reminder sending before E05-S04 and E05-S06 are done (guardrail); E01-S01 (encryption) after any story that stores PII.

## 17. Epics and stories (Epics)

Everything from here to §18 is written for the coding agent. Each epic is one fresh session (§18). Stories are in build order. The default Story DoD is §14.8; "Story DoD" lines list only additions. Test file names are the real names to create.

## Epic 00 — Foundation

**Goal** — A repository where `npm run check` is the definition of green, the module boundaries are enforced before any feature exists, and a staging bot answers `/start` through the full stack.
**Personas served** P0 (and P4 for staging access)
**Depends on** nothing.
**Preconditions for the session** — Empty repo on GitHub (private, GitHub Pro: OQ-06 default). Node 26.10.0 and Docker installed locally. gitleaks 8.30.1 binary installed. A Render account with access to Frankfurt. A staging bot token from @BotFather. `[VERIFY-AT-BUILD]` re-checks: latest patch versions of every package in §13.1 (record, don't upgrade); Node 26 LTS status; grammY Bot API support (S-59).
**Scope** — In: stories below. Out: any feature behaviour beyond the `/start` hello slice (E01).
**Flows and screens implemented** — none fully; SCR-01 message CP-SCR01-1 only, as the vertical slice.
**Data and API touched** — ENT-ProcessedUpdate, ENT-AuditEvent (table only); API-01, API-09.
**Risks specific to this epic** — R-05 (Node 26 not LTS until 2026-10-28), R-06 (TypeScript 6/7 split), R-12 (Render plan names and prices), R-17 (Node builds without Temporal).

### E00-S01 — Bootstrap the repository, runtime pins and project docs

**Story** — As **P0 (Builder)**, I want a repository with pinned runtime and package versions and the project's working documents in place, so that every future session starts from the same, versioned memory.
**Context** — Fresh sessions only remember what is in the repo (§18.1). This story creates the layout of §13.3, the docs from Appendix B, and exact pins from §13.1. Evidence: Node release data (S-32), npm registry versions (S-31).
**Scope** — In: `package.json` with exact pins, `package-lock.json`, `.node-version`, `tsconfig*.json`, empty source tree per §13.3, `docs/BLUEPRINT.md` (this file), `docs/PROGRESS.md`, `docs/DECISIONS.md` (DR-01 to DR-23 copied), `docs/REVIEW-CHECKLIST.md` (§14.4 verbatim), `docs/RUNBOOK.md` (skeleton headings from §12.8/§12.9), `docs/OPS-LOG.md`, `AGENTS.md`, `CLAUDE.md`, `.env.example`, `.gitignore`. Out: tool configs (E00-S02), CI (E00-S07).
**Acceptance Criteria**
- AC-1 (happy) Given a clean clone and the official Node.js 26.10.0 build (nodejs.org binary via nvm, fnm or `actions/setup-node`), When `npm ci` runs, Then it completes with exit 0 and `node -e "console.log(typeof Temporal)"` prints `object` (builds without Temporal support exist: S-91).
- AC-2 (pins) Given `package.json`, When inspected by `scripts/check-pins.ts` (run inside `npm test`), Then every dependency and devDependency version is an exact semver (no `^`, `~`, `*`, ranges or tags) and equals the version listed in BLUEPRINT §13.1.
- AC-3 (runtime) Given `.node-version` contains `26.10.0` and `package.json` has `"engines": { "node": ">=26.10.0 <27" }`, When `npm ci` runs on Node 24, Then npm prints an engine warning, and `scripts/check-pins.ts` fails the test with the message `Node 26.10.0+ required`.
- AC-4 (docs) Given the repo root, Then `AGENTS.md` and `CLAUDE.md` exist with byte-identical content from Appendix B, `docs/REVIEW-CHECKLIST.md` contains the §14.4 block verbatim (all 37 lines starting with `[ ]`), and `docs/DECISIONS.md` contains headings `DR-01` to `DR-23`.
- AC-5 (env template) Given `.env.example`, Then it lists every variable of §12.8 exactly once, each with a one-line comment, and no values other than obvious non-secret examples.
- AC-6 (copy) n/a — no user-facing text.
- AC-7 (accessibility) n/a — no UI.
- AC-8 (observability) n/a — no runtime yet.
**Edge cases** — EC-SEC-05 (no secret values in `.env.example`, checked by gitleaks in E00-S02); story-specific: a package whose latest version on install day differs from §13.1 → install the §13.1 version and record the newer one in PROGRESS.md (version drift rule §13.10).
**Tests**
- Unit: `scripts/check-pins.test.ts` — exact pins, versions equal to the §13.1 table (parsed from `docs/BLUEPRINT.md`), Node version gate — covers AC-2, AC-3.
- Unit: `test/arch/docs-present.test.ts` — AGENTS/CLAUDE identical, checklist present, DR headings present, `.env.example` variables equal the §12.8 table — covers AC-4, AC-5.
- Manual: `npm ci` on a clean machine; record in PROGRESS.md — covers AC-1.
- Run: `npm test` green; the new tests fail before the files exist.
**Tasks**
- E00-S01-T1 Create the layout of §13.3 with empty `index.ts` placeholders exporting nothing (knip-clean in E00-S02).
- E00-S01-T2 Write `package.json` with the exact pins of §13.1 and `engines`; run `npm install` once to produce the lockfile.
- E00-S01-T3 Write `tsconfig.json` / `tsconfig.build.json` per §13.5.
- E00-S01-T4 Copy Appendix B files into place; copy DRs into `docs/DECISIONS.md`; copy §14.4 into `docs/REVIEW-CHECKLIST.md`.
- E00-S01-T5 Write `scripts/check-pins.ts` and the two tests.
**Story DoD** — Default + PROGRESS.md "VERIFY-AT-BUILD results" lists each package's latest version found on the day.

### E00-S02 — Build the `check` quality gate

**Story** — As **P0 (Builder)**, I want one command that formats, lints, type-checks, checks dead code, duplication, dependencies and secrets, and runs tests with a coverage floor, so that "green" means the same thing in every session.
**Context** — §13.4 and §14.2 define the gate. Tools verified: Prettier 3.9.9, ESLint 10.11.0 + typescript-eslint 8.71.0 with `projectService` (S-77), knip 6.38.0, jscpd 5.3.3 (S-80), Vitest 5.0.2 coverage thresholds (S-79), npm audit (S-86), gitleaks (S-81).
**Scope** — In: configs for every gate tool, the `check` script, Vitest projects (unit, integration, e2e), coverage thresholds, lint rules of §13.5 including FF-09 and FF-12 restrictions and banned imports, `scripts/` placeholders for `db:verify` (implemented in E00-S05). Out: dependency-cruiser rules (E00-S03), Stryker (E00-S07 CI job; config here).
**Acceptance Criteria**
- AC-1 (happy) Given the skeleton repo, When `npm run check` runs, Then it exits 0 and prints each step's name, and the total wall time is recorded in PROGRESS.md (M-23 baseline).
- AC-2 (format) Given a file with a formatting deviation (e.g. double quotes in `src/main/start.ts`), When `npm run check` runs, Then it exits non-zero at `format:check` naming that file.
- AC-3 (lint strictness) Given a function returning a Promise that is called without `await` in `src/platform/`, When `npm run lint` runs, Then it fails with `@typescript-eslint/no-floating-promises`; and given `const x: any = 1`, Then it fails with `@typescript-eslint/no-explicit-any`.
- AC-4 (restricted syntax) Given `new Date()` in `src/modules/invoices/domain/x.ts`, Then lint fails (FF-12); given `ctx.reply("hello")` in `src/bot/`, Then lint fails (FF-09); given `import { PrismaClient } from "@prisma/client"`, Then lint fails (banned import, §13.2).
- AC-5 (types) Given a type error, When `npm run typecheck` runs, Then it exits non-zero; `tsc --showConfig` shows `strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes` all `true` and `lib` containing `esnext`.
- AC-6 (coverage floor) Given coverage below 80% lines or 75% branches, When `npm run test:cov` runs, Then Vitest exits non-zero naming the metric (S-79).
- AC-7 (duplication, dead code) Given two identical 30-line functions in different files under `src/`, Then `npm run dup` fails; given an unused export, Then `npm run deadcode` fails.
- AC-8 (supply chain and secrets) Given a file containing a string matching a Telegram bot token pattern committed to the tree, When `npm run secrets` runs, Then gitleaks exits 1 with the value redacted; and `npm run audit` exits non-zero when a high advisory exists (verified by running it against a scratch project with a known-vulnerable pinned package, recorded in PROGRESS.md).
**Edge cases** — EC-SEC-05 (secrets in repo → gitleaks), EC-SEC-06 (vulnerable dependency → audit), story-specific: gate tools must not prompt (non-interactive, `CI=true` safe); `check` must fail fast on the first failing step with a non-zero exit.
**Tests**
- Integration: `test/arch/gate.int.test.ts` — creates temporary fixture files in a temp copy of the repo (via `fs.cp` to `os.tmpdir()`), runs each gate script with `execa`-free `child_process.spawnSync`, asserts exit codes and messages — covers AC-2, AC-3, AC-4, AC-5, AC-7, AC-8 (secrets part).
- Manual: coverage threshold failure with a deliberately untested file; audit check against a scratch project — recorded in PROGRESS.md — covers AC-6, AC-8 (audit part).
- Run: `npm run check` green; each fixture test was seen failing before its rule was configured.
**Tasks**
- E00-S02-T1 `.prettierrc.json`, `.prettierignore` (ignore `docs/`, `data/` and `src/platform/db/types.gen.ts`: the docs change only through amendments, and generated files are not hand-formatted).
- E00-S02-T2 `eslint.config.mjs` per §13.5 with FF-09 / FF-12 `no-restricted-syntax` selectors, banned imports, `no-console`, `no-restricted-properties` for `process.env` outside `platform/config`.
- E00-S02-T3 `vitest.config.ts` with projects `unit` (`src/**/*.test.ts`, `test/arch/**`), `integration` (`test/integration/**`), `e2e` (`test/e2e/**`), coverage v8 thresholds 80/75.
- E00-S02-T4 `knip.json`, `.jscpd.json` (`threshold: 3`, `ignore` test fixtures), `stryker.config.json` (`testRunner: "vitest"`, `mutate` = the 4 critical globs, `thresholds: { high: 85, low: 70, break: 70 }`, S-78).
- E00-S02-T5 `check` and related scripts exactly as §13.4; `test/arch/gate.int.test.ts`.
**Story DoD** — Default + measured `check` runtime recorded as the M-23 baseline.

### E00-S03 — Create the module skeleton and architecture fitness functions

**Story** — As **P0 (Builder)**, I want the modules of §12.2 to exist with their public entry points, and the dependency rules to fail the build when violated, so that boundaries hold from the first feature onwards.
**Context** — A modular monolith without enforced boundaries degrades within a few sessions (§12.1). dependency-cruiser rule format with `path`/`pathNot` and group matching is verified (S-76). Implements FF-01 to FF-07 and FF-13.
**Scope** — In: `src/modules/{accounts,clients,invoices,nudging,deliverability,notifications,admin}/index.ts` (+ empty `domain/`, `repo/`), `src/platform/*/index.ts`, `src/bot/`, `src/web/`, `src/main/`; `.dependency-cruiser.cjs`; `test/arch/table-ownership.test.ts` (FF-07) with the ownership map; `test/arch/copy-catalog.test.ts` (FF-10) against an empty catalogue that only contains CP-SCR01-1 (added in E00-S06). Out: business code.
**Acceptance Criteria**
- AC-1 (happy) Given the skeleton, When `npm run arch` runs, Then it exits 0 and reports 0 violations.
- AC-2 (deep import) Given `src/modules/nudging/x.ts` importing `../invoices/domain/balance.ts`, When `npm run arch` runs, Then it fails naming rule FF-02 (`no-deep-module-import`).
- AC-3 (DAG) Given `src/modules/invoices/x.ts` importing `../nudging/index.ts`, Then `npm run arch` fails naming the FF-03 rule `invoices-not-to-nudging`; given `src/platform/log/x.ts` importing any module, Then it fails naming FF-04.
- AC-4 (framework confinement) Given `import { Bot } from "grammy"` in `src/modules/accounts/service.ts`, Then it fails naming FF-05; given `import { Kysely } from "kysely"` in `src/modules/accounts/service.ts` (outside `repo/`), Then it fails naming FF-06; given `import { Resend } from "resend"` outside `src/platform/email/`, Then it fails naming FF-13.
- AC-5 (cycles) Given two files importing each other, Then it fails naming FF-01 (`no-circular`).
- AC-6 (table ownership) Given `src/modules/clients/repo/x.ts` containing `.selectFrom('invoice')`, When the unit test project runs, Then `table-ownership.test.ts` fails with `clients may not query invoice (owned by invoices)`.
- AC-7 (copy) n/a — no user-facing text.
- AC-8 (observability) n/a.
**Edge cases** — story-specific: type-only imports (`import type`) across modules still go through `index.ts` (rules apply to type imports too); test files may import module internals of their own module only.
**Tests**
- Integration: `test/arch/fitness.int.test.ts` — writes each violating fixture into a temp copy, runs `depcruise`, asserts the rule name — covers AC-2, AC-3, AC-4, AC-5.
- Unit: `test/arch/table-ownership.test.ts` (the real FF-07 check) plus a self-test with an in-memory violating source string — covers AC-6.
- Run: `npm run check` green.
**Tasks**
- E00-S03-T1 Create folders and `index.ts` files; `src/main/start.ts` composition-root placeholder that imports nothing yet.
- E00-S03-T2 `.dependency-cruiser.cjs`: `no-circular`, FF-02 (two rules), one rule per module for FF-03, FF-04, FF-05, FF-06, FF-13, all `severity: "error"`.
- E00-S03-T3 `test/arch/table-ownership.test.ts` with the ownership map from §12.2 (`v_*` views allowed for `admin`).
- E00-S03-T4 Fixture-based integration test.
**Story DoD** — Default.

### E00-S04 — Fail-fast configuration, structured logging, error tracking and health endpoint

**Story** — As **P0 (Builder)**, I want configuration validated at startup, structured logs without personal data, error tracking and a health endpoint, so that production problems are diagnosable at 3am by someone who did not write the code.
**Context** — Implements §12.8 (config), §12.9 (logging, Sentry, health), FF-08 harness, EC-OPS-01/-09/-10. pino with `redact` (S-31); `@sentry/node` 11.1.0 (S-63); Fastify (S-75).
**Scope** — In: `platform/config` (zod schema for every §12.8 variable, `get()`), `platform/log` (pino root + child loggers, redaction, correlation IDs), `platform/errors` (`AppError` hierarchy), Sentry init (disabled without DSN), `web/server.ts` with API-09 `/healthz`, `test/harness/logCapture.ts`, `test/harness/sentryCapture.ts`. Out: database check in health (stubbed until E00-S05 adds the pool; AC-4 then extended there).
**Acceptance Criteria**
- AC-1 (happy) Given all required variables for `APP_ENV=staging` set with valid values, When the app starts, Then it logs one `app.started` event with `{ env, version, node }` and listens on `PORT`.
- AC-2 (startup validation) Given `TELEGRAM_WEBHOOK_SECRET=abc def` (contains a space) or `PUBLIC_BASE_URL=http://x` (not https), When the app starts, Then it exits with code 1 and one `config.invalid` log naming the variable and the rule (never the value); and given a Node build where `typeof Temporal !== "object"`, startup exits 1 with a `runtime.temporal_missing` log (S-91).
- AC-3 (email guard) Given `APP_ENV=production` and `EMAIL_RECIPIENT_OVERRIDE` set, Then startup exits 1 with `config.invalid` naming `EMAIL_RECIPIENT_OVERRIDE`; given `APP_ENV=staging` without it, Then startup exits 1 the same way (EC-OPS-10).
- AC-4 (health) Given the server is running, When `GET /healthz` is requested, Then it returns 200 `{"status":"ok","db":"ok","version":"<GIT_SHA or 'dev'>"}` within 2 s (DB part completed in E00-S05).
- AC-5 (redaction) Given a log call with `{ email: "fixture.client1@example.test", name: "Fixture Client 1", token: "x" }`, When captured, Then the output contains `[Redacted]` for those keys and none of the three values.
- AC-6 (error tracking) Given `SENTRY_DSN` set and a route handler throwing `new Error("boom")`, When the request is made, Then one event is captured by the Sentry test transport with the correlation ID tag and no request body, headers or user data, and the response is 500 with the error envelope `{"error":{"code":"internal","message":"Internal error"}}`.
- AC-7 (copy / a11y) n/a — no user-facing text.
- AC-8 (observability) Given any HTTP request, Then exactly one `http.request` log line with `method`, `route`, `status`, `duration_ms`, `correlation_id` is emitted.
**Edge cases** — EC-OPS-01 (missing variable), EC-OPS-09 (health), EC-OPS-10 (override guard), EC-PERM-04 (PII in logs), EC-INT-01 (Sentry unreachable → the error is still logged at `error`), EC-INT-04 (sandbox/production guard).
**Tests**
- Unit: `src/platform/config/schema.test.ts` — each variable's rules; guards — covers AC-2, AC-3.
- Unit: `src/platform/log/redact.test.ts` — covers AC-5.
- Integration: `test/integration/server.int.test.ts` — start via `buildServer(config)`, `inject` `/healthz`, a throwing test route registered only in tests, log capture, Sentry capture — covers AC-1, AC-4, AC-6, AC-8.
**Tasks**
- E00-S04-T1 zod schema + `get()` + startup validation with `process.exitCode = 1`.
- E00-S04-T2 pino config with redact paths (§12.9) and `correlation_id` child loggers; Fastify `genReqId`.
- E00-S04-T3 `AppError` types and the error envelope handler.
- E00-S04-T4 Sentry init + test transport harness (verify option names in the Sentry Node docs for 11.1.0 and record them in DECISIONS.md) [VERIFY-AT-BUILD].
- E00-S04-T5 `/healthz` route.
**Story DoD** — Default + the Sentry option names used are recorded in DECISIONS.md with the docs URL.

### E00-S05 — Database access, migrations, drift check and test database harness

**Story** — As **P0 (Builder)**, I want a typed database layer with forward-only migrations, a drift check and a real Postgres for tests, so that the schema is executable documentation and tests exercise the real database.
**Context** — DR-05. Kysely migrator with locking (S-34), `kysely-codegen --verify` (S-36), Testcontainers Postgres (S-31), `uuidv7()` (S-84). Implements FF-06, FF-11, EC-OPS-02, EC-OPS-07, EC-SEC-01.
**Scope** — In: `platform/db` (pool, `Kysely<DB>` instance, `withTransaction(fn)`, query counter for tests), `migrate.ts`, first migration creating `processed_update` and `audit_event` (§12.4), `types.gen.ts`, `scripts/db-verify.ts`, Vitest `globalSetup` for Postgres 18 via Testcontainers, `test/factories/` base, `scripts/seed-dev.ts` guard, `platform/audit.record()`. Out: feature tables (created by the stories that own them).
**Acceptance Criteria**
- AC-1 (happy) Given an empty Postgres 18, When `npm run db:migrate:dev` runs, Then tables `processed_update` and `audit_event` exist with the columns and indexes of §12.4, and `kysely_migration` lists 1 migration.
- AC-2 (drift) Given a migration that adds a column but `types.gen.ts` not regenerated, When `npm run db:verify` runs, Then it exits non-zero (kysely-codegen `--verify`, S-36); after regenerating, it exits 0.
- AC-3 (concurrency of migrations) Given two processes run `migrateToLatest()` simultaneously on an empty DB, Then both finish without error and the migration is applied once (Kysely lock, S-34).
- AC-4 (failure) Given a migration that throws halfway, When run, Then the process exits 1, the failed migration is not recorded as applied, and a `db.migration.failed` log names the migration file.
- AC-5 (seed guard) Given `APP_ENV=staging`, When `npm run db:seed:dev` runs, Then it exits 1 with `refusing to seed staging` and writes nothing (EC-OPS-07).
- AC-6 (health DB) Given the DB is stopped, When `GET /healthz`, Then 503 `{"status":"degraded","db":"down"}` within 2.5 s.
- AC-7 (injection) Given a lint fixture using `sql.raw(userInput)`, Then `npm run lint` fails with the §13.5 restricted-syntax message.
- AC-8 (observability) Given `withTransaction` rolls back due to an error, Then one `db.tx.rollback` log with `correlation_id` and error code is emitted, without query parameters.
**Edge cases** — EC-OPS-02 (migration failure), EC-OPS-07 (seed guard), EC-SEC-01 (injection), story-specific: Testcontainers unavailable (Docker not running) → tests fail with a message `Docker is required for integration tests (BLUEPRINT §13.4)`.
**Tests**
- Integration: `test/integration/db.int.test.ts` — migrate, schema assertions via `information_schema`, concurrent migrate, failing migration in a temp folder, transaction rollback log — covers AC-1, AC-3, AC-4, AC-8.
- Integration: `test/integration/db-verify.int.test.ts` — drift scenario in a temp copy — covers AC-2.
- Unit: `scripts/seed-dev.test.ts` — covers AC-5.
- Integration: extend `server.int.test.ts` for DB-down health — covers AC-6.
- Lint fixture in `gate.int.test.ts` — covers AC-7.
**Tasks**
- E00-S05-T1 `platform/db/index.ts` (pool from `DATABASE_URL`, Kysely instance, `withTransaction`, query counter hook for tests).
- E00-S05-T2 Migration runner and first migration file `202609300001_base.ts`.
- E00-S05-T3 `scripts/db-verify.ts` (Testcontainers → migrate → `kysely-codegen --dialect postgres --out-file src/platform/db/types.gen.ts --verify`).
- E00-S05-T4 Vitest globalSetup (one container per run), truncate helper, factories base.
- E00-S05-T5 `platform/audit.record(tx, event)`; seed guard.
**Story DoD** — Default.

### E00-S06 — Telegram webhook skeleton with update de-duplication and bot test harness

**Story** — As **P0 (Builder)**, I want the webhook endpoint wired to a grammY bot with secret-token checking, update de-duplication and a test harness that fakes Telegram, so that every later story can be tested end to end without the network.
**Context** — DR-09. `webhookCallback(bot, "fastify", { secretToken, timeoutMilliseconds, onTimeout })` (S-55, S-57); header `X-Telegram-Bot-Api-Secret-Token` (S-10); transformers and `handleUpdate` for tests (S-58, S-90); preset `botInfo` avoids `getMe` (S-90); updates are re-delivered after slow or failed handling (S-54). Vertical slice: `/start` replies with CP-SCR01-1 only.
**Scope** — In: `bot/bot.ts` (bot with `botInfo`, auto-retry for interactive calls: `maxRetryAttempts: 3`, `maxDelaySeconds: 5`, S-56), dedupe middleware (ENT-ProcessedUpdate insert-or-ignore), `platform/copy/catalog.ts` with CP-SCR01-1, `platform/copy/render.ts` (HTML escaping, variable check), `/start` handler, `web/routes/telegram.ts` (API-01), `test/harness/{fakeTelegram,updates,testApp}.ts`, `scripts/ops-set-webhook.ts`. Out: onboarding (E01).
**Acceptance Criteria**
- AC-1 (happy) Given a private-chat update `/start` from user 111 with first name "Maya", When POSTed to `/tg/webhook` with the correct secret header, Then the response is 200 and the fake Telegram API recorded exactly one `sendMessage` to chat 111 with text equal to CP-SCR01-1 rendered with `first_name=Maya`, and `parse_mode: "HTML"`.
- AC-2 (secret) Given the same update without the header or with a wrong value, Then the response is 401, no `sendMessage` is recorded, and no `processed_update` row is inserted (EC-AUTH-13).
- AC-3 (dedupe) Given the same `update_id` POSTed twice, Then both responses are 200 and exactly one `sendMessage` is recorded; a `bot.update.duplicate` log is emitted for the second.
- AC-4 (escaping) Given first name `<b>Mia & Co</b>`, Then the sent text contains `&lt;b&gt;Mia &amp; Co&lt;/b&gt;` (EC-INPUT-05).
- AC-5 (failure) Given the fake Telegram API returns 500 for `sendMessage` three times, Then the handler gives up after 3 retries (auto-retry), responds 200 to the webhook (so Telegram does not re-deliver), and logs `bot.reply.failed` with the method name.
- AC-6 (copy) Given the catalogue, Then `catalog.ts` contains CP-SCR01-1 with the exact text of §11.1, and FF-10 passes for the IDs present in the catalogue, with the full-equality check enabled from E01 onwards.
- AC-7 (accessibility) n/a — plain text message.
- AC-8 (observability) Given any update, Then one `bot.update.received` log with `update_id`, `update_type`, `correlation_id = "upd:<update_id>"` and no message text is emitted, and the handler time is logged as `duration_ms`.
**Edge cases** — EC-AUTH-13 (bad secret), EC-NET-03 / EC-BOT-05 (re-delivery → dedupe), EC-INPUT-05 (escaping), story-specific: an update type we do not subscribe to (e.g. `edited_channel_post`) → 200 and ignored with `bot.update.ignored`.
**Tests**
- Integration: `test/integration/telegram_webhook.int.test.ts` — `rejects_missing_secret`, `same_update_twice_processed_once`, `start_replies_welcome`, `escapes_html_in_names`, `reply_failure_does_not_fail_webhook` — covers AC-1 to AC-5, AC-8.
- Unit: `src/platform/copy/render.test.ts` — escaping, missing-variable error — covers AC-4, AC-6.
- Run: `npm run check` green.
**Tasks**
- E00-S06-T1 Copy renderer and catalogue with CP-SCR01-1.
- E00-S06-T2 Bot factory with `botInfo`, auto-retry config, error boundary replying CP-SCR20-4 (added to the catalogue when E01-S02 lands; until then, log only).
- E00-S06-T3 Dedupe middleware using the `processed_update` table created in E00-S05; the insert-or-ignore repository function lives in `src/bot/repo/`.
- E00-S06-T4 Fastify route with `webhookCallback(..., { secretToken, timeoutMilliseconds: 9000, onTimeout: "throw" })`.
- E00-S06-T5 Test harness (`fakeTelegram` transformer recording calls, `updates.ts` builders, `sendUpdate` via `inject`).
- E00-S06-T6 `scripts/ops-set-webhook.ts` calling `setWebhook` with `url`, `secret_token`, `allowed_updates: ["message","callback_query","my_chat_member"]`, `drop_pending_updates: false` (S-10).
**Story DoD** — Default.

### E00-S07 — CI pipeline, branch protection and staging deployment with rollback

**Story** — As **P0 (Builder)**, I want CI to run `check` on every push, `main` protected, and staging deployed automatically after CI passes, so that the gate cannot be bypassed and every epic ends deployable.
**Context** — DR-17, DR-18, §13.8. Render: build, pre-deploy migrations, start, health checks, "After CI Checks Pass" auto-deploy, zero-downtime with SIGTERM (S-41); `.node-version` honoured (S-62); GitHub Pro protected branches for private repos (S-51).
**Scope** — In: `.github/workflows/ci.yml` (jobs `check`, `mutation`), `render.yaml` (staging web service + staging Postgres; production defined but not yet created), graceful shutdown, `docs/RUNBOOK.md` deploy + rollback sections, staging webhook set, M-10/M-11/M-15/M-16/M-17/M-19 recording format in PROGRESS.md. Out: production go-live (E08-S03).
**Acceptance Criteria**
- AC-1 (happy) Given a push to a PR branch, When CI runs, Then job `check` runs `npm run check` on Node from `.node-version` with Docker available and reports success; the run takes ≤ 8 minutes including install.
- AC-2 (protection) Given branch protection on `main` requiring `ci / check`, When a PR has a failing `check`, Then GitHub blocks the merge; direct pushes to `main` are rejected (verified manually, screenshot path recorded in PROGRESS.md).
- AC-3 (deploy) Given a merge to `main` with green CI, Then Render staging deploys automatically, the pre-deploy step runs `npm run db:migrate`, and `/healthz` on the staging URL returns 200 with the new `version`.
- AC-4 (failed migration) Given a migration that fails in pre-deploy, Then the deploy fails and the previous release keeps serving `/healthz` with the old `version` (EC-OPS-02).
- AC-5 (graceful shutdown) Given SIGTERM during a request, Then in-flight requests complete, the process exits within 25 s with code 0, and `app.stopped` is logged (EC-OPS-11).
- AC-6 (smoke) Given the staging webhook is set, When the product owner sends `/start` to the staging bot from a phone, Then CP-SCR01-1 arrives within 3 s (manual, recorded).
- AC-7 (rollback) Given a deployed release N, When the documented rollback is executed, Then release N−1 serves `/healthz` within 10 minutes (manual rehearsal recorded in RUNBOOK and OPS-LOG) [VERIFY-AT-BUILD on Render's rollback UI].
- AC-8 (observability) Given the CI run, Then the `check` job uploads the jscpd JSON and the coverage-floor summary as artifacts (inputs to M-20 and the M-23 duration).
**Edge cases** — EC-OPS-02, EC-OPS-03 (backward-compatible migrations: documented rule in RUNBOOK), EC-OPS-11, story-specific: gitleaks binary download in CI pinned to 8.30.1 with a checksum check.
**Tests**
- Integration: `test/integration/shutdown.int.test.ts` — start the server, open a slow test route, send SIGTERM to a child process, assert completion and exit code — covers AC-5.
- Manual: AC-2, AC-3, AC-4 (with a deliberately failing migration on a throwaway branch deployed to staging, then reverted), AC-6, AC-7 — each recorded in PROGRESS.md with date and observation.
- CI itself: AC-1, AC-8.
**Tasks**
- E00-S07-T1 `ci.yml` with actions pinned to commit SHAs (record SHAs in DECISIONS.md), gitleaks install, artifacts.
- E00-S07-T2 Branch protection (manual, by the product owner if the agent lacks rights; checklist in RUNBOOK).
- E00-S07-T3 `render.yaml` + environment group for staging; create services; set env vars.
- E00-S07-T4 SIGTERM handling in `src/main/start.ts` (stop loops, `server.close()`, exit).
- E00-S07-T5 RUNBOOK: deploy, rollback, set-webhook; run `ops:set-webhook` for staging.
**Story DoD** — Default + staging URL recorded in PROGRESS.md "Current position".

**Epic-level tests** — `npm run check` green in CI on `main`; staging `/healthz` 200; `/start` on the staging bot returns CP-SCR01-1; all fixture-based gate and fitness tests green.
**Definition of Done**
- All 7 stories DONE with evidence lines in PROGRESS.md.
- `npm run check` green locally and in CI on the merge commit; M-23 baseline recorded.
- Staging deployed via CI-gated auto-deploy; webhook set; manual `/start` check recorded.
- DECISIONS.md contains DR-01 to DR-23 plus any new DRs (Sentry option names, action SHAs).
- PROGRESS.md: epic DONE, `[VERIFY-AT-BUILD]` results, next epic's preconditions (staging bot token and a Resend API key for E01).
- Blueprint Amendments log updated if any AC changed.
**Session handoff checklist** — Print the §18.5 summary: stories and status; `check` result with commit SHA; CI and staging status; self-review result (`reviewed: n items, n fixed, n accepted`); diff size per story; deviations and amendments; `[VERIFY-AT-BUILD]` results (package versions, Node LTS status); manual checks done by the product owner; preconditions for E01 (Resend account + API key for staging, sending domain decision OQ-07 if available); "Safe to terminate. Next: open a fresh session and paste the E01 prompt."

## Epic 01 — Onboarding and account

**Goal** — A new freelancer goes from `/start` to a verified reply-to email, a time zone, a default currency and optional payment details in about 2 minutes, with every piece of personal data encrypted.
**Personas served** P1, P2 (P0 for encryption)
**Depends on** E00: `check`, module skeleton, DB layer, webhook + bot harness, copy renderer, staging.
**Preconditions for the session** — PROGRESS.md shows E00 DONE; `check` green on `main`. A staging Resend API key and a verified sending domain on Resend for staging (OQ-07 default: `send-staging.<domain>`). Env vars `RESEND_API_KEY`, `EMAIL_FROM_ADDRESS`, `DATA_ENCRYPTION_KEYS`, `DATA_ENCRYPTION_ACTIVE_KEY_ID`, `BLIND_INDEX_KEY`, `CODE_PEPPER` set in staging. `[VERIFY-AT-BUILD]`: Resend free-tier limits (S-26) and SDK version (S-31); `Intl.supportedValuesOf('timeZone')` behaviour on Node 26.10 (S-64).
**Scope** — In: stories below. Out: settings edits after onboarding (E07); email change (E07-S02).
**Flows and screens implemented** F-01, F-14; SCR-01 to SCR-06, SCR-20.
**Data and API touched** ENT-Freelancer, ENT-EmailVerification, ENT-ConversationState (new tables); INT-email (verification only); CP-EML-23, CP-EML-24.
**Risks specific to this epic** R-10 (encryption key loss), R-13 (sending-domain setup delays verification in staging).

### E01-S01 — Field encryption and blind index for personal data

**Story** — As **P0 (Builder)**, I want one audited way to encrypt, decrypt and blind-index personal data, so that every later story stores PII the same safe way and a database dump reveals no contacts.
**Context** — DR-08; Telegram developer terms §4.4 (S-07). Uses `node:crypto` (AES-256-GCM, HMAC-SHA-256). Key IDs allow rotation (§12.8). Implements the primitives FF-14 relies on.
**Scope** — In: `platform/crypto` (`encrypt(plaintext) → Buffer`, `decrypt(buf) → string`, `blindIndex(kind, value) → Buffer`, `hmacToken`, `timingSafeEqual` wrapper, `randomCode(6)`), key loading from config, Kysely column helpers, `scripts/crypto-reencrypt.ts` skeleton (full procedure in E08-S01). Out: token format for client links (E05-S01).
**Acceptance Criteria**
- AC-1 (happy) Given active key ID 1, When `encrypt("fixture.client1@example.test")` then `decrypt(result)`, Then the plaintext is returned; the ciphertext starts with byte `0x01` (key ID), is ≥ 29 bytes longer than the plaintext (1 + 12 IV + 16 tag), and two encryptions of the same plaintext differ (random IV).
- AC-2 (tamper) Given a ciphertext with one flipped byte, When `decrypt` runs, Then it throws `CryptoError` with `code: "decrypt_failed"` and the message contains no plaintext or key material.
- AC-3 (rotation) Given data encrypted with key 1 and config `DATA_ENCRYPTION_KEYS=1:…,2:…` with active key 2, When decrypting old data, Then it succeeds with key 1, and new writes use key 2 (first byte `0x02`).
- AC-4 (unknown key) Given a ciphertext with key ID 7 not in config, When decrypting, Then `CryptoError` `code: "unknown_key_id"` with `key_id: 7` (EC-OPS-12).
- AC-5 (blind index) Given `blindIndex("email", " Fixture.Client1@Example.TEST ")` and `blindIndex("email", "fixture.client1@example.test")`, Then both return the same 32-byte value; a different `kind` returns a different value.
- AC-6 (startup validation) Given `DATA_ENCRYPTION_KEYS` with a key that does not decode to 32 bytes, When the app starts, Then it exits 1 with `config.invalid` naming the variable (EC-OPS-01).
- AC-7 (copy / accessibility) n/a — no user-facing text.
- AC-8 (observability) Given any crypto error, Then the log event `crypto.error` carries `code` and `key_id` only.
**Edge cases** — EC-OPS-12 (missing, malformed or rotated key), EC-PERM-04 (no plaintext in errors or logs), story-specific: empty string encrypts and decrypts to empty string; strings up to 10,000 characters supported (conversation data).
**Tests**
- Unit: `src/platform/crypto/crypto.test.ts` — round trip, randomness, tamper, rotation, unknown key, blind index normalisation, empty and long strings — covers AC-1 to AC-5.
- Unit: extend `src/platform/config/schema.test.ts` — bad key length — covers AC-6.
- Mutation: `src/platform/crypto` is not in the mutation set; the tests above assert concrete bytes where deterministic (key ID byte, lengths).
**Tasks**
- E01-S01-T1 Key parsing in config (`id:base64`, 32 bytes each, active ID present).
- E01-S01-T2 `encrypt` / `decrypt` with AES-256-GCM, format `key_id ‖ iv ‖ tag ‖ ciphertext`.
- E01-S01-T3 `blindIndex(kind, value)` = HMAC-SHA-256(`BLIND_INDEX_KEY`, `kind + ":" + normalise(value)`), with email normalisation = trim + lowercase.
- E01-S01-T4 `randomCode`, `timingSafeEqual` wrappers (injectable for tests).
**Story DoD** — Default.

### E01-S02 — Conversation engine and system fallbacks

**Story** — As **P1 (Maya, solo freelancer)**, I want the bot to remember where I am in a multi-step setup, answer anything I send, and never leave me stuck, so that I can finish setup in bits while on the move.
**Context** — Implements ENT-ConversationState (encrypted `data_enc`), F-14 and SCR-20. Telegram's guidance: respond to every message; edit keyboards in place (S-12). Stale buttons, media, groups, floods and edits are the bot's common failure modes (EC-BOT-*). grammY's session and conversations plugins are forbidden (§13.2).
**Scope** — In: `bot/conversation/engine.ts` (flows as typed step machines: `{ flow, step, data }`, `start`, `advance`, `clear`, 24 h expiry), middleware order (§12.3), fallback handler, stale-callback handling, media, group and edit handling, per-user flood limit (30 updates/min, in-memory token bucket), `/cancel`, `platform/text.normaliseLine`, callback-data parser (`v1:` zod schema), catalogue lines CP-SCR20-1 to CP-SCR20-12, CP-SCR06-7, CP-SCR06-8. Out: the onboarding steps themselves (E01-S03 to E01-S06).
**Acceptance Criteria**
- AC-1 (happy) Given a test flow `demo` with steps `a → b` registered, When the user starts it and sends "x" then "y", Then `conversation_state` holds `{ flow: "demo", step: "b" }` after the first message, the stored `data_enc` decrypts to `{"a":"x"}`, and after the second the state row is deleted.
- AC-2 (unknown input) Given no active flow, When the user sends "hello", Then CP-SCR20-1 is sent (EC-BOT-01).
- AC-3 (media and edits) Given an active flow, When the user sends a sticker, voice note, photo or document, Then CP-SCR20-3 is sent and the step is unchanged; When the user edits a previous message, Then CP-SCR20-9 is sent (EC-BOT-02, EC-BOT-09).
- AC-4 (expiry) Given a flow last updated 24 h + 1 s ago (fake clock), When the user sends anything, Then CP-SCR20-5 is sent and the state row is deleted (EC-BOT-06).
- AC-5 (stale button) Given a callback with `v1:demo_ok:<uuid>` for a flow that is no longer active, or unparseable callback data, Then `answerCallbackQuery` shows CP-SCR20-2 and nothing changes (EC-BOT-07).
- AC-6 (group and flood) Given the bot is added to a group (`my_chat_member` with a group chat), Then CP-SCR20-6 is sent once to the group and `leaveChat` is called (EC-BOT-03); Given 31 updates from one user within 60 s, Then the 31st gets CP-SCR20-7 with `seconds` ≥ 1 and is not processed further (EC-BOT-04).
- AC-7 (cancel and copy) Given an active flow, When `/cancel`, Then CP-SCR06-7 and the state is cleared; with no flow, CP-SCR06-8. Every button in the engine's keyboards has a text label (EC-A11Y-08), and every catalogue line renders within its §11 max length with the longest fixtures (EC-L10N-03).
- AC-8 (observability) Given an unexpected exception in a handler, Then the transaction is rolled back, CP-SCR20-4 is sent, one Sentry event and one `bot.handler.error` log (with `update_id`, no message text) are emitted; `bot.update.unhandled` is never logged, because every update is answered.
**Edge cases** — EC-INPUT-01, EC-INPUT-03 (normalisation), EC-INPUT-05 and EC-SEC-02 (escaping via the renderer), EC-MSG-09 (missing template variable fails a test), EC-BOT-01 to EC-BOT-04, EC-BOT-06, EC-BOT-07, EC-BOT-09, EC-A11Y-08, EC-L10N-03, EC-DEV-11 (manual check of keyboards on iOS and Android at epic end); story-specific: a forwarded text message is treated as normal text input.
**Tests**
- Unit: `src/bot/conversation/engine.test.ts` — transitions, expiry at exactly 24 h (expired) and 24 h − 1 s (active), clear — covers AC-1, AC-4.
- Unit: `src/platform/text/normalise.test.ts` — zero-width, RTL marks, tabs, NBSP, empty result — EC-INPUT-01, EC-INPUT-03.
- Unit: `src/platform/copy/lengths.test.ts` — renders every catalogue line with `test/fixtures/longest.ts` values, asserts ≤ max length — AC-7 (EC-L10N-03).
- Integration: `test/integration/fallbacks.int.test.ts` — unknown text, media, edits, stale callback, group add, flood, cancel, handler exception — covers AC-2, AC-3, AC-5, AC-6, AC-7, AC-8.
**Tasks**
- E01-S02-T1 `conversation_state` migration + `src/bot/repo/conversationRepo.ts` (encrypt via E01-S01).
- E01-S02-T2 Engine API and typed flow definitions.
- E01-S02-T3 Middleware: flood limiter, loader, engine, routers, fallback, error boundary.
- E01-S02-T4 Callback parser (zod) and stale handling.
- E01-S02-T5 Catalogue lines for SCR-20, CP-SCR06-7, CP-SCR06-8; lengths test; turn on FF-10 full equality for the IDs added so far (the FF-10 test reads a `CP_IDS_IMPLEMENTED` list until E07-S05 completes the catalogue, then asserts full equality).
**Story DoD** — Default + manual check of stale-button behaviour on one real Telegram client, recorded.

### E01-S03 — Start command and profile name steps

**Story** — As **P1 (Maya, solo freelancer)**, I want to start the bot and set the name clients will see in one tap, so that reminders carry my real name without any typing.
**Context** — F-01 steps 1–4; SCR-01, SCR-02. Creates ENT-Freelancer. The Telegram name is offered as the default (fewer taps). Returning users get their overdue count (0 / 1 / many variants).
**Scope** — In: `freelancer` migration (all §12.4 columns), `accounts.createFromTelegram`, `accounts.updateProfile`, onboarding flow steps `name` and `business`, CP-SCR01-1 to CP-SCR01-6, CP-SCR02-1 to CP-SCR02-8, returning-user message using `invoices.countOverdue` (returns 0 until E02). Out: email and later steps (E01-S04 to E01-S06).
**Acceptance Criteria**
- AC-1 (happy) Given user 111 (first name "Maya", last name "Costa") with no account, When `/start`, Then a `freelancer` row exists with `telegram_user_id = 111`, `onboarding_step = "name"`, and CP-SCR01-1 with button CP-SCR01-2 is sent; When "Set up" is tapped, Then CP-SCR02-1 + CP-SCR02-8 ("Step 1 of 5") with button "Use Maya Costa" (CP-SCR02-2) is shown.
- AC-2 (default name) When "Use Maya Costa" is tapped, Then `display_name_enc` decrypts to "Maya Costa", CP-SCR02-5 with button CP-SCR02-6 is shown; When "No business name" is tapped, Then `business_name_enc` is null and `onboarding_step = "email"`.
- AC-3 (validation) Given the name prompt, When the user sends "   " or a string of only U+200B, Then CP-SCR02-3 and no change; When a 61-grapheme name is sent (e.g. 61 × "é" as base + combining mark), Then CP-SCR02-4 with `length=61`; a 60-grapheme name with emoji "Maya 🎨 …" is accepted (EC-INPUT-01, EC-INPUT-02, EC-INPUT-04).
- AC-4 (business validation) When an 81-character business name is sent, Then CP-SCR02-7; 80 characters are accepted.
- AC-5 (returning) Given an onboarded freelancer with 0 / 1 / 3 overdue invoices, When `/start`, Then CP-SCR01-3 / CP-SCR01-4 / CP-SCR01-5 respectively, followed by the home keyboard; a `/start` with payload `ref_abc` behaves the same (EC-BOT-10); a freelancer mid-onboarding (< 24 h) gets CP-SCR01-6 and the pending prompt.
- AC-6 (copy) Every message in this story uses the exact CP lines listed in Scope, and no Telegram name is shown if it is empty (CP-SCR02-2 hidden).
- AC-7 (accessibility) Buttons are labelled in words; the step indicator is text ("Step 1 of 5"), not emoji.
- AC-8 (observability) `/start` for a new user emits `account.created` (freelancer ID only); profile updates emit `account.profile_updated` with changed field names only.
**Edge cases** — EC-INPUT-01, EC-INPUT-02, EC-INPUT-04, EC-DATA-02 (singular overdue copy), EC-BOT-10; story-specific: two `/start` updates from the same new user processed concurrently → one `freelancer` row (unique `telegram_user_id`, insert-or-select).
**Tests**
- Unit: `src/modules/accounts/domain/profile.test.ts` — grapheme length rules, trimming — covers AC-3, AC-4.
- Integration: `test/integration/onboarding_profile.int.test.ts` — new user, default name, business skip, validation, returning variants, payload, concurrent `/start` — covers AC-1, AC-2, AC-5, AC-6, AC-8.
- E2E: `test/e2e/onboarding.e2e.test.ts` (CE-1) first half — covers AC-1, AC-2.
**Tasks**
- E01-S03-T1 `freelancer` migration + repo (encrypting fields via E01-S01).
- E01-S03-T2 `accounts` service: `createFromTelegram` (insert on conflict do nothing, then select), `updateProfile`.
- E01-S03-T3 Onboarding flow definition (steps name, business) in `bot/handlers/onboarding.ts`.
- E01-S03-T4 `/start` handler with new / resumed / returning branches; `invoices.countOverdue` stub returning 0 (real in E03-S01).
- E01-S03-T5 Catalogue lines; tests.
**Story DoD** — Default.

### E01-S04 — Reply-to email verification by code

**Story** — As **P1 (Maya, solo freelancer)**, I want to confirm the email address client replies go to with a 6-digit code, so that replies reach me and nobody can make reminders point at someone else's inbox.
**Context** — F-01 steps 5–6; SCR-03; DR-16; EC-SEC-12. Email sent via the `EmailGateway` port (Resend adapter, S-72, S-89); verification emails are not rewritten in staging (§12.8). Code rules: assumption 26.
**Scope** — In: `platform/email` (port + Resend adapter + fake), `email_verification` migration, `accounts.startEmailVerification`, `accounts.confirmEmailCode`, onboarding steps `email` and `email_code`, CP-SCR03-1 to CP-SCR03-14, CP-EML-23, CP-EML-24. Out: changing the email later (E07-S02).
**Acceptance Criteria**
- AC-1 (happy) Given step `email`, When the user sends " Maya@Example.com ", Then CP-SCR03-3 then CP-SCR03-4 (showing `Maya@Example.com`: trimmed, case kept) are sent; the fake email gateway recorded one email to `Maya@Example.com` with subject CP-EML-23 and body CP-EML-24 containing a 6-digit code; `email_verification` holds only an HMAC of the code, `expires_at = now + 15 min`.
- AC-2 (confirm) When the correct code is sent within 15 min, Then `freelancer.email_enc` decrypts to "Maya@Example.com", `email_bidx` = blind index of "maya@example.com", `email_verified_at` = now, the verification row is consumed, CP-SCR03-12 is sent, and `onboarding_step = "time_zone"`.
- AC-3 (validation) When "maya@", "maya example.com" or a 255-char address is sent, Then CP-SCR03-2 and no email is sent.
- AC-4 (wrong / expired / locked) Given a pending code, When a wrong code is sent, Then CP-SCR03-7 with `attempts_left=4`; after the 5th wrong code, CP-SCR03-9 with `minutes=15` and further codes are rejected until lock expiry; When the right code is sent at `expires_at` exactly, Then CP-SCR03-8 (EC-AUTH-01, EC-AUTH-04, EC-TIME-09).
- AC-5 (resend limits) When "Send a new code" is tapped 10 s after the last code, Then CP-SCR03-10 with `seconds=50` and no email; tapped twice concurrently after the cooldown, Then exactly one email and one new row, and the previous code is invalid; a 6th code request within an hour → CP-SCR03-13 (EC-AUTH-02, EC-AUTH-03, EC-AUTH-11).
- AC-6 (failure and suppression) Given the gateway returns 5xx or times out after 10 s, Then CP-SCR03-11, no verification row remains usable, and the step stays `email`; Given the address has a global bounce suppression, Then CP-SCR03-14 and no email.
- AC-7 (accessibility) The code prompt states the expiry (15 minutes) in text, and "Send a new code" is available at every error state (EC-A11Y-06).
- AC-8 (observability) Events `email_verification.sent` / `.confirmed` / `.failed` (`reason`) with freelancer ID only; never the address or the code; the Resend request is tagged `kind=verification`.
**Edge cases** — EC-AUTH-01 to EC-AUTH-04, EC-AUTH-11, EC-INPUT-06, EC-TIME-06, EC-TIME-09, EC-SEC-12, EC-A11Y-06; story-specific: the user sends the code with spaces ("123 456") or Unicode digits → spaces stripped; non-ASCII digits are rejected as wrong codes (no digit normalisation needed for this market).
**Tests**
- Unit: `src/modules/accounts/domain/verification.test.ts` — attempts, lock, expiry boundary, cooldown and hourly cap arithmetic — covers AC-4, AC-5.
- Integration: `test/integration/email_verification.int.test.ts` — happy path with fake gateway, invalid addresses, concurrency, provider failure, suppression — covers AC-1, AC-2, AC-3, AC-5, AC-6, AC-8.
- Manual: receive a real code on staging at the product owner's address within 60 s; recorded.
**Tasks**
- E01-S04-T1 `EmailGateway` port (`send({ to, from, replyTo?, subject, text, html, headers?, tags?, idempotencyKey? })`) + Resend adapter (`emails.send(payload, { idempotencyKey })`, S-89) + fake.
- E01-S04-T2 `email_verification` migration + repo.
- E01-S04-T3 Service functions with limits; email syntax validation (one regex documented in code with its test cases; max 254).
- E01-S04-T4 Onboarding steps and buttons (Send a new code, Change email).
- E01-S04-T5 Catalogue lines; tests.
**Story DoD** — Default + manual staging receipt recorded.

### E01-S05 — Time-zone picker

**Story** — As **P1 (Maya, solo freelancer)**, I want to type my city and confirm the local time shown, so that reminders go out during my working hours.
**Context** — F-01 steps 7–8; SCR-04; DR-11. IANA IDs from `Intl.supportedValuesOf('timeZone')` (S-64); local time rendered with Temporal (S-60). City search matches the city part of IDs.
**Scope** — In: `platform/time` (`listZones()`, `searchZones(query)`, `nowIn(zone)`, `formatTime`), onboarding step `time_zone`, region lists (Europe, America, Asia, Africa, Australia + Pacific, UTC), paging, CP-SCR04-1 to CP-SCR04-11. Out: changing the zone later with re-planning (E07-S01).
**Acceptance Criteria**
- AC-1 (happy) Given step `time_zone` and fake clock `2026-10-05T12:00:00Z`, When the user sends "Lisbon", Then CP-SCR04-3 shows "Europe/Lisbon — it's 13:00 there now. Is that right?" with CP-SCR04-4 / CP-SCR04-5; When "Yes, that's right" is tapped, Then `time_zone = "Europe/Lisbon"`, CP-SCR04-11 is sent and `onboarding_step = "currency"`.
- AC-2 (normalisation) When the user sends "sao paulo", "SÃO PAULO" or "new york", Then the single matches `America/Sao_Paulo` and `America/New_York` are offered (EC-SEARCH-01).
- AC-3 (many) When the user sends "san", Then CP-SCR04-6 lists up to 8 matching zones as buttons (e.g. `America/Santiago`, `America/Santo_Domingo`, …) with `count` equal to the number of matches; more than 8 → first 8 plus CP-SCR04-9.
- AC-4 (none) When the user sends "Springfield", Then CP-SCR04-7 with the region buttons CP-SCR04-2 (EC-SEARCH-02).
- AC-5 (regions) When "Europe" is tapped, Then CP-SCR04-8 lists 8 zones alphabetically with CP-SCR04-9 / CP-SCR04-10 paging by editing the same message; "UTC" saves `Etc/UTC` directly.
- AC-6 (validation) Given a callback carrying a zone index that does not exist in `listZones()`, Then CP-SCR20-2 and no change; only IDs present in `Intl.supportedValuesOf('timeZone')` can ever be saved.
- AC-7 (accessibility) Zone labels replace "_" with spaces; the confirmation includes the local time in text.
- AC-8 (observability) `account.time_zone_set` with the zone ID (not PII).
**Edge cases** — EC-SEARCH-01, EC-SEARCH-02, EC-TIME-10 (validation of saved zones), story-specific: callback data carries a numeric index into the sorted list, never the zone string, to stay within 64 bytes (S-14).
**Tests**
- Unit: `src/platform/time/zones.test.ts` — search normalisation, accents, multi-match ordering, paging — covers AC-2, AC-3, AC-4, AC-5.
- Integration: `test/integration/onboarding_timezone.int.test.ts` — confirm flow with fake clock, regions, invalid index — covers AC-1, AC-5, AC-6, AC-8.
- E2E: CE-1 continues through this step.
**Tasks**
- E01-S05-T1 `platform/time` zone list, search (NFKD + strip diacritics + lowercase, "_" ↔ " "), `nowIn` with Temporal.
- E01-S05-T2 Step handlers, region keyboards, paging.
- E01-S05-T3 Catalogue lines; tests.
**Story DoD** — Default.

### E01-S06 — Default currency, payment instructions and home menu

**Story** — As **P1 (Maya, solo freelancer)**, I want to pick my usual currency and say how clients pay me, then land on a simple home menu, so that invoices and reminders need no repeated typing.
**Context** — F-01 steps 9–10; SCR-05, SCR-06. ISO 4217 List One published 2026-09-17 (S-68) is converted into `currencies.json` (code → minor units) by a script; this story adds the list and code validation only (parsing and formatting amounts is E02-S01). Payment instructions are encrypted (DR-08) and later appear in every reminder (CP-EML-16).
**Scope** — In: `data/iso4217-list-one.xml`, `scripts/gen-currencies.ts`, `platform/money/currencies.json` + `isCurrency(code)`, onboarding steps `currency` and `payment`, home keyboard, `/help`, CP-SCR05-1 to CP-SCR05-10, CP-SCR06-1 to CP-SCR06-6, CP-CMD lines stored in the catalogue (menu registration is E07-S05). Out: amount parsing (E02-S01).
**Acceptance Criteria**
- AC-1 (happy) Given step `currency`, When "EUR" is tapped, Then `default_currency = "EUR"` and CP-SCR05-5 with CP-SCR05-6 is shown; When "Bank transfer: IBAN PT50 0000 0000 0000 0000 0000 0" is sent, Then `payment_instructions_enc` decrypts to that text, `onboarding_step` is null, and CP-SCR05-8 with CP-SCR05-9 is sent.
- AC-2 (other currency) When "Other" then "chf" is sent, Then `default_currency = "CHF"`; When "XYZ" or "EURO" is sent, Then CP-SCR05-4 with `input` echoed (escaped) and no change.
- AC-3 (skip and length) When "Skip for now" is tapped, Then CP-SCR05-10 then CP-SCR05-8, and `payment_instructions_enc` is null; a 501-character text → CP-SCR05-7 with `length=501`; 500 characters are accepted (EC-INPUT-02).
- AC-4 (generation) Given the committed XML, When `npm run gen:currencies` runs, Then `currencies.json` contains 150+ active codes, with `EUR: 2`, `USD: 2`, `JPY: 0`, `KRW: 0`, `CLP: 0`, `INR: 2`, and no fund or precious-metal codes (entries with `CcyMnrUnts = "N.A."` excluded); re-running produces a byte-identical file.
- AC-5 (home and help) Given onboarding is complete, Then the home keyboard CP-SCR06-1 to CP-SCR06-5 is shown with "New invoice" first; `/help` sends CP-SCR06-6.
- AC-6 (not onboarded) Given onboarding is unfinished, When `/new`, `/owed`, `/clients` or `/settings` is sent, Then CP-SCR20-8 and the pending step's prompt (§12.6 matrix).
- AC-7 (accessibility) Currency buttons show ISO codes as text; payment instructions are shown back to the user inside `<code>` so Telegram does not auto-link IBAN-like strings.
- AC-8 (observability) `account.onboarded` with `duration_s` from `created_at` (feeds onboarding time); no payment text in logs.
**Edge cases** — EC-INPUT-02, EC-PAY-06 (currency list is the single source of minor units), story-specific: users who send a currency symbol ("€") at the Other prompt get CP-SCR05-4 (only ISO codes accepted there).
**Tests**
- Unit: `scripts/gen-currencies.test.ts` — sample XML fixture → expected JSON, exclusion of N.A., determinism — covers AC-4.
- Unit: `src/platform/money/currencies.test.ts` — `isCurrency` for EUR, eur, EURO, XYZ — covers AC-2.
- Integration: `test/integration/onboarding_finish.int.test.ts` — currency, other, skip, length, home, help, not-onboarded commands — covers AC-1, AC-2, AC-3, AC-5, AC-6, AC-8.
- E2E: `test/e2e/onboarding.e2e.test.ts` (CE-1) completed end to end: `/start` → home keyboard.
**Tasks**
- E01-S06-T1 Commit the XML (S-68) and the generator; generate `currencies.json`.
- E01-S06-T2 Steps `currency` and `payment`; home keyboard; `/help`.
- E01-S06-T3 Guard middleware for not-onboarded users (CP-SCR20-8).
- E01-S06-T4 Catalogue lines (including CP-CMD-1 to CP-CMD-8 as data); tests; CE-1.
**Story DoD** — Default.

**Epic-level tests** — CE-1 (`test/e2e/onboarding.e2e.test.ts`) green; FF-14 (`no-plaintext-pii.e2e.test.ts`) runs over the onboarding data and finds no plaintext name, email or payment text; full regression of E00 tests.
**Definition of Done**
- All 6 stories DONE with evidence lines.
- CE-1 green; `check` green; CI green on `main`.
- Deployed to staging; the product owner onboards on iOS or Android with a real code email: (1) Telegram name default works; (2) city search "Lisbon" confirms local time; (3) home keyboard shown. Recorded in PROGRESS.md.
- PROGRESS.md and DECISIONS.md updated; Amendments log updated if needed.
**Session handoff checklist** — §18.5 summary, plus: the sending domain used in staging and its SPF/DKIM status in Resend; confirmation that no PII appears in staging logs (spot-check 20 lines); preconditions for E02 (none beyond E01 DONE).

## Epic 02 — Clients and invoices

**Goal** — A freelancer logs an invoice for a new or existing client in under a minute, sees when the first reminder would go out, and can edit, cancel or delete it.
**Personas served** P1, P2
**Depends on** E01: onboarded freelancer (zone, currency), conversation engine, encryption, copy renderer.
**Preconditions for the session** — PROGRESS.md shows E01 DONE; `check` green. No new env vars. `[VERIFY-AT-BUILD]`: ISO 4217 list still current (S-68 published 2026-09-17; re-download only if a newer list exists and record it); Temporal API on Node 26.10 (S-60).
**Scope** — In: stories below. Out: owed list and payments (E03); nudge rows and sending (E04, E05). No reminders are created or sent in this epic; the plan is only previewed.
**Flows and screens implemented** F-02; F-10 (list, card, edit, archive); SCR-07 to SCR-10, SCR-12 (base card, edit, cancel, delete), SCR-16.
**Data and API touched** ENT-Client, ENT-Invoice (new tables); `platform/money`; `nudging/domain/plans.ts` and `schedule.ts` (pure).
**Risks specific to this epic** R-14 (amount and date parsing ambiguity).

### E02-S01 — Money parsing, formatting and totals

**Story** — As **P2 (Arjun, high-volume contractor)**, I want to type amounts the way I write them and see them back with ISO codes, so that multi-currency invoices are never misread.
**Context** — DR-10; §10.5; EC-INPUT-09, EC-INPUT-12, EC-PAY-06. Minor units from `currencies.json` (E01-S06, S-68). Display via `Intl.NumberFormat('en', { style: 'currency', currency, currencyDisplay: 'code' })`. Mutation-tested module (§14.2 row 11).
**Scope** — In: `platform/money/parse.ts` (`parseAmount(input, defaultCurrency) → Result<{ minor: bigint, currency }, ParseError>`), `format.ts` (`format(minor, currency)`, `formatTotals(Map<currency, minor>)`), boundary constants. Out: UI (E02-S03).
**Acceptance Criteria**
- AC-1 (happy) Given default EUR, Then `parseAmount("1200")` → `{ minor: 120000n, currency: "EUR" }`; `"1,200.50"` → 120050n; `"1.200,50"` → 120050n; `"1200,5"` → 120050n; `"1 200"` → 120000n; `"1'200"` → 120000n; `"€1,200"` → 120000n; `"1200 usd"` → `{ 120000n, "USD" }`; `"USD 99.99"` → `{ 9999n, "USD" }`; `"150000 JPY"` → `{ 150000n, "JPY" }`.
- AC-2 (thousands vs decimal) Then `"1,200"` → 120000n (comma + exactly 3 digits = thousands) and `"1,20"` → 120n (comma + 2 digits = decimal); `"1.234.567,89"` → 123456789n.
- AC-3 (validation) Then `"0"` and `"-5"` → `ParseError("not_positive")`; `"abc"` and `""` → `ParseError("unreadable")`; `"1.005"` in EUR → `ParseError("too_many_decimals", { minorDigits: 2 })`; `"150000.5 JPY"` → `ParseError("too_many_decimals", { minorDigits: 0 })`; `"100000000"` in EUR → `ParseError("too_large")`; `"99999999.99"` → accepted; `"12 XYZ"` → `ParseError("unknown_currency", { code: "XYZ" })`.
- AC-4 (formatting) Then (with U+00A0 between code and number, §10.5) `format(120050n, "EUR")` = `"EUR 1,200.50"`; `format(150000n, "JPY")` = `"JPY 150,000"`; `format(1n, "USD")` = `"USD 0.01"`; `formatTotals({ USD: 1240000n, EUR: 315000n, INR: 18000000n })` = `"INR 180,000.00 + USD 12,400.00 + EUR 3,150.00"` (descending by minor amount, ties by code).
- AC-5 (no floats) Given the module source, Then no `parseFloat`, `Number(` or `Math.round` appears outside the single audited digit-conversion function (lint rule, DR-10).
- AC-6 (copy) n/a at this layer; error codes map to CP-SCR08-2 to CP-SCR08-6 in E02-S03.
- AC-7 (accessibility) n/a.
- AC-8 (observability) n/a (pure functions).
**Edge cases** — EC-INPUT-09, EC-INPUT-12, EC-PAY-06, EC-L10N-07; story-specific: non-breaking spaces and narrow no-break spaces as thousands separators ("1 200") are accepted.
**Tests**
- Unit: `src/platform/money/parse.test.ts` — every example in AC-1 to AC-3 as table-driven cases — covers AC-1, AC-2, AC-3.
- Unit: `src/platform/money/format.test.ts` — covers AC-4.
- Mutation: `stryker run --mutate "src/platform/money/**"` score ≥ 70 (recorded in PROGRESS.md).
**Tasks**
- E02-S01-T1 Tokeniser: strip symbols and spaces, detect trailing or leading ISO code.
- E02-S01-T2 Separator rules → integer minor units with BigInt.
- E02-S01-T3 Formatters; totals ordering.
- E02-S01-T4 Lint restriction for float APIs in `platform/money`.
**Story DoD** — Default + Stryker score for `platform/money` recorded.

### E02-S02 — Client records: create, list, edit, archive

**Story** — As **P1 (Maya, solo freelancer)**, I want my clients stored once, with the email reminders go to and a greeting name, and I want to fix or archive them later, so that logging invoices is quick and reminders reach the right person.
**Context** — F-10; SCR-16; ENT-Client (encrypted name, email, contact; blind index for email). Duplicate detection (EC-DATA-11) and limits (EC-BIZ-01). Suppression status is read through `deliverability.isSuppressed` (always "allowed" until E05; the interface exists from E00's skeleton and returns `false` until E05-S04 implements it).
**Scope** — In: `client` migration, `clients` service (`createClient`, `listClients` paged and sorted in memory, `getClient`, `updateClient`, `archiveClient`, `findByEmail`, `countNewAddressesToday`), `/clients` list and card, edit name / email / greeting, archive with confirmation, CP-SCR16-1 to CP-SCR16-18, and the client-creation sub-flow used by the wizard (CP-SCR07-4 to CP-SCR07-14, CP-SCR07-16). Out: the wizard steps around it (E02-S03).
**Acceptance Criteria**
- AC-1 (happy) Given no clients, When the wizard sub-flow creates "Acme Ltd" / "ap@acme.test" / greeting "Sam", Then a `client` row exists with all three fields encrypted and `email_bidx` = blind index of "ap@acme.test"; `/clients` then shows CP-SCR16-1 with `count=1` and the line CP-SCR16-2 "Acme Ltd · ap@acme.test · nothing owed".
- AC-2 (empty and paging) Given no clients, `/clients` sends CP-SCR16-3 (EC-DATA-01); given 23 clients, page 1 shows 10 sorted case-insensitively by name then ID, with CP-SCR16-17; page 3 shows 3 (EC-DATA-03, EC-DATA-05).
- AC-3 (duplicate) Given "Acme Ltd" with "ap@acme.test", When a new client with " AP@acme.TEST " is entered, Then CP-SCR07-10 with CP-SCR07-11 / CP-SCR07-12; "Use Acme Ltd" selects the existing client; "Create new anyway" creates a second row (EC-DATA-11, EC-INPUT-10).
- AC-4 (validation and limits) Name of 81 graphemes → CP-SCR07-9; invalid email → CP-SCR07-8; greeting of 41 graphemes → CP-SCR02-4 wording with `length=41`; the 11th new client email address added on the same local day → CP-SCR07-13; the 201st active client → CP-SCR07-16 (EC-BIZ-01, EC-INPUT-02).
- AC-5 (edit and archive) Editing the email to "billing@acme.test" updates both `email_enc` and `email_bidx` and sends CP-SCR16-15; editing the name or greeting sends CP-SCR16-16; Archive → CP-SCR16-12 (with the open-invoice count) → "Archive" → `archived_at` set, CP-SCR16-18, the client disappears from `/clients` and from the wizard picker, but its invoices remain (EC-DATA-06).
- AC-6 (copy) The client card shows CP-SCR16-4 and, from E05 on, one of CP-SCR16-5 to CP-SCR16-7; in this epic, CP-SCR16-5 always.
- AC-7 (permissions and accessibility) A callback `v1:cli:<id>` for another freelancer's client → CP-SCR12-32 and no change (EC-PERM-02); all buttons are labelled in words.
- AC-8 (observability) `client.created` / `client.updated` (field names only) / `client.archived` events with IDs only.
**Edge cases** — EC-INPUT-02, EC-INPUT-04 (emoji and non-Latin names), EC-INPUT-06, EC-DATA-01, EC-DATA-03, EC-DATA-05, EC-DATA-06, EC-DATA-11, EC-BIZ-01, EC-L10N-09 (names like "Ваня Studio" and "山田デザイン" round-trip); story-specific: an archived client's email re-entered as a new client is treated as a duplicate of the archived one (offer "Use", which un-archives it).
**Tests**
- Unit: `src/modules/clients/domain/validation.test.ts` — limits, grapheme counting — covers AC-4.
- Integration: `test/integration/clients.int.test.ts` — create, duplicate, paging and sorting, edit, archive, other freelancer's ID, daily cap across local midnight in America/Los_Angeles — covers AC-1 to AC-5, AC-7, AC-8.
**Tasks**
- E02-S02-T1 `client` migration + repo (encryption, blind index).
- E02-S02-T2 Service functions with limits (`countNewAddressesToday` by the freelancer's local date).
- E02-S02-T3 Client creation sub-flow (reusable by the wizard).
- E02-S02-T4 `/clients` list, card, edit, archive handlers.
- E02-S02-T5 Catalogue lines; tests.
**Story DoD** — Default.

### E02-S03 — New-invoice wizard: client and amount steps

**Story** — As **P1 (Maya, solo freelancer)**, I want to start a new invoice by tapping a recent client and typing the amount, so that logging takes seconds.
**Context** — F-02 steps 1–3; SCR-07, SCR-08. Uses E02-S02's sub-flow and E02-S01's parser. The wizard timer starts at step 1 (M-1.1).
**Scope** — In: flow `new_invoice` steps `client` and `amount`; recent-clients query (last 6 by latest invoice, then by creation); "Show all clients" paging; CP-SCR07-1 to CP-SCR07-3, CP-SCR07-15, CP-SCR07-17, CP-SCR08-1 to CP-SCR08-7; 500-open-invoice limit check at `/new`. Out: due date onwards (E02-S04 to E02-S06).
**Acceptance Criteria**
- AC-1 (happy) Given clients "Acme Ltd" and "Bolt GmbH", When `/new`, Then CP-SCR07-1 with buttons "Acme Ltd", "Bolt GmbH", "New client", and CP-SCR07-15; When "Acme Ltd" is tapped, Then CP-SCR08-1 with `client=Acme Ltd`, `currency=EUR` and CP-SCR08-7; When "1,200.50" is sent, Then the step becomes `due_date` with draft `{ clientId, minor: 120050n, currency: "EUR" }`.
- AC-2 (currency override) When "950 GBP" is sent, Then the draft currency is GBP.
- AC-3 (validation) Each `ParseError` maps to its copy: unreadable → CP-SCR08-2 (with `input` truncated to 20 chars and escaped), not_positive → CP-SCR08-3, too_many_decimals → CP-SCR08-4 (with an example, e.g. "Try 1200.50" or "Try 150000"), too_large → CP-SCR08-5, unknown_currency → CP-SCR08-6; the step does not advance.
- AC-4 (no clients and new client) Given no clients, Then CP-SCR07-1 shows only "New client"; tapping it runs the E02-S02 sub-flow and then continues to the amount step.
- AC-5 (limit) Given 500 open invoices, When `/new`, Then CP-SCR07-17 is sent, the wizard does not start, and an `invoice.limit_reached` log is emitted (EC-BIZ-01).
- AC-6 (copy) Step indicators "Step 1 of 4 · Client" and "Step 2 of 4 · Amount" appear on the respective prompts; each answered prompt is edited to show the chosen value (client name or formatted amount).
- AC-7 (accessibility) Recent-client buttons are truncated at 24 characters with "…"; full names appear in the message text.
- AC-8 (observability) `wizard.started` with `flow=new_invoice`; the start instant is stored in the conversation data for M-1.1.
**Edge cases** — EC-INPUT-09, EC-INPUT-12, EC-BIZ-01, EC-BOT-06 (wizard expiry), EC-BOT-07 (tapping an old client button after an archive → CP-SCR20-2); story-specific: tapping "New client" while a previous `/new` is active restarts the wizard (one active flow per user).
**Tests**
- Integration: `test/integration/new_invoice_client_amount.int.test.ts` — recent clients, new client path, currency override, each error mapping, limit, stale button — covers AC-1 to AC-5, AC-8.
- E2E: `test/e2e/log-invoice.e2e.test.ts` (CE-2) first part.
**Tasks**
- E02-S03-T1 Flow definition and keyboards.
- E02-S03-T2 Recent-clients query in `clients` repo (by latest invoice date via a `clients` interface call to `invoices.lastInvoiceDates(freelancerId)`; no cross-table join).
- E02-S03-T3 Amount step with the error mapping.
- E02-S03-T4 Tests.
**Story DoD** — Default.

### E02-S04 — New-invoice wizard: due-date step

**Story** — As **P1 (Maya, solo freelancer)**, I want to set the due date with one tap or by typing it naturally, so that "overdue" means what my invoice says.
**Context** — F-02 step 4; SCR-09; EC-INPUT-13, EC-TIME-11, EC-TIME-14. Dates are `Temporal.PlainDate` in the freelancer's calendar (DR-11).
**Scope** — In: `platform/time/parseDate.ts` (`parseDueDate(input, todayLocal) → Result<PlainDate, DateError>`), shortcut buttons, step `due_date`, CP-SCR09-1 to CP-SCR09-12. Out: reference and save (E02-S06).
**Acceptance Criteria**
- AC-1 (happy) Given local today Mon 2026-10-05 (Europe/Lisbon), When "In 14 days" is tapped, Then the draft due date is 2026-10-19; "In 7 days" → 2026-10-12; "In 30 days" → 2026-11-04; "End of month" → 2026-10-31; on Sat 2026-10-31 "End of month" → 2026-11-30 (EC-TIME-14).
- AC-2 (typed formats) Given today 2026-10-05, Then "30 Oct" → 2026-10-30; "Oct 30" → 2026-10-30; "30 October 2026" → 2026-10-30; "2026-10-30" → 2026-10-30; "today" → 2026-10-05; "tomorrow" → 2026-10-06; "in 10 days" → 2026-10-15; "5 Jan" → 2027-01-05; "29 Feb 2028" → 2028-02-29; "29 Feb 2027" → unreadable (CP-SCR09-7).
- AC-3 (ambiguity) When "10/30/2026", "30/10/2026", "30.10.2026" or "10-30" is sent, Then CP-SCR09-8 and no change (EC-INPUT-13).
- AC-4 (bounds) Given today 2026-10-05, Then "2025-10-04" (366 days ago) → CP-SCR09-9; "2025-10-05" (365 days ago) → accepted with CP-SCR09-11 (`days=365`); "2028-10-05" (731 days ahead) → CP-SCR09-10; "2028-10-04" (730 days ahead) → accepted.
- AC-5 (already overdue) When "2026-09-28" is sent, Then CP-SCR09-11 with `days=7` is shown before moving on.
- AC-6 (copy) CP-SCR09-1 is shown with buttons CP-SCR09-2 to CP-SCR09-5, hint CP-SCR09-6 and indicator CP-SCR09-12.
- AC-7 (accessibility) Shortcut buttons state the offset in words ("In 14 days"), not only a date.
- AC-8 (observability) `wizard.step` with `step=due_date` and the input kind (`shortcut` / `typed`), never the raw text.
**Edge cases** — EC-INPUT-13, EC-TIME-11, EC-TIME-14, EC-TIME-09 (today = due date is "due today", tested in E03-S01); story-specific: input with trailing punctuation ("30 Oct.") is accepted.
**Tests**
- Unit: `src/platform/time/parseDate.test.ts` — table-driven cases from AC-1 to AC-4 with injected today — covers AC-1 to AC-5.
- Integration: `test/integration/new_invoice_due_date.int.test.ts` — step wiring and copy — covers AC-5, AC-6, AC-8.
- Mutation: `src/platform/time/**` included in the Stryker set (score recorded).
**Tasks**
- E02-S04-T1 Parser with explicit grammar (named months in English, ISO, relative words).
- E02-S04-T2 Shortcut computation with Temporal.
- E02-S04-T3 Step handler and copy; tests.
**Story DoD** — Default.

### E02-S05 — Reminder plan computation (presets, weekdays, send hour, DST)

**Story** — As **P1 (Maya, solo freelancer)**, I want reminders scheduled on sensible weekdays at my morning hour, spaced apart and never opening with a harsh tone, so that clients get reminders at civil times.
**Context** — AD-3; DR-11; DR-12; assumptions 16, 17, 19, 20. Pure functions in `nudging/domain/` (no I/O), used by the wizard preview (E02-S06), by nudge persistence (E04-S01) and by re-planning. Temporal types (S-60, S-61, S-88). Mutation-tested (§14.2 row 11, M-24).
**Scope** — In: `nudging/domain/plans.ts` (preset offsets and step kinds), `nudging/domain/schedule.ts` (`planSchedule(input) → PlannedStep[]`), `nudging/index.ts` exports `previewFirstReminderDate` and `reminderStatusPreview`. Out: persistence, digest, sending.
**Rules (normative for this story)**
1. Presets: `gentle` = [+3 friendly, +10 neutral, +21 final]; `standard` = [+1 friendly, +7 neutral, +14 firm, +30 final]; `firm` = [−3 pre_due, +1 friendly, +7 neutral, +14 firm, +21 final]; `off` = [].
2. Candidate local date = `dueDate + offset`. Saturday or Sunday → the following Monday.
3. Spacing: every step's date ≥ previous step's date + 2 days; if violated, move it to previous + 2, then re-apply rule 2.
4. Earliest date (digest on): the next digest day is today if today is Monday–Friday and the local time is before 18:00, otherwise the next Monday–Friday day; earliest = the first Monday–Friday day after the next digest day. (Digest off: earliest = today if today is Monday–Friday and now + 60 min ≤ today's send instant, else the next Monday–Friday day.)
5. Past collapse: steps whose date is before the earliest date are merged into **one** step on the earliest date, with the step kind of the **first** merged step (never open with a firm tone); later steps keep their kinds and obey rule 3.
6. Send instant = local date at `sendHour:00` in the zone → `Instant` (Temporal `ZonedDateTime`), disambiguation "compatible" [VERIFY-AT-BUILD: option name per Temporal docs].
7. Re-plan input can include `alreadySent: { stepIndex, localDate }[]`; sent steps are kept, and remaining steps start from the next index with spacing measured from the last sent date.
**Acceptance Criteria**
- AC-1 (happy, standard) Given due date Mon 2026-10-19, zone Europe/Lisbon, send hour 10, now 2026-10-05T09:00Z, plan `standard`, Then steps are: friendly Tue 2026-10-20 09:00Z (10:00 WEST), neutral Mon 2026-10-26 10:00Z (10:00 WET after the 2026-10-25 DST change), firm Mon 2026-11-02 10:00Z, final Wed 2026-11-18 10:00Z.
- AC-2 (weekend) Given due date Fri 2026-10-23 and `standard`, Then friendly (+1 = Sat 24th) moves to Mon 2026-10-26, and neutral (+7 = Fri 30th) stays Fri 2026-10-30 (≥ 2 days after Monday).
- AC-3 (spacing) Given `firm` with due date Mon 2026-10-19, Then pre_due Fri 2026-10-16, friendly Tue 2026-10-20, neutral Mon 2026-10-26, firm Mon 2026-11-02, final Mon 2026-11-09 (no two steps less than 2 days apart).
- AC-4 (past collapse) Given due date 2026-09-15 (20 days ago), `standard`, now Mon 2026-10-05 12:00 local, Then exactly two steps: friendly on Tue 2026-10-06 (earliest; merges +1, +7 and +14, with the first merged step's kind) and final on Thu 2026-10-15 (+30); given now 19:00 local, the first step is Wed 2026-10-07.
- AC-5 (DST, 5 zones) For each of Europe/Berlin (2026-03-29, 2026-10-25), America/New_York (2026-03-08, 2026-11-01), Australia/Sydney (2026-04-05, 2026-10-04), America/Sao_Paulo and Asia/Kolkata (no DST in 2026–2028), steps on the Monday after each transition are at 10:00 local, and the UTC instants differ by the expected offset change (EC-TIME-04).
- AC-6 (off and digest-off) Plan `off` → []; with digest off and now 08:30 local on a Tuesday, a past-due invoice's first step is today at 10:00.
- AC-7 (re-plan) Given due date Mon 2026-10-19, `standard`, `alreadySent` = friendly on Tue 2026-10-20, and the plan re-computed on Wed 2026-10-21 at 09:00 local after a pause, Then the remaining steps are neutral Mon 2026-10-26, firm Mon 2026-11-02, final Wed 2026-11-18 (future offsets unchanged); re-computed instead on Wed 2026-10-28 at 09:00 local, neutral (now past) moves to Thu 2026-10-29 (earliest), firm stays Mon 2026-11-02 and final Wed 2026-11-18.
- AC-8 (preview) `previewFirstReminderDate` returns the local date of the first step, or null for `off` and for invoices whose client address is suppressed.
**Edge cases** — EC-TIME-04, EC-TIME-06, EC-TIME-12, EC-MSG-05, EC-BIZ-10 (all steps sent → empty remaining plan); story-specific: due dates on 29 Feb 2028 with +1 offsets → 1 Mar 2028 (Temporal arithmetic).
**Tests**
- Unit: `src/modules/nudging/domain/schedule.test.ts` — AC-1 to AC-8 as table-driven cases with fixed `now` and zone.
- Unit: `src/modules/nudging/domain/plans.test.ts` — preset tables match assumption 19.
- Mutation: `stryker run --mutate "src/modules/nudging/domain/**"` score ≥ 70; surviving mutants listed in PROGRESS.md.
**Tasks**
- E02-S05-T1 Preset table.
- E02-S05-T2 `planSchedule` implementing rules 1–7 with Temporal only.
- E02-S05-T3 Public preview functions in `nudging/index.ts`.
- E02-S05-T4 Tests and Stryker run.
**Story DoD** — Default + mutation score for `nudging/domain` recorded (M-24 baseline).

### E02-S06 — New-invoice wizard: reference, link, reminder plan and save

**Story** — As **P1 (Maya, solo freelancer)**, I want to confirm the reference, optionally add a link, pick how firm reminders are, and save, so that the invoice is tracked and I know when the first reminder would go.
**Context** — F-02 steps 5–6; SCR-10; ENT-Invoice. Publishes `InvoiceChanged{kind:"created"}` on the in-process bus within the save transaction (§12.2); the `nudging` subscriber arrives in E04-S01. M-1.1 is recorded as `entry_duration_ms`.
**Scope** — In: `invoice` migration (all §12.4 columns), `invoices.createInvoice`, `platform/events` bus (sync, same transaction), steps `reference`, `link`, `plan`, `confirm`; CP-SCR10-1 to CP-SCR10-23. Out: nudge rows (E04-S01).
**Acceptance Criteria**
- AC-1 (happy) Given draft Acme Ltd / EUR 1,200.50 / due Mon 2026-10-19, freelancer default plan `standard`, `next_invoice_seq = 7`, When the reference step shows CP-SCR10-1 with "Use INV-007" and it is tapped, Then CP-SCR10-7 shows "Acme Ltd · EUR 1,200.50\nRef INV-007 · due Mon 19 Oct 2026\nReminders: Standard — first on Tue 20 Oct 2026" with buttons CP-SCR10-10, CP-SCR10-11, CP-SCR10-17 and CP-SCR10-23; When "Save invoice" is tapped, Then one `invoice` row exists (`status=open`, `nudge_preset=standard`, `plan_version=1`, `version=1`, `entry_duration_ms` > 0), `next_invoice_seq = 8`, one `InvoiceChanged{created}` event is published, the conversation state is deleted, and CP-SCR10-20 is sent followed by the home keyboard.
- AC-2 (custom reference and duplicate) When "A-2026/15" is sent, Then it is used and `next_invoice_seq` stays unchanged; a 41-character reference → CP-SCR10-3; a reference equal (case-insensitive) to an existing open or paid invoice for the same client → CP-SCR10-4 with CP-SCR10-5 / CP-SCR10-6 (EC-INPUT-10).
- AC-3 (plan picker) When "Reminders: Standard" is tapped, Then CP-SCR10-12 with CP-SCR10-13 to CP-SCR10-16; choosing "Off — just track it" updates the summary line to CP-SCR10-8, and saving sends CP-SCR10-21.
- AC-4 (link) When "Add invoice link" then "https://pay.example.test/inv/7" is sent, Then it is stored in `invoice_url`; "http://x.test", "javascript:alert(1)" or a 501-character URL → CP-SCR10-19 (EC-INPUT-14).
- AC-5 (suppressed client) Given `deliverability.isSuppressed(freelancerId, clientEmail)` returns true (stubbed in the test; real from E05), Then the summary shows CP-SCR10-9 and saving sends CP-SCR10-21.
- AC-6 (double tap and failure) When "Save invoice" is tapped twice within 200 ms, Then exactly one invoice row exists, and the second callback is answered with CP-SCR20-11 (EC-CONC-02); When the DB insert fails, Then CP-SCR10-22 is sent, no invoice or event exists, and the wizard state is kept so "Save invoice" works again.
- AC-7 (accessibility) The summary is plain text with bold amount and date; all buttons are labelled in words.
- AC-8 (observability) `invoice.created` with `invoice_id`, `currency`, `preset`, `entry_duration_ms`; never client names, emails or references.
**Edge cases** — EC-INPUT-02, EC-INPUT-10, EC-INPUT-14, EC-CONC-02; story-specific: the freelancer changes their default plan while a wizard is open → the wizard keeps the plan shown in its summary.
**Tests**
- Integration: `test/integration/new_invoice_save.int.test.ts` — full save, custom reference, duplicate reference, plan off, link validation, suppressed stub, double tap, DB failure (inject failure via a test-only repo wrapper) — covers AC-1 to AC-6, AC-8.
- E2E: `test/e2e/log-invoice.e2e.test.ts` (CE-2) completes `/new` → saved (the /owed assertion is added in E03-S02).
**Tasks**
- E02-S06-T1 `invoice` migration + repo; `platform/events` bus with typed events and transaction-bound dispatch.
- E02-S06-T2 `invoices.createInvoice` (sequence handling, duplicate-reference check).
- E02-S06-T3 Steps, summary rendering (uses `nudging.previewFirstReminderDate`), plan picker, link step.
- E02-S06-T4 Atomic wizard consumption (delete the state row in the save transaction; 0 rows → duplicate tap).
- E02-S06-T5 Tests; CE-2.
**Story DoD** — Default.

### E02-S07 — Invoice card: edit, cancel and delete

**Story** — As **P1 (Maya, solo freelancer)**, I want to open an invoice, fix a typo in the amount or date, cancel it, or delete a mistake, so that the tracker stays accurate.
**Context** — SCR-12 (base card, edit, cancel, delete, conflict, not found); EC-CONC-01, EC-DATA-07, EC-DATA-12, EC-BIZ-09, EC-PAY-13, EC-PERM-02. Changes publish `InvoiceChanged` (`edited`, `cancelled`, `deleted`). Until E04-S01 the reminder status line comes from `nudging.reminderStatusPreview`.
**Scope** — In: `invoices.getInvoiceCard`, `editInvoice` (amount, due date, reference, link) with the `version` check, `cancelInvoice`, `deleteInvoice`; card rendering CP-SCR12-1, CP-SCR12-2, CP-SCR12-7; buttons CP-SCR12-9 (wired in E03-S03), CP-SCR12-14, CP-SCR12-15; More menu with CP-SCR12-16 and CP-SCR12-20; confirms CP-SCR12-17 to CP-SCR12-23; edit CP-SCR12-28 to CP-SCR12-32; CP-SCR12-37. Out: payments (E03), pause/resume/send now/plan change (E04-S04, E05-S05), forward text (E06-S03).
**Acceptance Criteria**
- AC-1 (happy) Given invoice INV-007 (Acme Ltd, EUR 1,200.50, due Mon 2026-10-19, standard), When `v1:inv:<id>` is tapped, Then CP-SCR12-1 shows "Acme Ltd · INV-007\nAmount EUR 1,200.50 · paid EUR 0.00 · owed EUR 1,200.50\nDue Mon 19 Oct 2026 (due in 14 days)\nReminders: next on Tue 20 Oct 2026 (friendly)" with buttons "Paid in full", "Edit", "More".
- AC-2 (edit) When Edit → Amount → "1,350" is sent, Then `amount_minor = 135000`, `version = 2`, `InvoiceChanged{edited}` is published, CP-SCR12-30 is sent, and the card re-renders; editing the due date uses the E02-S04 parser rules; editing the reference applies the E02-S06 duplicate rule.
- AC-3 (conflict) Given the card was rendered at `version = 1` and the invoice is now `version = 2` (edited from another device), When an edit is submitted from the old card, Then CP-SCR12-31 is sent with the fresh card and nothing changes (EC-CONC-01, EC-DATA-12).
- AC-4 (cancel) When More → Cancel invoice → CP-SCR12-17 → "Cancel invoice", Then `status = cancelled`, `cancelled_at` set, `InvoiceChanged{cancelled}` published, CP-SCR12-37 sent; "Keep it" changes nothing (EC-BIZ-09).
- AC-5 (delete) When More → Delete → CP-SCR12-21 → "Delete for good", Then the invoice row and its dependent rows are gone (FK cascade), `InvoiceChanged{deleted}` is published before the delete in the same transaction, CP-SCR12-23 is sent, and an `audit_event` `invoice.deleted` with the invoice ID remains (EC-DATA-07).
- AC-6 (permissions) Given freelancer B's invoice ID in a callback from freelancer A, Then CP-SCR12-32 and no change; the same for a deleted invoice (EC-PERM-02).
- AC-7 (validation and accessibility) Editing the amount below the amount already paid is rejected with CP-SCR13-7 wording (active once payments exist, E03-S04; tested here with a seeded payment row) (EC-PAY-13); destructive confirms put "Keep it" first.
- AC-8 (observability) `invoice.edited` (field names), `invoice.cancelled`, `invoice.deleted` events with IDs only.
**Edge cases** — EC-CONC-01, EC-DATA-07, EC-DATA-12, EC-BIZ-09, EC-PAY-13, EC-PERM-02, EC-BOT-07 (buttons on an old card after delete → CP-SCR12-32).
**Tests**
- Integration: `test/integration/invoice_card.int.test.ts` — render, edits, conflict, cancel, delete, other freelancer, amount below paid — covers AC-1 to AC-8.
- Integration: `test/integration/authz.int.test.ts` — created here and extended by every later epic: for each callback action, replays with another freelancer's IDs and asserts no state change.
**Tasks**
- E02-S07-T1 Card query and renderer.
- E02-S07-T2 Edit sub-flows reusing the parsers; version check in the UPDATE (`where version = $n`).
- E02-S07-T3 Cancel and delete with events and audit.
- E02-S07-T4 `authz.int.test.ts` scaffold; tests.
**Story DoD** — Default.

**Epic-level tests** — CE-1 and CE-2 green; `authz.int.test.ts` green for all E02 callbacks; mutation scores for `platform/money`, `platform/time` and `nudging/domain` ≥ 70.
**Definition of Done**
- All 7 stories DONE with evidence lines.
- `check` green; CI green; staging deployed.
- Manual on staging (product owner): (1) log an invoice for a new client in ≤ 60 s using shortcuts; (2) type "1.200,50" and see "EUR 1,200.50" in the summary; (3) edit the due date from the card. Recorded.
- PROGRESS.md, DECISIONS.md updated; Amendments log updated if any AC changed.
**Session handoff checklist** — §18.5 summary, plus: measured M-1.1 on the manual run; Stryker scores; preconditions for E03 (none).

## Epic 03 — Owed overview and payments

**Goal** — A freelancer sees who owes what (per currency, overdue first) and records full or part payments in one or two taps. This is the tracker-only closed beta.
**Personas served** P1, P2
**Depends on** E02: invoices, cards, money formatting, plan preview.
**Preconditions for the session** — PROGRESS.md shows E02 DONE; `check` green; staging has 10+ real invoices from the E02 manual check (fine to keep). `[VERIFY-AT-BUILD]`: none new.
**Scope** — In: stories below. Out: reminders (E04, E05).
**Flows and screens implemented** F-03, F-04; SCR-11, SCR-12 (payment buttons), SCR-13.
**Data and API touched** ENT-Payment (new); ENT-Invoice (status transitions); `invoices.listOwed`, `countOverdue`, `recordPayment`, `undoPayment`, `reopenInvoice`.
**Risks specific to this epic** R-03 (a reminder after payment; groundwork for the invariant).

### E03-S01 — Invoice status and balance rules

**Story** — As **P1 (Maya, solo freelancer)**, I want "overdue", "due today" and balances computed from my own calendar and my recorded payments, so that the numbers I see match reality.
**Context** — Derived values (§12.4): `balance = amount − Σ payments`; overdue = open ∧ local today > due date (EC-TIME-02, EC-TIME-09). Pure domain in `invoices/domain/status.ts`, mutation-tested.
**Scope** — In: `payment` migration (§12.4), `status.ts` (`balance`, `dueState(dueDate, todayLocal) → overdue(n) | due_today | due_tomorrow | due_in(n)`), `countOverdue` real implementation (replaces the E01-S03 stub), status text rendering (CP-SCR11-6 to CP-SCR11-10). Out: UI lists (E03-S02).
**Acceptance Criteria**
- AC-1 (happy) Given an invoice due 2026-10-19 for EUR 1,200.50 with payments EUR 200.00 and EUR 300.50, Then `balance` = EUR 700.00 and `dueState` at local today 2026-10-21 = `overdue(2)` rendered as CP-SCR11-6 "2 days overdue".
- AC-2 (boundaries) `dueState` at 2026-10-19 = `due_today` (CP-SCR11-8), at 2026-10-20 = `overdue(1)` (CP-SCR11-7 "1 day overdue"), at 2026-10-18 = `due_tomorrow` (CP-SCR11-10), at 2026-10-14 = `due_in(5)` (CP-SCR11-9).
- AC-3 (zones) Given the freelancer in Pacific/Auckland and the instant 2026-10-19T11:30:00Z (00:30 on 20 Oct local), Then the invoice due 2026-10-19 is `overdue(1)`; for America/Los_Angeles at the same instant (04:30 on 19 Oct local) it is `due_today` (EC-TIME-02).
- AC-4 (count) Given 3 open invoices of which 2 are overdue, 1 paid overdue invoice and 1 cancelled overdue invoice, Then `countOverdue` = 2 and `/start` for this returning user shows CP-SCR01-5 with `overdue_count=2`.
- AC-5 (invariants) The domain rejects a payment list whose sum exceeds the amount (`InvariantError`); balances are BigInt minor units.
- AC-6 (copy) Status texts render exactly as CP-SCR11-6 to CP-SCR11-10.
- AC-7 (accessibility) n/a (domain).
- AC-8 (observability) n/a (pure functions).
**Edge cases** — EC-TIME-02, EC-TIME-09, EC-DATA-02 (singular "1 day"), EC-PAY-13 (invariant).
**Tests**
- Unit: `src/modules/invoices/domain/status.test.ts` — AC-1 to AC-3, AC-5 as table-driven cases.
- Integration: `test/integration/overdue_count.int.test.ts` — AC-4.
- Mutation: `invoices/domain/**` score ≥ 70.
**Tasks**
- E03-S01-T1 `payment` migration + repo.
- E03-S01-T2 `status.ts` functions with Temporal.
- E03-S01-T3 `countOverdue` query (open invoices with `due_date < local today`, computed in SQL with the date passed as a parameter from `platform/time`).
- E03-S01-T4 Tests; Stryker.
**Story DoD** — Default + mutation score for `invoices/domain` recorded.

### E03-S02 — "Who owes me" list with totals per currency

**Story** — As **P2 (Arjun, high-volume contractor)**, I want one message that shows what I'm owed per currency, overdue first, so that my Monday review takes seconds.
**Context** — F-03; SCR-11; EC-DATA-01/03/04/05, EC-PAY-14, EC-BOT-11. Totals via `formatTotals` (E02-S01). Budget: ≤ 300 ms server time at 500 open invoices (§12.9).
**Scope** — In: `invoices.listOwed(freelancerId, page)` (groups Overdue / Due in the next 7 days / Later; sort by due date, reference, ID; page size 8), `/owed` and the home button, paging by editing the message, reference buttons opening the card, markers CP-SCR11-17 and CP-SCR11-18 (the latter becomes reachable in E05), CP-SCR11-1 to CP-SCR11-16. Out: payment actions (E03-S03).
**Acceptance Criteria**
- AC-1 (happy) Given open invoices: A (Acme, EUR 1,200.00, due 2026-09-28), B (Bolt, USD 4,000.00, due 2026-10-02, USD 1,500.00 paid), C (Cora, INR 180,000.00, due 2026-10-09), D (Dune, EUR 950.00, due 2026-11-20), today 2026-10-05, When `/owed`, Then the message starts with CP-SCR11-1 "You're owed INR 180,000.00 + USD 2,500.00 + EUR 2,150.00 across 4 open invoices." (ordered by minor amount descending), followed by "Overdue" with A (7 days overdue) and B (3 days overdue, showing the balance USD 2,500.00), "Due in the next 7 days" with C (due in 4 days), "Later" with D; reference buttons in the same order; CP-SCR11-15 at the end.
- AC-2 (empty) Given no open invoices, Then CP-SCR11-11 (EC-DATA-01).
- AC-3 (paging) Given 17 open invoices, Then page 1 shows 8 with "Next" only, page 2 shows 8 with "Previous" and "Next", page 3 shows 1 with "Previous"; CP-SCR11-14 "Page 2 of 3"; paging edits the same message; with exactly 8 invoices there is no "Next" (EC-DATA-03, EC-DATA-04).
- AC-4 (stability) Given two invoices with the same due date, Then they are ordered by reference, then ID (EC-DATA-05); given an invoice paid between page 1 and page 2, Then page 2 is re-queried and shows no duplicate; a page callback beyond the end renders the last page.
- AC-5 (singular and markers) Given one open invoice, Then CP-SCR11-1 reads "…across 1 open invoice." (EC-DATA-02); given an invoice with `nudges_paused_reason = freelancer`, Then its line ends with CP-SCR11-17.
- AC-6 (limits and performance) Given 500 open invoices, Then each page's message is under 1,500 characters and `listOwed` completes in ≤ 300 ms with ≤ 4 queries (asserted with the query counter) (EC-BOT-11).
- AC-7 (failure and accessibility) Given the DB query fails, Then CP-SCR11-16; all figures appear in the message text (not only in buttons), and reference buttons show references, not emoji.
- AC-8 (observability) `owed.viewed` with `page`, `count`, `duration_ms`.
**Edge cases** — EC-DATA-01 to EC-DATA-05, EC-PAY-14 (never summed across currencies), EC-BOT-11.
**Tests**
- Integration: `test/integration/owed.int.test.ts` — AC-1 to AC-8 with factories and a fake clock; performance test with 500 invoices.
- E2E: `test/e2e/log-invoice.e2e.test.ts` (CE-2) adds: after saving, `/owed` shows the invoice and correct totals.
**Tasks**
- E03-S02-T1 `listOwed` query (paid sums via a grouped subquery on `payment`, one round trip) + grouping in the domain.
- E03-S02-T2 Renderer and paging keyboards.
- E03-S02-T3 Tests; CE-2 extension.
**Story DoD** — Default.

### E03-S03 — Mark paid in full, with undo and reopen

**Story** — As **P1 (Maya, solo freelancer)**, I want one tap to mark an invoice paid, with a short undo in case I tapped the wrong one, so that the tracker is right and reminders stop.
**Context** — F-04; SCR-13; EC-CONC-02, EC-CONC-07, EC-BOT-07, EC-PERM-02. Publishes `InvoiceChanged{paid}` / `{reopened}` (the nudging subscriber in E04-S01 cancels or re-plans nudges).
**Scope** — In: `invoices.recordPayment(kind: "full")`, `undoPayment` (≤ 15 min after `recorded_at`), `reopenInvoice`, card button CP-SCR12-9, CP-SCR13-1 to CP-SCR13-4, CP-SCR13-11 to CP-SCR13-13, audit events. Out: part payments (E03-S04); digest "Paid {ref}" and client-claim buttons (they call the same function in E04-S03 and E05-S03).
**Acceptance Criteria**
- AC-1 (happy) Given INV-007 open with balance EUR 1,200.50, When "Paid in full" is tapped at 10:00:00, Then a `payment` row (`kind=full`, `amount_minor=120050`) exists, `status=paid`, `paid_at=10:00:00`, `InvoiceChanged{paid}` is published, and the card message is edited to CP-SCR13-1 with the "Undo" button CP-SCR13-2.
- AC-2 (undo) When "Undo" is tapped at 10:14:59, Then the payment row is deleted, `status=open`, `paid_at=null`, `InvoiceChanged{reopened}` is published, and CP-SCR13-3 is shown with `date` = the next reminder date (or the card's status line if the plan is off); at 10:15:00 → CP-SCR13-4 and no change.
- AC-3 (reopen) Given a paid invoice, its card shows CP-SCR13-11; tapping it deletes the latest `full` payment, sets `open`, publishes `reopened` and shows CP-SCR13-12.
- AC-4 (double tap) When "Paid in full" is tapped twice within 200 ms, Then exactly one payment row exists and the second callback is answered with CP-SCR13-13 (EC-CONC-02).
- AC-5 (stale and permissions) Given the invoice was cancelled or already paid, When the old card's "Paid in full" is tapped, Then CP-SCR13-13 or CP-SCR12-32 and no change (EC-BOT-07); another freelancer's invoice ID → CP-SCR12-32 (EC-PERM-02; added to `authz.int.test.ts`).
- AC-6 (copy) Exact CP lines as listed; amounts formatted per §10.5.
- AC-7 (accessibility) The Undo button label is the word "Undo"; the acknowledgement states the amount in text.
- AC-8 (observability) `payment.recorded` (`kind`, IDs), `payment.undone`, `invoice.reopened`; audit events `invoice.paid`, `invoice.reopened` with actor `freelancer`.
**Edge cases** — EC-CONC-02, EC-CONC-07 (a nudge in `sending` state when Paid is tapped: documented here, and tested in E05-S02 once sending exists), EC-BOT-07, EC-PERM-02.
**Tests**
- Integration: `test/integration/payment_full.int.test.ts` — AC-1 to AC-5, AC-8 with a fake clock at 10:14:59 / 10:15:00.
- Integration: extend `test/integration/authz.int.test.ts` for `pay`, `undo`, `reopen`.
**Tasks**
- E03-S03-T1 `recordPayment` / `undoPayment` / `reopenInvoice` with events and audit; the idempotency check uses `SELECT … FOR UPDATE` on the invoice row plus the status check.
- E03-S03-T2 Handlers and card button wiring; Undo expiry via `recorded_at`.
- E03-S03-T3 Tests.
**Story DoD** — Default.

### E03-S04 — Record a part payment

**Story** — As **P2 (Arjun, high-volume contractor)**, I want to record a partial payment and see the new balance, so that reminders ask for the right amount.
**Context** — F-04 step 2; EC-PAY-13; CP-EML-15 (reminders mention the paid part, E05-S01).
**Scope** — In: "Part payment" button (CP-SCR12-10), sub-flow `part_payment`, CP-SCR13-5 to CP-SCR13-10, parser reuse (E02-S01) with the invoice's currency fixed. Out: payment history view (parking lot).
**Acceptance Criteria**
- AC-1 (happy) Given USD 4,000.00 open with no payments, When "Part payment" then "1,500" is sent, Then a `payment` (`kind=part`, 150000) exists, the invoice stays `open`, `InvoiceChanged{part_paid}` is published, and CP-SCR13-8 shows "Recorded USD 1,500.00 from Bolt. Still owed: USD 2,500.00. Reminders will mention the new balance."
- AC-2 (equals balance) When "2,500" is then recorded, Then CP-SCR13-9 then CP-SCR13-1 are shown, `status=paid`, and the payment is stored as `kind=part` with the invoice marked paid (sum equals amount).
- AC-3 (over balance) When "2,500.01" is sent against a balance of USD 2,500.00, Then CP-SCR13-7 and no payment (EC-PAY-13).
- AC-4 (validation) "abc" → CP-SCR13-6; "0" → CP-SCR08-3; "100 EUR" on a USD invoice → CP-SCR13-10; "100 USD" → accepted as USD.
- AC-5 (permissions and stale) Another freelancer's invoice → CP-SCR12-32; a paid invoice's old "Part payment" button → CP-SCR13-13.
- AC-6 (copy) Exact CP lines; balance formatting per §10.5.
- AC-7 (accessibility) The prompt states the current balance in text (CP-SCR13-5).
- AC-8 (observability) `payment.recorded` with `kind=part` and IDs.
**Edge cases** — EC-PAY-13, EC-INPUT-09, EC-INPUT-12, EC-BOT-07, EC-PERM-02; story-specific: two part payments submitted concurrently that together exceed the balance → the second is rejected with CP-SCR13-7 (row lock on the invoice).
**Tests**
- Integration: `test/integration/payment_part.int.test.ts` — AC-1 to AC-5, concurrency.
- Unit: extend `invoices/domain/status.test.ts` for sums equal to the amount.
**Tasks**
- E03-S04-T1 Sub-flow and handler.
- E03-S04-T2 `recordPayment(kind: "part")` with the balance check under `FOR UPDATE`.
- E03-S04-T3 Tests.
**Story DoD** — Default.

**Epic-level tests** — CE-1, CE-2 (with the /owed assertion) green; `authz.int.test.ts` covers all E02–E03 callbacks; full regression.
**Definition of Done**
- All 4 stories DONE with evidence lines.
- `check` green; CI green; staging deployed.
- **Closed beta 1**: 5–10 invited freelancers onboarded on production-like staging, or production with `NUDGES_LIVE=off` if E08-S03 was pulled forward. Otherwise staging only; the product owner decides and records it.
- Manual on staging: (1) /owed with 3 currencies shows correct totals; (2) Paid in full + Undo within 15 min; (3) part payment updates the balance. Recorded.
- PROGRESS.md, DECISIONS.md updated.
**Session handoff checklist** — §18.5 summary, plus: beta invitations sent (if any) and feedback captured in PROGRESS.md "Parking lot"; preconditions for E04 (none).

## Epic 04 — Nudge scheduling and evening digest

**Goal** — Every open invoice has persisted reminder rows that follow its plan and react to every change. Freelancers get reliable Telegram notices and an evening digest that announces tomorrow's reminders and lets them hold any. No email is sent yet.
**Personas served** P1 (P2 indirectly)
**Depends on** E03: payments and `InvoiceChanged` events for paid / part_paid / reopened; E02-S05 plan computation.
**Preconditions for the session** — PROGRESS.md shows E03 DONE; `check` green. `RUN_SCHEDULER=true` in staging. `[VERIFY-AT-BUILD]`: Telegram flood limits (S-08) and grammY auto-retry options (S-56) unchanged.
**Scope** — In: stories below. Out: sending emails, client pages, send-now (E05).
**Flows and screens implemented** F-05 steps 1–2, F-06 (pause, resume, change plan); SCR-12 (status lines and controls), SCR-14, SCR-15 (outbox infrastructure; individual notices are added by the stories that raise them).
**Data and API touched** ENT-Nudge, ENT-Notification (new); `nudging.planForInvoice` (event handler), `buildDigest`, `holdNudge`, `runSchedulerTick` (digest phase), `notifications.enqueue`, `runDispatchTick`.
**Risks specific to this epic** R-03, R-09 (scheduler drift).

### E04-S01 — Nudge rows lifecycle and re-planning on invoice events

**Story** — As **P1 (Maya, solo freelancer)**, I want reminders to follow my invoice automatically when I edit it, pay it, cancel it or pause it, so that I never have to manage reminders by hand.
**Context** — DR-07 state machine; §12.4 ENT-Nudge transitions; E02-S05 `planSchedule`; `InvoiceChanged` events from `invoices` handled synchronously in the same transaction (§12.2). Suppressed addresses (E05) make nudges "waiting" rather than planned; until then `isSuppressed` returns false.
**Scope** — In: `nudge` migration, `nudging/domain/transitions.ts`, `nudging.planForInvoice(event)` subscriber wired in `src/main/wire.ts`, `nudging.replanAllForFreelancer(freelancerId)` (used by settings in E07-S01), `nudging.reminderStatus(invoiceId)` (replaces the preview-based status on the card: CP-SCR12-2, CP-SCR12-3, CP-SCR12-6, CP-SCR12-7, CP-SCR12-44). Out: announcements (E04-S03), sending (E05-S02).
**Acceptance Criteria**
- AC-1 (happy) Given INV-007 created (due Mon 2026-10-19, `standard`, now 2026-10-05, Europe/Lisbon, digest on), When `InvoiceChanged{created}` is handled, Then 4 `nudge` rows exist with `plan_version=1`, steps 0–3, kinds friendly/neutral/firm/final, `scheduled_for` equal to E02-S05 AC-1's instants, `status=planned`, `announced_at=null`.
- AC-2 (edit re-plan) When the due date is edited to 2026-10-26, Then all `planned`/`announced` rows of plan 1 become `cancelled` (`cancel_reason=superseded`), 4 new rows with `plan_version=2` are created from the new date, and `sent` rows (none yet) are untouched; the invoice's `plan_version` = 2 (same transaction).
- AC-3 (terminal events) `paid` → all `planned`/`announced` rows `cancelled` with reason `paid`; `cancelled` → reason `cancelled`; `deleted` → rows removed by cascade after a `nudge.cancelled` log per row with reason `deleted`; `part_paid` → no change (amounts are rendered at send time).
- AC-4 (reopen) Given a paid invoice with 1 sent and 3 cancelled rows, When `reopened` (now 2026-10-28 09:00 local), Then a new plan version is created with the remaining steps re-planned per E02-S05 rule 7 (sent step kept as history).
- AC-5 (digest off) Given the freelancer has `digest_enabled=false`, Then new rows are created with `announced_at = created_at` (DR-12 exception).
- AC-6 (state machine) `transitions.ts` allows exactly the transitions in §12.4; any other transition throws `InvariantError` and is covered by a table-driven test of all 8 × 8 pairs.
- AC-7 (status line and finished plans) The card shows CP-SCR12-2 with the next `planned`/`announced` row's local date and step name; with all steps sent it shows CP-SCR12-6 (`n=4`) and the invoice stays in /owed (EC-BIZ-10); plan off → CP-SCR12-7.
- AC-8 (observability) `nudge.planned` (count, plan_version), `nudge.cancelled` (reason) per affected row; no PII.
**Edge cases** — EC-BIZ-10, EC-TIME-10 (re-plan on settings change via `replanAllForFreelancer`), EC-CONC-05 (re-planning runs inside the invoice transaction, which holds the invoice row lock), story-specific: an edit that does not change the due date, preset or pause state (e.g. reference only) does not re-plan.
**Tests**
- Unit: `src/modules/nudging/domain/transitions.test.ts` — AC-6.
- Integration: `test/integration/nudge_lifecycle.int.test.ts` — AC-1 to AC-5, AC-7, AC-8.
- Mutation: `nudging/domain/**` stays ≥ 70.
**Tasks**
- E04-S01-T1 `nudge` migration (constraints and partial indexes of §12.4).
- E04-S01-T2 Transition table and guard.
- E04-S01-T3 Event subscriber: create, supersede, cancel, re-plan; `replanAllForFreelancer`.
- E04-S01-T4 `reminderStatus` and card integration.
- E04-S01-T5 Tests.
**Story DoD** — Default.

### E04-S02 — Notification outbox and dispatcher

**Story** — As **P1 (Maya, solo freelancer)**, I want every notice about my reminders and clients to reach me once, even if Telegram hiccups, so that I can trust what the bot tells me.
**Context** — ENT-Notification; INT-telegram retry and 403 rules (§12.7); flood limits and `retry_after` (S-08, S-09); EC-BOT-04, EC-BOT-08, EC-NET-09, EC-INT-01, EC-INT-03. Notices are rendered at dispatch from IDs (no PII in the outbox).
**Scope** — In: `notification` migration, `notifications.enqueue(tx, { kind, freelancerId, refs, dedupeKey })`, `runDispatchTick(now)` (claims ≤ 50 due rows with `SKIP LOCKED`, sends sequentially, backoff 1 min / 5 min / 30 min / 2 h / 6 h, drop after 24 h), a renderer registry keyed by `kind`, the loop in `src/main/loops.ts`, `bot_blocked_at` handling and clearing. Out: specific notice kinds (added with their stories).
**Acceptance Criteria**
- AC-1 (happy) Given a `test_notice` kind registered, When enqueued inside a committed transaction and the dispatcher ticks, Then exactly one `sendMessage` is recorded for the freelancer's chat, the row is `sent` with `telegram_message_id`, and `sent_at` = now.
- AC-2 (rollback) Given `enqueue` inside a transaction that rolls back, Then no row exists and nothing is sent.
- AC-3 (dedupe) Given the same `dedupe_key` enqueued twice, Then one row exists (unique constraint; the second enqueue is a no-op).
- AC-4 (429 and 5xx) Given the fake API returns 429 with `retry_after: 7`, Then the row's `next_attempt_at` = now + 7 s and `attempt_count` = 1; given 5xx 5 times, Then attempts follow 1 min, 5 min, 30 min, 2 h, 6 h (±20% jitter, seeded), and the row is `dropped` once 24 h have passed since `created_at`, with a `notification.dropped` log (EC-NET-09, EC-INT-03).
- AC-5 (blocked bot) Given the fake API returns 403 "Forbidden: bot was blocked by the user", Then the row is `dropped`, `freelancer.bot_blocked_at` = now, and other pending rows for that freelancer are dropped without calls; When the freelancer later sends any message, Then `bot_blocked_at` is cleared (EC-BOT-08).
- AC-6 (concurrency) Given two dispatchers ticking at the same time over 20 due rows, Then each row is sent exactly once (EC-CONC-05).
- AC-7 (copy) Renderers use CP IDs only; a renderer referencing a missing catalogue ID fails the unit test for that kind.
- AC-8 (observability) `notification.sent` (kind, latency from `created_at`), `notification.retry` (kind, attempt), `notification.dropped` (kind, reason).
**Edge cases** — EC-BOT-04, EC-BOT-08, EC-NET-09, EC-INT-01, EC-INT-03, EC-CONC-05; story-specific: a notification whose referenced invoice was deleted before dispatch is `dropped` with reason `subject_gone`.
**Tests**
- Integration: `test/integration/outbox.int.test.ts` — AC-1 to AC-6, AC-8, subject-gone case.
- Unit: `src/modules/notifications/domain/backoff.test.ts` — schedule and jitter bounds.
**Tasks**
- E04-S02-T1 `notification` migration + repo.
- E04-S02-T2 `enqueue` (insert on conflict do nothing).
- E04-S02-T3 Dispatcher tick with error classification and backoff.
- E04-S02-T4 Loop runner (`await tick(); await sleep()`), stopped on SIGTERM.
- E04-S02-T5 Tests.
**Story DoD** — Default.

### E04-S03 — Evening digest with hold and mark-paid

**Story** — As **P1 (Maya, solo freelancer)**, I want a message at 18:00 listing tomorrow's reminders, with a Hold button for each, so that nothing goes to a client without my knowing.
**Context** — F-05 steps 1–2; SCR-14; DR-12 (eligible to send only if announced ≥ 12 h before the send instant); EC-TIME-12, EC-CONC-05, EC-CONC-08, EC-BOT-07. The digest is a `digest` notification. `announced_at` is written for the listed nudges **only when Telegram confirms delivery** (in the dispatcher's success transaction).
**Scope** — In: digest phase of `nudging.runSchedulerTick(now)` (on Monday–Friday, freelancers whose local time is ≥ 18:00 and whose `digest:<freelancer>:<localDate>` key does not yet exist, with planned nudges scheduled for the next working day, or pending client-paid claims; only freelancers for whom sending is live per `NUDGES_LIVE` (`on`, or listed in `NUDGES_ALLOWLIST`) get digests, so beta users without reminders never see announcements), `buildDigest`, digest renderer (CP-SCR14-1 to CP-SCR14-3, CP-SCR14-11), per-line buttons Hold (CP-SCR14-4) and Paid (CP-SCR14-5, calls E03-S03), hold handling (CP-SCR14-9, CP-SCR14-10, CP-SCR14-12), "Waiting for you" section (CP-SCR14-6 to CP-SCR14-8, reachable from E05-S03). Out: sending (E05-S02).
**Acceptance Criteria**
- AC-1 (happy) Given two nudges due Tue 2026-10-20 10:00 local (INV-007 friendly; INV-009 neutral) and the fake clock at Mon 2026-10-19 18:00:30 local, When the scheduler and dispatcher tick, Then one message is sent: CP-SCR14-1 with `count=2`, two CP-SCR14-2 lines ordered by client name, CP-SCR14-3, and buttons "Hold INV-007", "Paid INV-007", "Hold INV-009", "Paid INV-009"; both nudges get `status=announced` and `announced_at` = the delivery time.
- AC-2 (not delivered) Given Telegram returns 5xx until 22:00 local and succeeds at 22:05 local, Then `announced_at` = 22:05, which is less than 12 h before 10:00, so the nudges are not eligible and are re-planned to the next send window after the next digest (Wed 10:00; the digest on Tue 18:00 lists them again); a `digest.late` log is emitted.
- AC-3 (working days) Digests are sent Monday to Friday only, and each lists the reminders scheduled for the next working day: given Fri 2026-10-23 18:00 local, the digest lists nudges scheduled for Mon 2026-10-26; on Saturday and Sunday no digest is sent (EC-TIME-12).
- AC-4 (hold) When "Hold INV-007" is tapped, Then that nudge becomes `skipped`, its line is edited to CP-SCR14-9 with the date of the next step, or CP-SCR14-10 if it was the last; the remaining plan is unchanged; tapping Hold after the nudge was sent → toast CP-SCR14-12 (EC-CONC-08); tapping an old digest's button after a re-plan → CP-SCR20-2 (EC-BOT-07).
- AC-5 (overflow and empty) Given 13 nudges due tomorrow, Then 10 lines with buttons and CP-SCR14-11 with `more=3`; given none and no pending claims, Then no digest is sent.
- AC-6 (dedupe and overlap) Given two scheduler processes ticking at 18:00:30 and 18:00:31, Then exactly one digest notification exists for that freelancer and date (EC-CONC-05).
- AC-7 (accessibility) Each line carries all facts in text (time, client, reference, balance, days overdue, step name); buttons carry the reference so they stay unambiguous without seeing the line.
- AC-8 (observability) `digest.enqueued` (count), `digest.delivered` (count, delay_s), `nudge.announced` (count), `nudge.held`.
**Edge cases** — EC-TIME-12, EC-CONC-05, EC-CONC-08, EC-BOT-07, EC-BOT-08 (blocked bot → no digest → nudges never announced → never sent automatically; the freelancer sees them when they unblock and open /owed), EC-DATA-02 (count 1).
**Tests**
- Integration: `test/integration/digest.int.test.ts` — AC-1 to AC-6, AC-8 with a fake clock and fake Telegram.
- Unit: `src/modules/nudging/domain/eligibility.test.ts` — `unannounced_is_not_sendable`, `announced_less_than_12h_before_is_not_sendable`, `announced_16h_before_is_sendable`, `digest_off_skips_12h_rule` (the function takes `digestEnabled` as input; DR-12 exception), `send_now_bypasses_announcement` (the last one is exercised in E05-S05).
**Tasks**
- E04-S03-T1 Digest selection query (per-freelancer local 18:00 computed via `platform/time`; a batch of 50 freelancers per tick).
- E04-S03-T2 Digest renderer and keyboards; dispatcher success hook that sets `announced_at` for the IDs in `refs`.
- E04-S03-T3 Late-delivery re-plan rule; eligibility function.
- E04-S03-T4 Hold handler.
- E04-S03-T5 Tests.
**Story DoD** — Default + manual: receive a real digest on staging at 18:00 local and hold one reminder; recorded.

### E04-S04 — Pause, resume and change a reminder plan

**Story** — As **P1 (Maya, solo freelancer)**, I want to pause reminders for one invoice when the client tells me payment is coming, resume them later, or switch to a gentler or firmer plan, so that each client gets the right amount of chasing.
**Context** — F-06 steps 1, 2 and 4; SCR-12; events `paused` / `resumed` / `plan_changed` from `invoices` handled by E04-S01's subscriber (re-plan from today per E02-S05 rules 4, 5, 7).
**Scope** — In: `invoices.pauseReminders`, `resumeReminders`, `changePlan`; card buttons CP-SCR12-11, CP-SCR12-12, CP-SCR12-41; acknowledgements CP-SCR12-34, CP-SCR12-35, CP-SCR12-38, CP-SCR12-39; plan picker CP-SCR12-42, CP-SCR12-43 (reusing CP-SCR10-13 to CP-SCR10-16); status CP-SCR12-3. Out: resume refusal after a client opt-out (CP-SCR12-36, CP-SCR12-40 in E05-S04); send now (E05-S05).
**Acceptance Criteria**
- AC-1 (pause) Given INV-007 with 3 planned nudges, When "Pause reminders" is tapped, Then `nudges_paused_reason=freelancer`, the 3 nudges are `cancelled` (`paused`), the toast CP-SCR12-38 and the card with CP-SCR12-3 and the "Resume reminders" button are shown.
- AC-2 (resume) Given a paused invoice, When "Resume reminders" is tapped on Wed 2026-10-28 at 09:00 local, Then the pause is cleared, a new plan version is created per E02-S05 rule 7 (e.g. first remaining step Thu 2026-10-29), CP-SCR12-35 shows `date=Thu 29 Oct 2026`, and the toast CP-SCR12-39 appears.
- AC-3 (change plan) When More → "Change reminder plan" → "Gentle — 3 reminders", Then `nudge_preset=gentle`, remaining unsent steps are replaced by the gentle plan's remaining steps (sent history kept), and CP-SCR12-43 is shown.
- AC-4 (plan off) Choosing "Off — just track it" cancels all remaining nudges and the card shows CP-SCR12-7.
- AC-5 (permissions and stale) Other freelancer → CP-SCR12-32; pausing an already paused invoice from an old card → re-render with CP-SCR20-2.
- AC-6 (copy) Exact CP lines as listed.
- AC-7 (accessibility) Pause and Resume are distinct labelled buttons (never a toggle icon).
- AC-8 (observability) `invoice.reminders_paused` / `resumed` / `plan_changed` with IDs and the preset.
**Edge cases** — EC-BOT-07, EC-PERM-02, EC-TIME-12 (resume respects spacing and weekends).
**Tests**
- Integration: `test/integration/pause_resume_plan.int.test.ts` — AC-1 to AC-5, AC-8.
- Integration: extend `authz.int.test.ts` for `pause`, `resume`, `plan_set`.
**Tasks**
- E04-S04-T1 Invoice service functions + events.
- E04-S04-T2 Card buttons and plan picker reuse.
- E04-S04-T3 Tests.
**Story DoD** — Default.

**Epic-level tests** — New integration scenario `test/integration/plan_to_digest.int.test.ts`: create an invoice → rows planned → digest delivered at 18:00 → rows announced → hold one → pay the other → all cancelled. CE-1 and CE-2 still green; full regression.
**Definition of Done**
- All 4 stories DONE with evidence lines.
- `check` green (mutation job green for `nudging/domain`); CI green; staging deployed with `RUN_SCHEDULER=true`.
- Manual on staging: digest received at 18:00 local, Hold works, pausing an invoice removes it from the next digest. Recorded.
- PROGRESS.md, DECISIONS.md updated.
**Session handoff checklist** — §18.5 summary, plus: observed digest delivery delay on staging; preconditions for E05 (sending subdomain with SPF, DKIM and DMARC verified in Resend; `RESEND_WEBHOOK_SECRET`; `LINK_SIGNING_KEY`; `EMAIL_RECIPIENT_OVERRIDE` set in staging; OQ-07 answered or its default applied).

## Epic 05 — Client email nudges with baseline safety

**Goal** — Announced reminders go to clients by email exactly once, with honest provenance, the correct balance and payment details. Clients can say "I've already paid", stop reminders, or unsubscribe in one click, and bounces and complaints suppress addresses automatically.
**Personas served** P3, P1, P4
**Why the safety ships here** — The product sends email to people who never signed up for it (clients). Per the skill's guardrail, baseline safety (opt-out, "already paid", caps, bounce and complaint handling, audit) ships in the same epic as the core loop, not later.
**Depends on** E04: nudge rows, announcements, outbox; E01: email gateway, encryption; E03: payments.
**Preconditions for the session** — PROGRESS.md shows E04 DONE; `check` green. In staging: sending subdomain verified in Resend with SPF, DKIM and DMARC (OQ-07 default applied), `RESEND_WEBHOOK_SECRET` set and a Resend webhook pointing to `/webhooks/resend` with the events of §12.7, `LINK_SIGNING_KEY` set, `EMAIL_RECIPIENT_OVERRIDE` set, `NUDGES_LIVE=allowlist` with the product owner's Telegram ID. `[VERIFY-AT-BUILD]`: Resend idempotency semantics (S-27), rate limit (S-73), webhook event names (S-29) and payload field names for the message ID; Svix verification via `resend.webhooks.verify` in SDK 6.31.0 (S-89); that Resend's DKIM signature covers custom `List-Unsubscribe` headers (check a received test email's raw headers; S-47 requires it).
**Scope** — In: stories below. Out: auto-suspension and admin (E06).
**Flows and screens implemented** F-05 steps 3–5, F-06 step 3 (send now), F-07, F-08, F-09 (suppression part); SCR-15, SCR-21, SCR-22, SCR-23; email CP-EML-1 to CP-EML-22.
**Data and API touched** ENT-Nudge (sending states), ENT-Suppression, ENT-EmailEvent (new tables); API-02 to API-08; INT-email.
**Risks specific to this epic** R-01 (deliverability), R-03, R-04 (email law classification).

### E05-S01 — Compose reminder emails and signed action links

**Story** — As **P3 (Sam, the client-side payer)**, I want a reminder that names the freelancer, the reference, the amount still owed and how to pay, with an obvious way to say it's paid or to stop, so that I can act in one glance and trust the email.
**Context** — §10.3 tone ladder; §11.25 copy; DR-02 (From / Reply-To); DR-13 (RFC 8058 headers, S-47); client action tokens (§12.5); EC-MSG-09, EC-MSG-12, EC-MSG-13, EC-INPUT-05, EC-SEC-02, EC-SEC-11, EC-TIME-09. Pure composition in `nudging/domain/compose.ts` and `tokens.ts` (no I/O).
**Scope** — In: `composeReminder({ freelancer, client, invoice, paidMinor, nudge, sendLocalDate, baseUrl }) → { from, replyTo, to, subject, text, html, headers, tags }`, `tokens.ts` (`issue(nudgeId, action, exp)`, `verify(token) → Result`), HTML template with the §9.2 tokens (inline CSS, no images), plain-text part. Out: sending (E05-S02).
**Acceptance Criteria**
- AC-1 (happy, friendly) Given freelancer "Maya Costa" (no business, reply-to maya@example.test, payment instructions "Bank transfer: IBAN PT50 …"), client "Acme Ltd" with contact "Sam" at ap@acme.test, invoice INV-007 EUR 1,200.50 due Mon 2026-10-19, no payments, nudge `friendly` sent locally Tue 2026-10-20, Then `from` = `"Maya Costa via InvoiceNudge" <reminders@send.<domain>>`, `replyTo` = `maya@example.test`, `subject` = "Reminder: invoice INV-007 (EUR 1,200.50)", and the text part is exactly: CP-EML-8 ("Hi Sam,"), CP-EML-11, CP-EML-16, CP-EML-19, CP-EML-20, CP-EML-21, CP-EML-22, in that order, separated by blank lines.
- AC-2 (steps) For `pre_due`, `neutral`, `firm`, `final` the subject and body use CP-EML-2 / CP-EML-10, CP-EML-4 / CP-EML-12, CP-EML-5 / CP-EML-13, CP-EML-6 / CP-EML-14; `{days}` = local send date − due date; `{pay_by_date}` = send date + 7 days (e.g. firm sent Mon 2026-11-02 → "Mon 9 Nov 2026").
- AC-3 (variants) With a USD 1,500.00 part payment on USD 4,000.00, the body includes CP-EML-15 "Thanks for the USD 1,500.00 received so far; USD 2,500.00 is still open." and `{balance}` everywhere is USD 2,500.00; without payment instructions CP-EML-17 replaces CP-EML-16; with `invoice_url` CP-EML-18 is included; without a contact name CP-EML-9 "Hello," is used; with business "Costa Studio", `from` display = "Maya Costa (Costa Studio) via InvoiceNudge" and the sign-off has the business line.
- AC-4 (headers and links) `headers` contain `List-Unsubscribe: <https://…/u/<token>>` and `List-Unsubscribe-Post: List-Unsubscribe=One-Click`; body links are `https://…/c/<token>/paid` and `https://…/c/<token>/stop`; every token verifies to the nudge ID with actions `paid` / `stop` / `unsub` and `exp` = send instant + 60 days; `tags` = `[{ name: "kind", value: "reminder" }, { name: "step", value: <step_kind> }]`.
- AC-5 (tokens) `verify` rejects a token with one altered character, a token with a valid MAC but `exp <= now`, an unknown version byte, and a `paid` token presented to the `stop` route, each returning `invalid` with no detail (EC-SEC-11, EC-TIME-09); comparisons are constant-time.
- AC-6 (escaping and lengths) Given client name `<script>x</script> & Co` and reference `A&B<1>`, Then the HTML part contains only escaped forms and the text part the raw text; subjects are ≤ 60 characters with the typical fixtures and ≤ 100 with maximum-length fixtures (reference 40 chars, amount 99,999,999.99) (EC-MSG-12, EC-INPUT-05, EC-SEC-02).
- AC-7 (accessibility) The HTML part has `lang="en"`, a single-column layout ≤ 600 px, body text ≥ 16 px, and links shown as full visible URLs; there are no images.
- AC-8 (template integrity) Rendering every step × variant combination with fixtures never leaves a `{…}` placeholder in the output; a missing variable throws `TemplateError` (EC-MSG-09).
**Edge cases** — EC-INPUT-05, EC-SEC-02, EC-SEC-11, EC-MSG-09, EC-MSG-12, EC-MSG-13 (the footer states that replies go to the freelancer), EC-TIME-09 (token expiry boundary).
**Tests**
- Unit: `src/modules/nudging/domain/compose.test.ts` — `from_and_reply_to` (DR-02 compliance), each step, each variant, escaping, lengths, placeholder check — covers AC-1 to AC-4, AC-6 to AC-8.
- Unit: `src/modules/nudging/domain/tokens.test.ts` — AC-5 cases.
- Mutation: `nudging/domain/**` ≥ 70.
**Tasks**
- E05-S01-T1 Token issue and verify (HMAC-SHA-256 truncated to 128 bits, §12.5).
- E05-S01-T2 Text composition from CP IDs; HTML template with escaping.
- E05-S01-T3 Headers and tags.
- E05-S01-T4 Tests.
**Story DoD** — Default.

### E05-S02 — Send pipeline: claim, send once, record, retry

**Story** — As **P1 (Maya, solo freelancer)**, I want each announced reminder emailed exactly once at the planned time, and never after I've marked the invoice paid, so that my clients get one polite email and not a mess.
**Context** — DR-06, DR-07, DR-12, DR-21; AD-1, AD-3; INT-email retries and idempotency (`emails.send(payload, { idempotencyKey })`, S-27, S-89); rate limit 10 req/s (S-73); staging override (§12.8); EC-MSG-04, EC-NET-03, EC-NET-09, EC-INT-01, EC-INT-03, EC-INT-04, EC-INT-09, EC-CONC-05, EC-CONC-07, EC-TIME-13, EC-OPS-10, EC-OPS-11.
**Scope** — In: the sending phase of `runSchedulerTick`. (1) Claim ≤ 50 rows that are `announced` with `scheduled_for <= now`, or `sending` with `next_attempt_at <= now`, or `send_now` rows, with `FOR UPDATE SKIP LOCKED`; set `sending`, `claimed_at` (first claim only), `attempt_count + 1`, `next_attempt_at` = now + backoff (a lease). (2) Per row: an eligibility re-check in a short transaction; compose; call the gateway outside any transaction; record the outcome. Plus the `NUDGES_LIVE` gate, the recipient override in staging, notices CP-SCR15-1, CP-SCR15-2, CP-SCR15-13, CP-SCR15-15, and audit `nudge.sent`. Out: caps and send-now UI (E05-S05); events webhook (E05-S06).
**Acceptance Criteria**
- AC-1 (happy) Given INV-007's friendly nudge `announced` at Mon 18:00:05 local and `scheduled_for` Tue 2026-10-20 10:00 local, When the scheduler ticks at 10:00:10, Then the fake gateway records exactly one email (AC-1 content of E05-S01) with `idempotencyKey = "nudge/<id>"`; the nudge becomes `sent` with `sent_at`, `provider_message_id` and `recipient_bidx`; one `nudge_sent` notification (CP-SCR15-1) is enqueued; `nudge.sent` is logged with `latency_ms` = 10,000.
- AC-2 (not before time, not unannounced) Given the clock at 09:59:59, Then nothing is sent; given a `planned` (unannounced) nudge at its time, Then it is not claimed and is re-planned to the next window after the next digest (DR-12).
- AC-3 (re-check) Given the invoice was paid, cancelled, paused, or its address suppressed, or the freelancer suspended, unverified or deleted, between claim and send, Then no email is sent and the nudge is `cancelled` with the matching reason; given the plan version changed, reason `superseded`.
- AC-4 (race with payment) Given "Paid in full" commits after the re-check but before the provider responds, Then the email is sent once, the nudge is `sent`, and an audit event `nudge.raced_payment` is recorded by the paid-event handler (EC-CONC-07).
- AC-5 (retries) Given the gateway fails with 5xx, then times out after 10 s, then returns 429 with `retry-after: 3`, then succeeds, Then all four calls use the same idempotency key, attempts happen at +1 min, +5 min and +3 s (honouring `retry-after`), and exactly one `sent` state results (EC-NET-03, EC-INT-03); given 4 consecutive 5xx failures, Then the nudge is `failed`, CP-SCR15-2 is enqueued and a Sentry event is captured (EC-INT-01); given 409 `invalid_idempotent_request`, Then `failed` immediately with a Sentry error and no new key (EC-INT-09).
- AC-6 (unknown and catch-up) Given a nudge `sending` whose `claimed_at` is 23 h ago without a recorded success, Then it becomes `unknown` and CP-SCR15-13 is enqueued (never resent automatically); given `announced` nudges whose `scheduled_for` is more than 24 h in the past (outage), Then they are re-planned to the next window after the next digest and one CP-SCR15-15 per freelancer is enqueued (EC-TIME-13).
- AC-7 (exactly once under concurrency) Given two scheduler processes ticking simultaneously over 30 due nudges, Then 30 emails are recorded, each nudge once (`send_pipeline.int.test.ts::two_workers_one_email`) (EC-CONC-05); given SIGTERM mid-batch, Then the in-flight call completes or its row stays `sending` with `next_attempt_at` set, and the next tick resumes it with the same key (EC-OPS-11).
- AC-8 (gates and staging) Given `NUDGES_LIVE=off`, Then no claims happen; given `allowlist` without the freelancer, their nudges are not claimed. Given `APP_ENV=staging`, Then the `to` address is `delivered+<nudge_id>@resend.dev` (or `bounced+…` / `complained+…` for clients whose name starts with "Bounce Test" / "Complaint Test"), and the original address never reaches the provider (EC-INT-04, EC-OPS-10).
**Edge cases** — EC-MSG-04, EC-NET-03, EC-NET-09, EC-INT-01, EC-INT-03, EC-INT-04, EC-INT-09, EC-CONC-05, EC-CONC-07, EC-TIME-13, EC-OPS-10, EC-OPS-11; story-specific: the provider accepts but our process crashes before recording `sent` → the next attempt reuses the key within 24 h and Resend returns the original response (S-27), which is recorded as `sent`.
**Tests**
- Integration: `test/integration/send_pipeline.int.test.ts` — `happy_path_sends_once`, `not_before_time`, `unannounced_is_replanned`, `recheck_cancels_*` (6 cases), `raced_payment_audited`, `retry_uses_same_idempotency_key` (DR-06 compliance), `four_failures_mark_failed`, `conflict_409_fails_without_new_key`, `unknown_after_23h`, `outage_catch_up`, `two_workers_one_email` (DR-07 compliance), `sigterm_mid_batch`, `nudges_live_gate`, `staging_override` — covers AC-1 to AC-8.
- E2E: `test/e2e/nudge-cycle.e2e.test.ts` (CE-3) — the full cycle in §14.3.
- Manual: `npm run test:contract` against Resend test addresses from staging; one reminder viewed in the Resend dashboard with correct headers; raw headers show a DKIM signature whose `h=` includes `list-unsubscribe` [VERIFY-AT-BUILD]; recorded.
**Tasks**
- E05-S02-T1 Claim query + lease semantics.
- E05-S02-T2 Eligibility re-check (single function used by claim and send).
- E05-S02-T3 Sender: compose → gateway (10 s timeout) → outcome classification (2xx, retryable, non-retryable, 409).
- E05-S02-T4 Unknown and catch-up rules; staging recipient rewrite in the Resend adapter (only when `APP_ENV=staging`).
- E05-S02-T5 Paid-event hook for `nudge.raced_payment`; notices; tests; CE-3.
**Story DoD** — Default + contract test run and DKIM header check recorded in PROGRESS.md.

### E05-S03 — Client "I've already paid" page and freelancer confirmation

**Story** — As **P3 (Sam, the client-side payer)**, I want to tell the freelancer I've already paid with one click, so that the reminders stop and I don't have to write an email.
**Context** — F-07; SCR-21, SCR-23; API-03, API-04; EC-MSG-11 (GET never changes state), EC-SEC-07, EC-SEC-08, EC-SEC-11, EC-NET-06, EC-A11Y-01, EC-A11Y-03, EC-A11Y-04; DR-15 (plain HTML); FF-15.
**Scope** — In: web routes for API-03 and API-04 with `@fastify/formbody` and `@fastify/rate-limit` (30/min per IP); HTML views for SCR-21 and SCR-23; security headers (§12.9); `nudging.handleClientPaid(token)`; freelancer notice CP-SCR15-5 with buttons CP-SCR15-6 / CP-SCR15-7; "Not received yet" → resume with not-before = today + 3 days (CP-SCR15-8); digest "Waiting for you" section (CP-SCR14-6 to CP-SCR14-8) activated; card status CP-SCR12-4 and /owed marker CP-SCR11-18. Out: stop and unsubscribe (E05-S04).
**Acceptance Criteria**
- AC-1 (GET is safe) Given a valid `paid` token for INV-007, When `GET /c/<token>/paid`, Then 200 HTML with h1 CP-SCR21-1 "Tell Maya Costa you've paid", body CP-SCR21-2 with a `<dl>` of reference, balance and due date, one `<button type="submit">` CP-SCR21-3, footer CP-SCR21-9, `<title>` CP-SCR21-8; no state changes (EC-MSG-11).
- AC-2 (POST confirms) When `POST /c/<token>/paid` with `confirm=1`, Then the invoice gets `nudges_paused_reason = client_says_paid`, planned and announced nudges are `cancelled` (`paused`), one `client_says_paid` notification is enqueued (CP-SCR15-5, dedupe per invoice), an audit event with actor `client` is recorded, and the response is 200 HTML CP-SCR21-4 + CP-SCR21-5.
- AC-3 (idempotent and states) A second POST → CP-SCR21-6 and no second notification; if the invoice is already paid → CP-SCR21-7; if cancelled or deleted → 404 SCR-23 with CP-SCR23-6 + CP-SCR23-3.
- AC-4 (freelancer answers) When P1 taps "Yes, mark as paid", Then E03-S03's full payment runs (invoice `paid`); When P1 taps "Not received yet" on Wed 2026-10-28, Then the pause clears, the plan is re-planned with not-before Sat 31 Oct → first reminder Mon 2026-11-02, and CP-SCR15-8 shows "Mon 2 Nov 2026"; while unanswered, each weekday digest shows CP-SCR14-6 first.
- AC-5 (token failures) An altered, expired or wrong-action token, or one for a deleted nudge, → 404 with CP-SCR23-1 + CP-SCR23-2, byte-identical for all four cases (EC-SEC-07, EC-SEC-11); more than 30 requests/min from one IP → 429 with CP-SCR23-6 + CP-SCR23-5; an unexpected error → 500 with CP-SCR23-6 + CP-SCR23-4 and a Sentry event (EC-NET-06).
- AC-6 (headers) Every HTML response carries the CSP, `Referrer-Policy: no-referrer`, `X-Content-Type-Options: nosniff` and `Cache-Control: no-store` of §12.9 and sets no cookies (EC-SEC-08).
- AC-7 (accessibility) FF-15 passes for every SCR-21 and SCR-23 state (`lang`, one `h1`, `<title>`, labelled button, < 15 KB, no external resources); the button is ≥ 48 px high; no meaning by colour alone; manual check at 200% zoom and with a screen reader, recorded (EC-A11Y-01, EC-A11Y-03, EC-A11Y-04).
- AC-8 (observability) `client_action.paid` (invoice ID, outcome) and `http.request` logs; never the token, IP or any PII.
**Edge cases** — EC-MSG-11, EC-SEC-07, EC-SEC-08, EC-SEC-11, EC-NET-06, EC-A11Y-01, EC-A11Y-03, EC-A11Y-04, EC-BOT-07 (old notice buttons after the invoice was paid → CP-SCR13-13).
**Tests**
- Integration: `test/integration/client_paid_page.int.test.ts` — AC-1 to AC-3, AC-5, AC-6, AC-8.
- Integration: `test/integration/pages.int.test.ts` (FF-15) — SCR-21 and SCR-23 states.
- Integration: `test/integration/client_paid_answer.int.test.ts` — AC-4.
- E2E: `test/e2e/client-paid.e2e.test.ts` (CE-4).
- Manual: open a staging reminder's link on a phone; 200% zoom; VoiceOver or TalkBack reads the heading, facts and button; recorded.
**Tasks**
- E05-S03-T1 Routes, form parsing, rate limit, security headers (a Fastify `onSend` hook for HTML routes).
- E05-S03-T2 Views (tiny template functions producing escaped HTML from CP IDs).
- E05-S03-T3 `handleClientPaid` with events, notice and audit.
- E05-S03-T4 Freelancer answer handlers; digest waiting section.
- E05-S03-T5 Tests; CE-4.
**Story DoD** — Default + manual accessibility check recorded.

### E05-S04 — Stop reminders and one-click unsubscribe

**Story** — As **P3 (Sam, the client-side payer)**, I want to stop reminders about one invoice, or unsubscribe from all reminders from this freelancer, without logging in or writing back, so that I stay in control of my inbox.
**Context** — F-08; SCR-22; API-05 to API-08; DR-13; RFC 8058: POST with `List-Unsubscribe=One-Click`, no redirects, no cookies (S-47); ENT-Suppression; EC-MSG-06, EC-MSG-11. `deliverability.isSuppressed` becomes real here (it was stubbed until now).
**Scope** — In: `suppression` migration; `deliverability.isSuppressed`, `suppress`; routes and views for API-05 to API-08; `nudging.handleClientStop`, `handleUnsubscribe`; notices CP-SCR15-9, CP-SCR15-10; resume refusal CP-SCR12-36 / CP-SCR12-40; status lines CP-SCR12-5, CP-SCR12-44, CP-SCR16-6, CP-SCR16-7 (bounce status set by E05-S06); wizard warning CP-SCR07-14 and summary CP-SCR10-9 now driven by real suppression.
**Acceptance Criteria**
- AC-1 (stop one invoice) Given a valid `stop` token, When GET, Then SCR-22 with CP-SCR22-1, CP-SCR22-2, CP-SCR22-3 and no change; When POST `confirm=1`, Then the invoice gets `nudges_paused_reason = client_opt_out`, its planned and announced nudges are `cancelled` (`opted_out`), CP-SCR15-9 is enqueued, and CP-SCR22-4 + CP-SCR22-5 are returned.
- AC-2 (resume refused) Given an invoice stopped by the client, When P1 taps "Resume reminders", Then CP-SCR12-36 (toast CP-SCR12-40) and nothing changes; the card shows CP-SCR12-5.
- AC-3 (one-click) Given a valid `unsub` token, When a mail client sends `POST /u/<token>` with body `List-Unsubscribe=One-Click` (`application/x-www-form-urlencoded`, and separately `multipart/form-data`), Then the response is 200 with an empty body, no `Location` header and no `Set-Cookie`; a suppression row (freelancer, `recipient_bidx`, `unsubscribe`) exists; all planned and announced nudges of that freelancer whose client's current email blind index equals `recipient_bidx` are `cancelled` (`suppressed`); CP-SCR15-10 is enqueued once.
- AC-4 (browser unsubscribe) When the same URL is opened with GET, Then SCR-22 CP-SCR22-6 with button CP-SCR22-7 and no change; the form POST returns CP-SCR22-8 as HTML (`Accept: text/html`).
- AC-5 (effects elsewhere) After AC-3, creating a new invoice for any client with that address shows CP-SCR07-14 and CP-SCR10-9, and no nudges are planned for it; the client card shows CP-SCR16-6; the invoice card shows CP-SCR12-44.
- AC-6 (idempotency and speed) Repeating any POST returns the same done state without new rows or notices; the suppression is effective for the very next scheduler tick (≤ 60 s, AD-5) (EC-MSG-06).
- AC-7 (accessibility and headers) FF-15 passes for all SCR-22 states; §12.9 headers as in E05-S03.
- AC-8 (observability) `client_action.stop` / `.unsubscribe` with IDs and outcome; `suppression.added` with reason.
**Edge cases** — EC-MSG-06, EC-MSG-11, EC-SEC-11 (same token failure page as E05-S03), EC-CONC-05 (unsubscribe during a claim: the pipeline's re-check sees the suppression; if the email is already with the provider, it goes out once; audit records it).
**Tests**
- Integration: `test/integration/client_stop_unsubscribe.int.test.ts` — AC-1 to AC-8.
- E2E: `test/e2e/unsubscribe.e2e.test.ts` (CE-5).
**Tasks**
- E05-S04-T1 `suppression` migration and `deliverability` functions.
- E05-S04-T2 Routes and views for API-05 to API-08 (one-click returns an empty 200).
- E05-S04-T3 Nudging handlers and cancellation by recipient blind index.
- E05-S04-T4 UI integrations (resume refusal, status lines, wizard warning).
- E05-S04-T5 Tests; CE-5.
**Story DoD** — Default.

### E05-S05 — Sending caps, send-now and sent notices

**Story** — As **P1 (Maya, solo freelancer)**, I want to send a reminder right now when I need to, see a note each time one goes out, and have sensible daily limits, so that I stay in control and the service can't be abused.
**Context** — F-06 step 3; SCR-12 (send now), SCR-15; assumption 21 (caps); EC-BIZ-01, EC-SEC-13, EC-CONC-02, EC-MSG-10; AD-6 (no provider calls in webhook handlers: send-now marks the row and wakes the scheduler).
**Scope** — In: cap check in the eligibility re-check (30 per local day; 10 for accounts younger than 7 days; overflow postponed to the next working day's send hour, keeping `announced`), CP-SCR15-11 once per day; send-now button CP-SCR12-13 with confirm CP-SCR12-24 to CP-SCR12-26 and the 48 h rule CP-SCR12-27; in-process "wake" signal for the scheduler; batching of sent notices (more than 3 in one minute → one list message using the CP-SCR15-1 line format). Out: operator overrides of caps (E06-S02 records them as a parking-lot item).
**Acceptance Criteria**
- AC-1 (send now) Given INV-007 open with its next step (neutral) planned for Mon 2026-10-26, When "Send reminder now" → CP-SCR12-24 ("…It will be reminder 2 of 4.") → "Send now" is tapped at Thu 2026-10-22 11:00, Then the neutral nudge gets `send_now=true` and `scheduled_for=now`, the callback is answered with CP-SCR20-12 within 1 s, the scheduler wakes and sends within 10 s, CP-SCR15-1 arrives, and the remaining steps are re-planned from 2026-10-22 per E02-S05 rule 7.
- AC-2 (48 h rule) Given a reminder for INV-007 was sent Wed 2026-10-21 10:00, When send-now is attempted Thu 2026-10-22 09:59, Then CP-SCR12-27 with `date_next` = "Fri 23 Oct 2026"; at 10:00 on Fri 23 Oct it is allowed.
- AC-3 (visibility) The button is hidden when the invoice is paused by the client, opted out, paid or cancelled, when no step remains, when the address is suppressed, when the freelancer is suspended, or when sending is not live for them.
- AC-4 (double tap) Two "Send now" taps within 200 ms → one nudge marked, one email (EC-CONC-02).
- AC-5 (daily cap) Given a freelancer (account older than 7 days) with 30 reminders sent today (local) and 4 more due at 10:00, Then the 4 are rescheduled to the next working day at 10:00 (still `announced`), and exactly one CP-SCR15-11 (`cap=30`, `count=4`) is enqueued that day; for an account created 3 days ago the cap is 10 (EC-BIZ-01, EC-SEC-13).
- AC-6 (notices) Given 5 reminders sent within the same minute, Then one message lists 5 lines in the CP-SCR15-1 format instead of 5 messages; 1–3 sends produce individual CP-SCR15-1 messages.
- AC-7 (accessibility) The confirmation states the step number and total in text ("reminder 2 of 4").
- AC-8 (observability) `nudge.send_now`, `nudge.cap_postponed` (count), `notification.batched` (count).
**Edge cases** — EC-BIZ-01, EC-SEC-13, EC-CONC-02, EC-MSG-10, EC-BOT-07 (old send-now button after the invoice was paid → CP-SCR13-13).
**Tests**
- Integration: `test/integration/send_now_caps.int.test.ts` — AC-1 to AC-8.
- Unit: `src/modules/nudging/domain/eligibility.test.ts` adds `send_now_bypasses_announcement` and the cap cases.
**Tasks**
- E05-S05-T1 Cap computation (by the freelancer's local date) in eligibility.
- E05-S05-T2 Send-now handler, 48 h rule, visibility rules on the card.
- E05-S05-T3 Scheduler wake signal (an in-process event emitter awaited by the loop's sleep).
- E05-S05-T4 Notice batching in the dispatcher.
- E05-S05-T5 Tests.
**Story DoD** — Default.

### E05-S06 — Email events webhook: bounces and complaints

**Story** — As **P4 (Operator)**, I want bounces and spam complaints to suppress addresses automatically and tell the freelancer what to fix, so that one bad address or unhappy client doesn't damage delivery for everyone.
**Context** — F-09; API-02; ENT-EmailEvent; Resend event types (S-29) and Svix verification (S-30, S-89); Gmail spam-rate threshold 0.3% (S-46); EC-INT-07, EC-CONC-04, EC-CONC-06, EC-MSG-01, EC-MSG-02. Events are matched to nudges by `provider_message_id`; addresses are never read from the payload (we use the nudge's `recipient_bidx`).
**Scope** — In: `email_event` migration; route API-02 with raw-body capture scoped to that route (Fastify `addContentTypeParser` with `parseAs: "string"` [VERIFY-AT-BUILD against Fastify 5.12 docs]); `resend.webhooks.verify({ payload, headers: { id, timestamp, signature }, webhookSecret })`; `deliverability.recordEmailEvent`; handling of `email.bounced` (global suppression for 90 days, reason `bounce`; cancel all planned and announced nudges to that address across freelancers; CP-SCR15-3 to affected freelancers), `email.complained` (freelancer-scoped suppression, reason `complaint`; cancel that freelancer's nudges to the address; CP-SCR15-4; complaint counter used by E06-S01), `email.suppressed` (treated like a bounce), `email.delivered` / `delivery_delayed` / `failed` / `sent` (stored only); client and invoice status lines CP-SCR16-7, CP-SCR12-8; E01-S04's CP-SCR03-14 path now real.
**Acceptance Criteria**
- AC-1 (bounce) Given nudge N sent to ap@acme.test with `provider_message_id = m1`, When a signed `email.bounced` event for `m1` arrives, Then 200 `{"ok":true}`, one `email_event` row (`type=email.bounced`, `nudge_id=N`), one global suppression (reason `bounce`, `email_bidx = N.recipient_bidx`), all planned and announced nudges of any freelancer whose client email blind index matches are `cancelled` (`suppressed`), and CP-SCR15-3 is enqueued for each affected freelancer (EC-MSG-02).
- AC-2 (complaint) When a signed `email.complained` for `m1` arrives, Then a suppression (freelancer of N, reason `complaint`) is created, that freelancer's nudges to the address are `cancelled`, CP-SCR15-4 is enqueued, and `deliverability.complaintsInWindow(freelancerId, 30 days)` returns 1.
- AC-3 (signature and replay) An event with an invalid signature → 400 `{"error":{"code":"invalid_signature","message":"Invalid signature"}}` and nothing stored; the same valid event delivered twice (same `svix-id`) → 200 both times, one row and one set of effects (EC-INT-07).
- AC-4 (ordering) An event for an unknown message ID is stored with `nudge_id = null` and linked when a nudge later records that ID (EC-CONC-04); `email.delivered` arriving after `email.bounced` does not lift the suppression (EC-CONC-06).
- AC-5 (no resend) `email.delivery_delayed` with no later `delivered` triggers no resend and no notice (EC-MSG-01).
- AC-6 (fix path) After a bounce, editing the client's email to a different address (E02-S02) makes reminders plannable again (the new blind index is not suppressed), re-plans the invoice, and the client card shows CP-SCR16-5; re-entering the bounced address within 90 days shows CP-SCR07-14.
- AC-7 (accessibility and copy) Notices name the client and the address in text; exact CP lines.
- AC-8 (observability) `email_event.received` (type, matched yes/no), `suppression.added` (reason), `deliverability.complaint` (freelancer ID, 30-day count); no addresses in logs.
**Edge cases** — EC-INT-07, EC-CONC-04, EC-CONC-06, EC-MSG-01, EC-MSG-02; story-specific: an event whose timestamp is more than 5 minutes old is still accepted if the signature verifies (Svix's own tolerance applies) [VERIFY-AT-BUILD].
**Tests**
- Integration: `test/integration/email_events.int.test.ts` — the route verifies through a `WebhookVerifier` port whose production adapter calls `resend.webhooks.verify`; tests inject a fake verifier (valid / invalid) so no signing algorithm is re-implemented; covers AC-1 to AC-8.
- Contract (staging, manual): a real Resend event with a real signature is accepted and a tampered copy (one byte changed) is rejected with 400; recorded — covers AC-3 on the real path.
- Manual: send to `bounced@resend.dev` and `complained@resend.dev` from staging (fixture clients "Bounce Test …", "Complaint Test …"); observe suppression and notices; recorded.
**Tasks**
- E05-S06-T1 `email_event` migration; route with raw body and verification.
- E05-S06-T2 Event handlers and suppression effects.
- E05-S06-T3 Card and client status lines for bounces.
- E05-S06-T4 Tests; staging manual run.
**Story DoD** — Default + staging bounce and complaint run recorded.

**Epic-level tests** — CE-3, CE-4, CE-5 green (together with CE-1, CE-2); FF-08 and FF-14 over the full reminder cycle (no PII in logs or plaintext in the DB); `authz.int.test.ts` includes `now`, `now_ok`, `claim_ok`, `claim_no`.
**Definition of Done**
- All 6 stories DONE with evidence lines; CE-1 to CE-5 green; mutation job green.
- Staging: 20 reminders to Resend test addresses with correct subjects, headers and links; the DKIM `h=` list includes `list-unsubscribe`; bounce and complaint webhooks observed end to end.
- **Closed beta 2** switched on (`NUDGES_LIVE=allowlist`) only after the product owner has read one real reminder of each step kind, sent to their own address from staging with the override disabled for that single test (recorded as a deliberate exception in PROGRESS.md).
- PROGRESS.md, DECISIONS.md updated; any copy change goes through an Amendment.
**Session handoff checklist** — §18.5 summary, plus: deliverability status (SPF, DKIM, DMARC pass as shown in a received email's headers); contract test result; list of any reminders marked `unknown` or `failed` on staging; preconditions for E06 (`ADMIN_TELEGRAM_IDS` set).

## Epic 06 — Operator tools and manual forwarding

**Goal** — The operator can see the service's health, act on abuse, and rely on automatic suspension after complaints. Freelancers can paste a ready-made reminder into any chat app.
**Personas served** P4, P2
**Depends on** E05: complaints counter, suppression, sending pipeline.
**Preconditions for the session** — PROGRESS.md shows E05 DONE; `check` green; `ADMIN_TELEGRAM_IDS` set in staging to the product owner's ID. `[VERIFY-AT-BUILD]`: none new.
**Scope** — In: stories below. Out: settings, export, deletion (E07).
**Flows and screens implemented** F-13, F-15, F-09 (suspension part); SCR-19; SCR-12 (forward text); SCR-15 (CP-SCR15-12, CP-SCR15-14).
**Data and API touched** ENT-Freelancer (`sending_status`, `suspended_reason`), views `v_admin_stats`, `v_metrics_*`, ENT-AuditEvent.
**Risks specific to this epic** R-01, R-15 (false-positive suspensions).

### E06-S01 — Complaint-based auto-suspension

**Story** — As **P4 (Operator)**, I want a freelancer's reminder sending suspended automatically after two spam complaints in 30 days, and to be alerted, so that one account cannot damage delivery for everyone before I even look.
**Context** — Assumption 22; Gmail's 0.3% spam-rate threshold shared across our domain (S-46); AD-5; EC-SEC-13, EC-BIZ-02, EC-ADMIN-06.
**Scope** — In: policy in `deliverability/domain/complaintPolicy.ts`; on each complaint event (E05-S06), if `complaintsInWindow ≥ 2`, then `accounts.setSendingStatus(suspended, "auto: 2 complaints in 30 days")`, cancel due nudges (`suspended`), CP-SCR15-12 to the freelancer, CP-SCR19-7 to every operator; eligibility already refuses suspended accounts (E05-S02). Out: manual suspend and unsuspend commands (E06-S02).
**Acceptance Criteria**
- AC-1 (happy) Given freelancer F with one complaint 20 days ago, When a second complaint arrives, Then `sending_status = suspended`, `suspended_reason = "auto: 2 complaints in 30 days"`, all planned and announced nudges of F are `cancelled` (`suspended`), CP-SCR15-12 is enqueued for F, CP-SCR19-7 is enqueued for each ID in `ADMIN_TELEGRAM_IDS`, and an audit event `sending.suspended` (actor `system`) is recorded.
- AC-2 (window) Given one complaint 31 days ago and one today, Then no suspension.
- AC-3 (while suspended) Given F is suspended, Then the digest is not sent for F, new invoices still plan nudges (so the history is visible), the send pipeline does not claim F's nudges, and F's tracking features (/new, /owed, payments) work normally.
- AC-4 (idempotent) A third complaint while suspended does not create another alert or notice.
- AC-5 (permissions) Only the system (event handler) or an operator can change `sending_status`; no freelancer-facing command can.
- AC-6 (copy) Exact CP-SCR15-12 and CP-SCR19-7 with `support_email` from config.
- AC-7 (accessibility) n/a beyond the text notices.
- AC-8 (observability) `sending.suspended` (reason, complaints_30d) at `warn` level; an error-tracking breadcrumb, not an error.
**Edge cases** — EC-SEC-13, EC-BIZ-02, EC-ADMIN-06; story-specific: complaints arriving for two different freelancers at the same moment are handled independently (per-freelancer row lock).
**Tests**
- Unit: `src/modules/deliverability/domain/complaintPolicy.test.ts` — window boundaries (exactly 30 days = inside) — AC-2.
- Integration: `test/integration/auto_suspension.int.test.ts` — AC-1, AC-3 to AC-6, AC-8.
**Tasks**
- E06-S01-T1 Policy function and hook in the complaint handler.
- E06-S01-T2 Suspension effects and notices.
- E06-S01-T3 Digest and pipeline checks for suspended accounts (verify existing eligibility covers them).
- E06-S01-T4 Tests.
**Story DoD** — Default.

### E06-S02 — Operator admin commands

**Story** — As **P4 (Operator)**, I want terse admin commands in Telegram to see stats, look up a user, and suspend or restore sending with a reason, so that I can run the service from my phone in minutes a day.
**Context** — F-13; SCR-19; §12.6 (operators by `ADMIN_TELEGRAM_IDS`); EC-AUTH-14, EC-ADMIN-02, EC-ADMIN-03, EC-PERM-03. Stats come from the `v_admin_stats` view (counts only).
**Scope** — In: `/admin` router with subcommands `stats`, `user <telegram_id>`, `suspend <telegram_id> <reason>`, `unsuspend <telegram_id>` (`metrics` is added in E06-S04); `v_admin_stats` view migration; `admin` module functions; unsuspend effects (re-plan open invoices from today via `nudging.replanAllForFreelancer`, CP-SCR15-14 to the freelancer); CP-SCR19-1 to CP-SCR19-8; audit with actor `operator`. Out: metrics lines (E06-S04).
**Acceptance Criteria**
- AC-1 (stats) Given an operator, When `/admin stats`, Then CP-SCR19-1 with values equal to direct counts from fixtures (freelancers, active in 7 days by `last_seen_at`, open invoices, reminders sent and failed in 24 h, bounces and complaints in 24 h, suspended accounts); `activation_pct` and `paid14_pct` show "n/a" until E06-S04.
- AC-2 (user) `/admin user 111` → CP-SCR19-2 with counts only (no client names, emails or references) (EC-ADMIN-03); an unknown ID → CP-SCR19-6.
- AC-3 (suspend) `/admin suspend 111 spam reports from clients` → `sending_status = suspended`, reason stored (max 200 chars, no PII expected, but it is operator-entered and never shown to clients), nudges cancelled as in E06-S01, CP-SCR15-12 to the freelancer, CP-SCR19-4 to the operator, audit `sending.suspended` (actor `operator`) (EC-ADMIN-02, EC-PERM-03); `/admin suspend 111` without a reason → CP-SCR19-3.
- AC-4 (unsuspend) `/admin unsuspend 111` → `active`, open invoices re-planned from today (E02-S05 rules), CP-SCR15-14 to the freelancer, CP-SCR19-5 to the operator, audit `sending.restored`.
- AC-5 (not an operator) Any `/admin …` from a non-listed user → CP-SCR20-1, exactly like an unknown command; no audit event and no log line that names the command (EC-AUTH-14).
- AC-6 (help) `/admin` or an unknown subcommand → CP-SCR19-8.
- AC-7 (accessibility) Plain text with fixed field order.
- AC-8 (observability) `admin.command` (subcommand, target ID, outcome) for operators only.
**Edge cases** — EC-AUTH-14, EC-ADMIN-02, EC-ADMIN-03, EC-PERM-03, EC-ADMIN-06 (unsuspend after a false positive restores reminders from today, not retroactively).
**Tests**
- Integration: `test/integration/admin.int.test.ts` — AC-1 to AC-8.
**Tasks**
- E06-S02-T1 `v_admin_stats` view (FF-07 allows `v_*` for `admin`).
- E06-S02-T2 Router with allow-list check before any parsing.
- E06-S02-T3 Suspend and unsuspend with effects and audit.
- E06-S02-T4 Tests.
**Story DoD** — Default.

### E06-S03 — Text to forward for chat apps

**Story** — As **P2 (Arjun, high-volume contractor)**, I want a ready-made reminder I can paste into WhatsApp or Telegram for clients who ignore email, so that I chase them in their channel with the right numbers.
**Context** — F-15; `copy_text` buttons allow only 1–256 characters (S-53), so the draft is sent as its own message for long-press copy or forward; nothing is sent to the client by us; bots cannot message clients (S-82).
**Scope** — In: More → CP-SCR12-33 on the card; `nudging.forwardText(invoiceId)` rendering CP-SCR12-46 with the current balance, contact name and payment instructions; CP-SCR12-45 intro. Out: per-channel variants.
**Acceptance Criteria**
- AC-1 (happy) Given INV-007 (Acme, contact "Sam", EUR 1,200.50 owed, due Mon 2026-10-19, payment instructions set), When "Get text to forward" is tapped, Then two messages are sent: CP-SCR12-45, then CP-SCR12-46 exactly "Hi Sam, a quick reminder that invoice INV-007 for EUR 1,200.50 was due on Mon 19 Oct 2026. You can pay here: Bank transfer: IBAN PT50 … Thanks, Maya Costa" as plain text (no HTML parse mode, so it copies cleanly).
- AC-2 (variants) Without a contact name → "Hi there, …"; without payment instructions → no payment sentence; with a part payment → the balance is the remaining amount.
- AC-3 (allowed states) Works for open invoices in every reminder state, including opted-out and suppressed clients (the freelancer may contact their own client); hidden for paid and cancelled invoices.
- AC-4 (no side effects) No nudge row, email or notification is created.
- AC-5 (permissions) Another freelancer's invoice → CP-SCR12-32 (added to `authz.int.test.ts` as `fwd`).
- AC-6 (length) The draft is ≤ 700 characters with the longest fixtures.
- AC-7 (accessibility) The intro explains what the next message is for.
- AC-8 (observability) `forward_text.generated` with the invoice ID.
**Edge cases** — EC-PERM-02, EC-BOT-07 (old button on a paid invoice → CP-SCR13-13), EC-L10N-03.
**Tests**
- Integration: `test/integration/forward_text.int.test.ts` — AC-1 to AC-8.
**Tasks**
- E06-S03-T1 Renderer (plain text).
- E06-S03-T2 Handler and More-menu button.
- E06-S03-T3 Tests.
**Story DoD** — Default.

### E06-S04 — Product metrics in operator stats

**Story** — As **P4 (Operator)**, I want activation, payment speed and retention figures computed from our own data, so that I can tell whether the product works without adding tracking.
**Context** — §6 metrics M-1.2, M-2.1, M-2.2, M-5.1 (and M-1.1 from `entry_duration_ms`); AD-4 (no analytics SDK); views owned by `admin` (§12.4).
**Scope** — In: views `v_metrics_activation`, `v_metrics_paid_after_first_nudge`, `v_metrics_days_overdue`, `v_metrics_retention`; `/admin stats` fills `activation_pct` and `paid14_pct`; `/admin metrics` prints CP-SCR19-9 for the last 30 days (operator only). Out: charts, exports.
**Acceptance Criteria**
- AC-1 (activation, M-1.2) Given 10 freelancers created in the last 30 days of whom 6 saved a first invoice within 10 minutes of creation, Then `activation_pct = 60`.
- AC-2 (paid within 14 days, M-2.1) Given 5 invoices with ≥ 1 sent reminder, 4 of which were paid within 14 days of their first reminder's `sent_at`, Then `paid14_pct = 80`.
- AC-3 (days overdue, M-2.2) Given invoices paid 3, 5 and 20 days after their due dates, and 1 paid before its due date (excluded), Then the median is 5.
- AC-4 (retention, M-5.1) Given 4 freelancers activated 4–5 weeks ago, 2 of whom logged or updated an invoice in their 4th week after activation, Then retention = 50%.
- AC-5 (entry time, M-1.1) The median `entry_duration_ms` for invoices whose client already existed is shown in seconds.
- AC-6 (privacy) The views expose counts and dates only; FF-14 and a view-column test assert no `_enc` or `_bidx` columns appear in any `v_metrics_*` view.
- AC-7 (copy) `/admin metrics` renders CP-SCR19-9 exactly, with "n/a" for metrics without data; CP-SCR19-8 lists `metrics`.
- AC-8 (performance) Each view query runs in ≤ 500 ms at the scale ceiling (5,000 freelancers, 50,000 invoices) on the E08-S02 dataset.
**Edge cases** — EC-DATA-01 (no data → "n/a" instead of 0%), EC-TIME-02 (weeks computed in UTC for metrics; documented).
**Tests**
- Integration: `test/integration/metrics_views.int.test.ts` — AC-1 to AC-6 with factories.
- Performance: covered in E08-S02.
**Tasks**
- E06-S04-T1 View migrations.
- E06-S04-T2 Stats and metrics renderers.
- E06-S04-T3 Tests.
**Story DoD** — Default.

**Epic-level tests** — Scenario test `test/integration/complaint_to_restore.int.test.ts`: two complaints → auto-suspension → operator sees the user → unsuspend → reminders re-planned. CE-1 to CE-5 green.
**Definition of Done**
- All 4 stories DONE; `check` and CI green; staging deployed.
- Manual on staging: operator runs `/admin stats`, `/admin user`, suspend and unsuspend on a test account; the freelancer receives CP-SCR15-12 and CP-SCR15-14; text-to-forward pasted into WhatsApp shows correct numbers. Recorded.
- PROGRESS.md, DECISIONS.md and (if needed) the Amendments log updated.
**Session handoff checklist** — §18.5 summary, plus: current metric values on staging (for the M-ids); preconditions for E07 (none).

## Epic 07 — Settings, export and deletion

**Goal** — Freelancers can change every setting, take all their data, or delete their account at once. Clients and freelancers can read a plain privacy summary, and the bot's command menu is complete.
**Personas served** P1, P3
**Depends on** E06 (all features exist, so export and deletion cover everything); E04-S01 `replanAllForFreelancer`.
**Preconditions for the session** — PROGRESS.md shows E06 DONE; `check` green. `[VERIFY-AT-BUILD]`: Telegram document upload limit 50 MB (S-08); BotFather `/setdescription` still available (S-12).
**Scope** — In: stories below. Out: public launch (E08).
**Flows and screens implemented** F-11, F-12; SCR-17, SCR-18, SCR-24; command menu CP-CMD-1 to CP-CMD-8.
**Data and API touched** ENT-Freelancer (settings), all tables (export, deletion, retention), API-10.
**Risks specific to this epic** R-02 (Telegram policy), R-04 (legal text pending, OQ-02).

### E07-S01 — Settings screen and edits

**Story** — As **P1 (Maya, solo freelancer)**, I want one settings screen where I can change my name, time zone, reminder time, default plan, evening plan, currency and payment details, so that the bot keeps matching how I work.
**Context** — F-11; SCR-17; reuses onboarding steps (SCR-02, SCR-04, SCR-05) and the plan picker; changing the zone or send hour re-plans every open invoice (`nudging.replanAllForFreelancer`, EC-TIME-10); turning the digest off applies the DR-12 exception.
**Scope** — In: `/settings` summary CP-SCR17-1 and buttons CP-SCR17-2 and CP-SCR17-4 to CP-SCR17-9; reminder-time picker CP-SCR17-12, CP-SCR17-13; digest toggle CP-SCR17-15, CP-SCR17-16 (edit in place); zone change ack CP-SCR17-17; default plan (affects new invoices only). Out: reply-to change (E07-S02), export and delete buttons (E07-S03, E07-S04).
**Acceptance Criteria**
- AC-1 (summary) Given Maya (Europe/Lisbon, send hour 10, Standard, digest on, EUR, payment details set), When `/settings`, Then CP-SCR17-1 renders every line with current values ("Reminder time: 10:00 on weekdays", "Evening plan: on at 18:00", "Payment details: set") and the button grid.
- AC-2 (send hour) When "Reminder time" → "09:00", Then `send_hour = 9`, all planned and announced nudges of open invoices are re-planned to 09:00 local on their dates (new plan versions; announced ones return to `planned` and are re-announced by the next digest), and CP-SCR17-13 is shown.
- AC-3 (zone) When the zone is changed to America/New_York via SCR-04, Then the same re-plan happens in the new zone and CP-SCR17-17 is shown; past `sent` rows are untouched (EC-TIME-10).
- AC-4 (digest toggle) When "Evening plan on/off" is tapped while on, Then `digest_enabled = false`, all `planned` nudges get `announced_at = now` and `status = announced`, the eligibility rule's 12-hour condition no longer applies to this freelancer (DR-12 exception), and the message is edited to CP-SCR17-15; tapping again → `true`, future plans are announced by digests again, and CP-SCR17-16 is shown.
- AC-5 (other fields) Name and business reuse SCR-02 validations; currency reuses SCR-05 (affects new invoices only); payment details reuse CP-SCR05-5 / CP-SCR05-7 and apply to the next reminder rendered; default plan uses CP-SCR10-12 and affects only invoices created afterwards.
- AC-6 (stale and permissions) An old settings message's button → CP-SCR20-2 and a fresh summary; settings callbacks only ever act on the sender's own freelancer row.
- AC-7 (accessibility) Every setting's current value is in the summary text, not only on buttons; the reminder-time picker shows 24-hour times as text.
- AC-8 (observability) `settings.changed` with the field name; `nudge.replanned` with the count.
**Edge cases** — EC-TIME-04, EC-TIME-10, EC-TIME-12, EC-BOT-07, EC-INPUT-02.
**Tests**
- Integration: `test/integration/settings.int.test.ts` — AC-1 to AC-8, including a re-plan across the Europe/Lisbon DST change.
**Tasks**
- E07-S01-T1 Summary renderer and keyboard.
- E07-S01-T2 Reuse of onboarding step handlers in "edit mode" (return to the summary afterwards).
- E07-S01-T3 Re-plan calls; digest toggle effects.
- E07-S01-T4 Tests.
**Story DoD** — Default.

### E07-S02 — Change reply-to email with re-verification

**Story** — As **P1 (Maya, solo freelancer)**, I want to change the email client replies go to, with a new code, so that replies follow me without ever pointing at an unverified address.
**Context** — CP-SCR17-3, CP-SCR17-14; E01-S04 verification reused; EC-AUTH-12, EC-SEC-12.
**Scope** — In: "Reply-to email" in settings → CP-SCR17-14 → SCR-03 flow for the new address; the old address stays in `email_enc` until the new code is confirmed. Out: nothing else.
**Acceptance Criteria**
- AC-1 (happy) Given verified maya@example.test, When "Reply-to email" → "maya@costa.example" → correct code, Then `email_enc` = maya@costa.example, `email_verified_at` = now, CP-SCR03-12, and reminders composed after this moment use the new Reply-To.
- AC-2 (pending) While the new code is unconfirmed, reminders keep using maya@example.test (EC-AUTH-12); abandoning the flow leaves the old address untouched.
- AC-3 (limits) All E01-S04 limits apply (cooldown, attempts, hourly cap, bounce suppression).
- AC-4 (same address) Entering the current address → CP-SCR03-12 without sending a code.
- AC-5 (permissions) Only the sender's own row can change.
- AC-6 (copy) CP-SCR17-14 precedes the SCR-03 prompt; exact lines.
- AC-7 (accessibility) Expiry stated as in E01-S04.
- AC-8 (observability) `email_verification.sent` / `.confirmed` with `purpose=change`.
**Edge cases** — EC-AUTH-01 to EC-AUTH-04, EC-AUTH-11, EC-AUTH-12, EC-SEC-12.
**Tests**
- Integration: `test/integration/email_change.int.test.ts` — AC-1 to AC-8.
**Tasks**
- E07-S02-T1 Settings entry and the edit-mode verification flow.
- E07-S02-T2 Tests.
**Story DoD** — Default.

### E07-S03 — Export my data as CSV

**Story** — As **P1 (Maya, solo freelancer)**, I want all my clients, invoices, payments and reminders as spreadsheet files, so that I can give them to my accountant or leave with my data.
**Context** — F-12 step 1; SCR-18; EC-DATA-10, EC-DATA-13, EC-FILE-09, EC-PERM-05, EC-L10N-09; Telegram bots upload files up to 50 MB (S-08); assumption 34 (UTF-8 with BOM) [VERIFY-AT-BUILD by opening a file in a spreadsheet app].
**Scope** — In: `accounts.exportAccount(freelancerId)` producing 4 CSV buffers (decrypted values; streamed row by row into memory buffers), `sendDocument` ×4 with file names CP-SCR18-11, CP-SCR18-1 to CP-SCR18-4, 1 export per hour. Out: other formats.
**Acceptance Criteria**
- AC-1 (happy) Given 3 clients, 5 invoices, 2 payments, 9 nudges, When "Export my data", Then CP-SCR18-1, then 4 documents with the expected headers — `clients.csv`: `name,email,greeting,archived_at,created_at`; `invoices.csv`: `reference,client,amount,currency,due_date,status,paid_at,link,reminder_plan,created_at`; `payments.csv`: `invoice_reference,amount,currency,kind,recorded_at`; `reminders.csv`: `invoice_reference,step,scheduled_for,status,sent_at` — then CP-SCR18-2.
- AC-2 (encoding and quoting) Files start with the UTF-8 BOM; fields with commas, quotes or newlines are quoted per RFC 4180 with doubled quotes; names in Cyrillic and CJK round-trip byte-exact (EC-DATA-10, EC-L10N-09); amounts use a dot decimal ("1200.50"); timestamps are ISO 8601 UTC; dates are ISO.
- AC-3 (formula injection) A client named "=HYPERLINK(\"x\")" is written as "'=HYPERLINK(""x"")" (EC-DATA-13).
- AC-4 (limits) A second export within 60 minutes → CP-SCR18-4 with `minutes` and `wait`; at the scale ceiling for one freelancer (200 clients, 500 open + 2,000 closed invoices), the total size stays < 5 MB (EC-FILE-09).
- AC-5 (failure) Given `sendDocument` fails after retries, Then CP-SCR18-3 and the hourly limit is not consumed.
- AC-6 (permissions) The export contains only the sender's data (a fixture with a second freelancer proves it).
- AC-7 (accessibility) CP-SCR18-2 names the four files in text.
- AC-8 (observability) `account.exported` with row counts; no content in logs.
**Edge cases** — EC-DATA-10, EC-DATA-13, EC-FILE-09, EC-PERM-05, EC-L10N-09.
**Tests**
- Unit: `src/modules/accounts/domain/csv.test.ts` — quoting, BOM, formula prefix.
- Integration: `test/integration/export.int.test.ts` — AC-1, AC-4 to AC-8 with fake Telegram capturing documents.
- Manual: open the 4 files in one spreadsheet app; names display correctly; recorded.
**Tasks**
- E07-S03-T1 CSV writer (no dependency; about 40 lines, plus tests).
- E07-S03-T2 Export service across modules through their interfaces.
- E07-S03-T3 Handler with rate limit and typing action.
- E07-S03-T4 Tests.
**Story DoD** — Default + manual spreadsheet check recorded.

### E07-S04 — Delete my account and retention jobs

**Story** — As **P1 (Maya, solo freelancer)**, I want to delete my account and everything in it at once, knowing clients won't be contacted again, so that I can leave cleanly.
**Context** — F-12 steps 2–3; SCR-18; Telegram developer terms §4.2 (delete on request) (S-07); §12.4 retention table; EC-DATA-07, EC-PERM-05, EC-PERM-07; PITR backups age out within 3 or 7 days (S-38).
**Scope** — In: `accounts.deleteAccount(freelancerId)` in one transaction (cancel nudges with reason `account_deleted`, delete the freelancer row → cascades to clients, invoices, payments, nudges, notifications, suppressions, verifications; conversation state deleted by `telegram_user_id`; audit rows' `freelancer_id` set null by FK and one `account.deleted` audit row written without the ID), CP-SCR17-11, CP-SCR18-5 to CP-SCR18-10; a daily retention job in the scheduler for every retention row of §12.4. Out: backups (platform-managed).
**Acceptance Criteria**
- AC-1 (confirm) Given 12 clients, 40 invoices and 7 planned reminders, When "Delete my account", Then CP-SCR18-5 with `clients=12`, `invoices=40`, `nudges=7`, with "Keep my account" first and "Delete everything" second; "Keep my account" → CP-SCR18-8 and no change.
- AC-2 (delete) When "Delete everything", Then within one transaction all rows owned by the freelancer are gone (a count query per table returns 0 for their IDs), no nudge of theirs can be claimed afterwards, `audit_event` rows that referenced them have `freelancer_id = null`, one `account.deleted` audit row exists, and CP-SCR18-9 is sent; the whole operation takes ≤ 60 s for the largest account (500 open + 2,000 closed invoices).
- AC-3 (fresh start) After deletion, `/start` creates a brand-new freelancer and nothing from the old account reappears (EC-DATA-07).
- AC-4 (failure) Given the transaction fails, Then CP-SCR18-10 and every row still exists.
- AC-5 (in-flight) Given a nudge in `sending` at the moment of deletion, Then the provider call may complete (the email was already handed over), but no row is recorded afterwards (the recording finds no row and logs `nudge.orphan_result`), and nothing else is sent.
- AC-6 (retention job) Given rows older than their retention (processed updates > 48 h, verifications > 24 h after expiry, notifications > 30 days, email events > 90 days, bounce suppressions > 90 days, audit > 400 days), When the daily job runs at 03:00 UTC, Then exactly those rows are deleted in batches of 1,000, and the job is idempotent (EC-PERM-07).
- AC-7 (accessibility) The consequence text states the counts in words and numbers; the destructive button is second.
- AC-8 (observability) `account.deleted` (duration_ms, row counts) and `retention.purged` (table, count); no PII.
**Edge cases** — EC-DATA-07, EC-PERM-05, EC-PERM-07, EC-CONC-05 (a deletion during a scheduler tick: claims use `SKIP LOCKED`, and the cascade waits for row locks).
**Tests**
- Integration: `test/integration/account_delete.int.test.ts` — AC-1 to AC-5, AC-7, AC-8.
- Integration: `test/integration/retention.int.test.ts` — AC-6 with a fake clock.
- E2E: FF-14 re-run after a deletion shows none of that account's fixture strings in any table.
**Tasks**
- E07-S04-T1 Deletion service and FK review (every FK to `freelancer` is `ON DELETE CASCADE` or `SET NULL` as specified in §12.4).
- E07-S04-T2 Confirm flow and copy.
- E07-S04-T3 Retention job (daily, per table, batched).
- E07-S04-T4 Tests.
**Story DoD** — Default.

### E07-S05 — Privacy page and bot command menu

**Story** — As **P3 (Sam, the client-side payer)**, I want a plain-language page explaining what this service stores about me and how to stop it, so that I can trust a reminder that arrived from a service I never signed up for.
**Context** — SCR-24; API-10; Telegram requires an accessible privacy policy (S-07); commands via `setMyCommands` (S-12); the full legal text is pending (OQ-02). This story also completes the copy catalogue, so FF-10 asserts full equality with §11 from here on.
**Scope** — In: `/privacy` page (CP-SCR24-1 to CP-SCR24-6, FF-15 rules); footer links from client pages and emails; `scripts/ops-set-commands.ts` (`setMyCommands` with CP-CMD-1 to CP-CMD-6); BotFather description and short description set manually from CP-CMD-7 and CP-CMD-8, with the privacy-policy URL set in @BotFather (S-07, S-12); FF-10 switched to full equality. Out: legal review (OQ-02).
**Acceptance Criteria**
- AC-1 (page) `GET /privacy` → 200 HTML with h1 CP-SCR24-1 and sections CP-SCR24-2 to CP-SCR24-6 in order; FF-15 passes; security headers present.
- AC-2 (accuracy) A test asserts that the processors named in CP-SCR24-4 equal the integrations listed in §12.7 (Telegram, Resend, Render, Sentry), and that CP-SCR24-2 names every encrypted field category of DR-08.
- AC-3 (links) Every reminder email footer and every client page footer links to `/privacy`.
- AC-4 (commands) After `npm run ops:set-commands` on staging, `getMyCommands` returns exactly `new`, `owed`, `clients`, `settings`, `help`, `cancel` with CP-CMD descriptions; `/admin` is not listed.
- AC-5 (catalogue complete) FF-10 compares all CP IDs in §11 with `catalog.ts` in both directions and passes.
- AC-6 (BotFather) Manual: description, short description and privacy-policy URL set for the staging and production bots; screenshots referenced in PROGRESS.md.
- AC-7 (accessibility) The page meets §9.3 (one h1, `lang`, readable at 200% zoom).
- AC-8 (observability) `ops.commands_set` log with the command count.
**Edge cases** — EC-A11Y-01, EC-A11Y-04; story-specific: the legal-text slot (CP-SCR24-6) ships as written until OQ-02 is answered; replacing it is a copy Amendment.
**Tests**
- Integration: `test/integration/privacy_page.int.test.ts` — AC-1 to AC-3, AC-7.
- Unit: `test/arch/copy-catalog.test.ts` full equality — AC-5.
- Manual: AC-4 (staging), AC-6.
**Tasks**
- E07-S05-T1 Page view and route.
- E07-S05-T2 Footer links.
- E07-S05-T3 `ops-set-commands` script.
- E07-S05-T4 FF-10 full equality; tests.
**Story DoD** — Default + BotFather settings recorded.

**Epic-level tests** — A scenario test onboards, logs invoices, exports, deletes, and re-onboards with the same Telegram user (clean state); CE-1 to CE-5 green; FF-10 full equality green.
**Definition of Done**
- All 5 stories DONE; `check` and CI green; staging deployed.
- Manual: settings changes re-plan correctly (one reminder moved from 10:00 to 09:00, observed in the next digest); an export opened in a spreadsheet app; an account deleted on staging and re-created. Recorded.
- **Public beta ready**: OQ-02 answered, or the product owner explicitly accepts launching the beta with the plain-language page only (recorded as a decision in DECISIONS.md).
**Session handoff checklist** — §18.5 summary, plus: status of OQ-02; preconditions for E08 (production Render services created; production bot token; production Resend domain verified; `NUDGES_LIVE` plan for launch).

## Epic 08 — Launch hardening

**Goal** — Prove the service can be restored, holds its budgets at the scale ceiling, runs in production with authenticated email, and tells the operator when something goes wrong. Then launch.
**Personas served** P4, P0
**Depends on** E07 (feature-complete).
**Preconditions for the session** — PROGRESS.md shows E07 DONE; `check` green. Production Render services and Postgres created (Frankfurt; paid plans; Pro workspace for 7-day PITR, OQ-05 default); production bot token; production sending subdomain verified in Resend (SPF, DKIM, DMARC); production env vars set; the product owner's decision on OQ-02 recorded. `[VERIFY-AT-BUILD]`: Render PITR windows and restore behaviour (S-38); Render and Resend prices for the budget check (OQ-05; S-26); Telegram webhook IP ranges and ports (S-13).
**Scope** — In: stories below. Out: new features (Parking lot).
**Flows and screens implemented** none new; SCR-19 operational alerts (CP-SCR19-10 to CP-SCR19-14).
**Data and API touched** all (drill, load); `scripts/*` ops scripts; `docs/RUNBOOK.md`, `docs/OPS-LOG.md`.
**Risks specific to this epic** R-10, R-11, R-16 (launch-day deliverability).

### E08-S01 — Backup restore drill and runbook

**Story** — As **P4 (Operator)**, I want a rehearsed, timed restore procedure and a key-rotation procedure, so that a bad day costs hours, not the business.
**Context** — §12.9 restore procedure; RTO ≤ 4 h, RPO ≤ 15 min (assumption 30); Render PITR restores into a new instance and not within 10 minutes of now (S-38); key rotation (§12.8); EC-OPS-04, EC-INT-05, EC-OPS-12.
**Scope** — In: `scripts/ops-verify-restore.ts` (row counts per table, latest `updated_at`, decrypt 20 random `_enc` values with current keys, report); `scripts/crypto-reencrypt.ts` (batched, idempotent, resumable by `id`); RUNBOOK sections "Restore", "Rotate data key", "Rotate blind-index key", "Rotate link key", "Lost encryption key"; one full drill on staging; the monthly drill entry in the OPS-LOG template.
**Acceptance Criteria**
- AC-1 (drill) Given staging with at least 1,000 invoices, When the §12.9 restore procedure is executed to a point 30 minutes in the past, Then staging runs on the restored instance, `ops:verify-restore` reports all tables readable and 20/20 sample decryptions OK, and the total wall time (start of restore → scheduler re-enabled) is recorded in OPS-LOG and is ≤ 4 h.
- AC-2 (no duplicates after restore) Given nudges sent after the restore point, When step 5 of the procedure is followed, Then after re-enabling the scheduler no email is sent twice (verified on staging with the email sink or Resend test addresses).
- AC-3 (re-encrypt) Given data written under key 1, When key 2 is added and made active and `npm run crypto:reencrypt` runs, Then every `_enc` column starts with byte `0x02`; re-running changes nothing; interrupting it and re-running completes it (EC-INT-05, EC-OPS-12).
- AC-4 (blind index rotation) Running the script with a new `BLIND_INDEX_KEY` recomputes every `_bidx`, and duplicate detection and suppression lookups still work (integration test).
- AC-5 (missing key) Given a ciphertext with an unknown key ID, Then the app logs `crypto.error` with `unknown_key_id` and the affected feature shows CP-SCR20-4; startup with no keys refuses to start (EC-OPS-01).
- AC-6 (runbook) RUNBOOK contains every procedure above with exact commands, expected outputs and a "stop and ask" point before any destructive step; a second person (or a fresh agent session) can follow it without questions (manual read-through recorded).
- AC-7 (copy / accessibility) n/a — operator-only.
- AC-8 (observability) Scripts log start, progress per 1,000 rows and end, with counts and durations.
**Edge cases** — EC-OPS-04, EC-INT-05, EC-OPS-12, EC-OPS-01.
**Tests**
- Integration: `test/integration/reencrypt.int.test.ts` — AC-3, AC-4.
- Integration: `test/integration/verify_restore.int.test.ts` — the script's report on a seeded DB.
- Manual: AC-1, AC-2, AC-6 on staging; recorded in OPS-LOG and PROGRESS.md.
**Tasks**
- E08-S01-T1 Verify-restore script.
- E08-S01-T2 Re-encrypt script (key and blind index modes).
- E08-S01-T3 RUNBOOK sections.
- E08-S01-T4 Staging drill.
**Story DoD** — Default + drill timings recorded.

### E08-S02 — Load test at the scale ceiling and performance budgets

**Story** — As **P0 (Builder)**, I want the budgets of §12.9 measured on the real staging instance at the scale ceiling, so that launch does not discover them.
**Context** — Scale ceiling (§2); budgets (§12.9); AD-6, AD-7; exit trigger 1 of §12.1 (webhook p95). Uses `EMAIL_GATEWAY_MODE=sink` and `TELEGRAM_GATEWAY_MODE=sink` on staging so no real messages are sent (§12.8).
**Scope** — In: `scripts/load-test.ts` (seeds 5,000 freelancers, 50,000 open invoices, 3,000 nudges due in one send window; replays synthetic webhook updates at 20/s for 10 minutes with the secret header, mixing `/owed`, card opens, payments and wizard steps; measures response times client-side); a report in `docs/OPS-LOG.md`; fixes for any budget miss (as separate commits in this story, or new stories if large).
**Acceptance Criteria**
- AC-1 (webhook) At 20 updates/s for 10 minutes, webhook acknowledgement p95 < 1 s and p99 < 3 s, with 0 non-2xx responses except intentional 401 probes.
- AC-2 (sending) 3,000 due nudges are sent (to the sink) within 15 minutes of their `scheduled_for` (M-4.1 ≥ 99% within 15 min), at ≤ 5 sends/s, and each exactly once.
- AC-3 (queries) `/owed` for a freelancer with 500 open invoices ≤ 300 ms server time; DB queries per update ≤ 12 (from the query counter in debug logs).
- AC-4 (memory and CPU) RSS stays ≤ 350 MB on the `0.5c-512mb` instance; no restarts during the test.
- AC-5 (metrics views) Each `v_metrics_*` query ≤ 500 ms on the seeded data (E06-S04 AC-8).
- AC-6 (cost) The measured resource needs fit the planned plans, and the monthly estimate (compute + DB + Resend Pro for about 90,000 emails, S-26) is ≤ US$60 (AD-7) using prices confirmed on the day [UNKNOWN → OQ-05]; if not, a DR proposes the change.
- AC-7 (cleanup) After the test, seeded data is removed by a guarded script (`APP_ENV=staging` only) and sink modes are switched off; verified.
- AC-8 (observability) The report includes p50/p95/p99, throughput, errors, RSS and CPU samples, and the scheduler tick durations from logs.
**Edge cases** — EC-DATA-03, EC-BOT-11, EC-CONC-05, EC-OPS-07 (seed and cleanup scripts refuse to run in production).
**Tests**
- Manual (staging): the load run and report; recorded.
- Integration: `test/integration/loadtest_guard.int.test.ts` — the seed and cleanup scripts refuse `APP_ENV=production`; sink modes refused outside staging.
**Tasks**
- E08-S02-T1 Load script and seeding.
- E08-S02-T2 Run, report, fix.
- E08-S02-T3 Cleanup and guard tests.
**Story DoD** — Default + report in OPS-LOG.

### E08-S03 — Production environment, domain authentication and go-live smoke

**Story** — As **P4 (Operator)**, I want production deployed through the same gate, with authenticated email and a scripted smoke test, so that launch day is uneventful.
**Context** — §13.8 (production branch, Render auto-deploy after CI); Gmail sender rules: SPF or DKIM, TLS, spam rate < 0.3%, plus DMARC and alignment for bulk (S-46); webhook set-up (S-10, S-13); `NUDGES_LIVE` rollout.
**Scope** — In: production `render.yaml` services live; production webhook and command menu set; DNS: SPF, DKIM (Resend) and DMARC `p=none` with reporting to the operator's mailbox for the first 30 days, then `p=quarantine` (a DR records the date); first production deploy via the `production` branch; the smoke checklist in RUNBOOK; rollout: `NUDGES_LIVE=allowlist` for the beta users, then `on` after 7 days with complaint rate < 0.1% and no unknown or failed nudges unexplained.
**Acceptance Criteria**
- AC-1 (deploy) Moving `production` to a green `main` commit with `git merge --ff-only` deploys to production with pre-deploy migrations; `/healthz` returns 200 with that commit's version.
- AC-2 (bot) `/start` on the production bot from the operator's phone returns CP-SCR01-1 within 3 s; `getWebhookInfo` shows the production URL, no pending errors, and `allowed_updates` as configured.
- AC-3 (email authentication) A reminder sent to the operator's Gmail and Outlook addresses (a real client fixture owned by the operator) shows SPF pass, DKIM pass and DMARC pass in the received headers, and the one-click unsubscribe link is offered by the mail client or present in the raw headers.
- AC-4 (guards) Production refuses to start with `EMAIL_RECIPIENT_OVERRIDE` set or any sink mode (tested once by a deliberately bad deploy to a throwaway service, then deleted; recorded).
- AC-5 (rollout) The `NUDGES_LIVE` steps are executed and recorded with dates; switching to `on` happens only after the 7-day criteria are met.
- AC-6 (rollback) A production rollback is rehearsed once, before launch, with a harmless release (recorded) [VERIFY-AT-BUILD Render rollback steps].
- AC-7 (copy / accessibility) The operator's own test invoice in production produces one real reminder; its "I've paid" and "stop" links and the footer privacy link open SCR-21, SCR-22 and SCR-24, each checked by hand against the §9.3 list (one h1, `lang`, visible focus, 200% zoom) and recorded.
- AC-8 (observability) Production logs visible in Render; Sentry receives a deliberate test error (then resolved); no PII in the first 200 production log lines (spot check).
**Edge cases** — EC-OPS-02, EC-OPS-03, EC-OPS-10, EC-INT-04, EC-SEC-08.
**Tests**
- Manual: AC-1 to AC-8 on production, recorded in PROGRESS.md and OPS-LOG.
- Integration: existing config guard tests (E00-S04) cover AC-4 in CI.
**Tasks**
- E08-S03-T1 DNS records and DMARC policy DR.
- E08-S03-T2 Production services, env vars, webhook, commands, BotFather texts.
- E08-S03-T3 Smoke checklist execution.
- E08-S03-T4 Rollout steps.
**Story DoD** — Default + production smoke recorded.

### E08-S04 — Alerts on error rate, send failures and backlog

**Story** — As **P4 (Operator)**, I want to be told in Telegram when errors spike, sends fail, the scheduler stalls or the outbox backs up, so that I find out before my users do.
**Context** — §12.9 alert list; M-12 to M-14 need OPS-LOG entries; M-18 churn script; external uptime check [UNKNOWN → OQ-16]; EC-OPS-06.
**Scope** — In: `operator_alert` notification kind with CP-SCR19-10 to CP-SCR19-14, an alert evaluator in the scheduler (every 5 minutes, at most once per hour per kind), `scripts/churn.ts` for M-18, OPS-LOG template sections for M-12 to M-14 and M-4.2, and the OQ-16 default applied (a free external uptime monitor on `/healthz` every 5 minutes, alerting the operator by email).
**Acceptance Criteria**
- AC-1 (error rate) Given more than 5% of updates in 15 minutes ended in the error boundary, Then one operator alert is sent, and not again within the hour.
- AC-2 (send failures) Given more than 10 nudges `failed` in 1 hour, Then one alert.
- AC-3 (stall) Given more than 20 nudges `announced` and more than 30 minutes past `scheduled_for`, Then one alert (scheduler stall).
- AC-4 (backlog) Given more than 200 `pending` notifications, Then one alert.
- AC-5 (complaints) Every complaint produces an operator alert with the freelancer's Telegram ID and 30-day complaint count.
- AC-6 (uptime) The external monitor is configured and a deliberate stop of the staging service triggers its email within 10 minutes (recorded).
- AC-7 (copy) Alert texts are exactly CP-SCR19-10 to CP-SCR19-14.
- AC-8 (observability) `alert.fired` (kind, value, threshold) logs; `scripts/churn.ts` prints the M-18 value for the last 14 days and is referenced in the handoff.
**Edge cases** — EC-OPS-06, EC-BOT-08 (operator blocked the bot → alerts dropped; the uptime monitor still emails).
**Tests**
- Integration: `test/integration/alerts.int.test.ts` — AC-1 to AC-5 with a fake clock, rate limiting per kind.
- Unit: `scripts/churn.test.ts` on a fixture git history.
- Manual: AC-6.
**Tasks**
- E08-S04-T1 Catalogue lines CP-SCR19-10 to CP-SCR19-14.
- E08-S04-T2 Alert evaluator and notification kind.
- E08-S04-T3 Churn script; OPS-LOG sections.
- E08-S04-T4 Uptime monitor set-up (OQ-16 default).
- E08-S04-T5 Tests.
**Story DoD** — Default + uptime monitor test recorded.

**Epic-level tests** — Full regression; CE-1 to CE-5 green in CI; the production smoke checklist executed.
**Definition of Done**
- All 4 stories DONE with evidence.
- Restore drill ≤ 4 h recorded; load test within budgets recorded; production live with SPF, DKIM and DMARC passing; alerts firing in a test; uptime monitor active.
- **Public launch**: `NUDGES_LIVE=on` after the 7-day beta criteria; the date recorded in PROGRESS.md.
- PROGRESS.md, DECISIONS.md and (if needed) the Amendments log updated.
**Session handoff checklist** — §18.5 summary, plus: launch status; the first week's operator checklist (daily `/admin stats`; complaint rate; unknown and failed nudges); open OQs that remain and their owners.

## 18. Session protocol for Claude Code / Codex (Session Protocol)

### 18.1 Why fresh sessions
Long agent sessions drift: they forget constraints, half-remember decisions and compress context. Each epic therefore starts in a clean session, and the repository is the memory. `docs/BLUEPRINT.md` says *what* to build, `docs/PROGRESS.md` says *where we are*, `docs/DECISIONS.md` says *why*, and `npm run check` says *whether it works*. Nothing is assumed to be remembered.

### 18.2 What lives in the repo (created in E00-S01)
```
docs/BLUEPRINT.md          this document
docs/PROGRESS.md           state: epic/story status, AC evidence, deviations, VERIFY-AT-BUILD results, next preconditions
docs/DECISIONS.md          DR-01…DR-23 copied from §4.2, plus every decision taken in sessions
docs/REVIEW-CHECKLIST.md   §14.4 verbatim
docs/RUNBOOK.md            deploy, rollback, restore, key rotation, webhook and command set-up
docs/OPS-LOG.md            dated operational events (deploys needing intervention, drills, incidents) for M-12…M-14, M-4.2
AGENTS.md / CLAUDE.md      identical entry points (Appendix B)
.env.example               every variable of §12.8
npm run check              the gate (§13.4)
```

### 18.3 Session lifecycle
**Boot (human).** Open a fresh Claude Code or Codex session at the repo root and paste the epic's prompt from Appendix A.

**Orient (agent).**
1. Read `AGENTS.md` / `CLAUDE.md`, then all of `docs/PROGRESS.md`, then §12–§16, §18, §21 of this document, the target epic in §17, and every F-, SCR-, CP-, ENT-, API-, INT- and EC- ID the epic references, plus `docs/REVIEW-CHECKLIST.md`.
2. Verify preconditions: previous epics DONE in PROGRESS.md; `git status` clean on `main`; `npm run check` green on a clean tree; the epic's env vars present; every `[VERIFY-AT-BUILD]` item for this epic (§18.10) re-checked and the result recorded in PROGRESS.md.
3. If a precondition fails, stop and report exactly what failed and the proposed fix. Do not build.

**Plan (agent).**
4. Restate the epic goal and story list in ≤ 15 lines. List any AC that is unclear or references a missing ID; there should be none, and any found is a blueprint defect (§18.7).
5. Produce the session plan: story order; per story the tests to write (from its Tests block), the tasks, and the manual checks the human will do at the end. If the epic does not fit the session, propose which stories move to a follow-up epic (e.g. `E05b`) and record it as a deviation.
6. Wait for "go" unless the prompt says to proceed without stopping.

**Build (agent), story by story.**
7. Branch `epic/E0n` once per epic.
8. Write the story's tests first; run them; they must fail. Record "seen red" per AC in PROGRESS.md.
9. Implement the tasks; run `npm run check` after each story (and more often); keep it green. User-facing text comes only from CP IDs.
10. Commit per story: `feat(nudging): send reminders exactly once [E05-S02]`. Mark the story DONE in PROGRESS.md with one evidence line per AC.
11. Any decision the blueprint does not make (a library detail, an index, a helper's location) → a `DR-nn` entry in DECISIONS.md in the same commit. Any AC that turns out wrong or impossible → §18.7, never a silent change.

**Verify (agent).**
12. Full `npm run check`; the epic-level tests; the full regression; CI green on the pushed branch.
13. **Self-review.** Walk `docs/REVIEW-CHECKLIST.md` over the epic's diff, story by story, reading the diff rather than your memory of writing it. Fix what fails. Report `reviewed: <n> items, <n> fixed, <n> accepted with reason`, never a bare tick.
14. If any story's diff exceeded 400 changed lines, say so in the handoff; it signals mis-sizing and deserves an Amendment for the next epic.
15. Deploy to staging if the DoD requires it; run the smoke checks; record results.
16. Walk the epic's Definition of Done line by line; each line gets evidence, or the epic is not done.

**Handoff (agent).**
17. Update PROGRESS.md (epic and story status, AC evidence, deviations, accepted risks, VERIFY-AT-BUILD results, next preconditions, one session-log line), DECISIONS.md, and the Amendments log if anything changed.
18. Final commit; open or merge the PR per §13.7 once CI is green.
19. Print the handoff summary (§18.5) and say: "Safe to terminate this session. Next: open a fresh session and paste the E0n+1 prompt."

**Terminate (human).** Close the session and review the PR with the same checklist. Next epic: new session.

### 18.4 Gates (an epic is DONE only when all are true)
- Every story DONE with an evidence line per AC (a passing test name, or a dated manual observation).
- `npm run check` green on the final commit; CI green (including the mutation job when triggered).
- Epic-level tests and all previous epics' tests green.
- Staging smoke checks passed where the DoD requires them.
- The review checklist applied to every story, with results recorded (which items failed and what was done).
- No gate rule weakened, disabled or bypassed; if one was, a DR explains why and a follow-up story restores it.
- No new skipped tests or open markers without a matching PROGRESS.md "accepted risk" line with owner and reason. <!-- lint-ignore -->
- No new flaky test; any flake is quarantined with a follow-up story.
- PROGRESS.md, DECISIONS.md and (if needed) the Amendments log updated and committed.

A gate that cannot be met leaves the epic IN PROGRESS with an exact list of what remains. Partial is fine; pretending is not.

### 18.5 Handoff summary format (≤ 20 lines)
```
## Handoff — E05 Client email nudges with baseline safety — 2026-11-12
Status: DONE | IN PROGRESS (remaining: E05-S06 AC-4; staging bounce run)
Stories: E05-S01 DONE · E05-S02 DONE · E05-S03 DONE · E05-S04 DONE · E05-S05 DONE · E05-S06 IN PROGRESS
check: green (commit abc1234) · CI: green · mutation: 74 (nudging/domain) · staging: deployed, smoke 3/3
Self-review (§14.4): reviewed 41 items, 3 fixed (missing freelancer_id scope in stop handler;
  one test asserted only length; CP-SCR22-8 rendered with an improvised heading), 1 accepted
  (jscpd flags two HTML view helpers; extraction deferred with reason in PROGRESS.md).
Diff size: E05-S02 was 520 lines, over 400 — mis-sized; proposed split recorded as amendment A-02.
Deviations: none | E05-S05 AC-5 cap 30 → 25 (Amendment A-03, approved by product owner on 2026-11-11)
Accepted risks: EC-MSG-01 relies on provider behaviour; no automatic resend by design
VERIFY-AT-BUILD: Resend idempotency ✔ 2026-11-10 · DKIM covers list-unsubscribe ✔ · Svix verify ✔
Decisions added: DR-24 (raw-body parser scope), DR-25 (notice batching threshold)
Next epic E06 preconditions: set ADMIN_TELEGRAM_IDS in staging; nothing else
Manual checks for you: open a staging reminder's "I've paid" link on your phone; confirm the notice
Safe to terminate. Next: fresh session, paste the E06 prompt.
```

### 18.6 PROGRESS.md and DECISIONS.md
- PROGRESS.md is a **state file, not a diary**: current status per epic and story, evidence lines, deviations, accepted risks, VERIFY-AT-BUILD results, preconditions, and a one-line-per-session log at the bottom. Evidence is specific: `AC-3 ✔ test/integration/send_pipeline.int.test.ts::recheck_cancels_paid` or `AC-6 ✔ manual 2026-11-12 — staging /start answered in 1.4 s on Android`.
- DECISIONS.md holds `DR-nn` entries (context → options → decision → consequences → evidence), newest last. DR-01 to DR-23 are copied from §4.2 in E00; IDs are never reused; superseded entries stay with their status changed.
- Formats: Appendix B.

### 18.7 Amendments log (changing this blueprint from a session)
The blueprint is the source of truth, so when reality disagrees, the blueprint changes explicitly:
1. Never implement something different from an AC silently.
2. Add a row to the table below.
3. Edit the affected AC, copy line or architecture text in place and mark it `(amended A-nn)`.
4. If other epics are affected, add a line to their preconditions.
5. Scope additions are not amendments: they are new stories in a later epic; note them in PROGRESS.md "Parking lot".

Ask the product owner **before** amending anything that changes user-visible behaviour (including copy), money, data retention or security. Smaller technical amendments can be made and reported in the handoff.

| ID | Date | Epic / story | What changed | Why | Approved by |
|---|---|---|---|---|---|
| A-00 | 2026-09-30 | — | Log created with the approved blueprint v1.0; no amendments yet | Baseline | Product owner ("go") |

### 18.8 Recovery
- **Session crashed mid-epic.** The next session reads PROGRESS.md, runs `git status` and `git log`, inspects `epic/E0n`, runs `check`, and resumes from the first story or AC without evidence. Uncommitted work is inspected, then either committed with a note or discarded; PROGRESS.md records which.
- **`check` red at boot.** The previous session left a violated gate. Fix it first as a hotfix commit `fix: restore green` with its own PROGRESS.md evidence, then continue.
- **Blueprint defect** (missing ID, contradictory ACs). Treat it as §18.7; if it blocks, stop and ask.
- **Environment broken** (a service down, credentials expired). Record it in PROGRESS.md, report, stop.

### 18.9 Claude Code vs Codex
- Claude Code reads `CLAUDE.md`; Codex reads `AGENTS.md`. Both files are identical and point to the docs above.
- The protocol depends only on repo files, not on any skill being installed at build time.
- Paste the Appendix A prompt as the first message. If the tool exposes an effort or thinking level, use the maximum for planning and verification.
- Every command the agent runs must be non-interactive (no watch mode, no prompts); `npm run check` exits non-zero on failure.

### 18.10 `[VERIFY-AT-BUILD]` re-check list
At Orient time the agent re-checks the items for its epic (a docs page, a registry query, or a sandbox call) and records `✔ date` or `✘ date + what changed` in PROGRESS.md. On ✘ it stops and reports before building on the item.

| # | Item (where tagged) | How to re-check | First epic |
|---|---|---|---|
| V-01 | Package versions in §13.1 (record newer versions, don't upgrade) | `npm view <pkg> version` for each | E00 |
| V-02 | Node 26 LTS status (LTS from 2026-10-28, S-32) | nodejs.org release schedule | E00 (again E08) |
| V-03 | Bot API version and grammY support (§3.6, S-10, S-59) | Bot API changelog; grammY releases | E00 |
| V-04 | Webhook ports, TLS, source IPs (§3.6, S-13) | Telegram webhook guide | E00 (again E08) |
| V-05 | GitHub-hosted runners include Docker (§13.1) | first CI run of Testcontainers tests | E00 |
| V-06 | Sentry free-plan limits and SDK privacy option names (§12.7) | Sentry pricing and Node SDK docs | E00 |
| V-07 | Render rollback steps (§13.8) | Render docs + rehearsal | E00 (again E08) |
| V-08 | `node:crypto` algorithms available (§13.1) | E01-S01 unit tests | E01 |
| V-09 | Telegram message length limit 4,096 (§3.6) | Bot API `sendMessage` reference | E01 |
| V-10 | Resend free-tier limits and pricing (§12.7, S-26) | Resend pricing page | E01 (again E08) |
| V-11 | ISO 4217 list current (S-68) | SIX list "Pblshd" date | E02 |
| V-12 | Temporal `disambiguation` option name (E02-S05 rule 6) | MDN / TC39 Temporal docs | E02 |
| V-13 | Telegram flood limits and grammY auto-retry options (S-08, S-56) | Bots FAQ; plugin docs | E04 |
| V-14 | Resend idempotency semantics, webhook event names and message-ID field, SDK `webhooks.verify` (S-27, S-29, S-89) | docs + contract test | E05 |
| V-15 | Resend DKIM signature covers `List-Unsubscribe` headers (S-47) | raw headers of a received email | E05 |
| V-16 | Fastify `addContentTypeParser` with `parseAs: "string"` (E05-S06) | Fastify 5 docs | E05 |
| V-17 | Svix timestamp tolerance (E05-S06) | Resend / Svix docs | E05 |
| V-18 | Telegram document upload limit and BotFather `/setdescription` (S-08, S-12) | Bots FAQ; BotFather | E07 |
| V-19 | CSV with BOM opens correctly (assumption 34) | open in a spreadsheet app | E07 |
| V-20 | Render PITR windows, plans and prices (S-38, OQ-05) | Render docs and pricing | E08 |
| V-21 | Render's Node 26.10.0 runtime has Temporal enabled (DR-04 build caveat, S-91) | `/healthz` or a one-off shell: `node -e "console.log(typeof Temporal)"` on the staging instance | E00 |

## 19. Risks and mitigations (Risks)

| ID | Risk | Likelihood | Impact | Mitigation | Owner | Trigger to revisit |
|---|---|---|---|---|---|---|
| R-01 | Shared sending-domain reputation damaged by one abusive account or bad address lists | Medium | High: every user's reminders land in spam | Verified reply-to (DR-16); templates only; caps (assumption 21); one-click unsubscribe (DR-13); bounce and complaint suppression (E05-S06); auto-suspension (E06-S01); DMARC reporting (E08-S03) | P4 | Complaint rate ≥ 0.1% over 30 days (M-3.3) |
| R-02 | Telegram changes bot rules, limits or terms, or restricts the bot | Low | High | Follow developer terms (S-07); pinned grammY; VERIFY-AT-BUILD V-03; data export (E07-S03) keeps users' data portable | P4 | A Bot API changelog entry affecting webhooks or messaging |
| R-03 | A reminder reaches a client after the invoice was paid | Medium | High (trust) | Digest with Hold (DR-12); one-tap Paid; eligibility re-check just before sending (E05-S02); client "I've paid" page (E05-S03) | P0 | M-3.2 > 2% |
| R-04 | Reminder emails are classed as commercial, or our GDPR role differs from the design | Medium | Medium–High | OQ-02, OQ-03 to counsel before public launch; opt-out built in regardless; no marketing content in reminders | Product owner | Counsel's answer |
| R-05 | Node 26 is "Current", not LTS, until 2026-10-28 | Low | Low | Pin 26.10.0; upgrade to the first LTS patch; V-02 | P0 | 2026-10-28 |
| R-06 | TypeScript 7 moves on while typescript-eslint supports only < 6.1 | Medium | Low | Pin TS 6.0.3 (DR-04); re-evaluate when typescript-eslint supports TS 7 | P0 | typescript-eslint release supporting TS 7 |
| R-07 | Competitors add the same wedge (e.g. invioTrack starts emailing clients) | Medium | Medium | Differentiate on trust features (digest, "I've paid", opt-out) and tracking invoices issued anywhere | Product owner | A competitor announcement |
| R-08 | Freelancers don't want a bot emailing their clients | Medium | High (adoption) | Plans include Off; per-invoice pause; text to forward (E06-S03); honest From line | Product owner | Share of invoices with plan Off > 50% after 8 weeks |
| R-09 | Scheduler stalls or misses runs (outage, bug) | Low | Medium | Catch-up rules (DR-21, EC-TIME-13); stall alert (E08-S04); M-4.1 | P0 | M-4.1 < 99% |
| R-10 | Loss of `DATA_ENCRYPTION_KEYS` makes all PII unreadable | Low | Critical | Two copies of every key (§12.8); rotation runbook; restore drill verifies decryption (E08-S01) | P4 | Any key-handling change |
| R-11 | Infrastructure cost exceeds AD-7 (US$60/month) | Medium | Low–Medium | Measure in E08-S02; OQ-05 prices; Resend plan tiering (S-26) | P4 | Monthly bill > US$60 |
| R-12 | Render plan names or prices change (plans were renamed 2026-08-26, S-42), so the wrong plan is chosen at setup | Low | Low | Choose by plan ID (`0.5c-512mb`); confirm prices on the day (OQ-05) | P4 | Render pricing change notice |
| R-13 | Sending-domain DNS setup delays verification emails in staging | Medium | Low (delays E01) | Set up DNS during E00 (OQ-07); the E01 precondition lists it | Product owner | E01 blocked |
| R-14 | Amount or date parsing produces a wrong value the user doesn't notice | Low | Medium | Strict rules; ambiguous date formats rejected; the summary shows the parsed value before Save; mutation-tested parsers | P0 | A user-reported wrong amount |
| R-15 | Auto-suspension hits a good user (two complaints from one hostile client) | Low | Medium | Operator alert and review within 2 working days; unsuspend restores (E06-S02) | P4 | Any unsuspend after an auto-suspension |
| R-16 | A new sending domain has no reputation on launch day, so early reminders go to spam | Medium | Medium | Staged rollout (allowlist, then on after 7 days); DMARC reports; low early volume | P4 | Complaint or bounce spike in week 1 |
| R-17 | A Node 26 build without Temporal support (as observed in a Homebrew build, S-91) breaks AD-3's runtime assumption on a developer machine or on the host | Medium | Medium | Official nodejs.org binaries; startup check (E00-S04 AC-2); V-21 on Render; fallback `temporal-polyfill` behind `platform/time` (DR-04) | P0 | `runtime.temporal_missing` at startup anywhere |

## 20. Open questions (Open Questions)

| ID | Question | Why it matters | Owner | Blocking? | Proposed default if unanswered |
|---|---|---|---|---|---|
| OQ-01 | Final product name and Telegram bot usernames (staging, production); trademark check | Copy, domain and bot identity | Product owner | E08 (production bot); not blocking earlier | Keep "InvoiceNudge" as the working name; staging bot uses any available username |
| OQ-02 | Operator's legal entity and jurisdiction; GDPR roles (controller for freelancer data; processor for client contacts on behalf of freelancers?); data processing terms; full privacy policy text | Legal basis for handling third-party contact data; Telegram requires a privacy policy (S-07) | Legal counsel | Public beta (E07 DoD) | Invite-only beta with the plain-language page (CP-SCR24-2 to CP-SCR24-5) |
| OQ-03 | Are reminders "transactional or relationship" messages under CAN-SPAM (S-06) and exempt from equivalent EU/UK marketing rules? Is a postal address required in the footer? | Footer content and legal exposure | Legal counsel | Public launch (E08) | Keep one-click opt-out; add the operator's postal address to CP-EML-22 only if counsel says the emails are commercial |
| OQ-04 | Does field-level encryption with environment-held keys plus provider disk encryption satisfy Telegram developer terms §4.4 (S-07)? | Compliance with platform terms | Legal counsel / Telegram | No | DR-08 as designed |
| OQ-05 | Render prices for `0.5c-512mb` web services, Postgres plans and workspace tiers (Hobby 3-day vs Pro 7-day PITR, S-38); the pricing page could not be read this session | Cost ceiling AD-7; backup window | Product owner | E00 (plan choice), E08 (budget check) | Smallest paid web instance and smallest paid Postgres per environment; Pro workspace before public launch |
| OQ-06 | GitHub account: Pro (protected branches on private repos, S-51) or a public repository? | Whether the gate can be enforced on `main` | Product owner | E00-S07 | GitHub Pro, private repo |
| OQ-07 | Product domain and sending subdomains; who manages DNS | Email authentication and deliverability | Product owner | E01 (staging verification email), E05 | Register a domain for the final name; `send-staging.<domain>` and `send.<domain>` with SPF, DKIM, DMARC |
| OQ-08 | Monetization after the beta (price, free limits, Telegram Stars vs other) | Business model; Stars required for digital goods sold in Telegram (S-07) | Product owner | No | Free during beta; decide after 8 weeks of M-5.1 data |
| OQ-09 | May copy mention late fees, statutory interest or legal remedies (EU EUR 40, S-05; NY FIFA, S-04)? | Tone and legal exposure | Legal counsel | No | Never mention them in v1 |
| OQ-10 | Support email address and response-time promise | CP-SCR15-12 and CP-SCR20-10 show it | Product owner | E06 (`SUPPORT_EMAIL`) | `support@<domain>`, answered within 2 working days |
| OQ-11 | Resend sending region for data residency (regions are selectable per domain, S-74) | EU data locality | Product owner / vendor | No | Choose an EU region if offered when creating the domain; record in DECISIONS.md |
| OQ-12 | How do Telegram's iOS, Android and Desktop apps expose inline keyboards to screen readers? | Accessibility of the bot surface | Design / QA | No | Text-first rule (§9.3) and manual checks per UI epic |
| OQ-13 | Should reminders use the client's time zone when known? | Civil send times for cross-continent clients | Product owner | No | Freelancer's zone in v1 |
| OQ-14 | Do some users need working days other than Monday–Friday? | Weekend rules | Product owner | No | Monday–Friday for everyone in v1 (assumption 16) |
| OQ-15 | Should the "I've paid" page accept a short free-text note from the client? | Richer signal vs moderation and abuse | Product owner | No | No free text in v1 |
| OQ-16 | Which external uptime monitor watches `/healthz`? | The app cannot alert about its own death | Operator | E08-S04 AC-6 | Any free monitor with 5-minute checks and email alerts; record in DECISIONS.md |

## 21. Glossary

| Term | Definition | Also known as / rejected synonyms | Where used |
|---|---|---|---|
| Freelancer | The bot user who issues invoices and wants to be paid (P1, P2) | not "user" in copy, "account holder" | everywhere |
| Client | The business or person who owes the freelancer money | not "customer", "debtor" | §11, ENT-Client |
| Contact name | The first name greeted in reminder emails | greeting | CP-EML-8, SCR-16 |
| Invoice | A bill the freelancer already issued elsewhere and logs here | not "bill", "request" | everywhere |
| Reference | The freelancer's invoice number or code, shown to clients | invoice number, ref | ENT-Invoice, CP-EML |
| Due date | The calendar date by which payment is due, in the freelancer's calendar | — | DR-11 |
| Overdue | Open with the freelancer's local today after the due date | late | §10.5 |
| Balance / owed | Amount minus recorded payments | outstanding | CP-SCR11, CP-EML |
| Paid | Fully settled (payments sum equals the amount) | settled | ENT-Invoice |
| Part payment | A payment smaller than the balance | not "partial", "instalment" | SCR-13 |
| Cancelled | Voided by the freelancer; kept in history; no reminders | void | SCR-12 |
| Reminder | An email sent to a client about an unpaid invoice (user-facing word) | "nudge" in code only | §11 |
| Nudge | Internal name for one scheduled reminder (a row in ENT-Nudge) | reminder (user-facing) | §12 |
| Reminder plan | The preset schedule for an invoice: Gentle, Standard, Firm, Off | cadence, sequence | CP-SCR10-12 |
| Step | One reminder within a plan, with a kind: pre_due, friendly, neutral, firm, final | stage | ENT-Nudge |
| Evening plan / digest | The 18:00 message listing tomorrow's reminders | preview | SCR-14 |
| Announced | A nudge listed in a delivered digest; the precondition for automatic sending | — | DR-12 |
| Hold | Skip one reminder from the digest | — | CP-SCR14-4 |
| Pause | Stop all reminders for an invoice until resumed | — | CP-SCR12-11 |
| Send now | Send the next reminder immediately (bypasses the digest) | — | CP-SCR12-13 |
| Send hour | The local hour (08–17) at which reminders go out on weekdays | reminder time | SCR-17 |
| Reply-to email | The freelancer's verified address that receives client replies | — | DR-16 |
| Stop reminders | Client action that ends reminders for one invoice | — | SCR-22 |
| Unsubscribe | Client action that ends all reminders from one freelancer to that address | opt-out | DR-13 |
| One-click unsubscribe | RFC 8058 mechanism: POST `List-Unsubscribe=One-Click` (S-47) | — | API-07 |
| Suppression | A record that an address must not receive reminders (bounce, complaint, unsubscribe) | block list | ENT-Suppression |
| Bounce | Permanent delivery failure reported by the provider (S-29) | hard bounce | E05-S06 |
| Complaint | Recipient marked a reminder as spam (S-29) | spam report | E05-S06 |
| Dunning | Finance term for the process of chasing overdue invoices | chasing | §3.1 |
| Operator | The person running the service (P4) | admin | SCR-19 |
| Outbox | Table of pending Telegram notifications to freelancers | — | ENT-Notification |
| Blind index | HMAC of a normalised value, used to find encrypted values without decrypting | — | DR-08 |
| Idempotency key | A key that makes a retried request produce one effect (`nudge/<id>`, S-27) | — | DR-06 |
| Fitness function | An automated check that enforces an architectural rule | architecture test | §12.10 |
| `check` | The one command that defines green (`npm run check`) | the gate | §13.4 |
| DR / Amendment | Decision record / an explicit change to this blueprint | ADR | §4.2, §18.7 |

## 22. Sources

Every source below was read during this session (2026-09-30) through web search, page fetches, or direct reads of public registries and repositories (npm, PyPI, Node.js, GitHub API, SIX). Tier-4 and tier-5 sources (reviews, listicles) were used only as leads and are not cited as evidence. Where two sources conflicted (S-20 vs S-21), both are listed and the choice is explained in §3.2.

| ID | Title | URL | Accessed | Supports |
|---|---|---|---|---|
| S-01 | invioTrack launch post (Indie Hackers) | https://www.indiehackers.com/post/launched-inviotrack-a-telegram-invoicing-bot-for-freelancers-no-app-no-card-fees-3a33ac9c8e | 2026-09-30 | invioTrack features: PDF invoices, reminders notify freelancer, no payment processing, pricing free/3 inv, Pro $20, Business $40; Python/FastAPI/SQLite/Hetzner; launched May 2026 |
| S-02 | Bonsai — How often do freelancers get paid late? (pub 2026-01-08) | https://www.hellobonsai.com/blog/late-freelance-payment | 2026-09-30 | 29% invoices paid late; 100k+ freelancers, 3y data; >75% of late invoices paid within 14 days of due, 90% within a month; >$20k 3x more likely late than <$100 |
| S-03 | Remote — Reversing late payment culture (pub 2025-02-19) | https://remote.com/blog/contractor-management/reversing-late-payment-culture | 2026-09-30 | 85% freelancers paid late at least sometimes; ~21% paid late >half the time; 49% companies manual contractor billing (Remote Contractor Mgmt Report 2025) |
| S-04 | NY DOL — Freelance Isn't Free Act | https://dol.ny.gov/freelance-isnt-free-act | 2026-09-30 | NYS FIFA effective 2024-08-28, Article 44-A GBL; complaints via NY AG; model contract |
| S-05 | Your Europe — B2B late payments | https://europa.eu/youreurope/business/finance-and-tax/making-receiving-payments/late-payment/index_en.htm | 2026-09-30 | EU B2B default 30 days; EUR 40 fixed compensation per late invoice; member-state interest rates; not for B2C; page updated 2026-08-05 |
| S-06 | FTC — CAN-SPAM Act: A Compliance Guide for Business | https://www.ftc.gov/business-guidance/resources/can-spam-act-compliance-guide-business | 2026-09-30 | transactional/relationship exemption; primary purpose test; commercial reqs (address, opt-out 10 business days); both sender and promoted company liable; up to $53,088 per email |
| S-07 | Telegram Bot Platform Developer Terms of Service | https://telegram.org/tos/bot-developers | 2026-09-30 | §4 privacy policy required; §4.2 delete user data on request; §4.4 data encrypted at rest, stored separately from key; §5.2 no unsolicited spam; §6.2.5 30 msg/s free broadcast, 0.1 Star above; digital goods via Stars only |
| S-08 | Telegram Bots FAQ | https://core.telegram.org/bots/faq | 2026-09-30 | ~1 msg/s per chat; 20 msg/min per group; ~30 msg/s broadcast (paid up to 1000); webhook ports 443/80/88/8443; download 20 MB, upload 50 MB; no long polling while webhook set |
| S-09 | grammY — Scaling Up IV: Flood Limits | https://grammy.dev/advanced/flood | 2026-09-30 | on 429 wait retry_after then retry; auto-retry plugin; against pre-emptive throttling |
| S-10 | Telegram Bot API reference | https://core.telegram.org/bots/api | 2026-09-30 | Bot API 10.3 (2026-08-24); updates kept ≤24 h; setWebhook secret_token 1-256 chars [A-Za-z0-9_-] header X-Telegram-Bot-Api-Secret-Token; max_connections default 40 (1-100); retries non-2xx "reasonable amount of attempts"; User.language_code IETF tag |
| S-11 | Telegram Bot API changelog | https://core.telegram.org/bots/api-changelog | 2026-09-30 | 2026 releases 9.6 (Apr 3), 10.0 (May 8), 10.1 (Jun 11), 10.2 (Jul 14), 10.3 (Aug 24) |
| S-12 | Telegram Bot Features | https://core.telegram.org/bots/features | 2026-09-30 | deep link start param A-Za-z0-9_- ≤64 chars; commands ≤32 chars, setMyCommands scopes/language; inline vs reply keyboards; privacy mode; Stars (XTR) for digital goods; respond to all messages |
| S-13 | Telegram — Marvin's Marvellous Guide to All Things Webhook | https://core.telegram.org/bots/webhooks | 2026-09-30 | ports 443/80/88/8443; TLS1.2+; IPv4 only; source subnets 149.154.160.0/20 and 91.108.4.0/22 |
| S-14 | grammY API reference — InlineKeyboardButton.CallbackButton | https://grammy.dev/ref/types/inlinekeyboardbutton/callbackbutton | 2026-09-30 | callback_data 1-64 bytes |
| S-15 | grammY API reference — Api (getFile) | https://grammy.dev/ref/core/api | 2026-09-30 | getFile link valid at least 1 hour; bots download up to 20MB |
| S-16 | invioTrack homepage | https://inviotrack.com | 2026-09-30 | Telegram bot; /new 4 questions → PDF; follow-ups days 1,3,7,14,30 (recipient/channel ambiguous on page); Free 3 invoices/2 clients; Pro $20/mo; Business $40/mo; EN/ES/PT; no payment processing; WhatsApp coming soon |
| S-17 | FreshBooks pricing | https://www.freshbooks.com/pricing | 2026-09-30 | Lite $23/mo (5 billable clients), Plus $43 (50), Premium $70 (unlimited), Select custom; promos; all plans: scheduled late fees + automated late payment reminders; 30-day trial |
| S-18 | Zoho Invoice pricing | https://www.zoho.com/us/invoice/pricing/ | 2026-09-30 | free; 500 invoices/yr; 2 users; automated payment reminders; "Powered by Zoho Invoice" branding |
| S-19 | Chaser pricing | https://www.chaserhq.com/pricing | 2026-09-30 | Compact $259/mo, Core $779, Complete $1,169 (US); email + SMS reminders, automated calls; Xero/QuickBooks/Sage integrations |
| S-20 | Landolio blog — automated invoice reminder tools (pub 2026-02-19) | https://landolio.com/blog/automated-invoice-reminder-tools-freelancers | 2026-09-30 | describes Landolio as automated email escalation (free 3 clients, Pro £9/mo); lists FreshBooks, Xero, Invoice Ninja, Bonsai |
| S-21 | Landolio homepage | https://landolio.com | 2026-09-30 | CONFLICT with S-20: homepage now sells one-time £19 "Getting-Paid Toolkit" (37 email templates, escalation system) + free tools; no SaaS plan shown |
| S-22 | Bonsai pricing | https://www.hellobonsai.com/pricing | 2026-09-30 | Basic $15/Essentials $25/Premium $39/Elite $59 per user/mo monthly ($9/$19/$29/$49 yearly); reminders not listed on pricing table |
| S-23 | GitHub API — invoiceninja/invoiceninja | https://api.github.com/repos/invoiceninja/invoiceninja | 2026-09-30 | source-available (license NOASSERTION), Laravel, not archived, pushed 2026-09-29, ~10.1k stars |
| S-24 | Postmark pricing | https://postmarkapp.com/pricing | 2026-09-30 | Free 100/mo; Basic $15/10k; Pro $16.50/10k; Platform $18/10k; separate transactional/broadcast streams; 45-day retention |
| S-25 | Postmark support — What is an idempotency key? | https://postmarkapp.com/support/article/what-is-an-idempotency-key | 2026-09-30 | Postmark does not currently support idempotency keys |
| S-26 | Resend pricing | https://resend.com/pricing | 2026-09-30 | Free $0: 3,000/mo, 100/day, 3 domains, 30-day retention; Pro $20/mo 50k ($0.90/1k overage), $35/100k; Scale from $90 |
| S-27 | Resend docs — Idempotency keys | https://resend.com/docs/dashboard/emails/idempotency-keys | 2026-09-30 | Idempotency-Key header ≤256 chars, kept 24 h; same key+payload returns same response; different payload → 409 invalid_idempotent_request; POST /emails and /emails/batch |
| S-28 | Resend docs — Send test emails | https://resend.com/docs/dashboard/emails/send-test-emails | 2026-09-30 | delivered@/bounced@/complained@/suppressed@resend.dev with +labels |
| S-29 | Resend docs — Webhook event types | https://resend.com/docs/webhooks/event-types | 2026-09-30 | email.sent/delivered/bounced/complained/delivery_delayed/opened/clicked/failed/suppressed/scheduled/received |
| S-30 | Resend docs — Verify webhook requests | https://resend.com/docs/webhooks/verify-webhooks-requests | 2026-09-30 | Svix signing; svix-id/svix-timestamp/svix-signature; resend.webhooks.verify(); raw body required |
| S-31 | npm registry (package metadata, latest dist-tags) | https://registry.npmjs.org/ | 2026-09-30 | grammy 1.46.0 MIT (2026-08-26); @grammyjs/types 5.0.0; @grammyjs/auto-retry 2.0.2; fastify 5.12.5; kysely 0.29.6 (node>=22); kysely-codegen 0.20.0 (--verify); pg 8.23.0; zod 4.6.5; pino 10.3.1; vitest 5.0.2 (node ^22.12//^24//>=26); @vitest/coverage-v8 5.0.2; eslint 10.11.0; @eslint/js 10.0.1; typescript-eslint 8.71.0 (peer typescript >=4.8.4 <6.1.0); typescript latest 7.0.2, 6.0.3 available; prettier 3.9.9; @biomejs/biome 2.5.14; dependency-cruiser 18.4.0; @stryker-mutator/core + vitest-runner 10.0.0; jscpd 5.3.3; knip 6.38.0; @testcontainers/postgresql 12.2.0; @sentry/node 11.1.0; resend 6.31.0; postmark 5.1.0; drizzle-orm 0.45.3 (1.0.0-rc.5 in flight); @prisma/client 7.10.0, prisma CLI latest tag 8.0.0-rc.19; pg-boss 12.35.0; graphile-worker 0.18.0; tsx 4.23.15; @types/node 26.6.3; eslint-config-prettier 10.1.8; @vitest/eslint-plugin 1.6.27; @fastify/rate-limit 11.2.0; @fastify/formbody 9.0.0; temporal-polyfill 1.0.5; luxon 3.7.2 |
| S-32 | Node.js release index + Release schedule | https://nodejs.org/dist/index.json ; https://raw.githubusercontent.com/nodejs/Release/main/schedule.json | 2026-09-30 | v24.21.0 (2026-09-07) LTS Krypton, maintenance from 2026-10-20, EOL 2028-04-30; v26.10.0 Current, LTS from 2026-10-28, EOL 2029-04-30; v22 Jod EOL 2027-04-30 |
| S-33 | Announcing TypeScript 7.0 (Microsoft devblog) | https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/ | 2026-09-30 | TS 7.0 GA 2026-07-08, Go native; no stable programmatic API until 7.1; tools like typescript-eslint rely on TS 6 via @typescript/typescript6 alias; strict default since 6.0 |
| S-34 | Kysely docs — Migrations | https://kysely.dev/docs/migrations | 2026-09-30 | Migrator + FileMigrationProvider, migrateToLatest, up/down, DB-level lock, alphanumeric order enforced |
| S-35 | Kysely API — SelectQueryBuilder | https://kysely-org.github.io/kysely-apidoc/interfaces/SelectQueryBuilder.html | 2026-09-30 | forUpdate(), skipLocked(), noWait() |
| S-36 | kysely-codegen README | https://github.com/RobinBlomberg/kysely-codegen | 2026-09-30 | --verify (types up to date), --out-file, --dialect postgres, --camel-case, --runtime-enums |
| S-37 | PostgreSQL versioning policy | https://www.postgresql.org/support/versioning/ | 2026-09-30 | PG 18.6 current (18.0 2025-09-25, EOL 2030-11-14); 5-year support |
| S-38 | Render docs — Postgres backups | https://render.com/docs/postgresql-backups | 2026-09-30 | PITR paid DBs: Hobby 3 days, Pro+ 7 days; restores into new instance; logical backups retained 7 days |
| S-39 | Render docs — Regions | https://render.com/docs/regions | 2026-09-30 | Oregon, Ohio, Virginia, Frankfurt, Singapore |
| S-40 | Render docs — Deploy for Free | https://render.com/docs/free | 2026-09-30 | free web services spin down after 15 min idle; free Postgres expires after 30 days |
| S-41 | Render docs — Deploys | https://render.com/docs/deploys | 2026-09-30 | build / pre-deploy (migrations; paid only) / start; zero-downtime with health check; SIGTERM + 30 s default shutdown delay; auto-deploy "After CI Checks Pass" |
| S-42 | Render docs — Compute plans | https://render.com/docs/compute-plans | 2026-09-30 | plan IDs e.g. 0.5c-512mb (Starter), 1c-2g (Standard); renamed 2026-08-26 no price change; prices not on docs page |
| S-43 | Fly.io docs — Pricing | https://docs.fly.io/about/pricing/ | 2026-09-30 | shared-cpu-1x 256MB ~$1.97/mo, 512MB ~$2.96, 1GB ~$4.94 (Ashburn base); MPG pricing on separate page |
| S-44 | GitHub API — gitleaks releases | https://api.github.com/repos/gitleaks/gitleaks/releases/latest | 2026-09-30 | gitleaks v8.30.1 (2026-03-21) |
| S-45 | GitHub API — osv-scanner releases | https://api.github.com/repos/google/osv-scanner/releases/latest | 2026-09-30 | osv-scanner v2.6.0 (2026-09-14) |
| S-46 | Google Workspace Admin Help — Email sender guidelines | https://support.google.com/a/answer/81126 | 2026-09-30 | all senders: SPF or DKIM, PTR, TLS, spam rate <0.3%; bulk (5,000+/day): SPF and DKIM, DMARC, alignment, one-click unsubscribe for marketing/subscribed |
| S-47 | RFC 8058 — Signaling One-Click Functionality for List Email Headers | https://www.rfc-editor.org/rfc/rfc8058 | 2026-09-30 | List-Unsubscribe HTTPS URI + List-Unsubscribe-Post: List-Unsubscribe=One-Click; POST; no redirect/cookies; headers DKIM-signed; prefetch rationale |
| S-48 | W3C — WCAG 2.2 | https://www.w3.org/TR/WCAG22/ | 2026-09-30 | W3C Recommendation 12 Dec 2024; 2.5.8 Target Size (Minimum) AA; 3.3.7 Redundant Entry; 3.3.8 Accessible Authentication (Minimum) |
| S-49 | OWASP Top 10:2025 | https://top10.owasp.org/2025 | 2026-09-30 | A01 Broken Access Control … A03 Software Supply Chain Failures … A10 Mishandling of Exceptional Conditions |
| S-50 | DORA — DORA's software delivery performance metrics | https://dora.dev/guides/dora-metrics/ | 2026-09-30 | five metrics: change lead time, deployment frequency, failed deployment recovery time (throughput); change fail rate, deployment rework rate (instability); updated 2026-01-05 |
| S-51 | GitHub Docs — GitHub's plans | https://docs.github.com/en/get-started/learning-about-github/githubs-plans | 2026-09-30 | Free personal: no protected branches in private repos; 2,000 Actions min/mo; Pro: protected branches in private repos, 3,000 min/mo |
| S-52 | Veracode — Spring 2026 GenAI Code Security Update (pub 2026-03-24) | https://www.veracode.com/blog/spring-2026-genai-code-security/ | 2026-09-30 | security pass rate 55%, syntax 95%+, 150+ models, Java/JS/C#/Python, 80 tasks, 4 CWEs |
| S-53 | grammY — CopyTextButton reference | https://grammy.dev/ref/types/copytextbutton | 2026-09-30 | copy_text 1-256 characters |
| S-54 | grammY — Long Polling vs. Webhooks | https://grammy.dev/guide/deployment-types | 2026-09-30 | slow webhook → Telegram re-sends update (duplicates); webhookCallback default 10 s timeout; use a task queue for long work |
| S-55 | grammY — webhookCallback reference | https://grammy.dev/ref/core/webhookcallback | 2026-09-30 | options onTimeout, timeoutMilliseconds, secretToken |
| S-56 | grammY — auto-retry plugin | https://grammy.dev/plugins/auto-retry | 2026-09-30 | retries 429 (retry_after), 5xx and network errors with backoff from 3 s capped 1 h; options maxRetryAttempts, maxDelaySeconds, rethrowInternalServerErrors, rethrowHttpErrors |
| S-57 | grammY — Hosting: VPS | https://grammy.dev/hosting/vps | 2026-09-30 | webhookCallback(bot, "fastify") example |
| S-58 | grammY — Deployment checklist | https://grammy.dev/advanced/deployment | 2026-09-30 | bot.catch / framework errors; lint floating promises; graceful shutdown; no long ops in middleware (duplicates); testing via API transformers + bot.handleUpdate |
| S-59 | GitHub API — grammyjs/grammY release v1.46.0 | https://github.com/grammyjs/grammY/releases/tag/v1.46.0 | 2026-09-30 | v1.46.0 (2026-08-26) supports Bot API 10.3 |
| S-60 | Node.js 26.0.0 release notes | https://nodejs.org/en/blog/release/v26.0.0 | 2026-09-30 | released 2026-05-05; Temporal enabled by default; V8 14.6; LTS in Oct 2026; removals (writeHeader, legacy streams, --experimental-transform-types) |
| S-61 | TypeScript 6.0 release notes | https://www.typescriptlang.org/docs/handbook/release-notes/typescript-6-0.html | 2026-09-30 | Temporal types via lib esnext / esnext.temporal; strict default true; types default []; module esnext; deprecations removed in 7.0 |
| S-62 | Render docs — Setting your Node.js version | https://render.com/docs/node-version | 2026-09-30 | NODE_VERSION env > .node-version > .nvmrc > engines; default 24.21.0 (services after 2026-09-17); bound ranges |
| S-63 | Sentry pricing | https://sentry.io/pricing/ | 2026-09-30 | Developer free: 1 user, 5k errors/mo, 30-day lookback; Team $26/mo annual |
| S-64 | MDN — Intl.supportedValuesOf() | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/supportedValuesOf | 2026-09-30 | returns sorted IANA primary time zone IDs (400+) |
| S-65 | aiogram docs — CopyTextButton (aiogram 3.31.0) | https://docs.aiogram.dev/en/latest/api/types/copy_text_button.html | 2026-09-30 | aiogram 3.31.0 current docs; copy_text 1-256 |
| S-66 | PyPI JSON API + endoflife.date (Python, Go) | https://pypi.org/pypi/aiogram/json ; https://endoflife.date/api/python.json ; https://endoflife.date/api/go.json | 2026-09-30 | aiogram 3.31.0 (python <3.15,>=3.10); mypy 2.3.1; ruff 0.16.9; pytest 9.1.1; SQLAlchemy 2.1.1; Python 3.14.7 (EOL 2030-10); Go 1.27.1 |
| S-67 | GitHub API — go-telegram/bot, riverqueue/river releases | https://api.github.com/repos/go-telegram/bot/releases/latest ; https://api.github.com/repos/riverqueue/river/releases/latest | 2026-09-30 | go-telegram/bot v1.27.0 (2026-09-11); river v0.47.0 (2026-08-31) |
| S-68 | ISO 4217 List One (SIX, maintenance agency), published 2026-09-17 | https://www.six-group.com/dam/download/financial-information/data-center/iso-currrency/lists/list-one.xml | 2026-09-30 | minor units: USD/EUR/GBP/CAD/AUD/INR/BRL… 2; JPY/KRW/VND/CLP 0 |
| S-69 | Chaser blog — Payment reminder email: 8 templates and a sequence guide (upd. 2026-09-25) | https://www.chaserhq.com/blog/email-templates-friendly-late-payment-reminders-for-your-customers | 2026-09-30 | sequence −7, 0, +1–3, +7, +14, +30, +60, +90; tone warm→formal; subject <60 chars w/ invoice no.; "overdue" from ~day 7; mid-morning weekdays |
| S-70 | Zoho ZeptoMail — Overdue invoice reminder emails | https://www.zoho.com/zeptomail/articles/overdue-invoice-reminder-emails.html | 2026-09-30 | include invoice no., amount, due date, days overdue, one payment link; +1–3, +7–10, +20–25, +30; Tue–Thu mid-morning |
| S-71 | European Commission — eInvoicing in Belgium | https://ec.europa.eu/digital-building-blocks/sites/spaces/DIGITAL/pages/467108877/eInvoicing+in+Belgium | 2026-09-30 | mandatory structured B2B e-invoices for Belgian VAT enterprises from 2026-01-01; Peppol BIS Billing 3.0 / EN 16931 |
| S-72 | Resend API — Send email | https://resend.com/docs/api-reference/emails/send-email | 2026-09-30 | from "Name <email>", to ≤50, reply_to, html/text, custom headers, tags [A-Za-z0-9_-] ≤256, scheduled_at; Idempotency-Key header |
| S-73 | Resend API — Rate limit | https://resend.com/docs/api-reference/rate-limit | 2026-09-30 | default 10 requests/s per team; ratelimit-* and retry-after headers; 429 |
| S-74 | Resend docs — Domains | https://resend.com/docs/dashboard/domains/introduction | 2026-09-30 | verify domain; region selectable; send from subdomain recommended; DMARC after verification |
| S-75 | Fastify docs — Testing guide | https://github.com/fastify/fastify/blob/main/docs/Guides/Testing.md | 2026-09-30 | fastify.inject() fake HTTP injection for tests |
| S-76 | dependency-cruiser — Rules reference | https://github.com/sverweij/dependency-cruiser/blob/main/doc/rules-reference.md | 2026-09-30 | forbidden rules with from/to path + pathNot regex; group matching $1 for peer folders |
| S-77 | typescript-eslint — Shared configs & Typed linting docs | https://github.com/typescript-eslint/typescript-eslint/blob/main/docs/users/Shared_Configurations.mdx ; https://github.com/typescript-eslint/typescript-eslint/blob/main/docs/getting-started/Typed_Linting.mdx | 2026-09-30 | strictTypeChecked (not "stable": rules may change outside majors); flat config + parserOptions.projectService: true |
| S-78 | StrykerJS docs — Vitest runner & configuration | https://github.com/stryker-mutator/stryker-js/blob/master/docs/vitest-runner.md ; https://github.com/stryker-mutator/stryker-js/blob/master/docs/configuration.md | 2026-09-30 | testRunner "vitest"; vitest.related; mutate globs; thresholds {high, low, break} exit 1 below break |
| S-79 | Vitest docs — coverage config | https://github.com/vitest-dev/vitest/blob/main/docs/config/coverage.md | 2026-09-30 | coverage.provider v8 default; coverage.thresholds lines/functions/branches/statements; perFile |
| S-80 | jscpd README | https://github.com/kucherenko/jscpd | 2026-09-30 | --threshold; reporters incl. console/json/sarif; --history trend |
| S-81 | gitleaks README | https://github.com/gitleaks/gitleaks | 2026-09-30 | commands git / dir; --exit-code (default 1); --redact |
| S-82 | Telegram — Bots: An introduction for developers | https://core.telegram.org/bots | 2026-09-30 | bots can't start conversations; user must message first; limited cloud storage, older messages may be removed |
| S-83 | Render docs — Create and connect to Render Postgres | https://render.com/docs/postgresql-creating-connecting | 2026-09-30 | AES-256 encryption at rest; external TLS always; PG major 13–18 for new instances |
| S-84 | PostgreSQL 18 docs — UUID functions | https://www.postgresql.org/docs/current/functions-uuid.html | 2026-09-30 | uuidv7([shift]) time-ordered, uuidv4() |
| S-85 | OSV-Scanner README | https://github.com/google/osv-scanner | 2026-09-30 | osv-scanner scan source -r <dir>; V2 |
| S-86 | npm Docs — npm audit (v11) | https://docs.npmjs.com/cli/v11/commands/npm-audit | 2026-09-30 | --audit-level info/low/moderate/high/critical/none sets non-zero exit threshold; --omit=dev |
| S-87 | Python docs — datetime (Aware and Naive Objects) | https://docs.python.org/3/library/datetime.html | 2026-09-30 | aware and naive are the same datetime class; ordering comparison between them raises TypeError at runtime |
| S-88 | MDN — Temporal | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Temporal | 2026-09-30 | Temporal.Instant, ZonedDateTime, PlainDate, PlainTime, PlainDateTime, Duration, Now |
| S-89 | resend-node SDK source v6.31.0 | https://github.com/resend/resend-node | 2026-09-30 | emails.send(payload, { idempotencyKey }) → Idempotency-Key header; CreateEmailOptions from/to/subject/html/text/headers/replyTo/tags; webhooks.verify({ payload, headers: { id, timestamp, signature }, webhookSecret }) via Svix |
| S-90 | grammY source (core/client.ts, bot.ts) | https://github.com/grammyjs/grammY | 2026-09-30 | Transformer(prev, method, payload, signal); bot.handleUpdate(); botInfo option avoids getMe |
| S-91 | Local runtime check: `node -p process.config` and `Intl.NumberFormat` on a Homebrew Node.js v26.3.0 build (macOS) | (local command; no URL) | 2026-09-30 | `v8_enable_temporal_support: 0`, `typeof Temporal === "undefined"` on that build; `Intl` currency code format uses U+00A0 between code and number ("EUR 1,200.50") |

## Appendix A — Session prompts (Appendix Session Prompts)

One prompt per epic, generated from the skill's `assets/session-prompt.md` with the placeholders filled. Paste the whole block as the **first message** of a fresh Claude Code or Codex session opened at the repo root. Use the strongest model available and the maximum effort or thinking level the tool offers. Terminate the session after the handoff summary.

### Prompt — Epic 00: Foundation

```text
You are starting **Epic 00 — Foundation** of InvoiceNudge. This is a fresh session: you remember nothing from previous sessions. The repo is the memory.

Read, in order: `AGENTS.md` (or `CLAUDE.md`), `docs/PROGRESS.md` (all of it), then `docs/BLUEPRINT.md` sections 12, 13, 14, 15, 16, 18, 21 and **Epic 00** in section 17, plus every flow (F-*), screen (SCR-*), copy (CP-*), entity (ENT-*), API (API-*), integration (INT-*) and edge case (EC-*) ID that epic references, and `docs/REVIEW-CHECKLIST.md`.

Then follow the session protocol in BLUEPRINT §18 exactly:

1. **Orient** — confirm preconditions: previous epics DONE in PROGRESS.md; clean `git status` on main; `npm run check` green; required env vars present; every `[VERIFY-AT-BUILD]` item this epic depends on re-checked and recorded. If anything fails, stop and report — do not build.
2. **Plan** — restate the goal and stories in ≤ 15 lines; list the tests you will write per story and the tasks; flag any AC that is ambiguous or references a missing ID (treat as a blueprint defect, do not guess). Wait for my 'go' before building.
3. **Build** — story by story in order: tests first (they must fail), implement, keep `npm run check` green, copy only from CP-* IDs, one commit per story with `[E00-Snn]` in the message, PROGRESS.md evidence line per AC, DR-nn in DECISIONS.md for every decision the blueprint does not make.
4. **Verify** — full `check`, epic-level flow tests, regression of all previous epics, CI green, staging deploy + smoke if the DoD requires it. Then **self-review**: walk `docs/REVIEW-CHECKLIST.md` (BLUEPRINT §14.4) over the epic's diff, story by story, reading the diff rather than recalling what you wrote — check correctness against each AC, module boundaries, failure paths, test meaning (would a test fail if the logic were wrong?), authorization per object, behaviour at 100x data, observability, copy IDs, and diff size. Fix what fails; do not batch it. Walk the epic's Definition of Done line by line with evidence.
5. **Handoff** — update PROGRESS.md (status, evidence, deviations, accepted risks, VERIFY-AT-BUILD results, next epic preconditions, session log line), DECISIONS.md, and the Amendments log if anything changed; final commit; then print the ≤ 20-line handoff summary from BLUEPRINT §18 and tell me it is safe to terminate.

Rules that override everything: never mark DONE without evidence; never improvise user-facing text; never silently change an AC; never touch scope outside this epic; never weaken the quality gate to get to green; never use an API you have not verified exists in the pinned version; when in doubt, record and ask.

Epic 00 creates the repository itself: `docs/BLUEPRINT.md` is this document, and the files in Appendix B are copied literally.
```

### Prompt — Epic 01: Onboarding and account

```text
You are starting **Epic 01 — Onboarding and account** of InvoiceNudge. This is a fresh session: you remember nothing from previous sessions. The repo is the memory.

Read, in order: `AGENTS.md` (or `CLAUDE.md`), `docs/PROGRESS.md` (all of it), then `docs/BLUEPRINT.md` sections 12, 13, 14, 15, 16, 18, 21 and **Epic 01** in section 17, plus every flow (F-*), screen (SCR-*), copy (CP-*), entity (ENT-*), API (API-*), integration (INT-*) and edge case (EC-*) ID that epic references, and `docs/REVIEW-CHECKLIST.md`.

Then follow the session protocol in BLUEPRINT §18 exactly:

1. **Orient** — confirm preconditions: previous epics DONE in PROGRESS.md; clean `git status` on main; `npm run check` green; required env vars present; every `[VERIFY-AT-BUILD]` item this epic depends on re-checked and recorded. If anything fails, stop and report — do not build.
2. **Plan** — restate the goal and stories in ≤ 15 lines; list the tests you will write per story and the tasks; flag any AC that is ambiguous or references a missing ID (treat as a blueprint defect, do not guess). Wait for my 'go' before building.
3. **Build** — story by story in order: tests first (they must fail), implement, keep `npm run check` green, copy only from CP-* IDs, one commit per story with `[E01-Snn]` in the message, PROGRESS.md evidence line per AC, DR-nn in DECISIONS.md for every decision the blueprint does not make.
4. **Verify** — full `check`, epic-level flow tests, regression of all previous epics, CI green, staging deploy + smoke if the DoD requires it. Then **self-review**: walk `docs/REVIEW-CHECKLIST.md` (BLUEPRINT §14.4) over the epic's diff, story by story, reading the diff rather than recalling what you wrote — check correctness against each AC, module boundaries, failure paths, test meaning (would a test fail if the logic were wrong?), authorization per object, behaviour at 100x data, observability, copy IDs, and diff size. Fix what fails; do not batch it. Walk the epic's Definition of Done line by line with evidence.
5. **Handoff** — update PROGRESS.md (status, evidence, deviations, accepted risks, VERIFY-AT-BUILD results, next epic preconditions, session log line), DECISIONS.md, and the Amendments log if anything changed; final commit; then print the ≤ 20-line handoff summary from BLUEPRINT §18 and tell me it is safe to terminate.

Rules that override everything: never mark DONE without evidence; never improvise user-facing text; never silently change an AC; never touch scope outside this epic; never weaken the quality gate to get to green; never use an API you have not verified exists in the pinned version; when in doubt, record and ask.
```

### Prompt — Epic 02: Clients and invoices

```text
You are starting **Epic 02 — Clients and invoices** of InvoiceNudge. This is a fresh session: you remember nothing from previous sessions. The repo is the memory.

Read, in order: `AGENTS.md` (or `CLAUDE.md`), `docs/PROGRESS.md` (all of it), then `docs/BLUEPRINT.md` sections 12, 13, 14, 15, 16, 18, 21 and **Epic 02** in section 17, plus every flow (F-*), screen (SCR-*), copy (CP-*), entity (ENT-*), API (API-*), integration (INT-*) and edge case (EC-*) ID that epic references, and `docs/REVIEW-CHECKLIST.md`.

Then follow the session protocol in BLUEPRINT §18 exactly:

1. **Orient** — confirm preconditions: previous epics DONE in PROGRESS.md; clean `git status` on main; `npm run check` green; required env vars present; every `[VERIFY-AT-BUILD]` item this epic depends on re-checked and recorded. If anything fails, stop and report — do not build.
2. **Plan** — restate the goal and stories in ≤ 15 lines; list the tests you will write per story and the tasks; flag any AC that is ambiguous or references a missing ID (treat as a blueprint defect, do not guess). Wait for my 'go' before building.
3. **Build** — story by story in order: tests first (they must fail), implement, keep `npm run check` green, copy only from CP-* IDs, one commit per story with `[E02-Snn]` in the message, PROGRESS.md evidence line per AC, DR-nn in DECISIONS.md for every decision the blueprint does not make.
4. **Verify** — full `check`, epic-level flow tests, regression of all previous epics, CI green, staging deploy + smoke if the DoD requires it. Then **self-review**: walk `docs/REVIEW-CHECKLIST.md` (BLUEPRINT §14.4) over the epic's diff, story by story, reading the diff rather than recalling what you wrote — check correctness against each AC, module boundaries, failure paths, test meaning (would a test fail if the logic were wrong?), authorization per object, behaviour at 100x data, observability, copy IDs, and diff size. Fix what fails; do not batch it. Walk the epic's Definition of Done line by line with evidence.
5. **Handoff** — update PROGRESS.md (status, evidence, deviations, accepted risks, VERIFY-AT-BUILD results, next epic preconditions, session log line), DECISIONS.md, and the Amendments log if anything changed; final commit; then print the ≤ 20-line handoff summary from BLUEPRINT §18 and tell me it is safe to terminate.

Rules that override everything: never mark DONE without evidence; never improvise user-facing text; never silently change an AC; never touch scope outside this epic; never weaken the quality gate to get to green; never use an API you have not verified exists in the pinned version; when in doubt, record and ask.
```

### Prompt — Epic 03: Owed overview and payments

```text
You are starting **Epic 03 — Owed overview and payments** of InvoiceNudge. This is a fresh session: you remember nothing from previous sessions. The repo is the memory.

Read, in order: `AGENTS.md` (or `CLAUDE.md`), `docs/PROGRESS.md` (all of it), then `docs/BLUEPRINT.md` sections 12, 13, 14, 15, 16, 18, 21 and **Epic 03** in section 17, plus every flow (F-*), screen (SCR-*), copy (CP-*), entity (ENT-*), API (API-*), integration (INT-*) and edge case (EC-*) ID that epic references, and `docs/REVIEW-CHECKLIST.md`.

Then follow the session protocol in BLUEPRINT §18 exactly:

1. **Orient** — confirm preconditions: previous epics DONE in PROGRESS.md; clean `git status` on main; `npm run check` green; required env vars present; every `[VERIFY-AT-BUILD]` item this epic depends on re-checked and recorded. If anything fails, stop and report — do not build.
2. **Plan** — restate the goal and stories in ≤ 15 lines; list the tests you will write per story and the tasks; flag any AC that is ambiguous or references a missing ID (treat as a blueprint defect, do not guess). Wait for my 'go' before building.
3. **Build** — story by story in order: tests first (they must fail), implement, keep `npm run check` green, copy only from CP-* IDs, one commit per story with `[E03-Snn]` in the message, PROGRESS.md evidence line per AC, DR-nn in DECISIONS.md for every decision the blueprint does not make.
4. **Verify** — full `check`, epic-level flow tests, regression of all previous epics, CI green, staging deploy + smoke if the DoD requires it. Then **self-review**: walk `docs/REVIEW-CHECKLIST.md` (BLUEPRINT §14.4) over the epic's diff, story by story, reading the diff rather than recalling what you wrote — check correctness against each AC, module boundaries, failure paths, test meaning (would a test fail if the logic were wrong?), authorization per object, behaviour at 100x data, observability, copy IDs, and diff size. Fix what fails; do not batch it. Walk the epic's Definition of Done line by line with evidence.
5. **Handoff** — update PROGRESS.md (status, evidence, deviations, accepted risks, VERIFY-AT-BUILD results, next epic preconditions, session log line), DECISIONS.md, and the Amendments log if anything changed; final commit; then print the ≤ 20-line handoff summary from BLUEPRINT §18 and tell me it is safe to terminate.

Rules that override everything: never mark DONE without evidence; never improvise user-facing text; never silently change an AC; never touch scope outside this epic; never weaken the quality gate to get to green; never use an API you have not verified exists in the pinned version; when in doubt, record and ask.
```

### Prompt — Epic 04: Nudge scheduling and evening digest

```text
You are starting **Epic 04 — Nudge scheduling and evening digest** of InvoiceNudge. This is a fresh session: you remember nothing from previous sessions. The repo is the memory.

Read, in order: `AGENTS.md` (or `CLAUDE.md`), `docs/PROGRESS.md` (all of it), then `docs/BLUEPRINT.md` sections 12, 13, 14, 15, 16, 18, 21 and **Epic 04** in section 17, plus every flow (F-*), screen (SCR-*), copy (CP-*), entity (ENT-*), API (API-*), integration (INT-*) and edge case (EC-*) ID that epic references, and `docs/REVIEW-CHECKLIST.md`.

Then follow the session protocol in BLUEPRINT §18 exactly:

1. **Orient** — confirm preconditions: previous epics DONE in PROGRESS.md; clean `git status` on main; `npm run check` green; required env vars present; every `[VERIFY-AT-BUILD]` item this epic depends on re-checked and recorded. If anything fails, stop and report — do not build.
2. **Plan** — restate the goal and stories in ≤ 15 lines; list the tests you will write per story and the tasks; flag any AC that is ambiguous or references a missing ID (treat as a blueprint defect, do not guess). Wait for my 'go' before building.
3. **Build** — story by story in order: tests first (they must fail), implement, keep `npm run check` green, copy only from CP-* IDs, one commit per story with `[E04-Snn]` in the message, PROGRESS.md evidence line per AC, DR-nn in DECISIONS.md for every decision the blueprint does not make.
4. **Verify** — full `check`, epic-level flow tests, regression of all previous epics, CI green, staging deploy + smoke if the DoD requires it. Then **self-review**: walk `docs/REVIEW-CHECKLIST.md` (BLUEPRINT §14.4) over the epic's diff, story by story, reading the diff rather than recalling what you wrote — check correctness against each AC, module boundaries, failure paths, test meaning (would a test fail if the logic were wrong?), authorization per object, behaviour at 100x data, observability, copy IDs, and diff size. Fix what fails; do not batch it. Walk the epic's Definition of Done line by line with evidence.
5. **Handoff** — update PROGRESS.md (status, evidence, deviations, accepted risks, VERIFY-AT-BUILD results, next epic preconditions, session log line), DECISIONS.md, and the Amendments log if anything changed; final commit; then print the ≤ 20-line handoff summary from BLUEPRINT §18 and tell me it is safe to terminate.

Rules that override everything: never mark DONE without evidence; never improvise user-facing text; never silently change an AC; never touch scope outside this epic; never weaken the quality gate to get to green; never use an API you have not verified exists in the pinned version; when in doubt, record and ask.
```

### Prompt — Epic 05: Client email nudges with baseline safety

```text
You are starting **Epic 05 — Client email nudges with baseline safety** of InvoiceNudge. This is a fresh session: you remember nothing from previous sessions. The repo is the memory.

Read, in order: `AGENTS.md` (or `CLAUDE.md`), `docs/PROGRESS.md` (all of it), then `docs/BLUEPRINT.md` sections 12, 13, 14, 15, 16, 18, 21 and **Epic 05** in section 17, plus every flow (F-*), screen (SCR-*), copy (CP-*), entity (ENT-*), API (API-*), integration (INT-*) and edge case (EC-*) ID that epic references, and `docs/REVIEW-CHECKLIST.md`.

Then follow the session protocol in BLUEPRINT §18 exactly:

1. **Orient** — confirm preconditions: previous epics DONE in PROGRESS.md; clean `git status` on main; `npm run check` green; required env vars present; every `[VERIFY-AT-BUILD]` item this epic depends on re-checked and recorded. If anything fails, stop and report — do not build.
2. **Plan** — restate the goal and stories in ≤ 15 lines; list the tests you will write per story and the tasks; flag any AC that is ambiguous or references a missing ID (treat as a blueprint defect, do not guess). Wait for my 'go' before building.
3. **Build** — story by story in order: tests first (they must fail), implement, keep `npm run check` green, copy only from CP-* IDs, one commit per story with `[E05-Snn]` in the message, PROGRESS.md evidence line per AC, DR-nn in DECISIONS.md for every decision the blueprint does not make.
4. **Verify** — full `check`, epic-level flow tests, regression of all previous epics, CI green, staging deploy + smoke if the DoD requires it. Then **self-review**: walk `docs/REVIEW-CHECKLIST.md` (BLUEPRINT §14.4) over the epic's diff, story by story, reading the diff rather than recalling what you wrote — check correctness against each AC, module boundaries, failure paths, test meaning (would a test fail if the logic were wrong?), authorization per object, behaviour at 100x data, observability, copy IDs, and diff size. Fix what fails; do not batch it. Walk the epic's Definition of Done line by line with evidence.
5. **Handoff** — update PROGRESS.md (status, evidence, deviations, accepted risks, VERIFY-AT-BUILD results, next epic preconditions, session log line), DECISIONS.md, and the Amendments log if anything changed; final commit; then print the ≤ 20-line handoff summary from BLUEPRINT §18 and tell me it is safe to terminate.

Rules that override everything: never mark DONE without evidence; never improvise user-facing text; never silently change an AC; never touch scope outside this epic; never weaken the quality gate to get to green; never use an API you have not verified exists in the pinned version; when in doubt, record and ask.

This epic sends real email: keep `NUDGES_LIVE=allowlist` in staging and confirm the recipient override before the first send.
```

### Prompt — Epic 06: Operator tools and manual forwarding

```text
You are starting **Epic 06 — Operator tools and manual forwarding** of InvoiceNudge. This is a fresh session: you remember nothing from previous sessions. The repo is the memory.

Read, in order: `AGENTS.md` (or `CLAUDE.md`), `docs/PROGRESS.md` (all of it), then `docs/BLUEPRINT.md` sections 12, 13, 14, 15, 16, 18, 21 and **Epic 06** in section 17, plus every flow (F-*), screen (SCR-*), copy (CP-*), entity (ENT-*), API (API-*), integration (INT-*) and edge case (EC-*) ID that epic references, and `docs/REVIEW-CHECKLIST.md`.

Then follow the session protocol in BLUEPRINT §18 exactly:

1. **Orient** — confirm preconditions: previous epics DONE in PROGRESS.md; clean `git status` on main; `npm run check` green; required env vars present; every `[VERIFY-AT-BUILD]` item this epic depends on re-checked and recorded. If anything fails, stop and report — do not build.
2. **Plan** — restate the goal and stories in ≤ 15 lines; list the tests you will write per story and the tasks; flag any AC that is ambiguous or references a missing ID (treat as a blueprint defect, do not guess). Wait for my 'go' before building.
3. **Build** — story by story in order: tests first (they must fail), implement, keep `npm run check` green, copy only from CP-* IDs, one commit per story with `[E06-Snn]` in the message, PROGRESS.md evidence line per AC, DR-nn in DECISIONS.md for every decision the blueprint does not make.
4. **Verify** — full `check`, epic-level flow tests, regression of all previous epics, CI green, staging deploy + smoke if the DoD requires it. Then **self-review**: walk `docs/REVIEW-CHECKLIST.md` (BLUEPRINT §14.4) over the epic's diff, story by story, reading the diff rather than recalling what you wrote — check correctness against each AC, module boundaries, failure paths, test meaning (would a test fail if the logic were wrong?), authorization per object, behaviour at 100x data, observability, copy IDs, and diff size. Fix what fails; do not batch it. Walk the epic's Definition of Done line by line with evidence.
5. **Handoff** — update PROGRESS.md (status, evidence, deviations, accepted risks, VERIFY-AT-BUILD results, next epic preconditions, session log line), DECISIONS.md, and the Amendments log if anything changed; final commit; then print the ≤ 20-line handoff summary from BLUEPRINT §18 and tell me it is safe to terminate.

Rules that override everything: never mark DONE without evidence; never improvise user-facing text; never silently change an AC; never touch scope outside this epic; never weaken the quality gate to get to green; never use an API you have not verified exists in the pinned version; when in doubt, record and ask.
```

### Prompt — Epic 07: Settings, export and deletion

```text
You are starting **Epic 07 — Settings, export and deletion** of InvoiceNudge. This is a fresh session: you remember nothing from previous sessions. The repo is the memory.

Read, in order: `AGENTS.md` (or `CLAUDE.md`), `docs/PROGRESS.md` (all of it), then `docs/BLUEPRINT.md` sections 12, 13, 14, 15, 16, 18, 21 and **Epic 07** in section 17, plus every flow (F-*), screen (SCR-*), copy (CP-*), entity (ENT-*), API (API-*), integration (INT-*) and edge case (EC-*) ID that epic references, and `docs/REVIEW-CHECKLIST.md`.

Then follow the session protocol in BLUEPRINT §18 exactly:

1. **Orient** — confirm preconditions: previous epics DONE in PROGRESS.md; clean `git status` on main; `npm run check` green; required env vars present; every `[VERIFY-AT-BUILD]` item this epic depends on re-checked and recorded. If anything fails, stop and report — do not build.
2. **Plan** — restate the goal and stories in ≤ 15 lines; list the tests you will write per story and the tasks; flag any AC that is ambiguous or references a missing ID (treat as a blueprint defect, do not guess). Wait for my 'go' before building.
3. **Build** — story by story in order: tests first (they must fail), implement, keep `npm run check` green, copy only from CP-* IDs, one commit per story with `[E07-Snn]` in the message, PROGRESS.md evidence line per AC, DR-nn in DECISIONS.md for every decision the blueprint does not make.
4. **Verify** — full `check`, epic-level flow tests, regression of all previous epics, CI green, staging deploy + smoke if the DoD requires it. Then **self-review**: walk `docs/REVIEW-CHECKLIST.md` (BLUEPRINT §14.4) over the epic's diff, story by story, reading the diff rather than recalling what you wrote — check correctness against each AC, module boundaries, failure paths, test meaning (would a test fail if the logic were wrong?), authorization per object, behaviour at 100x data, observability, copy IDs, and diff size. Fix what fails; do not batch it. Walk the epic's Definition of Done line by line with evidence.
5. **Handoff** — update PROGRESS.md (status, evidence, deviations, accepted risks, VERIFY-AT-BUILD results, next epic preconditions, session log line), DECISIONS.md, and the Amendments log if anything changed; final commit; then print the ≤ 20-line handoff summary from BLUEPRINT §18 and tell me it is safe to terminate.

Rules that override everything: never mark DONE without evidence; never improvise user-facing text; never silently change an AC; never touch scope outside this epic; never weaken the quality gate to get to green; never use an API you have not verified exists in the pinned version; when in doubt, record and ask.
```

### Prompt — Epic 08: Launch hardening

```text
You are starting **Epic 08 — Launch hardening** of InvoiceNudge. This is a fresh session: you remember nothing from previous sessions. The repo is the memory.

Read, in order: `AGENTS.md` (or `CLAUDE.md`), `docs/PROGRESS.md` (all of it), then `docs/BLUEPRINT.md` sections 12, 13, 14, 15, 16, 18, 21 and **Epic 08** in section 17, plus every flow (F-*), screen (SCR-*), copy (CP-*), entity (ENT-*), API (API-*), integration (INT-*) and edge case (EC-*) ID that epic references, and `docs/REVIEW-CHECKLIST.md`.

Then follow the session protocol in BLUEPRINT §18 exactly:

1. **Orient** — confirm preconditions: previous epics DONE in PROGRESS.md; clean `git status` on main; `npm run check` green; required env vars present; every `[VERIFY-AT-BUILD]` item this epic depends on re-checked and recorded. If anything fails, stop and report — do not build.
2. **Plan** — restate the goal and stories in ≤ 15 lines; list the tests you will write per story and the tasks; flag any AC that is ambiguous or references a missing ID (treat as a blueprint defect, do not guess). Wait for my 'go' before building.
3. **Build** — story by story in order: tests first (they must fail), implement, keep `npm run check` green, copy only from CP-* IDs, one commit per story with `[E08-Snn]` in the message, PROGRESS.md evidence line per AC, DR-nn in DECISIONS.md for every decision the blueprint does not make.
4. **Verify** — full `check`, epic-level flow tests, regression of all previous epics, CI green, staging deploy + smoke if the DoD requires it. Then **self-review**: walk `docs/REVIEW-CHECKLIST.md` (BLUEPRINT §14.4) over the epic's diff, story by story, reading the diff rather than recalling what you wrote — check correctness against each AC, module boundaries, failure paths, test meaning (would a test fail if the logic were wrong?), authorization per object, behaviour at 100x data, observability, copy IDs, and diff size. Fix what fails; do not batch it. Walk the epic's Definition of Done line by line with evidence.
5. **Handoff** — update PROGRESS.md (status, evidence, deviations, accepted risks, VERIFY-AT-BUILD results, next epic preconditions, session log line), DECISIONS.md, and the Amendments log if anything changed; final commit; then print the ≤ 20-line handoff summary from BLUEPRINT §18 and tell me it is safe to terminate.

Rules that override everything: never mark DONE without evidence; never improvise user-facing text; never silently change an AC; never touch scope outside this epic; never weaken the quality gate to get to green; never use an API you have not verified exists in the pinned version; when in doubt, record and ask.

This epic touches production: stop and ask before every production change listed in E08-S03.
```

## Appendix B — Files to create in Epic 0 (Appendix Epic 0 Files)

E00-S01 copies these files literally, so it doesn't invent them. In the repository, `BLUEPRINT.md`, `PROGRESS.md`, `DECISIONS.md` and `REVIEW-CHECKLIST.md` live under `docs/`; `AGENTS.md`, `CLAUDE.md` and `.env.example` live at the root.

### B.1 `docs/PROGRESS.md`

````markdown
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
````

### B.2 `docs/DECISIONS.md`

The header below, followed by DR-01 to DR-23 copied verbatim from §4.2, with each `#### DR-` heading changed to `## DR-`. New decisions are appended from DR-24.

````markdown
# DECISIONS — InvoiceNudge

> One entry per decision, newest last. Blueprint decision records DR-01 to DR-23 are copied here in E00 so this is the single place to look.
> Format for new entries: context → options → decision → consequences (including what the rejected option would have bought) → compliance → reversibility → evidence. Keep each under 15 lines. IDs are never reused; superseded entries stay with their status changed and a pointer to the new one.
> `AD-n` drivers and `FF-nn` fitness functions are defined in BLUEPRINT §12; `S-nn` sources in BLUEPRINT §22.
````

### B.3 `AGENTS.md` and `CLAUDE.md` (byte-identical)

````markdown
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
````

### B.4 `docs/REVIEW-CHECKLIST.md`

A one-line header ("Review checklist — InvoiceNudge; copied verbatim from BLUEPRINT §14.4") followed by the §14.4 code block, unchanged.

### B.5 `.env.example`

```bash
# InvoiceNudge — environment variables (BLUEPRINT §12.8). Copy to .env for local development.
# Never commit real values. Production and staging values live in Render environment groups.

# local | test | staging | production
APP_ENV=local
# HTTP port (Render sets PORT in deployed environments)
PORT=3000
# Base URL for client links in emails (https only outside local)
PUBLIC_BASE_URL=http://localhost:3000
# Postgres connection string (test runs use Testcontainers instead)
DATABASE_URL=postgres://invoicenudge:invoicenudge@localhost:5432/invoicenudge
# Bot token from @BotFather (staging bot for local tunnels)
TELEGRAM_BOT_TOKEN=
# Bot username without @, used in CP-SCR20-6 and botInfo
TELEGRAM_BOT_USERNAME=
# Webhook secret_token: 1-256 chars of A-Z a-z 0-9 _ - (S-10)
TELEGRAM_WEBHOOK_SECRET=
# Resend API key
RESEND_API_KEY=
# Resend webhook signing secret (Svix)
RESEND_WEBHOOK_SECRET=
# Sender address on the verified sending subdomain, e.g. reminders@send-staging.example.com
EMAIL_FROM_ADDRESS=
# Staging only: rewrite reminder recipients to @resend.dev test addresses (must be unset in production)
EMAIL_RECIPIENT_OVERRIDE=
# Shown to suspended users (CP-SCR15-12) and blocked accounts (CP-SCR20-10)
SUPPORT_EMAIL=
# Comma list of id:base64 32-byte keys, e.g. 1:BASE64KEY (generate with node -e "console.log(require('crypto').randomBytes(32).toString('base64'))")
DATA_ENCRYPTION_KEYS=
# Key ID used for new writes
DATA_ENCRYPTION_ACTIVE_KEY_ID=1
# base64 32-byte HMAC key for blind indexes
BLIND_INDEX_KEY=
# base64 32-byte HMAC key for client action links
LINK_SIGNING_KEY=
# base64 32-byte key for hashing verification codes
CODE_PEPPER=
# Comma list of operator Telegram user IDs
ADMIN_TELEGRAM_IDS=
# Sentry DSN; empty disables error tracking
SENTRY_DSN=
# pino level: fatal | error | warn | info | debug | trace
LOG_LEVEL=info
# Run scheduler and outbox loops in this process
RUN_SCHEDULER=true
# Loop period in seconds (5-120)
SCHEDULER_TICK_SECONDS=30
# off | allowlist | on
NUDGES_LIVE=off
# Telegram user IDs allowed to send when NUDGES_LIVE=allowlist
NUDGES_ALLOWLIST=
# resend | sink (sink only when APP_ENV=staging, for load tests)
EMAIL_GATEWAY_MODE=resend
# api | sink (sink only when APP_ENV=staging, for load tests)
TELEGRAM_GATEWAY_MODE=api
```

### B.6 The `check` script

The `scripts` block of §13.4 is the literal content of `package.json` `scripts`. `scripts/db-verify.ts` starts a Postgres 18 Testcontainer, runs `migrateToLatest()`, then runs `kysely-codegen --dialect postgres --out-file src/platform/db/types.gen.ts --verify` with the container's URL, and exits with its status (FF-11).

