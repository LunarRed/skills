---
name: as-me
description: Draft or revise text in the configured personal voice for anything the user will send as themselves, including messages in any app, email, pull request text, commit text, and documents they sign. Use when the user explicitly asks for writing "in my voice," asks to apply their voice, or invokes $as-me. Do not apply this voice automatically to ordinary assistant responses.
---

# Write As Me

Write text that the user can send as their own. Preserve the intended meaning, facts, audience, and level of formality while applying every rule below.

## Core rules

Treat these as hard requirements and scan every draft against them before delivering it.

1. Never use an em dash. Replace it with a comma or a period.
2. Use colons rarely, usually only to introduce a list. Never use a colon to join two clauses or set up an explanation.
3. When avoiding a colon, rewrite the sentence as a natural first-person sentence rather than a comma-join or a clipped fragment. Write "I would first check if `.env.stage` exists," not "First thing to check, does `.env.stage` exist" and not "First thing to check. Does `.env.stage` exist." Write "I think you should know X," not "One thing worth knowing, X."
4. Use parentheses only for a true aside, meaning an important thought that is irrelevant to the current sentence. Never write `i.e.` or `e.g.` outside parentheses (i.e. like this). For clarification or examples, prefer rewording the sentence so no abbreviation is needed; if an abbreviation is genuinely needed, it goes inside parentheses.
5. Write development environment names in all caps in every context. Always use `PROD`, `STAGE`, and `DEV`, including phrases such as "the PROD database."

## Workflow

1. Identify what the user will send, who will receive it, and what the draft needs to accomplish.
2. For any messaging app or casual message to a colleague, read [references/messaging-register.md](references/messaging-register.md) completely before drafting. It contains the configured register patterns distilled from real edits of earlier drafts.
3. Draft in the appropriate register. Do not invent facts, commitments, opinions, or technical confidence that the user did not provide.
4. Run the final scan below and silently fix every issue.
5. Return clean, sendable copy. Keep explanations outside the draft unless the user asks for them.

## Final scan

- Search for every em dash character and remove it.
- Inspect every colon. Keep it only if it is rare and genuinely introduces a list.
- Inspect every introductory phrase joined by a comma or clipped into its own sentence. Rewrite it as a natural first-person sentence, e.g. "I would first check if X" or "I think you should know X."
- Inspect every parenthetical. Keep it only if the content is a true aside. Search for `i.e.` and `e.g.` anywhere outside parentheses and reword them away.
- Search case-insensitively for `prod`, `stage`, and `dev`, then capitalize every occurrence that names an environment.
- For messaging apps or casual colleague messages, verify the draft against the bundled register reference.

## Precedence

Apply the core rules above over any older example or observation in the messaging register. Match the register without copying a punctuation pattern that violates a core rule. Treat the user's direct instructions and corrections as the strongest evidence of their voice.
