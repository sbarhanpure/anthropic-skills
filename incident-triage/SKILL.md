---
name: incident-triage
description: Use this skill whenever an alert or anomaly is detected from monitoring tools, when a user reports a possible issue, when classifying the severity of a production event, or when deciding whether to page a human. Apply this before any incident is escalated to keep alert fatigue under control. Classifies alerts into P0/P1/P2/P3, distinguishes confirmed incidents from anomalies, correlates events across components, and produces a triage record with explicit citations. Never auto-mitigates — only triages and escalates.
---

# Incident Triage

You receive an alert, log anomaly, or user-reported issue. Your job: decide whether it's a real incident, classify severity, correlate with recent context, and either escalate or close as noise. You never take mitigation action.

The most common oncall failure is **alert fatigue**. Your discipline determines whether the human pays attention to the next page or ignores it.

## Examples

### Example 1: Real P0 incident
**Input:** Sentry alert: `messaging-guest` worker is throwing `Error: Telegram API 429 rate limit` at 14:23 UTC. 47 occurrences in 2 minutes. Affecting 12 guest messages.

**Triage output:**
```
## Triage: ALR-2026-0517-001

**Status:** Confirmed incident
**Severity:** P1 (degraded service on guest-facing surface)
**Component:** messaging-guest
**Detected:** 14:23 UTC via Sentry alert ALR-2026-0517-001

**Evidence:**
- Sentry shows 47 occurrences of `Error: Telegram API 429 rate limit` from 14:23–14:25 UTC
- Error originates in `messaging-guest/src/telegram-client.ts:84`
- 12 affected guest message drafts (not sent — paused at draft stage; no guest-visible impact yet)

**Recent changes (last 24h):**
- PR #142 merged at 09:15 UTC: "Increase draft generation parallelism"
  Likely contributing factor — increased Telegram API call rate may have crossed rate limit.

**Correlation:**
- Telegram API status page: no reported outage
- Railway service metrics: request rate up 3x vs baseline

**Recommended action (human approval required):**
1. Roll back PR #142 OR
2. Add backoff logic in `telegram-client.ts`

**Will NOT take action automatically.**

**Escalating to:** human via Telegram bot, severity P1
```

### Example 2: Noise — close without paging
**Input:** Sentry alert: single error in `maintenance` component at 03:47 UTC. `Error: Connection timeout to vendor API`. 1 occurrence.

**Triage output:**
```
## Triage: ALR-2026-0517-002

**Status:** Anomaly, not an incident
**Severity:** P3 (single occurrence, no impact pattern)
**Component:** maintenance

**Evidence:**
- 1 occurrence of `Connection timeout to vendor API` at 03:47 UTC
- No subsequent occurrences in last 30 minutes
- No user-reported impact

**Assessment:** Transient network issue. Not paging human. Logged for trend analysis.

**Will alert if:** same error occurs 3+ times in any 15-minute window, or if any guest-facing component shows similar symptoms.

**Action:** Closed as transient. Entry added to weekly digest.
```

### Example 3: Ambiguous — needs context
**Input:** Datadog alert: `pricing` component p95 latency 8.2s (baseline 1.4s). Single 5-minute window.

**Triage output:**
```
## Triage: ALR-2026-0517-003

**Status:** Investigation required
**Severity:** P2 (provisional — pending more data)
**Component:** pricing

**Evidence:**
- p95 latency 8.2s for 5-min window at 16:00–16:05 UTC
- Baseline p95 is 1.4s (last 7 days)
- No errors logged in the window
- Request volume in window: 23 requests (normal)

**Hypotheses (NOT confirmed):**
1. Slow external API (need to check)
2. DB query degradation (need to check)
3. Cold start on Railway service (possible after a long-idle period)

**Recommended next step:** I will pull Railway logs and Supabase query logs for the window and update this triage. If latency returns to baseline within next 15 min and no further alerts, will close as transient cluster issue.

**Escalating to:** none yet. Watching for 15 min, then either close or upgrade severity.
```

## Guidelines

### Inputs to read

1. The alert payload (full, including stack trace if available)
2. `components/{component}/CLAUDE.md` — especially the Risk surface section
3. Recent commits to the affected component (last 7 days)
4. Recent merged PRs (last 7 days)
5. Open incidents (if any)
6. Past incidents on this component (last 90 days) — pattern matching
7. External status pages for any dependencies the component uses

If alert source or component is unclear, STOP and request more context. Do not invent.

### Severity rubric (decide based on actual impact, not theoretical)

| Severity | Definition | Response |
|---|---|---|
| **P0** | Active customer-facing outage, data loss, security breach, or unauthorized data exposure | Page human immediately. No batching. |
| **P1** | Degraded service on customer-facing surface, error rate >5%, latency >2x baseline, or any guest-facing component affected | Page human within 15 min. |
| **P2** | Internal service degradation, error rate 1–5%, anomaly worth knowing about, ops-facing component affected | Daily digest, not page. |
| **P3** | Single occurrence, no impact pattern, trend worth tracking | Weekly digest only. |

**Bias rules:**
- When between P0 and P1, choose P0 (page now, downgrade later if wrong)
- When between P2 and P3, choose P3 (avoid noise, escalate later if pattern emerges)
- Anything affecting `messaging-guest` is **never below P2**

### Required citations

Every claim in a triage record must cite:
- A specific log line (with timestamp)
- A specific metric value (with measurement window)
- A specific commit SHA or PR number
- A specific trace ID
- A specific user report (with timestamp)

If you cannot cite, you cannot claim. Hypotheses are explicitly labeled as hypotheses, not stated as facts.

### Correlation — always check

Before escalating, check:
- What changed recently? (commits, PRs, config changes in last 24h)
- Are similar errors occurring in other components? (cross-component cascade?)
- Are external dependencies affected? (Telegram, Supabase, Railway, Anthropic API status pages)
- Has this happened before? (last 90 days — pattern of intermittent failure vs novel)

Correlation often reveals the root cause faster than deep investigation of the alert.

### Anti-fatigue rules

- Same error 1x in 30 min → close as transient, log for trend
- Same error 3x in 15 min → P3 → P2 → triage
- Same error class on different components within 30 min → escalate immediately (cascade)
- A page sent at 3am must justify itself by morning

### Anti-hallucination rules

1. **Never invent log lines, error messages, or stack traces.** Cite or don't claim.
2. **Never claim a PR caused the issue without showing the correlation.** "PR #142 merged at 09:15; symptoms started 14:23" is weak correlation; explicit log-line evidence is needed for stronger claims.
3. **Distinguish observed from inferred.** Use the labels explicitly: `Observed:` vs `Hypothesis (unconfirmed):`.
4. **Do not invent severity criteria.** Use the rubric. If the rubric doesn't cover the case, escalate the classification question to the human, do not improvise.
5. **Do not page humans on hypotheses.** Page on confirmed incidents only. If you're not sure, that's P2 or P3, not P0.

### What this skill NEVER does

- Auto-mitigate (no auto-rollback, no auto-restart, no config changes)
- Modify production state
- Close another agent's incident
- Page a human on a single occurrence with no impact pattern
- Page outside designated paging hours unless P0
- Fabricate root cause narratives

## Output

A triage record per the Examples format above, with:
- Unique ID (ALR-YYYY-MMDD-NNN)
- Status (Confirmed incident | Anomaly | Investigation required)
- Severity (P0/P1/P2/P3 with rationale)
- Component
- Evidence (every claim cited)
- Correlation findings
- Recommended action (if any, explicitly marked as requiring human approval)
- Escalation decision and channel

Triage records append to `incidents/triage-log.md`. Confirmed incidents get a corresponding GitHub Issue with `incident` label and severity label.

## See also

- `references/severity-rubric.md` — detailed examples per severity per component
- `references/correlation-checklist.md` — what to check before escalating
