---
name: home-thread-setup
description: Create a project's status page and make this T3 Code thread its home thread. Use when the user asks to set up a home thread or create a status page for this project.
disable-model-invocation: false
---

# Home Thread Setup

The **home thread** is the one T3 Code thread, running Claude, where the user runs `/home-thread-setup` and `/home-thread-update` for a project. Setup publishes the project's **status page**, a private claude.ai artifact page, and records where everything lives in the **locator file**, `.home-thread.json` in the project folder. `/home-thread-update` reads that file later to fill the page.

A **machine** is one of the user's computers running T3 Code. T3 calls it an environment and gives it an `environmentId`. `$T3CODE_THREAD_ID` is this thread. `<project folder>` is the project's `workspaceRoot` from `t3_project_read`, using the `projectId` that `t3_thread_read` reports for this thread; the shell does not set a variable for it, and a worktree thread's working directory is not the project folder.

## Steps

1. **Confirm the environment.** This skill needs the claude-artifacts tool `publish_files` and the T3 Code tool `t3_environment_links`. If either is missing, reply in one line that this thread lacks the claude-artifacts tools (or the T3 Code tools) so setup cannot run here, and stop. Suggest no fix; the tools come with the Claude thread or they do not.

   Done when both tools are present.

2. **Check for an existing status page.** Read `<project folder>/.home-thread.json`.
   - No file: continue to step 3.
   - The file exists and `get_artifact` on its `page.slug` succeeds: this project already has a status page. Report `page.url`. In one message, ask whether to make this thread the home thread (only when `home.threadId` differs from `$T3CODE_THREAD_ID`) and whether to pin this thread and rename it to `🏠 <project name>`, where the project name is the file's `project`. On a yes to the first, set `home` in the locator file to this machine and this thread. Pin and rename apply only when this thread is the home thread after that answer. Then go to step 7.
   - The file exists but `get_artifact` answers that the page is not found: tell the user the page is gone, and continue to step 3 as a fresh setup.
   - The file exists but `get_artifact` fails for any other reason, such as an expired login or a service error: reply in one line that the status page could not be checked, and stop without publishing anything.

   Done when you know which of the four cases applies.

3. **Find the project on each machine.** `t3_environment_links` returns this machine's `environmentId` at the top level and the other machines in `links`, each with a `label` (its hostname) and a `status`.
   - This machine: `t3_project_list` with its `environmentId`, paging with `cursor` until `nextCursor` is absent. The project is the one whose `workspaceRoot` equals `<project folder>`. Its `title` is the project name used in every later step.
   - Each other machine with `status: "reachable"`: `t3_project_list` with that `environmentId`, paged the same way. Propose the project whose `repositoryIdentity.canonicalKey` matches this project's, else the one whose `title` matches.
   - A machine that is not reachable, does not answer, or has no matching project gets no `projectId`.

   Done when every machine from `t3_environment_links` has a proposed `projectId` or is marked as having none.

4. **Ask once.** Send one message that:
   - lists each machine's label with its proposed project (`title` and `workspaceRoot`), or "no project found", and asks the user to confirm or correct the matches,
   - asks whether to pin this thread and rename it to `🏠 <project name>`,
   - asks the user to confirm publishing the status page.

   Done when the user has answered all three. If they decline publishing, stop.

5. **Publish the status page.**
   - `publish_files` with no `slug`, `files: [{ as: "index.html", path: "<skill base directory>/assets/status-page.html" }]`, `title: "Status page: <project name>"`, and `icon: "list"`. `<skill base directory>` is the folder Claude Code names when it loads this skill. Keep the `slug` and `url` from the result.
   - If `publish_files` refuses the path because it is outside the project folder, copy the template to `<project folder>/.home-thread-page-<current unix time>.html`, publish from there, then remove the copy with `rm -f` and confirm with `ls` that it is gone. If a file with that name already exists, do not copy over it; tell the user and stop.
   - `read_declaration` with the `slug` for its `version`, then `declare_capabilities` with the `slug`, `base_version: <that version>`, and `capabilities: { db: {} }`.

   Done when `declare_capabilities` succeeds and any template copy made in this step is gone.

6. **Write the locator file and check the page.**
   - Write `<project folder>/.home-thread.json` in the shape under "Locator file" below. `home` is this machine and `$T3CODE_THREAD_ID`. `machines` lists every machine, this one included, with the `projectId` the user confirmed, and leaves `projectId` out for a machine with none. `pinnedAndRenamed` is the user's answer from step 4.
   - Keep the file out of git without changing any `.gitignore`. Get the exclude file's path with `git -C "<project folder>" rev-parse --path-format=absolute --git-path info/exclude` (this also resolves correctly in a git worktree), and append the line `.home-thread.json` unless the file already has it.
   - `get_artifact` on the `slug` and confirm it returns the page `url`. If a browser tool is available, open the URL and confirm the page reads "No snapshot yet. Run /home-thread-update in the home thread."

   Done when the locator file parses as JSON, the exclude file contains `.home-thread.json` exactly once, and `get_artifact` returns the URL.

7. **Pin, rename, and hand off.** If the user said yes to pinning and renaming, call `t3_thread_organize` with `action: "pin"` and `t3_thread_update` with `action: "rename"` and `title: "🏠 <project name>"`, both without `threadId` so they act on this thread. Set `pinnedAndRenamed` in the locator file to `true` when both calls succeeded, and to `false` when either failed or the user declined; if a call failed, say so. Reply with the page URL in one line and tell the user to run `/home-thread-update` to fill it: in this thread when `home.threadId` is this thread, otherwise in the home thread.

   Done when the pin and rename succeeded or were declined, the locator file's `pinnedAndRenamed` matches what happened, and the reply holds the URL.

## Locator file

```json
{
  "project": "mm-ai-email-classification",
  "page": { "url": "https://claude.ai/artifact/<slug>", "slug": "<slug>" },
  "home": { "environmentId": "<environmentId>", "threadId": "<threadId>" },
  "machines": [
    { "environmentId": "<environmentId>", "label": "hiphopopotamus", "projectId": "<projectId on that machine>" },
    { "environmentId": "<environmentId>", "label": "rhymenocerous", "projectId": "<projectId on that machine>" }
  ],
  "pinnedAndRenamed": true
}
```

`project` is the project name. `page` is the status page. `home` is the home thread and the machine it runs on. `machines` has one entry per machine: `label` is its hostname and `projectId` is this project's id on that machine.
