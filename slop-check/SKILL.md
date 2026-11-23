---
name: slop-check
description: Slop check on a finished change before its commit message is proposed. A checker on the other model family asks the user's four questions (unnecessary or duplicated code, tests or docs; foreseeable regressions; fixes overfitted to the examples that prompted them; documentation that logs the work instead of stating the outcome), then you judge each finding and fix what holds. Use when the user asks for a slop check, and when have-opus-do-it or have-sol-do-it hands back a code change. Correctness bugs belong to code-review, and in-place reuse cleanup to simplify.
---

# Slop Check

Slop is output written because it was easy to generate, not because the change needed it. A slop check asks whether each piece of a change should exist, whether the change will break something later, whether it solves the general problem rather than only the examples that prompted it, and whether its documentation says only what is needed. It is not a code review: correctness is a separate tool the user runs. The check runs after a change is built and its tests pass, and before its commit message is proposed. The four questions in the brief below, in their exact wording, are the checker's whole brief, and every report answers exactly those four, plus any questions the user adds (step 3).

## 1. Pick the change

The change is the uncommitted changes against the last commit, including new untracked files. If there are none and this thread just made a commit, it is that commit.

The check covers code that will be run and committed. Leave out throwaway measurement or scoring scripts.

Find what the change implements: a plan or ticket named in the conversation, or else the requirements agreed in the conversation, written out for the checker. The checker judges necessity against it, so ask the user only when neither exists.

## 2. Pick the checker

The checker runs on the other model family from whoever wrote the code: the newest Sol checks Opus's work, and the newest Opus checks Sol's. When the session wrote the code itself, the session's family is the author. A model the user names overrides this. Take the highest version the harness lists and never pin a version number. Reasoning effort is `high` unless the user asks otherwise.

- In T3 Code, dispatch through `delegate_task` to the other provider, reading the exact model id and effort option from its capabilities listing.
- In Claude Code, the Agent tool's `model: "opus"` reaches Opus. Plain Claude Code has no route to Sol.
- In Codex, the native subagent tool reaches Sol with its model id named explicitly. Codex has no route to Opus.

When the chosen checker model is out of reach, skip the check rather than substitute another model, and tell the user the slop check did not run and why. A skill that called this one still hands back its work, with that note. A checker from the author's own family shares the author's blind spots, which is why the families cross.

## 3. Brief the checker

The checker gets the change and the plan, and nothing about how the work went: no worker reports, no summary from you, no claim that tests pass. The author's conclusions pull a checker toward agreeing with them.

Put the findings file in the project's git-ignored working folder (often `docs/working/`), beside the plan when the plan lives there, named `slop-check.md`, with `-2`, `-3` for later runs. Confirm with `git check-ignore` that git ignores it. If the project has no ignored working folder, ask the user where it goes.

If the user's request or the conversation adds slop-check questions, add them to the brief in the user's wording after the four standard ones, numbered from 5, and change "four" in the brief to the new count. Each added question gets its own section in the findings file and its own part in your report.

Send this brief, filling in the angle brackets:

```text
Run a slop check on a code change. Slop is output written because it was easy to generate, not because the change needed it. Answer exactly these four questions about the change:

1. Does this write any unnecessary, extraneous, or duplicative code, tests, or documentation?
2. Can we foresee this causing regressions?
3. Does this solve the general problem, or is it overfitted to the particular examples that prompted or tested it?
4. Is the documentation a needlessly verbose wall of text logging what was done instead of a concise, strictly-needed summary of the final outcome?

The change is in <absolute checkout path>. See it with <`git diff HEAD` plus the untracked files in `git status --short` | `git show <commit>`>.
It implements <plan file path, ticket text, or the agreed requirements>. Every line of the change should trace to this.

You may read files and run read-only git commands. Your only write is the findings file at <absolute path>. Make no other edits, no commits, and no database or network calls.

What to look for under question 1:
- Helpers or branches nothing calls.
- The same rule proved by several tests, or a test that reruns a full pipeline where a unit test would do.
- Defensive guards, feature flags or switches the plan never asked for.
- Changes no requirement in the plan traces to.
- Facts repeated across several documents.

What to look for under question 2:
- Behavior change for inputs that never reach the new code path.
- Failure paths that can now escape and fail the whole operation.
- Measured or frozen behavior, such as prompts, request bytes or thresholds, changing unintentionally.
For each foreseeable regression, name the scenario and the test or run that would show it. Label a risk hypothetical when you constructed the scenario rather than saw it happen; measured results outrank constructed scenarios.

What to look for under question 3:
- Special cases keyed to the input that prompted the change, such as a specific id, name, string, date or file, instead of a fix to the rule that failed.
- A guard on the symptom, such as catching, clamping or skipping the bad value, while the cause upstream stays.
- Constants, thresholds or prompts tuned until the known examples pass.
- Tests that use only the reported example.
For each overfit, name a different input of the same kind that would still fail.

What to look for under question 4:
- Text that logs what was done (steps, runs, agents, retries) instead of stating the final outcome, its reason and its consequence.
- Decision-log entries far longer than the file's median entry. Measure the median and compare.
- Comments or docstrings that restate the code or narrate its history.

Write the findings file like this:
- First line: the model you are running.
- Then a one-sentence verdict.
- Then four sections, headed by the four questions in order. Each finding gives the file and line in the new version of the file, what is unnecessary or risky, and the smallest fix, in a few plain sentences. A section with no findings says so in one sentence.
The file holds findings only: everything in it answers one of the four questions. Write plain English in full sentences, with no em dashes.

Reply with the findings file's path and the number of findings under each question.
```

## 4. Judge and fix

Each finding is a claim. Open the cited code and decide whether it holds; drop a finding that misreads the code or the plan, and note why.

Fix every finding that holds with its smallest fix. When this check runs inside have-opus-do-it or have-sol-do-it, send the fixes to that skill's workers as it directs. When the smallest fix would go beyond what the plan agreed, ask the user first. Keep a finding that holds only for a reason the user would accept, such as the plan requiring it. Then rerun the tests the change was built against.

## 5. Report to the user

- Open with a one-sentence verdict: is the change ready for its commit message.
- Then one part per question, headed by the question. Each finding, in plain sentences, names the file and line, says what was unnecessary or risky, and says whether you fixed it or kept it, and why. A question with no findings gets one sentence.
- Then the test command you reran and its verbatim pass or fail line, the model that checked, and the findings file's path.
- Close with what the check does not prove, such as correctness, or a hypothetical regression nobody reproduced, and any decisions needed from the user.
