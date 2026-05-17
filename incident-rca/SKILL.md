---
name: incident-rca
description: Use this skill whenever a confirmed incident is being investigated post-resolution, when writing a blameless post-mortem, when identifying contributing factors to a production issue, or when proposing preventive action items. Apply this for every P0 and P1 incident, and any P2 that recurs. Produces structured root cause analysis grounded in cited evidence — log lines, metrics, traces, code references — and explicitly distinguishes observed facts from inferred hypotheses. Catches the most common RCA failure mode of constructing plausible narratives that aren't actually supported by the evidence.
---

# Incident RCA

You write blameless post-mortems for confirmed incidents. The single most important property of your output: **every claim must be grounded in evidence, and inference must be explicitly labeled as inference.**

LLMs are skilled at constructing coherent narratives. Real incidents are often less coherent than they appear. Your discipline is to resist narrative coherence in favor of cited truth.

## Examples

### Example 1: Well-grounded RCA

**Input:** Resolved P1 incident — `messaging-guest` rate limit errors on 2026-05-17 from 14:23–14:38 UTC.

**Output:**
```markdown
# Post-Mortem: INC-2026-0517-001

## Summary
On 2026-05-17, between 14:23 and 14:38 UTC (15 minutes), the messaging-guest component experienced sustained Telegram API rate limit errors, preventing 12 guest message drafts from completing. No drafts were sent in error. No guest-facing impact (drafts were paused, not auto-sent). Resolved by manual rollback of PR #142.

## Severity
P1

## Timeline (all UTC)

- **09:15** — PR #142 merged ("Increase draft generation parallelism") [observed: GitHub commit log]
- **09:23** — Production deploy of PR #142 completed [observed: Railway deploy log]
- **09:23 – 14:22** — No anomalies detected. Normal traffic. [observed: 5 hours of metrics show no deviation]
- **14:23** — First `429 rate limit` error logged in messaging-guest [observed: Sentry, error ID a7b2c1]
- **14:23 – 14:25** — 47 occurrences of same error [observed: Sentry count for window]
- **14:25** — Triage agent classified as P1 confirmed incident [observed: triage-log.md entry]
- **14:25** — Page sent to Bryan via Telegram [observed: Telegram log]
- **14:31** — Bryan acknowledged page [observed: ack timestamp]
- **14:35** — Bryan initiated rollback of PR #142 [observed: GitHub revert PR #143]
- **14:38** — Rollback deployed; errors stopped [observed: Sentry error rate dropped to baseline]

## Root cause

**Observed:** PR #142 increased the parallelism factor for draft generation from 1 to 5. Before the change, the `messaging-guest` component sent at most ~5 Telegram API requests per second. After the change, request rate could spike to ~25/s.

**Observed:** Telegram Bot API rate limit for this bot type is 30 messages/second across all chats, but per-chat limits are stricter (1 msg/sec) [cited: Telegram Bot API docs at time of incident].

**Inferred (high confidence):** The parallelism increase caused multiple drafts targeting different guest chats to be sent concurrently. While this stayed within the 30/sec global limit, it likely caused bursts toward individual chat limits when multiple drafts targeted the same property's guest chat.

**Inferred (low confidence):** No specific log evidence ties the bursts to per-chat limits vs global limits. Telegram's 429 response does not specify which limit was hit. This inference is plausible but not confirmed.

## Contributing factors

**Observed:** The PR #142 LLD did not enumerate the Telegram API rate limit as a failure mode [verified: `decisions/lld-parallel-drafting.md` does not mention rate limits in the Failure modes section].

**Observed:** The component's CLAUDE.md "Risk surface" section did not call out Telegram rate limits [verified: `components/messaging-guest/CLAUDE.md` Risk surface section].

**Observed:** No staging test exists that exercises the parallel path at the rate that production hit.

## What went well

- Triage agent detected and classified within 2 minutes (good)
- Rollback path was documented in PR #142 description and worked as documented (good)
- No guest-facing impact because drafts pause at human approval gate (the Outbound gate prevented harm)
- Total time-to-resolve: 15 minutes (good for a non-P0)

## What didn't go well

- Failure mode was foreseeable but not caught in design review
- No automated rate-limit-aware backoff in `telegram-client.ts`
- Staging environment did not catch the issue because it doesn't replicate production load

## Action items

Each action item is specific, has an owner, and has a verification step.

1. **Add Telegram rate limit failure mode to messaging-guest CLAUDE.md "Risk surface" section.**
   - Owner: Bryan
   - Verification: file updated; visible in next `prd-validator` run on related PRDs
   - Deadline: This week
   - Tracked: Issue #144

2. **Add automatic exponential backoff to `telegram-client.ts` on 429 responses.**
   - Owner: messaging-guest implementation-agent (next ticket)
   - Verification: test reproduces 429, observes backoff, eventual success
   - Deadline: Next sprint
   - Tracked: Issue #145, tier M

3. **Add a "rate limit awareness" item to the LLD failure-modes checklist in TEMPLATES.md.**
   - Owner: Bryan
   - Verification: template updated; verified by next `tier-classifier` invocation
   - Deadline: This week
   - Tracked: Issue #146

4. **Document Telegram per-chat vs global rate limits in component reference docs.**
   - Owner: Bryan
   - Verification: doc exists; linked from `components/messaging-guest/CLAUDE.md`
   - Deadline: Next sprint
   - Tracked: Issue #147

## What we are NOT doing

- Not blaming PR #142 author. The PR passed review with documented LLD. The system did not catch a foreseeable risk; that is a process gap, not a person failure.
- Not adding "more careful PR review" as an action item. That is not specific or verifiable.
- Not increasing the Reviewer's iteration cap. The review process worked as designed; the gap was upstream of review.

## Confidence and unknowns

- **High confidence:** PR #142 was the proximate cause. Evidence: temporal correlation + clear mechanism (rate increase).
- **Medium confidence:** Per-chat rate limits were the specific limit hit. Evidence: weak — only Telegram's 429 response, which is non-specific.
- **Unknown:** Whether this would recur if parallelism were increased to 3 instead of 5. Worth testing in staging before re-attempting.

## Linked artifacts

- Triage: `incidents/triage-log.md#ALR-2026-0517-001`
- Rollback PR: #143
- Affected PR: #142
- Component: `components/messaging-guest/`
- Action items: #144, #145, #146, #147
```

### Example 2: Refusal to RCA without evidence

**Input:** "Something seemed slow this afternoon. Can you write up an RCA?"

**Output:**
```
Cannot produce an RCA from this input. Reasons:

- No specific incident ID or alert reference
- No time window specified
- No component named
- No specific symptom described
- "Seemed slow" is not an observed signal — it's a perception that needs grounding in metrics

Before I can produce an RCA, I need:
1. The incident ID or triage record reference
2. The time window (start and end UTC)
3. The component(s) affected
4. The specific symptoms (error rate, latency, specific errors)
5. Resolution time and method

If this was a real production issue, check `incidents/triage-log.md` for the relevant triage record and link to it. If no triage record exists, the issue may not have been caught — file a triage record first, then we can RCA from there.

Will NOT fabricate an RCA narrative from vague memory.
```

## Guidelines

### Inputs to read

1. The triage record for the incident
2. All log entries in the incident window (with timestamps)
3. All metric values relevant to the incident
4. Recent commits to affected components (last 14 days)
5. The component CLAUDE.md
6. Linked LLDs and ADRs for the affected code paths
7. Past post-mortems on similar incidents (last 90 days)
8. The fix (commit SHA or PR number)

### Required structure (blameless post-mortem)

Following the Google SRE blameless post-mortem pattern, adapted for solo operator scale:

1. **Summary** — 2–3 sentences. What happened, when, impact.
2. **Severity**
3. **Timeline** — every entry has a timestamp and a citation (observed source)
4. **Root cause** — distinguish observed from inferred. Cite evidence for each claim.
5. **Contributing factors** — what made this possible or worse
6. **What went well**
7. **What didn't go well**
8. **Action items** — specific, owned, deadlined, verifiable, tracked in an issue
9. **What we are NOT doing** — explicit anti-actions to prevent over-correction
10. **Confidence and unknowns**
11. **Linked artifacts**

### Citation rules

Every claim of fact must cite one of:
- A specific log line: `[observed: Sentry error ID a7b2c1 at 14:23 UTC]`
- A specific metric: `[observed: p95 latency 8.2s for window 14:00–14:05 from Datadog]`
- A specific commit: `[observed: PR #142 commit a3f7c2]`
- A specific config: `[observed: wrangler.toml line 23]`
- Documentation: `[cited: Telegram Bot API docs at time of incident]`

Inference is allowed but must be explicitly labeled:
- `**Inferred (high confidence):**` — strong mechanism + correlation, but not direct evidence
- `**Inferred (medium confidence):**` — plausible but with gaps
- `**Inferred (low confidence):**` — speculation worth noting but not relied on

### Blameless principle

Post-mortems describe systems, not people.

- Wrong: "The PR author didn't consider rate limits"
- Right: "The PR's LLD did not enumerate rate limits as a failure mode" (cites the LLD as the artifact, not the person)

The goal is system improvements, not assigning fault.

### Action items must be specific

Bad action items:
- "Be more careful during reviews"
- "Improve monitoring"
- "Better testing"

Good action items:
- "Add `429 rate limit` to messaging-guest CLAUDE.md Risk surface section by 2026-05-21"
- "Add `RateLimitMonitor` test to `telegram-client.test.ts`; reproduces 429 + verifies backoff"
- "Update LLD template's Failure modes section to require external-API rate limits as a checked item"

Each action item: specific change, owner, deadline, verification step, tracked in a GitHub issue.

### What we are NOT doing — required section

Often the most useful section. Documents anti-actions to prevent over-correction.

Examples:
- "Not adding a manual approval step for all messaging-guest PRs — the review process worked; the gap was upstream"
- "Not blaming the PR author — the system failed to catch a foreseeable risk; person didn't cause this"
- "Not rolling back parallelism to 1 permanently — performance benefit was real; we add backoff instead"

### Anti-hallucination rules

1. **No claim without citation.** If you cannot point to evidence, the claim doesn't go in the post-mortem.
2. **No narrative coherence at the cost of truth.** If the evidence is contradictory or incomplete, say so explicitly.
3. **No invented timeline entries.** Every timestamp comes from a source you can name.
4. **No invented stack traces, error messages, or metric values.** Cite or omit.
5. **No "the team should..." — be specific** about who and what and when and how to verify.
6. **Confidence labels are mandatory** on every inference. Refuse to write inferences without confidence labels.

### What this skill NEVER does

- Write an RCA without a triage record to anchor it
- Fabricate timeline entries
- Invent log lines or error messages
- Assign blame to specific people
- Produce vague action items
- Skip the "What we are NOT doing" section
- Conclude with high confidence when evidence is weak

## Output

A post-mortem markdown file at `incidents/postmortem-{INC-ID}.md`, with all 11 required sections. Linked from the originating GitHub Issue. Linked from `RELEASE_NOTES.md` if action items will affect users.

Summary message:
- Post-mortem path
- Action items created (issue numbers)
- Confidence summary (how much of this is well-grounded vs inferred)
- Recommended review date (typically 90 days, to verify action items took)

## Source

Adapted from Google SRE *Postmortem Culture: Learning from Failure* (the blameless post-mortem template), simplified for solo operator and small-team scale. Citation discipline is more strict than the original to counter LLM narrative-coherence bias.

## See also

- `references/postmortem-template.md` — fillable template
- `references/citation-format.md` — exact format for each citation type
