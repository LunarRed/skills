---
name: have-opus-do-it
description: Hand an agreed task to subagents running the newest Opus model while you direct, review, and verify. Use when the user asks to "have Opus do it", "have Opus build it", or to use Opus subagents, for any kind of task: code, research, drafting, analysis. Use only when the user names Opus. Never infer it from the size or type of the task.
disable-model-invocation: false
---

# Have Opus Do It

The user wants this task done by Opus subagents under your direction, review, and supervision, so your token allowance goes to judgment rather than keystrokes. If the task is not yet agreed with the user, get agreement on its scope before dispatching anything.

## Choose the worker

Every subagent runs the newest Opus model the harness offers. Never pin a version number. In Claude Code and T3 Code the Agent tool's `model: "opus"` alias resolves to the current Opus, so use that. If the harness instead lists Opus models by version, pick the highest. If you are not a Claude model and the harness delegates across vendors, route the dispatch to Claude's newest Opus through the harness's delegation tool.

## Dispatch

Opus subagents do the hands-on work of every step, whether that is research, drafting, file wrangling, analysis, or code. When the task is a code change, they write every implementation edit. You keep direction, review, and the judgment calls. Where the harness lets you set reasoning effort, default to `high` and raise it to `xhigh` only for the hardest steps. Never use any level above `xhigh`, whatever it is called: `max`, `ultracode`, or anything added later.

Split the work into steps sized for one subagent each. Run independent steps in parallel and dependent steps in sequence.

Keep the irreversible and outward-facing actions out of the dispatches and in your own hands: committing, sending, publishing, deleting, and anything else the user would want to approve first. Subagents produce the thing; you bring it back to the user for the final call.

Subagents start blank, with no memory of this conversation, so every dispatch must be self-contained:

- the context behind the task, compressed to what the step needs,
- the step's exact scope, file paths, sources, and constraints,
- the named deliverable the subagent reports back (a diff, a written file, a findings report, or command output), plus an instruction to report where it lives and any surprise found,
- an instruction that any claim of done or working comes with the output of the check that was run, and that a claim with no check behind it says so. This asks for honest reporting, not extra checking.

## Review

Review each deliverable against the agreed task before dispatching anything that depends on it. Send rework back to a subagent with your specific findings; fix something yourself only when it is a one-or-two-line correction.

A worker's report is a progress report, not proof of completion. Check it against the brief's definition of done. A claim with no check output behind it is an assumption; treat it as not yet verified and check it yourself if it is cheap, or send it back if it is not. If items are open and no blocker is stated, send the worker back with a short message naming them, at most twice, then stop and report the stall to the user.

Every change in the deliverable must trace to something the brief asked for, and everything the brief asked for must appear in the deliverable. Reject changes, files, and rewrites that no requirement called for; two models handing work back and forth can widen a task well past what the user agreed to.

The task is done when every step's deliverable has passed your review.

## Verify and hand back

Check the deliverables against reality wherever reality is cheap to check: re-run the command, open the file that was supposedly written, spot-check a claim against the source it cites. For a code change, run the relevant checks and tests. Deterministic checks cost no model tokens, so run them directly. When one comes back wrong, send the evidence to an Opus subagent for the fix, then re-check.

Hand back to the user with what was produced, where it lives, and the verbatim results of any checks you ran. For a code change, leave the diff uncommitted. The final irreversible action is the user's decision.
