---
name: tier-classifier
description: Use this skill whenever a ticket or task needs classification for engineering rigor, when sizing work for sprint planning, when determining which artifacts (HLD, LLD, ADR, rollback plan) are required, or when an existing classification looks wrong given the actual scope. Apply this before decomposition begins on any ticket. Classifies work as Small, Medium, or Large with an explicit rubric and bias-toward-higher-tier when uncertain. Prevents the most common decomposition failure mode of treating L work as M and skipping required artifacts.
---

# Tier Classifier

You classify every ticket as Small (S), Medium (M), or Large (L). The classification determines which engineering rigor artifacts are required. Misclassifying upward is cheap; downward causes silent drift and missing artifacts.

**Default to the higher tier when uncertain.**

## Examples

### Example 1: Clear S
**Input:** "Fix typo in error message displayed when a phone number is invalid."

**Output:**
- Tier: **S**
- Rationale: Single line change in user-facing text. No logic. No tests beyond existing ones. No architectural impact.
- Required artifacts: Task spec only. No LLD. No ADR.
- Recommended next step: Move directly to Ready; assign to component's implementation-agent.

### Example 2: Looks-like-M-but-actually-L
**Input:** "Add support for international phone numbers in messaging-guest."

**Output:**
- Tier: **L**
- Rationale: Sounds like a parsing addition but actually involves:
  - New external library or service (libphonenumber)
  - Schema migration (existing US-format records need normalization)
  - Validation rule changes affecting downstream message routing
  - PII handling implications (country codes can imply location)
  - Affects guest-facing surface (deploy gate applies)
- Required artifacts: HLD, LLD, ADR (library choice), test strategy, rollback plan
- Recommended next step: Pair with `adr-writer` skill on library choice and `prd-validator` on PRD if not done.

### Example 3: M with reasoning
**Input:** "Refactor `MessageQueue.enqueue()` to accept a priority parameter."

**Output:**
- Tier: **M**
- Rationale: New parameter on a public API, contract change. Existing callers must continue to work. Requires LLD section showing the new signature and migration plan for callers. Touches one module only.
- Required artifacts: Task spec, LLD section, test strategy (existing-caller compatibility tests required), no ADR needed (no new external dependency)
- Recommended next step: Lead Engineer pairing for LLD. Then component implementation-agent.

### Example 4: Bias toward higher tier
**Input:** "Add a new endpoint to expose property details." Caller says it's "just a GET endpoint, should be easy."

**Output:**
- Tier: **M** (not S, despite caller's framing)
- Rationale: New public surface. Authz check required (whose data?). Caching policy. Rate limiting. Response shape decision. These need at least an LLD section.
- Required artifacts: Task spec, LLD section, test strategy including security (authz) and contract tests
- Pushback: "Just a GET" framing is the most common source of S-tier scope creep. New endpoints are always at least M.

## Guidelines

### Inputs to read

1. The ticket draft (in chat or in the backlog tool)
2. Linked PRD if any
3. `STANDARDS.md` — for the tier rubric and required artifacts
4. `components/{component}/CLAUDE.md` for the components likely touched
5. Existing ADRs that might apply (`decisions/adr-*.md`)
6. Last 10 merged tickets in the component to calibrate "what S/M/L looks like here"

### Classification rubric

| Indicator | S | M | L |
|---|---|---|---|
| Lines of code (rough) | <20 | 20–500 | 500+ |
| Files touched | 1 | 1–5 | 5+ |
| New external dependencies | 0 | 0 | 1+ |
| Schema changes | none | minor (column add/rename) | major (table, FK, migration with backfill) |
| Public API changes | none | additive | breaking or new endpoint |
| Cross-component impact | none | none or one consumer | multiple consumers |
| New failure modes | none | 0–1 | 2+ |
| Security surface | unchanged | unchanged | new |
| Rollback complexity | revert PR | revert PR | needs documented rollback plan |
| Required for `messaging-guest` | n/a | n/a | always L |

A ticket is the highest tier any single row points to. Pick the highest.

### Tier-based required artifacts

| Tier | Required |
|---|---|
| **S** | Task spec only. Inline ADR comment in code if non-obvious. |
| **M** | Task spec + LLD section + test strategy. ADR if architecture touched. |
| **L** | HLD + LLD + ADR(s) + test strategy + rollback plan. Sampling re-review mandatory. |

### Bias rule

When between two tiers:
- S vs M → choose M
- M vs L → choose L

Reasoning: rework cost of under-classifying is much higher than overhead of slight over-classifying. Tickets surprise upward, not downward.

### Anti-hallucination rules

1. Do not estimate effort in days or hours. Tier is the only output. Time estimates are out of scope.
2. Do not classify based on the caller's framing. A "quick" task that touches a public API is M, not S.
3. Cite the specific rubric row that drove the classification. Don't say "looks complex" — say "5+ files touched and new external dependency."
4. If you can't determine scope from the ticket, ask 1–2 questions or escalate to OPEN_QUESTIONS.md. Do not guess.
5. If existing ADRs forbid the approach implied by the ticket, flag it as part of the classification.

### What this skill NEVER does

- Estimate time, hours, or story points
- Approve a tier (you propose; human accepts)
- Classify a ticket without reading the rubric
- Bypass the bias rule under time pressure
- Combine tier classification with implementation planning — keep concerns separate

## Output

A short classification block:

```
Tier: {S | M | L}

Rationale:
- {rubric row 1 that drove the decision}
- {rubric row 2}

Required artifacts:
- {list per tier}

Recommended next step:
- {S: assign to component impl-agent | M/L: pair with adr-writer/prd-validator next}
```

If the ticket cannot be classified without more info, return 1–2 specific questions instead.

## See also

- `references/tier-rubric.md` — full rubric with worked examples per row
