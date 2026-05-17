---
name: tdd-implementer
description: Use this skill whenever implementing new functionality, fixing bugs, refactoring code that needs test coverage, or working on any task where confidence in correctness matters more than speed of first commit. Apply this for every coding task to enforce test-first discipline: write failing test, run to confirm failure, implement minimum code, run to confirm pass, refactor with tests as safety net. Catches the most common code-agent failure mode of writing plausible-looking code without verifying it actually does what it claims.
---

# TDD Implementer

You write code test-first. Tests come before implementation, always. This is not a style preference — it is the most effective hallucination guard available for code-generating agents.

The discipline is: write a failing test that captures the intent, run it to confirm it fails for the right reason, write the minimum code to make it pass, run it again to confirm green, refactor with the test as a safety net.

## Examples

### Example 1: New function

**Task:** Add `normalizePhoneNumber(input: string): string` that strips formatting and returns E.164.

**Wrong (implementation-first):**
```typescript
function normalizePhoneNumber(input: string): string {
  return input.replace(/[^\d+]/g, '');
}
// "Looks right, let me add a test if I have time"
```

**Right (test-first):**
1. Write the test:
```typescript
test('normalizePhoneNumber strips spaces and dashes', () => {
  expect(normalizePhoneNumber('+1 555-123-4567')).toBe('+15551234567');
});
test('normalizePhoneNumber preserves leading +', () => {
  expect(normalizePhoneNumber('+15551234567')).toBe('+15551234567');
});
test('normalizePhoneNumber strips parentheses', () => {
  expect(normalizePhoneNumber('(555) 123-4567')).toBe('5551234567');
});
test('normalizePhoneNumber returns empty for empty input', () => {
  expect(normalizePhoneNumber('')).toBe('');
});
```
2. Run: 4 failures (function not yet defined)
3. Implement:
```typescript
export function normalizePhoneNumber(input: string): string {
  return input.replace(/[^\d+]/g, '');
}
```
4. Run: 4 passes
5. Add edge cases test discovered during implementation:
```typescript
test('normalizePhoneNumber strips embedded letters', () => {
  expect(normalizePhoneNumber('+1-CALL-NOW')).toBe('+1');
});
```
6. Run: 5 passes
7. Done.

### Example 2: Bug fix

**Task:** Fix bug where `MessageQueue.dequeue()` returns null when queue has items.

**Wrong:**
```typescript
// Inspect code, find suspicious lock condition, fix it.
// Tests pre-existing, hope they still pass.
```

**Right:**
1. Reproduce the bug as a failing test first:
```typescript
test('dequeue returns the item when queue has items (regression for bug #234)', () => {
  const q = new MessageQueue();
  q.enqueue({ id: '1' });
  expect(q.dequeue()).toEqual({ id: '1' });
});
```
2. Run: failure (confirms the bug)
3. Fix the code
4. Run: pass
5. Run full suite: ensure no regression
6. The new test stays in the suite as a regression guard.

### Example 3: Refactor

**Task:** Refactor `MessageQueue.enqueue()` to accept a priority parameter while preserving existing callers.

**Right:**
1. Existing tests must still pass with the refactor (these guard the contract)
2. Add new tests for the new behavior:
```typescript
test('enqueue with priority places item ahead of lower-priority items', () => {
  const q = new MessageQueue();
  q.enqueue({ id: '1' }, 1);
  q.enqueue({ id: '2' }, 10);
  expect(q.dequeue()).toEqual({ id: '2' });
});
test('enqueue without priority defaults to priority 5 (backward compat)', () => {
  const q = new MessageQueue();
  q.enqueue({ id: '1' });
  // Existing callers still work
});
```
3. Run new tests: failures
4. Refactor the implementation
5. Run all tests: pass
6. The existing test suite is the safety net that proves the refactor didn't break callers.

## Guidelines

### The discipline

For every coding task:
1. **Plan in writing first.** What is the function/change supposed to do? What inputs, what outputs, what edge cases?
2. **Write the failing test.** Capture the intent as a test that will fail until the work is done.
3. **Run it. Confirm it fails for the right reason.** A test that "fails" because of a syntax error doesn't count.
4. **Write minimum code to pass.** Resist the urge to add adjacent improvements.
5. **Run again. Confirm green.**
6. **Refactor if needed, with tests as safety net.**
7. **Add edge case tests discovered during implementation.**
8. **Run the full component test suite.** Per `components/{name}/CLAUDE.md` test commands.

### What "failing for the right reason" means

A test should fail because:
- The function being tested doesn't exist yet, OR
- The function exists but doesn't have the new behavior, OR
- The bug being fixed is reproducing as expected

A test should NOT count as a valid failure if:
- It has a syntax error
- It can't import a module that exists
- The test framework itself is broken
- The assertion is wrong (testing the wrong thing)

Read the failure output. Make sure it's the failure you expected.

### Coverage expectations

Per `STANDARDS.md` §4:
- Unit tests on new logic: always
- Negative path tests on public functions: always (one failure mode minimum)
- Coverage on changed lines: ≥80% target

For bug fixes: the test that reproduces the bug stays in the suite as a regression guard. Without that, you'll fix the same bug again.

### When TDD doesn't apply

- Pure exploratory spikes where you don't yet know what you're building (still write tests before merge though)
- Generated code (e.g., schema-generated types) — tested by code that uses them
- Throwaway scripts (verify by running, not by automated test)

These are exceptions, not the default. Default is test-first.

### Anti-hallucination rules

1. **Never claim "tests pass" without running them.** Every claim of green must come with the command and exit code.
2. **Never write a test you don't first run failing.** A test you've never seen fail can be silently broken.
3. **Never invent a test framework API.** Read the existing tests in the component to learn the patterns.
4. **Never delete or skip a failing test to "make it green."** That's hallucination of progress. Fix the code or escalate.
5. **Never claim coverage without running the coverage tool.** Coverage commands are in `components/{name}/CLAUDE.md`.
6. **If a test fails on retry, that's signal, not noise.** Investigate before re-running.

### What this skill NEVER does

- Skip the failing-test step "to save time"
- Mark tests as `skipped` or `xfail` without a linked tracking issue
- Add tests after the fact and claim TDD was followed
- Modify existing test assertions to make the test pass (that's hallucination)
- Reduce coverage on changed lines

## Output

For each coding task, the deliverable includes:
1. The plan (in PR description)
2. The new tests (committed before or with the implementation)
3. The implementation
4. Evidence of test runs: commands and exit codes in PR description
5. Coverage report or coverage delta on changed lines

## See also

- `references/tdd-workflow.md` — step-by-step workflow for the test-first cycle
