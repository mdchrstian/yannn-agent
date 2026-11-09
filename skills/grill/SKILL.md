---
name: grill
description: Relentlessly interview the user about a plan or design until every decision is settled, in rounds of numbered questions with a recommended answer each. Run with /yannn-agent:grill to stress-test a plan before building.
argument-hint: "[specs/plan/<slug>.md | plan in words]"
disable-model-invocation: true
---

Adapted from Matt Pocock's grilling skill (MIT, github.com/mattpocock/skills) and
andrej-karpathy-skills "Think before coding" (MIT). Talk in ${user_config.language}.

Plan: $ARGUMENTS (a file path: read it; words: use them; empty: ask for the plan in one line)

Interview the user until you reach a shared understanding. Map the plan as a **design
tree**: every decision branches into the decisions that hang off it.

Work in **rounds**. The **frontier** is every decision whose prerequisites are settled.
Ask the whole frontier in one round, numbered, each with your recommended answer, then
wait. Word each question so "yes" accepts the recommendation:

```
Q1 <title>: <question, with choices if any>
-> <recommended answer, and the one-line reason>
```

- Max 5 questions per round. More on the frontier: ask the 5 that unblock the most.
- Each answer reshapes the tree. Recompute the frontier; a question that depends on one
  still open this round belongs to a later round.
- Facts are your job, decisions are theirs. A question that needs a fact from the code:
  spawn `yannn-agent:scout` for it instead of asking; ask the rest of the frontier now.
- Push back. A choice that adds work without serving the Goal: say so and recommend
  cutting it to "Not doing". A simpler path exists: say so.
- Grill the hard parts first: data that can be lost, money, permissions, who sees what,
  what happens on error, what "done" looks like for each must-have.

Done when the frontier is empty: every branch visited, nothing silently assumed. Then:
1. Write the settled decisions into the plan file (create `specs/plan/<slug>.md` if
   there was none): a `## Decisions` section, one line each `question -> answer`, and
   empty the `## Open` section. Keep the file under 60 lines.
2. Ask the user to confirm the file is right. Do not act on it before they confirm.
3. End with one line: next `/yannn-agent:plan specs/plan/<slug>.md`, after `/clear`.

No code, no tickets.
