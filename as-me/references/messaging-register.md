# Personal messaging register

Use these patterns for casual business messages sent through any messaging app. They define the configured voice and are distilled from real edits of earlier drafts.

Formatting copied from a messaging app is not reliable because apps can alter whitespace, quotes, markup, and inline code. Learn word choice, sentence shape, and structure from pasted examples. Learn formatting only from the user's direct instructions or a screenshot.

The core punctuation rules in `SKILL.md` take precedence over every observation below.

## Register and structure

1. Open with a conversational orienting line such as "Okay, here's what I figured out" only when the user is reporting their own multi-part findings. Plain status replies get no opener and start at the verdict, e.g. "PR #230 is good, and I merged it."

2. End a topic with the user's next action and what the recipient should expect, e.g. "Let me know when it's rebased, and I'll merge it." Attach the closer to the end of its topic's paragraph rather than automatically putting it at the end of the whole message. When a thread is fully wrapping, a short upbeat close with a real emoji can fit, e.g. "Everything else looks good! 😎" When the message puts a proposal to the recipient, close with a binary that offers both outcomes, e.g. "Does that work or am I missing anything?" rather than only inviting correction with "Anything I'm missing there?"

3. Use first-person active sentences when the user owns an action, e.g. "I restored," "I decided the best fix was," "I'm using X to do Y," "I also handled," and "I added." Avoid subjectless fragments such as "Pushed on a branch" and tool-as-agent phrasing such as "git does the matching now."

4. Frame decisions as choices, e.g. "I decided the best fix was to delete the parser instead of rewriting it," not "The fix was deleting, not rewriting."

5. Keep system behavior that the user did not personally cause in third-person declarative language. Casual wording can fit, e.g. "`node_modules` never gets copied anymore" or "the stale snapshot thing is dead."

6. For instructions to a colleague, give a literal click path and exact text or commands. Use quoted UI labels. Because parentheses are reserved for true asides, integrate navigation details directly into the sentence with commas or `i.e.` where clarification is needed.

7. Format almost everything code-shaped as inline code, including commands, flags, file names, patterns, error names, version numbers, and branch names. Examples include `node_modules`, `npm ci`, `eslint`, `22.x`, and `25.8.1`. A classifier noun often helps, e.g. "the `node_modules` directory." Named systems and UI labels can take quotation marks. Quotation marks also go around a term borrowed from the recipient's own document and around a term the message is defining, e.g. the "stand-down" rule, what counts as a "human touching the case." Use real emoji characters rather than shortcodes.

8. Cut verification evidence, scratch work, instructions the recipient already followed, and open items that are unrelated to the current thread's action. Prefer what a technical detail means for the recipient over the underlying implementation history. Instructions usually do not need an explanation unless the recipient is likely to push back without it.

9. Avoid punchy tag fragments. Write a complete sentence about the action, e.g. "I added `!.env` to fix that," rather than "Added `!.env`, hole closed."

10. Attribute agent analysis when the user relays it. Do not turn an agent's findings into the user's own discoveries. A pattern such as "Also, a correction from my coding agent, ..." keeps the attribution clear while following the current punctuation rules.

11. Prefer neutral wording, e.g. "your audit issues" rather than "your audit gripe."

12. Lead with the circumstance as a full clause, e.g. "While I was testing, I caught X." Do not open with a comma-led noun fragment such as "One thing I caught while testing, X," which the user has explicitly rejected as not their voice. Short cause-and-effect sentences fit.

13. Soften verdicts. Prefer "Yeah, that all looks normal" over "that's normal." Words such as "just" and "all" can help when natural. Orient to the recipient's world with second person, e.g. "Your old setup," rather than an abstraction such as "The old flow."

14. Delegate colleague action to their agent when appropriate, e.g. "The one thing you should have your coding agent fix is ..." or "Your coding agent can take care of this for you." Give the exact words to relay, e.g. "Tell your coding agent to `rebase it on main`." Use "Tell your agent to," "Tell your agent that," and "Have it" as natural verbs.

15. Write only words the user fully understands and would use themselves. Match the vocabulary level they have used in the conversation. If they have just asked what a technical term means, do not casually put that term in their mouth. If an unavoidable term needs explanation, explain it to the user outside the sendable draft.

16. This voice sometimes uses asterisks around a modal as an uncertainty hedge, e.g. "It *should* just be the Node version fix." Use this sparingly and only when the claim is genuinely a best guess.

17. Do not write blame lines or validation lines. Avoid phrases such as "that's my fault," "that's the right instinct," and "good call." State the cause neutrally and move to the fix. Keep positivity about the state of things, e.g. "Everything else looks good," rather than praising the person.

18. Shape short replies as one paragraph with no line breaks. Put approval first as its own short sentence, e.g. "Yeah, go ahead." Follow with complete imperatives, e.g. "Push the branch, and open the PR." Use "Also," to connect the next topic. Put the handoff on the recipient, e.g. "Let me know when your PR is up, and I'll review and merge it." The same one-paragraph shape applies to the closing section of a longer message. Merge trailing short paragraphs into one, joining with "So" to carry on the current topic and "Also," to start the next.

19. Write pull request references as plain text in the form "PR #230," not as inline code. Fuse a verdict and action when natural, e.g. "PR #230 is good, and I merged it."

20. Expand compressed idioms into plain, concrete words, e.g. "and that will fix that issue" rather than "and that clears." Skip enumerator preambles such as "Two more things" when "Also," can start the next topic. Give a memorable concrete example inline with `e.g.` when an abstract category needs one. Use "since" for causal joins. Add a qualifier such as "unrelated" when the recipient might misread the scope.

21. Frame a request as work in progress rather than a stated requirement. Describe what the user is doing, e.g. "I'm trying to narrow the list of what counts as X" rather than "I need the specific list of what counts as X," and "I'm thinking the rule should be" rather than "I'm reading it as." This applies even when the user genuinely does need the answer.

22. State a risk as a purpose clause rather than a consequence clause. Write "so a case manager adding a note doesn't silently stop billing a client" rather than "otherwise a case manager adding a note could silently stop billing that client for good." Drop intensifiers while doing it, e.g. "a client" rather than "that client for good." The point is what the rule is protecting, not how bad the failure would be.

23. Introduce a list with a word plus a colon rather than a bare colon, e.g. "anything a person does that affects payment like:" rather than ending the lead-in at "payment:."

24. Keep a phrasal verb intact and attach the object after it, e.g. "switch off AutoPay for the case" rather than "switch AutoPay off on that case."

25. Never use the "that X is on me" construction in any variant, e.g. "that's on me" or "the rethink is on me." Rejected emphatically. When the user asks for self-deprecation, write plain statements such as "I fucked up" or "I never properly thought it through," never ownership idioms.

26. A comma-join is only an introductory phrase before its subject, e.g. "So for tonight, ...". Never splice two independent sentences with a comma; "Your code is fine, it does what the ticket asks" is wrong, use "and" or a period. When the introductory phrase is a noun fragment, rewrite it as a natural first-person sentence instead, per rule 12 and core rule 3.

27. Verdicts read like speech, not like a review checklist. "The code itself is totally fine" fits; spec-conformance phrasing such as "does what the ticket asks" or "matches the requirements" does not ("no human talks like that").

28. When reporting on work the user reviewed or verified, never narrate the process or grade the result. No lists of what was traced or checked, no verdict openers such as "the logic checks out," and no praise constructions such as "a real fix" or any "real X." Say everything looks good except the specific issues found, then give only the issues.

29. When the detail lives somewhere the recipient can see, point to it with a short parenthetical, e.g. "(see my comments)," rather than describing its location in prose. This pointer is an allowed exception to the parentheses rule.

30. Keep counts honest and minimal. Name exactly what is outstanding and join small items with "and," e.g. "one failing test and a doc tweak," never inflate with vague quantifiers such as "a couple of" or "plus some."

31. Use first person singular for the behavior of code the user owns, even in shared code, e.g. "I never return that field externally," not "we never return."

32. Name the actual identifier in inline code instead of an English paraphrase of it. Write the real field, function, or ticket name; do not translate it into a description the recipient has to map back.

33. Lead causal reasoning with "Since," putting the reason before the decision, e.g. "Since that predates your branch, I'll fix it separately," rather than stating the fact and chaining the consequence with "and."

34. Release the recipient from an item with a full second-person sentence, e.g. "so you don't need to do anything for this," never a clipped fragment such as "so no action needed there."

35. Correct a stale reference or assumption with "actually," stating what is true now, e.g. "the new entry is actually DEC-311," rather than narrating how the change happened.

## Personalization

Treat direct corrections as authoritative. Apply demonstrated wording and sentence patterns across messaging apps, but do not infer formatting choices from copied messages. Update this register only when the user asks to preserve new patterns. Generalize identity and platform references without weakening or replacing the configured voice.
