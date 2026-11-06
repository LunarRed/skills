---
name: keep-warm
description: Keep a T3 Code thread's prompt cache warm during a break with a scheduled keepalive ping. Use when the user asks to keep this thread warm, stop keeping it warm, or show its keepalive status.
---

# Keep Warm

A keepalive is two T3 scheduled tasks bound to this thread. The **ping** fires every interval and gets a tool-free `Still Alive 🍰`. The **check** is what ends keep-warm, because T3 tasks have no expiry. Every T3 run queues behind an active turn and never interrupts it. Never set `freshSession` on either task: a fresh session discards the cache keep-warm exists to keep.

Two modes, set by the check:

- **Idle timeout**, the default. The check repeats once per window, continues if the user sent a message since the previous check, else deletes both tasks. Keep-warm ends one to two windows after the user goes quiet.
- **Fixed window**, only when the user asks for a hard stop ("until 6pm", "for exactly two hours", "no matter what"). The check runs once, at the stop time, and deletes both tasks.

## Find this thread's tasks

`$T3CODE_THREAD_ID` is the thread. `t3_thread_read` it (`limit: 1`) for `providerInstanceId`. In `list_scheduled_tasks`, this thread's tasks have that `boundThreadId` and titles `keep-warm ping <threadId>` and `keep-warm check <threadId>`.

## Enable

1. Interval: the user's, else by provider. The scheduler minimum is 1 minute. A user interval past the cache lifetime rebuilds the cache instead of keeping it, so say that once.
   - `claudeAgent`: run the check below. `ephemeral_1h` means **50 minutes**. `ephemeral_5m` or no output means **4 minutes**.

     ```bash
     sid=$(ps -o command= -p $PPID | grep -o 'session-id=[^ ]*' | cut -d= -f2)
     f=$(ls -t ~/.claude/projects/*/${sid:-*}.jsonl | head -1)
     tail -n 200 "$f" | grep -o '"ephemeral_[15][hm]_input_tokens":[1-9]' | tail -1
     ```
   - `codex` on GPT-5.5 or later: **25 minutes**.
   - Anything else: ask. Its cache lifetime is unverified.
2. Window: idle timeout, the user's duration, else **4 hours**. Fixed window, the hard stop as a time with an offset, e.g. `2026-11-06T18:00:00-08:00`.
3. Create the tasks with `schedule_task` and `bindToCurrentThread: true`, or update them by `scheduledTaskId` when they exist:
   - ping: `{type: "interval", everyMs: <interval ms>}`, prompt `Keep-warm ping. No tools or other work. Reply exactly "Still Alive 🍰".`
   - check: idle timeout `{type: "interval", everyMs: <window ms>}`, fixed window `{type: "once", runAt: <stop time>}`. A new check needs its own id in its prompt, so create it with a placeholder prompt, then set the real one. When it exists, set `enabled: false` and then `enabled: true` with the new schedule, which restarts the window from now.
     - idle timeout: `Keep-warm check. If I sent a message since your last "Keep-warm continues." or the reply that turned keep-warm on, ignoring keep-warm prompts, reply exactly "Keep-warm continues." and use no tools. Otherwise call delete_scheduled_task for <pingId>, then <checkId>, and reply exactly "Keep-warm stopped."`
     - fixed window: `Keep-warm stop. Call delete_scheduled_task for <pingId>, then <checkId>. Do nothing else. Reply exactly "Keep-warm stopped."`

   Done when this thread has exactly one ping and one check. Confirm in one line, e.g. `Keep-warm on: pings every 50 min, stops 4 to 8 h after you go quiet.` or `Keep-warm on: pings every 25 min until 6:00 PM.`

## Stop

Delete both tasks. A ping already queued behind this turn survives that, so `t3_queue` `list` this thread and `cancel` any queued ping. Reply `Keep-warm stopped.`

## Status

One line from `list_scheduled_tasks`: mode, interval, and the check's `nextRunAt` in local time. No tasks means keep-warm is off. A ping whose check has no `nextRunAt` means the stop ran but failed, so delete both and say so.

## Limits

Mention only when they bear on the request: T3 must stay running and the host awake, and each ping is billed as a full cached-prefix read.
