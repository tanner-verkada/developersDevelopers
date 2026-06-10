---
name: verification-before-completion
description: Use when about to claim work is complete, fixed, passing, or ready. Requires running the verification command and showing its output before any success claim.
---

# Verification Before Completion

Fires when Claude is about to claim "done", "fixed", "passing", "ready", or any equivalent. Before any such claim, evidence must be produced.

## Procedure

1. Identify the relevant verification command (typecheck, test, build, lint — whichever matches the claim).
2. Run it. Capture the output.
3. Re-read the original ask and confirm every stated requirement is actually covered — green gates on half the scope is the most common false "done".
4. Pass: include the output verbatim in the response, then make the claim.
5. Fail: do NOT claim done. Surface the failure with the output. Never delete or skip a failing test to get to green.

## Example

Bad: "Tests are passing now."

Good:
```
$ bun run test:run
✓ src/lib/foo.test.ts (12)
Test Files  1 passed (1)
     Tests  12 passed (12)
```
Tests pass.
