# PRD Checklist — Good vs Bad Examples

## User persona

**Bad:** "Users"  
**Bad:** "Our customers"  
**Bad:** "Property managers and their guests"  (two personas, not one)  
**Good:** "Bryan, a part-time STR operator managing 5–25 units"  
**Good:** "Maya, a guest checking in after 10pm to a property she's never visited"

Single named persona, one sentence. If the work serves multiple personas, split into multiple PRDs.

## Use case

**Bad:** "Users will be able to check booking status"  
**Bad:** "The system supports message drafting"  
**Good:** "At 9pm, Maya texts the listing wondering if late check-in is possible. The system drafts a reply within 30 seconds, surfaces it to Bryan, who taps approve. Maya gets a confirmation message and the door code at 9:01pm."

Scenario, not capability. Concrete actors, concrete time, concrete outcome.

## Acceptance criteria

**Bad:** "The drafting feature should be fast"  
**Bad:** "Users find the system easy to use"  
**Bad:** "We support guest messaging"  
**Good:** "A drafted reply is ready within 30 seconds of inbound message ingestion in 95% of cases over a 7-day measurement window"  
**Good:** "Outbound message can be approved with a single tap from the Telegram bot interface"  
**Good:** "If guest message contains a question about check-in, the draft includes the property's check-in instructions verbatim from the property record"

Each criterion: a number, a state, or an observable behavior. Writable as a test.

## Out of scope (at least 3)

**Bad:** "We're not doing everything"  
**Good:**
- Not handling Airbnb platform messaging in v1 (Telegram only)
- Not auto-sending messages to guests in v1 (drafts only, human approves)
- Not handling cleaner or owner communications in this epic (separate component)

Forces clarity by negation. The Out of Scope section often reveals scope creep the PRD author didn't notice.

## Dependencies

**Bad:** "Some other systems"  
**Good:**
- Internal: `messaging-ops` component for cleaner/owner channels
- External: Telegram Bot API (existing in fitness-intel reference stack)
- Data: Property records from Supabase `properties` table; guest records from `guests` table

Specific named systems. Version pins where they matter.

## Risks and unknowns

**Bad:** "There might be issues"  
**Good:**
- Risk: A bad auto-drafted message reaches a paying guest. Mitigation: Outbound gate (human approval) in v1.
- Risk: LLM token cost per draft is unknown until measured. Mitigation: Track $ per draft from day 1; cap at $0.50/draft as a hard ceiling.
- Unknown: Whether Bryan will trust drafts enough to approve them quickly. Will be resolved by 2-week real-use measurement.

Honest, specific, with how each will be addressed.

## Definition of Done

**Bad:** "Feature ships"  
**Good:** "Maya scenario above runs end-to-end in production for 7 consecutive days with ≥95% draft-within-30s rate and zero guest complaints attributed to the drafting feature."

One sentence. Testable.
