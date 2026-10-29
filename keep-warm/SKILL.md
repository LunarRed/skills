---
name: keep-warm
description: Keep a T3 Code thread's prompt cache warm during a break with a scheduled keepalive ping. Use when the user asks to keep this thread warm, stop keeping it warm, or show its keepalive status.
---

# Keep Warm

Keep this T3 thread's prompt cache alive during a break. A keepalive is two T3 scheduled tasks bound to this thread:

- the **ping**, an interval task whose run gets the minimal reply, titled `keep-warm ping <threadId>`,
- the **stop**, an interval task whose interval is the whole window, so its first run lands at the stop time and deletes both tasks, titled `keep-warm stop <threadId>`.

T3 has no expiry field on scheduled tasks, so the stop task is how the window ends. T3 runs it on time. The model in that run deletes both tasks, and if the delete fails, the stop task fires again one window later.

The window runs in one of two modes:

- **Idle timeout**, the default. A ping that finds a user message since the last keep-warm reply restarts the stop task, so the window ends a full window after the user goes quiet.
- **Fixed window**, only when the user asks for a hard stop: "until 6pm", "for exactly four hours", "fixed window", "no matter what". The window ends at that time whatever the user does.

## Find this thread and its tasks

1. Thread id: `$T3CODE_THREAD_ID`. Read it with `t3_thread_read` (`limit: 1`) to get `providerInstanceId`, `model`, and `link`.
2. Call `list_scheduled_tasks` and keep the tasks whose `boundThreadId` is this thread and whose `title` starts with `keep-warm`. These are this thread's ping and stop.

## Enable

1. Pick the interval: the user's explicit interval if given, else the default for this thread's provider:
   - `claudeAgent`: check the cache tier this session writes:

     ```bash
     sid=$(ps -o command= -p $PPID | grep -o 'session-id=[^ ]*' | cut -d= -f2)
     f=$(ls -t ~/.claude/projects/*/${sid:-*}.jsonl | head -1)
     tail -n 200 "$f" | grep -o '"ephemeral_[15][hm]_input_tokens":[1-9]' | tail -1
     ```

     `ephemeral_1h` means the 1-hour cache: use **50 minutes**. `ephemeral_5m` or no output means the 5-minute cache: use **4 minutes**.
   - `codex` on GPT-5.5 or later: **25 minutes** (OpenAI's documented 30-minute minimum cache lifetime). OpenAI's Codex lead said in October 2026 that signed-in Codex keeps its cache indefinitely. That is not in OpenAI's docs yet, and local logs still show gpt-6.1-sol losing the cache after 41 to 52 idle minutes, so keep the interval.
   - anything else: ask the user for an interval. The cache lifetime is unverified.

   The scheduler minimum is 1 minute. If an explicit interval is at or past the cache lifetime, say once that pings will land after expiry and rebuild the cache rather than preserve it, then use it.
2. Pick the mode and window. Idle timeout: the user's duration, else **4 hours**. Fixed window: the time from now to the user's hard stop.
3. Pick the ping prompt for the mode, filling in the stop id and window:
   - idle timeout: `Cache keepalive. If a message other than a keep-warm prompt arrived after your last keep-warm reply ("Still Alive 🍰" or the reply that turned keep-warm on), first call schedule_task for <stopId> with enabled false, then again with enabled true and schedule {"type": "interval", "everyMs": <window ms>}. Use no other tool, resume no previous work, change no files, and create no tasks. Reply exactly "Still Alive 🍰".`
   - fixed window: `Cache keepalive only. Reply exactly "Still Alive 🍰". Do not use tools, resume previous work, change files, or create additional tasks.`
4. No tasks yet: create them in this order, all with `schedule_task` and `bindToCurrentThread: true`:
   1. the stop, titled `keep-warm stop <threadId>`, with `schedule: {type: "interval", everyMs: <window ms>}` and a placeholder prompt,
   2. the ping, titled `keep-warm ping <threadId>`, with `schedule: {type: "interval", everyMs: <interval ms>}` and the ping prompt,
   3. the stop's real prompt, set by its `scheduledTaskId`:

      `Keep-warm window ended. Call delete_scheduled_task for <pingId>, then for <stopId>. Use no other tool and resume no previous work. Reply exactly "Keep-warm stopped."`
5. Both tasks exist: update the ping by `scheduledTaskId` with the new schedule and ping prompt. Then call `schedule_task` on the stop with `enabled: false`, then again with `enabled: true` and the window schedule. That pair restarts the window from now.
6. Only one of the two exists: delete it, then create both fresh.

Done when `list_scheduled_tasks` shows exactly one ping and one stop for this thread, both enabled. Report the mode in plain words ("stops 4 hours after you go quiet" or "stops at 6:00 PM no matter what"), the interval and why it was picked, the stop time (the stop task's `nextRunAt` in local time), the thread `link`, and both task ids.

## Stop

Delete this thread's ping and stop with `delete_scheduled_task`. A ping already queued behind the current turn survives the delete, so also `t3_queue` `list` this thread and `cancel` each queued run whose text is the ping prompt. Done when `list_scheduled_tasks` shows no `keep-warm` task for this thread and the queue holds no ping. Leave every other task alone.

## Status

Report each `keep-warm` task for this thread: the mode (idle timeout when the ping prompt calls `schedule_task`, fixed window otherwise), interval, `nextRunAt` and `lastRunStatus` for the ping, and the stop time. In idle-timeout mode, say the stop time moves out whenever a ping finds the user active. No tasks means keep-warm is off.

## Limits to tell the user

Mention these when they bear on the request:

- T3 must stay running and the host awake. Pings missed while asleep fire once on wake, after the cache has expired.
- The ping interval is fixed. T3 exposes no last-activity reset, so pings continue while the user is active. In idle-timeout mode, the stop time counts from the last ping that found the user active, so expect up to one interval of slack.
- A ping that comes due mid-turn queues behind that turn. It never steers or interrupts, and it runs once the turn ends. Pings that come due during a long turn pile up and run back to back.
- Each ping is a real turn and is billed as a full cached-prefix read. An idle-timeout ping that finds the user active makes two more requests to restart the window. Changing the model, tools, or system prompt, or compacting the thread, invalidates the cache anyway.
