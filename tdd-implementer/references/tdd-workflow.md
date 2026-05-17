# TDD Workflow — Step by Step

Use this when you need the precise sequence to follow. The SKILL.md body explains the principle; this file is the procedural detail.

## Pre-flight

Before writing any code or test:

1. Read the task spec end to end
2. Read `components/{name}/CLAUDE.md` for test commands and conventions
3. Look at the 3 most recent tests in the component to learn the test framework patterns
4. Write the plan in the PR description:
   - What this change does
   - What tests will be added
   - What edge cases are anticipated
   - What is explicitly out of scope

## The cycle

### Step 1: Write a failing test

Pick the smallest meaningful behavior the task implies. Write a test that captures it. The test name should describe the behavior, not the code:

Bad: `test('returns string')`  
Good: `test('normalizePhoneNumber strips spaces and dashes from US-format numbers')`

### Step 2: Run the test, confirm it fails

Use the unit-test command from `components/{name}/CLAUDE.md`. Read the failure output. Confirm it fails because the behavior isn't implemented, not because of a typo.

Capture the exact output (command + exit code) in your PR draft.

### Step 3: Write minimum code to make the test pass

Resist:
- Adding unrelated improvements
- Handling cases not in any test
- "While I'm here" refactors

Just make the test pass. Cleanup happens in step 6.

### Step 4: Run the test, confirm green

Same command. Exit 0.

If it's red, fix the implementation. Do not modify the test to be more lenient.

### Step 5: Write next test

Continue with the next behavior the task implies. Repeat steps 1–4.

Tests typically include, at minimum:
- One positive path
- One negative path (invalid input, missing data)
- Edge case (empty, null, boundary value)

### Step 6: Refactor with tests as safety net

Once green for the desired behavior, look for:
- Duplication you introduced
- Names that aren't quite right
- Functions that could be smaller
- Code that's hard to read

Refactor. Re-run tests after each change. The test suite is what makes refactoring safe.

### Step 7: Run the full component test suite

Before opening the PR:

```bash
# From components/{name}/CLAUDE.md
<unit command>
<integration command>
<regression command>
<lint command>
<type-check command>
```

All exit 0, or stop and investigate.

### Step 8: Verify coverage

Run the coverage command from `components/{name}/CLAUDE.md`. Confirm coverage on changed lines is ≥80% (or component-specific target).

If below, add more tests until threshold is met.

## When you discover an edge case during implementation

Common scenario: you're implementing the happy path and realize there's a case you didn't think about.

**Right response:**
1. Stop the implementation
2. Write a test for the new edge case
3. Run it — confirm it fails
4. Update implementation to handle it
5. Run — confirm green

**Wrong response:**
- Adding a `// TODO: handle X` comment and moving on
- Writing code to handle X without first writing a test for it
- Assuming "I'll add the test later"

## When a test that was green goes red

Common scenario: refactor caused a regression elsewhere.

**Right response:**
1. Stop. The regression matters more than the new feature.
2. Read the failure output.
3. Decide: is the test wrong, or is my code wrong?
4. If code is wrong, fix it.
5. If test is wrong (rare), update test — but you must justify in PR description with reasoning.

**Wrong response:**
- Disabling the test
- Adding `skipIf` conditions
- Claiming "this test was flaky anyway"

## When you're stuck

If you've spent more than 30 minutes on one failing test:

1. Re-read the task spec
2. Verify your test asserts the right thing
3. Try writing the simplest version of the test (no edge cases, just happy path)
4. If still stuck, write to `OPEN_QUESTIONS.md` and stop. Escalate.

Spending hours fighting one test means the task isn't clearly defined or the design is wrong. Escalation is the right move, not heroic persistence.

## PR description requirements

After the cycle is complete, the PR description must include:

```
## Tests
- Unit: `<command>` → exit 0
- Integration: `<command>` → exit 0
- Regression: `<command>` → exit 0
- Coverage on changed lines: N% (was M%)

## Test files added
- path/to/test1.ts (N new tests)
- path/to/test2.ts (M new tests)

## Edge cases covered
- empty input
- malformed input
- boundary value at maximum

## Acceptance criteria verified
- [x] Criterion 1 — verified by `test_name_1`
- [x] Criterion 2 — verified by `test_name_2`
```

Without this, the `pr-reviewer` skill will reject the PR.
