---
name: wait-what
description: Stop. That last message did not land, re-pitch it.
disable-model-invocation: true
---

Wait, I don't understand where you've got to here. Re-pitch it with a little context, in ASD-STE100 Simplified Technical English, using the ubiquitous language of this project.

## Give me the context I lost

Say where we are and how we got here before you say anything new. Name the thing you were working on, the problem or decision in front of you, and why this step follows the last one. Assume I lost the thread, not that I forgot the project.

Two or three sentences. This is a re-pitch, not a status report.

## Write in Simplified Technical English

Apply the ASD-STE100 rules that matter most:

1. One idea per sentence. 20 words maximum for an instruction, 25 for a description.
2. Active voice. Name the actor, i.e. "the handler reads the token" rather than "the token is read".
3. One word, one meaning. Pick a term and repeat it. Never reach for a synonym to vary the prose.
4. Simple tenses. Write "we ran the migration" rather than "we have been running the migration".
5. No noun cluster longer than three words. Break up "user session token cache" with a preposition.
6. Keep the articles and keep the sentence whole. To get under the word limit, split the sentence, do not drop words.

## Keep the project's own words

ASD-STE100 approves domain terms as Technical Names and Technical Verbs, so our vocabulary survives the simplification and only the sentences around it get simpler. Never paraphrase a project term into plainer English, that defeats the point of the re-pitch.

Source the vocabulary in this order:

1. The terms we have already used in this conversation.
2. The terms in `docs/`, if that directory exists. A glossary, a domain doc, or an ADR all count.
3. The terms in the code, i.e. type, module, and function names.

If `docs/` does not exist, proceed silently. Do not tell me it is missing and do not offer to create it.
