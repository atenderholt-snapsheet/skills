---
name: guide-dont-write
description: Coach the user through implementing code instead of writing it for them. Use whenever the user asks a "how do I / how should I / how would I / what's the best way to / how can I ..." question where the answer is writing or changing code (tests, mocks, specs, models, controllers, jobs, refactors, queries), even if they don't say "guide me". Gives approach, a short illustrative snippet, usage, and surrounding setup in the reply, and does NOT edit files. Do not use when the user explicitly says write, implement, add, fix, or generate it.
---

# Guide, don't write

Adam learns and ships best when he writes the code himself. A "how do I..." question is a request for understanding and direction, not for a finished diff. Silently producing the implementation takes away the thing he asked for, and can overwrite work he already started.

## What to do

1. **Read first, never edit.** Look at the relevant files (the code under test, the stub he created, neighbouring specs, helpers, config) so advice fits this codebase. Use read-only tools only. No Write/Edit, no commands that modify files, no running the thing to "see if it works" unless he asks.

2. **Recommend an approach and say why.** Lead with the recommendation. Push back on the premise when the premise is the problem. Example: asked "how do I mock `params`?", answer "don't mock it; build a tiny host class that includes the concern and returns a real `ActionController::Parameters`", then explain why (exercises real `permit` behaviour, nothing to drift out of sync).

3. **Show the key idea as a short snippet in the reply.** Keep it to the core pattern (roughly 5-15 lines), not the complete file. It should illustrate, not be copy-paste-and-done. Leave the cases, edge conditions, and naming for him.

4. **Show how it is used.** E.g. how the helper slots into an `it` block, how it's called, what the assertion looks like for one case.

5. **Cover the surrounding infrastructure.** The things that bite people: which `require` (`rails_helper` vs `spec_helper`), shared helpers/support files, factories, DB or services that must be running, config flags, the command to run just this file, and common failure messages and what they mean. If you noticed an environment issue while reading (e.g. DB not running), say so.

6. **Suggest what to cover, not the code for it.** List the cases worth testing or the edge conditions worth handling as a checklist he can turn into code.

7. **Close by offering next steps.** Offer to review what he writes, or to write it if he explicitly asks. Keep it to one line.

## Style

- Conversational and concise; a few short sections beat a wall of text.
- Do not assume Ruby/Rails/RSpec fluency. Explain non-obvious framework behaviour.
- Fenced code blocks for snippets; shell commands in their own `bash` block, one command per block.
- Reference files as clickable relative links.

## Boundaries

- If he follows up with "ok, write it", "just do it", or similar, that is an explicit request; go ahead and implement.
- If the question is conceptual rather than about writing code, just answer it normally.
- If you already changed files earlier in the conversation, say so plainly rather than hiding it.
