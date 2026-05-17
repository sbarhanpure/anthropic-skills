# Severity Rubric — Component-Specific Examples

Concrete examples for Cadence's components. Use as reference when the abstract rubric is ambiguous.

## messaging-guest

| Event | Severity | Why |
|---|---|---|
| Draft generation throwing errors on >50% of attempts | P0 | Guest-facing critical path broken |
| Draft auto-send to a guest without human approval (any occurrence) | P0 | Safety invariant violation |
| Draft generation latency >30s p95 | P1 | Degraded service on guest-facing path |
| Single draft generation failure with retry success | P3 | Transient, no user impact |
| Draft generation cost per draft >$0.50 (budget threshold) | P1 | Economic anomaly worth knowing immediately |

## messaging-ops

| Event | Severity | Why |
|---|---|---|
| Cleaner notification system not sending for 30+ min | P1 | Affects turnover SLAs |
| Owner statement generation failing | P2 | Recoverable, not time-critical |
| Single message send retry | P3 | Normal operation |

## turnover

| Event | Severity | Why |
|---|---|---|
| Turnover assignment loop fails to assign for >2h before next check-in | P1 | Operational impact on cleaner schedule |
| Supply tracking off by more than 10% in count | P2 | Anomaly worth investigating |
| Photo upload to vendor failing intermittently | P3 | Usually transient |

## maintenance

| Event | Severity | Why |
|---|---|---|
| Vendor dispatch API returning errors on emergency tickets | P1 | Could delay urgent fixes |
| Routine maintenance ticket creation failing | P2 | Recoverable manually |
| Vendor API timeout single occurrence | P3 | Common, retry handles it |

## pricing

| Event | Severity | Why |
|---|---|---|
| Pricing suggestion engine returning prices >2x or <0.5x current price | P0 | Could cause severe revenue loss if acted on |
| Calendar gap analysis hasn't run in 24h | P1 | Operator missing opportunity signals |
| Single price suggestion fails | P3 | Transient, suggested-only anyway |

## Cross-cutting

| Event | Severity | Why |
|---|---|---|
| Railway deploy failure | P1 | Blocks shipping |
| Supabase outage affecting all components | P0 | Total Cadence outage |
| Anthropic API rate limit hit | P1 | All LLM-using paths blocked |
| Telegram bot non-responsive | P1 | Bryan can't approve drafts |
| Token cost spike >2x daily baseline | P2 | Investigate before continuing autoscale |

## Severity tie-breakers

When between two severities, apply in order:

1. **Customer-facing?** If yes, choose higher severity.
2. **Reversible?** If a wrong action could cause data loss or guest impact, choose higher severity.
3. **Pattern or one-off?** Single occurrence with no pattern, lean toward lower severity (P3). Pattern of recurrence, lean higher.
4. **Time of day?** Don't downgrade just because it's 3am. Page based on impact, not convenience.

## After-hours paging

For Cadence v1 (single operator):
- P0: Page Bryan via Telegram regardless of time
- P1: Page Bryan if within 7am–11pm local time; batch to morning otherwise
- P2/P3: Never page; appears in daily digest at 7am

When Cadence has multiple operators / customers, revisit this rule.
