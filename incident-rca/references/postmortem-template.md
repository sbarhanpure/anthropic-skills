# Post-Mortem Template

Fillable template. Every section is required.

```markdown
# Post-Mortem: INC-YYYY-MMDD-NNN

## Summary
{2-3 sentences. What happened, when, customer impact.}

## Severity
P0 | P1 | P2 | P3

## Timeline
All times UTC. Every entry has a citation.

- **HH:MM** — {event} [observed: {source}]
- **HH:MM** — {event} [observed: {source}]
- ...

## Root cause

**Observed:** {fact} [citation]

**Inferred (high|medium|low confidence):** {hypothesis with reasoning} [evidence cited]

## Contributing factors

**Observed:** {fact} [citation]

## What went well

- {specific thing that worked, with citation if possible}

## What didn't go well

- {specific thing that did not work}

## Action items

Each must have: owner, verification step, deadline, tracked issue.

1. **{Specific change}**
   - Owner: {name}
   - Verification: {how we know it's done}
   - Deadline: {date}
   - Tracked: Issue #NNN

## What we are NOT doing

- Not {anti-action 1, with reasoning}
- Not {anti-action 2, with reasoning}

## Confidence and unknowns

- **High confidence:** {what we know with strong evidence}
- **Medium confidence:** {what we believe but evidence is weaker}
- **Unknown:** {what would help to know but we don't}

## Linked artifacts

- Triage: {path}
- Fix: PR #NNN
- Affected PR (if applicable): PR #MMM
- Component: components/{name}/
- Action items: #NNN, #MMM, ...
```

## Citation format reference

Inline citations follow this format:

| Source | Format |
|---|---|
| Log line | `[observed: Sentry error ID a7b2c1 at 14:23 UTC]` |
| Metric | `[observed: p95 latency 8.2s for 14:00-14:05 UTC, Datadog]` |
| Commit | `[observed: PR #142 commit a3f7c2]` |
| Config | `[observed: wrangler.toml line 23]` |
| Documentation | `[cited: Telegram Bot API docs, accessed YYYY-MM-DD]` |
| Past post-mortem | `[reference: incidents/postmortem-INC-...md]` |

If you can't fit any of these formats, you don't have evidence — don't claim.

## Confidence label rules

| Label | When to use |
|---|---|
| `**Observed:**` | Direct evidence exists in logs/metrics/code. Citation provided. |
| `**Inferred (high confidence):**` | Strong mechanism + clear correlation, but no direct evidence. E.g., "PR raised the rate; rate limits hit shortly after; mechanism is clear." |
| `**Inferred (medium confidence):**` | Plausible explanation, multiple alternatives also plausible. E.g., "The slow query could have caused this, but cold start is also consistent." |
| `**Inferred (low confidence):**` | Speculation worth noting, weak support. E.g., "Possibly related to the recent dep upgrade, no direct evidence." |
| `**Unknown:**` | We don't know, and we know we don't know. Worth flagging for future investigation. |

A post-mortem with mostly `Inferred (low confidence)` claims is honest about its uncertainty. A post-mortem that confidently asserts speculation is dangerous.
