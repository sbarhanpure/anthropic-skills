---
name: adr-writer
description: Use this skill whenever an architecturally significant decision needs to be documented, when capturing trade-offs between technical options, when superseding a prior architectural decision, or when a code review reveals an implicit architectural choice that should be recorded. Apply this for every L-tier ticket and any M-tier ticket that touches architecture. Produces Architecture Decision Records in Michael Nygard's standard format with proper status tracking, immutability rules, and linkable consequences.
---

# ADR Writer

You produce Architecture Decision Records in the Michael Nygard format. ADRs are short, immutable records of architecturally significant decisions. One ADR per decision.

## Examples

### Example 1: Choosing a queue technology

**Trigger:** L-tier ticket requires picking between Postgres-as-queue, Redis, and SQS for the orchestrator's job queue.

**Output:**
```markdown
# ADR-0007: Use Postgres for the orchestrator job queue

## Status
Accepted (2026-05-16)

## Context
The orchestrator needs a job queue to manage agent tasks across multiple components. We considered three options:
- Postgres (Supabase) with row-locking via `SELECT FOR UPDATE SKIP LOCKED`
- Redis with reliable queue patterns (Lua scripts)
- AWS SQS

Constraints:
- Single-developer project; operational simplicity matters more than absolute throughput
- Already using Supabase for application state
- Volume estimate: <100 jobs/day in first 6 months

## Decision
We will use Postgres (Supabase) for the job queue, with row-level locking via `SELECT FOR UPDATE SKIP LOCKED` to coordinate workers.

## Consequences
**Positive:**
- One fewer service to operate, secure, and pay for
- Transactional consistency between job state and application state for free
- Native to existing stack; no new tooling

**Negative:**
- Won't scale beyond ~1000 jobs/minute without architectural change
- Less mature tooling for queue introspection than Redis/SQS ecosystems
- Polling-based; small added latency vs push-based queues

**Accepted because:** volume estimates put us well under the scaling ceiling for the foreseeable future.
```

### Example 2: Superseding an earlier ADR

**Trigger:** ADR-0003 said "Use webhooks for cross-component events." After 6 months, we've hit reliability issues. Need to switch to a message bus.

**Output:**
```markdown
# ADR-0012: Use a message bus for cross-component events (supersedes ADR-0003)

## Status
Accepted (2026-05-16). Supersedes ADR-0003.

## Context
ADR-0003 chose webhooks for cross-component events. After 6 months in production we have data:
- Webhook delivery failures: 2.3% (no retries by default)
- Lost events leading to data inconsistency: 11 incidents in Q1
- Recovery: manual reconciliation scripts each time

The original tradeoffs in ADR-0003 (simplicity, no new infrastructure) are now outweighed by operational cost.

## Decision
We will introduce a message bus (Redpanda or similar Kafka-compatible) for cross-component events. Components publish events to the bus; consumers subscribe. Webhooks remain only for external integrations.

## Consequences
**Positive:**
- Built-in retries and dead-letter queue
- Replay for recovery
- Loose coupling

**Negative:**
- One new service to operate
- More complexity for simple use cases (a single new dependency that was a webhook is now bus subscribe + handler)
- 1–2 week migration cost

**Accepted because:** 11 incidents in one quarter is unsustainable for a small team.
```

### Example 3: Refusing to write an ADR

**Trigger:** Someone asks for an ADR on "use camelCase for variable names."

**Output:**
> This is not architecturally significant. Nygard's criterion: ADRs capture decisions that affect "structure, non-functional characteristics, dependencies, interfaces, or construction techniques."
> 
> Naming conventions belong in `components/{name}/CLAUDE.md` under Conventions, not in an ADR. Do not write an ADR for this.

## Guidelines

### When to write an ADR

Write an ADR when the decision affects any of:
- System structure (services, components, modules)
- Non-functional characteristics (perf, reliability, cost)
- External dependencies (libraries, services, infra)
- Interfaces (APIs, contracts, protocols)
- Construction techniques (build, deploy, test approach)

Do NOT write an ADR for:
- Naming conventions (goes in component CLAUDE.md)
- Bug fixes (goes in commit message)
- Variable choices, code style (goes in linter config)
- Decisions reversible in one PR (probably not architecturally significant)

### Format — Nygard 1.0

Five required sections. No additions.

```markdown
# ADR-NNNN: {Short noun phrase title}

## Status
{Proposed | Accepted | Deprecated | Superseded by ADR-NNNN}

## Context
The forces at play. Technological, business, constraints. Value-neutral language. Describes facts, doesn't yet take a position.

## Decision
Stated in present tense, active voice: "We will use X because..."

## Consequences
Both positive AND negative. What becomes easier; what becomes harder; what we accept as cost.
```

### Numbering and storage

- Location: `decisions/adr-NNNN-{slug}.md`
- NNNN: zero-padded, monotonically increasing
- Numbers never reused, even if an ADR is deleted
- Slug: short noun phrase, lowercase, hyphenated

### Immutability rule

Once `Status: Accepted`, the ADR is immutable. To change a decision:
1. Write a new ADR
2. The new ADR's title says "(supersedes ADR-NNNN)"
3. Update the old ADR's Status to `Superseded by ADR-NNNN`
4. Do not edit any other content of the old ADR

### Style guidelines (from Nygard's original 2011 essay)

- Title: short noun phrase
- One ADR per decision
- 1–2 pages, no longer
- Write as a conversation with a future developer
- Full sentences, organized into paragraphs
- Bullets for visual style only, not as sentence fragments
- Present tense for the decision: "We will use X" not "We chose X"

### Anti-hallucination rules

1. Do not invent options that weren't considered. If only one option was considered, say so.
2. Do not invent consequences. If you're not sure about a negative consequence, leave it out rather than guess.
3. Cite specific systems, libraries, versions when relevant — not generic technologies.
4. Do not claim a decision is reversible if it requires data migration or breaking API changes.
5. If you're inferring context from incomplete information, flag the inferred sections explicitly.

### What this skill NEVER does

- Make the decision. Only humans accept ADRs.
- Mark status as Accepted (only the human does that)
- Write an ADR for non-architectural decisions
- Edit an Accepted ADR's content (always supersede with a new one)
- Combine multiple decisions into one ADR

## Output

A markdown file at `decisions/adr-NNNN-{slug}.md` in Nygard format.

Summary message:
- ADR path
- Status (always Proposed until human accepts)
- Whether it supersedes any prior ADRs (with links)
- Reminder: this is Proposed, awaiting human acceptance

## Source

Michael Nygard, *Documenting Architecture Decisions*, Cognitect, November 15, 2011. The five-section format has been the de facto standard since adoption by Thoughtworks Technology Radar in 2018.

## See also

- `references/nygard-template.md` — bare template ready to fill
- `references/when-to-adr.md` — decision tree for whether something deserves an ADR
