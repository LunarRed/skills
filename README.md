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

### [`wait-what`](./wait-what)

Stops an agent that has lost me and makes it re-pitch, with a little context, in ASD-STE100 Simplified Technical English, using the project's own vocabulary.

- Codex invocation, `$wait-what`
- Claude Code invocation, `/wait-what`
- Automatic invocation, never. This one is mine to trigger.

Adapted from the `wait-what` skill in [mattpocock/skills](https://github.com/mattpocock/skills). His version sources the ubiquitous language from a `CONTEXT.md` file. Mine sources it from the conversation first, then `docs/` if that directory exists, then the code, so it works in a repo that keeps no `CONTEXT.md`.

## Install

Clone the repository into a stable location:

```bash
mkdir -p "$HOME/GitHub/LunarRed"
git clone https://github.com/LunarRed/skills.git "$HOME/GitHub/LunarRed/skills"
```

Create the personal skill directories used by Codex and Claude Code:

```bash
mkdir -p "$HOME/.agents/skills" "$HOME/.claude/skills"
```

Link each skill into the shared Agent Skills directory, then link Claude Code to that shared entry:

```bash
for skill in as-me wait-what; do
  ln -s "$HOME/GitHub/LunarRed/skills/$skill" "$HOME/.agents/skills/$skill"
  ln -s "$HOME/.agents/skills/$skill" "$HOME/.claude/skills/$skill"
done
```

Codex discovers the skill through `~/.agents/skills`. Claude Code discovers it through `~/.claude/skills`. Both use the same files in this repository, so every edit has one source of truth.

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

## Customize `wait-what`

Edit [`wait-what/SKILL.md`](./wait-what/SKILL.md) to change which ASD-STE100 rules the agent applies, or to change where it looks for the project's vocabulary. The sourcing list is deliberately ordered, so a repo with its own glossary path only needs that one line changed.

Keep `disable-model-invocation: true` in the frontmatter. The skill body is written as something I say, so an agent should never fire it at itself.

## Update

Pull the latest changes into the canonical checkout:

```bash
git -C "$HOME/GitHub/LunarRed/skills" pull
```

The symlinks continue pointing at the updated files.

## Add another skill

1. Create a new top-level directory named after the skill.
2. Add and validate its `SKILL.md` file.
3. Add supporting references, scripts, assets, or metadata only when the skill needs them.
4. Link the skill into `~/.agents/skills` and `~/.claude/skills`.
5. Add the skill to the catalog in this README.
