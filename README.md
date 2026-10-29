# Agent Skills

My personal AI agent skills for Claude Code, Codex, and other tools that support the open Agent Skills format.

This repository is the source of truth for the skills I use across projects and machines. Each skill is focused on one job, stored as ordinary files, and designed to be easy to inspect, customize, and change.

## Available skills

### [`as-me`](./as-me)

Drafts and revises anything I will send as myself, including messages in any app, email, pull request text, commit text, and documents I sign.

- Codex invocation, `$as-me`
- Claude Code invocation, `/as-me`
- Automatic invocation, ask the agent to write something in my voice

The skill includes a platform-neutral messaging register built from my edits of earlier drafts. It loads that reference only for messaging apps and other casual colleague messages. The wording is anonymous and portable, but the configured voice and outcomes remain mine.

### [`have-opus-do-it`](./have-opus-do-it)

Does an agreed task, whether a code change, research, drafting, file wrangling, or analysis, through subagents running the newest Opus model, directed, reviewed, and supervised by whichever model is running my session, so that model's token allowance goes to judgment instead of keystrokes. I do the analysis and planning in normal conversation, then invoke this at the doing moment. Subagents produce the deliverable, the session model reviews it, and irreversible actions such as committing, sending, or publishing stay with me.

If the task is not yet agreed, the skill makes the session model confirm its scope before any subagent runs. The skill never names an Opus version. It dispatches through the harness's `opus` alias, which resolves to the current Opus, so a new Opus release needs no edit here. In a harness that delegates across vendors, such as T3 Code, a non-Claude session model routes the dispatch to Claude's newest Opus.

- Codex invocation, `$have-opus-do-it`
- Claude Code invocation, `/have-opus-do-it`
- Automatic invocation, ask the agent to have Opus do, build, or handle something

### [`have-sol-do-it`](./have-sol-do-it)

The same workflow as `have-opus-do-it`, with OpenAI's newest Sol model as the worker instead of Opus. The two files are kept as separate copies on purpose, so each can carry instructions for the quirks of its own model family. Sol is reachable from Codex and from T3 Code's cross-vendor delegation; plain Claude Code has no route to it, and the skill says so rather than substituting a model.

- Codex invocation, `$have-sol-do-it`
- Claude Code invocation, `/have-sol-do-it`
- Automatic invocation, ask the agent to have Sol do, build, or handle something

### [`wait-what`](./wait-what)

Stops an agent that has lost me and makes it re-pitch, with a little context, in ASD-STE100 Simplified Technical English, using the project's own vocabulary.

- Codex invocation, `$wait-what`
- Claude Code invocation, `/wait-what`
- Automatic invocation, never. This one is mine to trigger.

Adapted from the `wait-what` skill in [mattpocock/skills](https://github.com/mattpocock/skills). His version sources the ubiquitous language from a `CONTEXT.md` file. Mine sources it from the conversation first, then `docs/` if that directory exists, then the code, so it works in a repo that keeps no `CONTEXT.md`.

## Install

Clone this repository and the companion [`dotfiles`](https://github.com/LunarRed/dotfiles) repository side by side:

```bash
mkdir -p "$HOME/GitHub/LunarRed"
git clone https://github.com/LunarRed/skills.git "$HOME/GitHub/LunarRed/skills"
git clone https://github.com/LunarRed/dotfiles.git "$HOME/GitHub/LunarRed/dotfiles"
```

The symlinks are not made by hand. Each skill has two entries in `dotfiles/mise.toml`, one for `~/.agents/skills/<skill>` and one for `~/.claude/skills/<skill>`, and `mise bootstrap dotfiles apply` creates them. The `sync-agents` script in `dotfiles/bin` pulls both repositories with `git pull --ff-only`, refreshes approved third-party skills, and runs that apply step. It refuses to run while either checkout has uncommitted changes.

```bash
"$HOME/GitHub/LunarRed/dotfiles/bin/sync-agents"
```

Codex discovers a skill through `~/.agents/skills`. Claude Code discovers it through `~/.claude/skills`. Both link to the same files in this repository, so every edit has one source of truth.

Start a new agent session if a newly linked skill does not appear immediately.

## Repository structure

```text
skills/
├── README.md
├── as-me/
│   ├── SKILL.md
│   ├── agents/
│   │   └── openai.yaml
│   └── references/
│       └── messaging-register.md
├── have-opus-do-it/
│   └── SKILL.md
├── have-sol-do-it/
│   └── SKILL.md
├── wait-what/
│   ├── SKILL.md
│   └── agents/
│       └── openai.yaml
└── .gitignore
```

Every skill has a `SKILL.md` entry point with its name, triggering description, and instructions. A skill can also include references, scripts, assets, and agent-specific metadata when they are useful.

## Customize `as-me`

Edit [`as-me/SKILL.md`](./as-me/SKILL.md) to change universal writing rules. Edit [`as-me/references/messaging-register.md`](./as-me/references/messaging-register.md) to preserve personal wording, structure, and register patterns learned from real message edits.

Keep messaging-app formatting out of the register unless it came from a direct instruction or screenshot. Copied messages can lose or transform whitespace, markup, quotes, and inline code.

## Customize `have-opus-do-it` and `have-sol-do-it`

Edit [`have-opus-do-it/SKILL.md`](./have-opus-do-it/SKILL.md) or [`have-sol-do-it/SKILL.md`](./have-sol-do-it/SKILL.md) to change the dispatch rules or how much rework the session model may apply directly during review. The "Choose the worker" section is where each file names its model family and carries that family's quirks. The rest of the two files is meant to stay identical, so a change to the shared workflow goes in both.

Both skills are model-invocable, so I can ask for them in a plain sentence. Their descriptions tell the agent to use them only when I name Opus or Sol and never to infer them from the task, because they spawn paid subagents. Keep that guard in the description if you edit it.

## Customize `wait-what`

Edit [`wait-what/SKILL.md`](./wait-what/SKILL.md) to change which ASD-STE100 rules the agent applies, or to change where it looks for the project's vocabulary. The sourcing list is deliberately ordered, so a repo with its own glossary path only needs that one line changed.

Keep `disable-model-invocation: true` in the frontmatter. The skill body is written as something I say, so an agent should never fire it at itself.

## Update

Run `sync-agents` on each machine. On my Mac I run it by hand; on the headless server a systemd timer runs it daily. It pulls `main` of both repositories, so a change reaches a machine only after it is merged and pushed.

Removing a skill needs one manual step. `mise` only creates and repairs the links it is configured for; it never deletes one. After a skill's entries leave `mise.toml`, remove its two stale links by hand on each machine:

```bash
rm "$HOME/.agents/skills/<skill>" "$HOME/.claude/skills/<skill>"
```

## Add another skill

1. Create a new top-level directory named after the skill.
2. Add and validate its `SKILL.md` file.
3. Add supporting references, scripts, assets, or metadata only when the skill needs them.
4. Add the skill to the catalog in this README.
5. Add its two symlink entries to `mise.toml` in the `dotfiles` repository, merge this repository first, then `dotfiles`, and run `sync-agents`.
