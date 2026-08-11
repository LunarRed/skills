---
name: have-opus-build
description: Build the agreed change through Opus 5 subagents under Fable's direction, review, and supervision.
disable-model-invocation: true
---

# Have Opus Build

The user wants this change built by Opus 5 subagents under your direction, review, and supervision, so their Fable allowance goes to judgment rather than keystrokes. If the change to build is not yet agreed with the user, get agreement on its scope before dispatching anything.

## Dispatch

Opus 5 subagents write every implementation edit. Dispatch them with the Agent tool using `model: "opus"`, and where the harness lets you set reasoning effort, match it to the step's difficulty, up to max for the hardest steps.

Split the work into steps sized for one subagent each. Run independent steps in parallel and dependent steps in sequence.

Subagents start blank, with no memory of this conversation, so every dispatch must be self-contained:

- the context behind the change, compressed to what the step needs,
- the step's exact scope, file paths, and constraints,
- what done looks like, plus an instruction to report every file changed and any surprise found.

## Review

Review each subagent's diff against the agreed change before dispatching anything that depends on it. Send rework back to a subagent with your specific findings; apply a correction yourself only when it is a one-or-two-line review fix.

The build is done when every step's diff has passed your review.

## Verify and hand back

Run the relevant checks and tests. Test runs cost no model tokens, so run them directly. When one fails, send the failing output back to an Opus 5 subagent for the fix, then re-run.

Hand back to the user with what changed, the verbatim test results, and the diff left uncommitted. Committing is the user's decision.
