---
name: have-opus-do-it
description: Do the agreed task through Opus 5 subagents under Fable's direction, review, and supervision.
disable-model-invocation: true
---

# Have Opus Do It

The user wants this task done by Opus 5 subagents under your direction, review, and supervision, so their Fable allowance goes to judgment rather than keystrokes. If the task is not yet agreed with the user, get agreement on its scope before dispatching anything.

## Dispatch

Opus 5 subagents do the hands-on work of every step, whether that is research, drafting, file wrangling, analysis, or code. You keep direction, review, and the judgment calls. Dispatch subagents with the Agent tool using `model: "opus"`, and where the harness lets you set reasoning effort, match it to the step's difficulty, up to max for the hardest steps.

Split the work into steps sized for one subagent each. Run independent steps in parallel and dependent steps in sequence.

Keep the irreversible and outward-facing actions out of the dispatches and in your own hands: committing, sending, publishing, deleting, and anything else the user would want to approve first. Subagents produce the thing; you bring it back to the user for the final call.

Subagents start blank, with no memory of this conversation, so every dispatch must be self-contained:

- the context behind the task, compressed to what the step needs,
- the step's exact scope, file paths, sources, and constraints,
- the named deliverable the subagent reports back (a diff, a written file, a findings report, or command output), plus an instruction to report where it lives and any surprise found.

## Review

Review each deliverable against the agreed task before dispatching anything that depends on it. Send rework back to a subagent with your specific findings; fix something yourself only when it is a one-or-two-line correction.

The task is done when every step's deliverable has passed your review.

## Verify and hand back

Check the deliverables against reality wherever reality is cheap to check: re-run the command, open the file that was supposedly written, spot-check a claim against the source it cites. Deterministic checks cost no model tokens, so run them directly. When one comes back wrong, send the evidence to an Opus 5 subagent for the fix, then re-check.

Hand back to the user with what was produced, where it lives, and the verbatim results of any checks you ran. The final irreversible action is the user's decision.
