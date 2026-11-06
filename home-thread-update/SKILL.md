---
name: home-thread-update
description: Refresh a project's status page from its T3 Code threads on every machine. Use when the user asks to refresh or update this project's status page. Manual only; never run it on a schedule or on your own.
disable-model-invocation: false
---

# Home Thread Update

Reads every thread of this project on every machine, decides each thread's **phase**, groups the threads into **tasks**, and writes the result as one **snapshot**: a single database document at `status/current` on the project's **status page**. The page renders itself from the snapshot. This runs only in the **home thread**, and only when the user asks.

- **task**: one Asana DEV ticket, like `DEV-5086`. Tasks group threads. **Unassigned** (key `unassigned`) holds threads whose title names no ticket, or two or more.
- **phase**: one of seven keys, most urgent first: `waiting`, `blocked`, `unknown`, `coding`, `testing`, `monitoring`, `done`. The page shows them as "Waiting on you", "Blocked", "Unknown", "Coding", "Testing", "Monitoring", "Done".
- **machine**: one of the user's computers running T3 Code, identified by its `environmentId`.
- **locator file**: `.home-thread.json` in the project folder, written by `/home-thread-setup`.

The only thing this skill writes is `status/current`. The page stays as published, nothing gets scheduled, and the `notes` collection (the user's own task notes) is left alone.

## Steps

1. **Check this is the home thread.** If the claude-artifacts tool `db_write` or the T3 Code tool `t3_thread_list` is missing, reply in one line that this thread lacks those tools so update cannot run here, and stop. Then read `<project folder>/.home-thread.json`, where `<project folder>` is the project's `workspaceRoot` from `t3_project_read`, using the `projectId` that `t3_thread_read` reports for `$T3CODE_THREAD_ID`.
   - Missing: reply "This project has no status page yet. Run /home-thread-setup in the thread you want as the home thread." and stop.
   - `home.threadId` differs from `$T3CODE_THREAD_ID`: reply "This is not the home thread for <project>. Run /home-thread-update there." and stop.

   Done when the locator file is loaded and its `home.threadId` is this thread.

2. **Load the previous snapshot.** `db_get` with `slug: <page.slug>` and `path: "status/current"`. When `exists` is true, keep its data and its `version`.

   Done when you know whether a previous snapshot exists, and hold its `version` if it does.

3. **List each machine's threads.** For each entry in the locator file's `machines`, call `t3_thread_list` with its `projectId` and `environmentId`, `includeSubagents: false`, and `limit: 100`, paging with `cursor` until `nextCursor` is absent. Drop the home thread. Give the machine a result:
   - `ok`: every page listed, and every read in step 4 succeeds.
   - `partial`: listing stopped after some pages, or some reads in step 4 fail. Use the threads you got, and carry forward as `unknown` any previous row for this machine missing from the partial list.
   - `unreachable`: the first call failed, or the entry has no `projectId`. Carry forward every previous row for this machine.

   A carried-forward row gets `phase: "unknown"`, `basis: "recorded"`, `readFailed: true`, and `reason: "machine unreachable"`, and keeps its `now`, `needs`, and `readAt`. Set the machine's `attemptedAt` to now. Set `succeededAt` to now for `ok`, else keep the previous value. Write `note` in plain words for any result other than `ok`, e.g. "Did not answer. Its threads are shown as Unknown with their last known text."

   Done when every machine has a result and every listed thread is either queued for step 4 or carried forward.

4. **Decide each thread's phase.** Reading threads is the expensive part, so re-read a thread only when it has no previous row, its `updatedAt` differs from the previous row's `threadUpdatedAt`, or the previous row has `readFailed: true`. Every other thread keeps its previous row, `readAt` included, with `title`, `model`, and `threadUpdatedAt` refreshed from the list. For each thread to re-read:
   - One `t3_thread_read` with `afterPosition` set to the list's `itemCount` minus 40 (at least 0), `limit: 12`, and `maxCharsPerItem: 1200`. It returns the thread record (`pendingRequestCount`, `archived`), `recentRuns`, and the messages in that window, oldest first. While a result's `hasMore` is true, read again with `afterPosition` set to that result's `nextPosition`, until `hasMore` is false. Judge from the newest messages seen. Positions count every timeline item, tool calls included, so a window of 40 usually holds 3 to 10 messages. If these reads hold fewer than 3 messages in total and the first `afterPosition` was above 0, read again the same way from `itemCount` minus 120.
   - A record with `archived: true` is dropped from the snapshot.
   - Apply the phase checklist below. When `pendingRequestCount` is above 0, `t3_pending_request` with `action: "list"` shows the waiting questions; any pending request it leaves out is an approval.
   - Write `now` (one short sentence: what the thread is doing), `needs` (one short sentence: what it needs next, or "Nothing."), and `reason` (a few words: why this phase, e.g. "1 pending approval"). Set `readAt` to now and `readFailed: false`.
   - If a read fails, the row gets `phase: "unknown"`, `basis: "recorded"`, `readFailed: true`, and `reason: "read failed"`, keeps its previous `now` and `needs`, and the machine becomes `partial`.

   Done when every listed thread has a row with a `phase`, `basis`, and `reason`.

5. **Group threads into tasks.** Find `DEV-<digits>` in each thread's title. Exactly one distinct ticket: `taskKey` is that ticket. No ticket, or two or more: `taskKey: "unassigned"`. One hint applies: a title with no ticket whose `parentThreadId` is a thread with exactly one ticket takes the parent's ticket, and its `reason` ends with "ticket from parent thread".

   Done when every thread row has a `taskKey`.

6. **Write each task's summary.** One row per ticket in use, plus the Unassigned row exactly as in the snapshot shape below. For a ticket:
   - `shortTitle`: the text after the ticket in its clearest thread title, trimmed of spaces and leading punctuation.
   - `fullTitle`: the ticket's full name as its threads state it, else the `shortTitle`.
   - `done`, `next`, `blocking`: one short sentence each, drawn from its threads' rows. `blocking` reads "None." when nothing blocks.

   Keep a task's previous row only when its set of threads is the same as in the previous snapshot and none of them were re-read in step 4; otherwise write the row again. A ticket with no threads left gets no row.

   Done when every `taskKey` in use has a row in `tasks`.

7. **Write the snapshot.** Build the document in the shape below, with `refreshedAt` set to now. Send one `db_write` with the `slug` and `ops: [{ op: "set", path: "status/current", data: <snapshot> }]`, adding `ifVersion: <version from step 2>` when a previous snapshot existed. If the write is rejected for a version mismatch, report that to the user and stop. Then `db_get` `status/current` again.

   Done when the stored `refreshedAt` equals the one you wrote.

8. **Reply** in one or two lines: thread count, how many are `waiting`, which machines answered, and `page.url`. For example: `42 threads, 3 waiting on you. hiphopopotamus ok, rhymenocerous unreachable. <url>`

## Phase checklist

Recorded, exact:

1. `pendingRequestCount` above 0, or `status` is `waiting`: `waiting`, basis `recorded`.

Judged from the last few messages, basis `judged`. Check in this order; the first match wins:

2. The latest assistant message ends by asking the user a question that is required to continue, and no later user message answers it: `waiting`.
3. The thread says it is stopped on something outside the user's control, such as an answer from another person, a system that is down, or a dependency not yet merged: `blocked`.
4. `status` is `failed` or `interrupted` with no later run in `recentRuns`: `blocked`, reason "run failed" or "run interrupted".
5. The thread says a PR is merged, the task is shipped, or the work is complete with nothing further, or the thread is `settled`: `done`.
6. The thread is watching a deploy, a job, logs, or a rollout: `monitoring`.
7. The thread is running tests, QA, an evaluation, or a review of results: `testing`.
8. Anything else, including planning and reviewing diffs: `coding`.

A finished run alone is not `done`; it falls through to the rules after 5. An optional offer or a courtesy question, like "Want me to continue?", is not `waiting`; only a question the work cannot continue without counts for rule 2.

`unknown` comes only from step 3 or a failed read in step 4, never from this checklist.

## Snapshot shape

Store exactly these fields. The page works out each task's phase, the counts, and the sort order itself.

```jsonc
{
  // the locator file's project, and when this refresh ran (UTC)
  "project": "mm-ai-email-classification",
  "refreshedAt": "2026-11-06T17:32:00Z",
  // one per machine in the locator file; result is ok, partial, or unreachable
  "machines": [
    { "environmentId": "...", "label": "rhymenocerous", "result": "unreachable", "attemptedAt": "2026-11-06T17:32:00Z",
      "succeededAt": "2026-11-05T21:10:00Z", "note": "Did not answer. Its threads are shown as Unknown with their last known text." }
  ],
  // one per ticket in use, plus the Unassigned row exactly as shown
  "tasks": [
    { "key": "DEV-5086", "shortTitle": "Third-party deal emails", "fullTitle": "Fix third-party deal emails logging as contact Email",
      "done": "Fix implemented; BluePeak QA complete.", "next": "Approve the deploy order.", "blocking": "Your approval of the deploy order." },
    { "key": "unassigned", "shortTitle": "Unassigned", "fullTitle": "Threads with no ticket in their title", "done": "", "next": "", "blocking": "" }
  ],
  // one per thread except the home thread; key is "<environmentId>/<threadId>", threadId exactly as T3 gives it
  "threads": [
    { "key": "<environmentId>/<threadId>", "environmentId": "...", "threadId": "mcp:0ef0228b-...", "taskKey": "DEV-5086",
      "title": "⭐ DEV-5086: Third-party deal emails", "model": "claude-fable-5-1",
      "phase": "waiting", "basis": "recorded", "reason": "1 pending approval",
      "now": "Fix is ready; QA is complete.", "needs": "Deploy order approval.",
      "threadUpdatedAt": "2026-11-06T17:20:00Z", "readAt": "2026-11-06T17:32:00Z", "readFailed": false }
  ]
}
```
