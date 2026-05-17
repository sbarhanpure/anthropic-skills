# Nygard ADR Template

Bare template. Fill in the four content sections; Status starts as Proposed.

```markdown
# ADR-NNNN: {Title}

## Status
Proposed

## Context

{The forces at play. What's the situation that requires a decision? What constraints are operating? What business, technical, social, or political factors matter? Use value-neutral language — describe facts, don't yet take a position. Usually 1–3 paragraphs. Where multiple options are being considered, list them here with brief description.}

## Decision

{Stated in present tense, active voice. "We will use X because..." Include the reasoning, not just the conclusion. Usually 1–3 paragraphs.}

## Consequences

{Both positive and negative. What becomes easier? What becomes harder? What new risks or obligations does this create? What do we accept as cost?

Format suggestion:

**Positive:**
- ...
- ...

**Negative:**
- ...
- ...

**Accepted because:** {one-sentence justification for why we accept the negatives}}
```

## When superseding

```markdown
# ADR-NNNN: {New title} (supersedes ADR-MMMM)

## Status
Proposed. Will supersede ADR-MMMM.

## Context
{Include why the prior decision is being revisited. What changed? What data do we have now that we didn't?}

## Decision
{New decision in present tense.}

## Consequences
{...}

## Supersedes
ADR-MMMM: {original title}. Original was correct given the information at the time. Revised because: {one sentence}.
```

Then update the original ADR's status line to:
```
## Status
Superseded by ADR-NNNN ({new title})
```

Do not modify any other content of the original.
