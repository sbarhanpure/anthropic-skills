# Tier Rubric — Worked Examples Per Row

For each rubric row, examples of what S, M, L look like in practice.

## Lines of code (rough)

- **S (<20):** Typo, single-condition bug fix, dependency version bump, copy edit, config tweak
- **M (20–500):** New function with tests, refactor of one module, schema column add
- **L (500+):** New service, new component, multi-file migration, integration with new external system

## Files touched

- **S (1):** Single edit, single test
- **M (1–5):** Function + its tests + maybe a caller update
- **L (5+):** Genuinely cross-cutting change

## New external dependencies

- **S/M (0):** No new libraries, no new services, no new APIs
- **L (1+):** Any new third-party library, any new service call, any new API integration. The dependency itself is the L signal — even a "simple" library add has version, license, security, and ops implications worth an ADR.

## Schema changes

- **S (none):** No DB changes
- **M (minor):** Add a nullable column, rename a column with simple backfill, add an index
- **L (major):** New table, new FK relationship, breaking migration, migration requiring backfill of >10k rows, splitting a table

## Public API changes

- **S (none):** Internal change only, no contract surfaces affected
- **M (additive):** New optional parameter, new field in response, new endpoint that doesn't interact with existing ones
- **L (breaking or new):** Removing/renaming a parameter, changing response shape, deprecating an endpoint, adding an endpoint that requires authz design

## Cross-component impact

- **S (none):** Change is invisible to other components
- **M (one consumer):** One other component will need to know but no contract change to them
- **L (multiple consumers):** Multiple components or external systems must be updated in coordination

## New failure modes

- **S (none):** Worst case is the existing failure modes still apply
- **M (0–1):** One new way this can fail (e.g., network timeout to a new service)
- **L (2+):** Multiple new failure surfaces; needs explicit enumeration in LLD

## Security surface

- **S (unchanged):** No new input handling, no new data exposure, no new authz path
- **M (unchanged):** Same as S; if security surface changes at all, it's L
- **L (new):** Any new input from outside the system, any new data egress, any new authz check

Rule of thumb: when in doubt about security, it's L.

## Rollback complexity

- **S (revert PR):** `git revert` is sufficient
- **M (revert PR):** Same as S — if it requires more, it's not M
- **L (documented rollback plan):** Requires data migration reversal, requires coordination with other components, requires manual steps

## Always L

- Any ticket touching `messaging-guest` component (Cadence specific)
- Anything affecting payment, billing, or financial records
- Anything affecting PII handling
- Anything requiring legal or compliance review

## Common misclassifications

**"It's just a config change" → often M, sometimes L**
- If the config controls behavior in production, the impact is the size, not the diff

**"It's just a copy change" → almost always S, but check**
- Copy that goes to guests in `messaging-guest`: still L because the deploy gate applies

**"It's just adding a column" → M, never S**
- Schema changes always need migration plan, even simple ones

**"It's just a small refactor" → M, not S, if changing public-facing behavior**
- Public API touched → M minimum
