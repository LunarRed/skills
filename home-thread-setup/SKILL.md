---
name: home-thread-setup
description: Create a project's status page and make this T3 Code thread its home thread. Use when the user asks to set up a home thread or create a status page for this project.
disable-model-invocation: false
---

# Home Thread Setup

The **home thread** is the one T3 Code thread, running Claude, where the user runs `/home-thread-setup` and `/home-thread-update` for a project. Setup publishes the project's **status page**, a private claude.ai artifact page, and records where everything lives in the **locator file**, `.home-thread.json` in the project folder. Setup ends by running `/home-thread-update` once, so the page is already filled when the user opens it; later refreshes are `/home-thread-update` on its own.

A **machine** is one of the user's computers running T3 Code. T3 calls it an environment and gives it an `environmentId`. `$T3CODE_THREAD_ID` is this thread. `<project folder>` is the project's `workspaceRoot` from `t3_project_read`, using the `projectId` that `t3_thread_read` reports for this thread; the shell does not set a variable for it, and a worktree thread's working directory is not the project folder.

## Steps

1. **Confirm the environment.** This skill needs Claude Code's `Artifact` and `ArtifactData` tools and the T3 Code tool `t3_environment_links`. A tool listed as deferred counts; load it with ToolSearch. If any is missing, reply in one line that this thread lacks the Artifact tools (or the T3 Code tools) so setup cannot run here, and stop. Suggest no fix; the tools come with the Claude thread or they do not.

   Done when all three tools are present.

2. **Check for an existing status page.** Read `<project folder>/.home-thread.json`.
   - No file: continue to step 3.
   - The file exists and `Artifact` with `action: "read"` on its `page.url` succeeds: this project already has a status page. Report `page.url`. In one message, ask whether to make this thread the home thread (only when `home.threadId` differs from `$T3CODE_THREAD_ID`) and whether to pin this thread and rename it to `🏠 <project name>`, where the project name is the file's `project`. On a yes to the first, set `home` in the locator file to this machine and this thread. Pin and rename apply only when this thread is the home thread after that answer. Then go to step 7.
   - The file exists but the read answers that the page is not found: tell the user the page is gone, and continue to step 3 as a fresh setup.
   - The file exists but the read fails for any other reason, such as an expired login or a service error: reply in one line that the status page could not be checked, and stop without publishing anything.

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
   - Copy `<skill base directory>/assets/status-page.html` to `status-page.html` in your scratchpad directory. `<skill base directory>` is the folder Claude Code names when it loads this skill. The Artifact tool publishes only from the working directory or the scratchpad, and the scratchpad keeps the copy out of the project.
   - In the copy, change `<title>Status page</title>` to `<title>Status page: <project name></title>`. The page's `<title>` names the artifact; the tool's `title` input does not.
   - Load the `artifact-capabilities` skill, then publish with `Artifact`: `file_path` set to the copy, `icon: "list"`, and `capabilities: { db: {} }`. Keep the `url` from the result.

   Done when the publish result shows the `url` and `capabilities db`.

6. **Write the locator file and check the page.**
   - Write `<project folder>/.home-thread.json` in the shape under "Locator file" below. `home` is this machine and `$T3CODE_THREAD_ID`. `machines` lists every machine, this one included, with the `projectId` the user confirmed, and leaves `projectId` out for a machine with none. `pinnedAndRenamed` is the user's answer from step 4.
   - Keep the file out of git without changing any `.gitignore`. Get the exclude file's path with `git -C "<project folder>" rev-parse --path-format=absolute --git-path info/exclude` (this also resolves correctly in a git worktree), and append the line `.home-thread.json` unless the file already has it.
   - `ArtifactData` with `action: "list"`, `url` set to the `url` from step 5, and `collection: "status"`, and confirm it answers with no documents. If a browser tool is available, open the URL and confirm the page reads "No snapshot yet. Run /home-thread-update in the home thread."

   Done when the locator file parses as JSON, the exclude file contains `.home-thread.json` exactly once, and the `ArtifactData` list answers.

7. **Pin, rename, and fill the page.** If the user said yes to pinning and renaming, call `t3_thread_organize` with `action: "pin"` and `t3_thread_update` with `action: "rename"` and `title: "🏠 <project name>"`, both without `threadId` so they act on this thread. Set `pinnedAndRenamed` in the locator file to `true` when both calls succeeded, and to `false` when either failed or the user declined; if a call failed, say so. Then, when `home.threadId` is this thread, run `/home-thread-update` here right away so the page shows real data; its one-line reply, which carries the page URL, is the hand-off. When the home thread is elsewhere, reply with the page URL and tell the user to run `/home-thread-update` in the home thread.

   Done when the pin and rename succeeded or were declined, the locator file's `pinnedAndRenamed` matches what happened, and either the first refresh has written a snapshot or the reply names the home thread to run it in.

## Locator file

```json
{
  "project": "mm-ai-email-classification",
  "page": { "url": "https://claude.ai/artifact/<id>" },
  "home": { "environmentId": "<environmentId>", "threadId": "<threadId>" },
  "machines": [
    { "environmentId": "<environmentId>", "label": "hiphopopotamus", "projectId": "<projectId on that machine>" },
    { "environmentId": "<environmentId>", "label": "rhymenocerous", "projectId": "<projectId on that machine>" }
  ],
  "pinnedAndRenamed": true
}
```

`project` is the project name. `page` is the status page. `home` is the home thread and the machine it runs on. `machines` has one entry per machine: `label` is its hostname and `projectId` is this project's id on that machine.
