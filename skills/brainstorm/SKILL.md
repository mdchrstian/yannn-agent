---
name: brainstorm
description: Shape a vague idea into a short design with 2 or 3 approaches and a clear "Not doing" list, saved to specs/plan/<slug>.md. Run with /yannn-agent:brainstorm when an idea has no clear plan yet.
argument-hint: "<idea>"
disable-model-invocation: true
---

Adapted from Superpowers brainstorming (MIT, github.com/obra/superpowers) and
andrej-karpathy-skills "Think before coding" (MIT). Talk in ${user_config.language}.

Idea: $ARGUMENTS

1. Look before you ask. In an existing project, spawn `yannn-agent:scout` once for:
   stack, what already exists that this idea touches or can reuse. Never ask the user
   what the code answers.
2. First message, max 150 words: the idea played back in your own words (what, for whom,
   why), what you found in the code, and the ONE open question whose answer changes what
   gets built the most. If they have not said why they want it, ask that.
3. Then one question per message, each with your recommended answer. State every
   assumption you make out loud; if two readings exist, name both. If a simpler approach
   exists than the one they describe, say so.
4. When the goal is clear, give 2 or 3 approaches, one line each with what it costs.
   Recommend the smallest that meets the goal. They pick.
5. Write `specs/plan/<slug>.md`, max 40 lines:
   ```
   # <Idea>
   ## Goal        1 to 3 sentences, in the user's words
   ## Users       who, and the one thing they must be able to do
   ## Approach    the chosen one, and one line on why not the others
   ## Must have   bullets
   ## Not doing   bullets, each with a one-line reason
   ## Open        questions still unsettled (grill will settle them)
   ```
6. End with one line: next `/yannn-agent:grill specs/plan/<slug>.md` if "Open" is not
   empty, otherwise `/yannn-agent:plan specs/plan/<slug>.md`. Suggest `/clear` first;
   everything needed is in the file.

Stop as soon as the file is written. No code, no tickets.
