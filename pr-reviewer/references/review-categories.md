# Review Categories — Detailed Checklists

Loaded only when the reviewer needs more detail than the SKILL.md body provides.

## Spec compliance

- [ ] Every acceptance criterion in the task spec is satisfied by code in the diff
- [ ] Each criterion is verified by an automated test OR explicit manual verification step in PR description
- [ ] No acceptance criterion is interpreted differently than written
- [ ] PRD acceptance criteria (parent of task) are not violated by this change

Common failures:
- Implementation handles happy path but task spec says "and error case X" — partial compliance is blocking
- Test exists but tests a slightly different thing than the criterion says

## Test coverage

- [ ] Every new public function has at least one positive-path test
- [ ] Every new public function has at least one negative-path test
- [ ] Coverage on changed lines does not decrease from main
- [ ] All test commands in `components/{name}/CLAUDE.md` ran and exited 0
- [ ] Exit codes captured in PR description; if uncertain, re-run

Common failures:
- New error handling code with no test exercising the error path
- Coverage went up overall but down on the changed file specifically

## Error handling

- [ ] Every thrown error is caught at an appropriate boundary OR explicitly propagated
- [ ] All failure modes enumerated in the LLD are handled in code
- [ ] User-facing error messages do not leak internals (stack traces, paths, secrets)
- [ ] Retries follow component's documented retry policy (in CLAUDE.md)

Common failures:
- `try/catch` with empty catch block (swallows errors)
- Network call without timeout

## Security

- [ ] All external input is validated before use
- [ ] No secrets in code or commits
- [ ] Authz check exists for any operation reading or modifying data belonging to a user
- [ ] PII (in Cadence: guest data, payment info) is handled per component CLAUDE.md
- [ ] No SQL injection or command injection surfaces

Common failures:
- New endpoint reads `req.query.userId` and looks up the user without checking the caller has authz
- Hardcoded API key in a config file

## Performance

Only flag if NFRs are declared in LLD OR component CLAUDE.md specifies budgets.

- [ ] No obvious N+1 query patterns
- [ ] No unbounded loops over user-supplied input
- [ ] No synchronous I/O in hot paths
- [ ] Memory usage stays within budget if specified

## Convention

- [ ] Naming matches component CLAUDE.md conventions
- [ ] File structure matches last 5 PRs in component
- [ ] Error-handling pattern matches component's documented pattern
- [ ] Logging follows component's logging convention
- [ ] No mixing of paradigms (e.g., callback style in a promise-based module)

## Diff size

- 0–100 LOC: pass
- 100–400 LOC: pass; verify the work justifies the size
- 400+ LOC: blocking unless explicitly justified in PR description (e.g., generated file, mechanical migration)

Large diffs hide bugs. If the work is genuinely large, it should be split into multiple PRs against a feature branch.

## Scope discipline

- [ ] Every file changed is necessary for the task as specified
- [ ] No drive-by refactors of adjacent code
- [ ] No "while I was here" improvements
- [ ] Changes outside the task scope are listed in PR description as `## Out of scope changes` with justification

Adjacent improvements go in separate tickets. Scope discipline is non-negotiable.
