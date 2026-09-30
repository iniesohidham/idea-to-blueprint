# Example output: Telegram invoice-nudging bot

This folder is a complete, real run of the `idea-to-blueprint` skill. Nothing was cut down for the demo.

- **Input (verbatim):** "I want a Telegram bot that helps freelancers track invoices and nudges late clients."
- **Intake:** the skill asked one batch of 15 questions, each with a default. The reply was "defaults", so every answer is recorded as `[ASSUMED — intake default]` (BLUEPRINT §4.1, rows 1–15).
- **Decision brief:** approved with "go".
- **Model:** Claude Opus 5.5 (1M context), `claude-opus-5-5[1m]`, running in Claude Code. The effort setting was not visible to the agent.
- **Generated:** 2026-09-30.
- **Research:** 30 web searches and about 100 page fetches (about 80 succeeded), plus direct reads of the npm and PyPI registries, the Node.js release index, the GitHub API and the ISO 4217 list. That produced 90 cited web sources and 1 local runtime check (§22).
- **Lint:** `scripts/lint_blueprint.py --strict` → `PASS — no findings` (9 epics, 47 stories).
- **Size:** `BLUEPRINT.md` is 90,167 words (about 180 pages at 500 words per page). The whole folder is about 98,000 words.
- **Unedited by humans.** Every change after generation was made by the agent itself, inside the skill's own self-review (template §6), human-pass and lint loop (Phase 4). The agent's in-run revisions included renumbering stories, removing hedged ACs and correcting two claims it could not verify. One edit was made after the run: a persona's sample email address was moved to a reserved `.example` domain.

## Files

| File | What it is |
|---|---|
| [BLUEPRINT.md](BLUEPRINT.md) | The blueprint (in a real repo: `docs/BLUEPRINT.md`) |
| [PROGRESS.md](PROGRESS.md) · [DECISIONS.md](DECISIONS.md) · [REVIEW-CHECKLIST.md](REVIEW-CHECKLIST.md) | Epic-0 state, decisions and review files (in a real repo: under `docs/`) |
| [AGENTS.md](AGENTS.md) · [CLAUDE.md](CLAUDE.md) | Identical entry points for Codex and Claude Code |

## Jump into the blueprint

- [1. Executive summary](BLUEPRINT.md#1-executive-summary) · [3. Research and benchmark](BLUEPRINT.md#3-research-findings-research) · [4. Assumptions and decisions](BLUEPRINT.md#4-assumptions-and-decision-records-assumptions--decisions)
- [5. Personas](BLUEPRINT.md#5-personas) · [7. Flows](BLUEPRINT.md#7-user-flows-flows) · [11. Copy tables](BLUEPRINT.md#11-copy-tables-copy)
- [12. Architecture](BLUEPRINT.md#12-architecture) · [13. Tech stack](BLUEPRINT.md#13-tech-stack-and-engineering-conventions-tech-stack) · [14. Quality](BLUEPRINT.md#14-quality-strategy-review-and-metrics-quality)
- [15. Edge cases](BLUEPRINT.md#15-edge-case-register-edge-cases) · [16. Delivery plan](BLUEPRINT.md#16-delivery-plan--sprints-and-epic-order-delivery-plan) · [17. Epics](BLUEPRINT.md#17-epics-and-stories-epics)
- [18. Session protocol](BLUEPRINT.md#18-session-protocol-for-claude-code--codex-session-protocol) · [20. Open questions](BLUEPRINT.md#20-open-questions-open-questions) · [22. Sources](BLUEPRINT.md#22-sources) · [Appendix A: session prompts](BLUEPRINT.md#appendix-a--session-prompts-appendix-session-prompts)

Facts about third-party products, prices and versions were verified on 2026-09-30 and will drift. The blueprint marks volatile ones `[VERIFY-AT-BUILD]`.
