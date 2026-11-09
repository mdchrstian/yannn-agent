---
name: plan
description: Turn a plan file, idea or PRD into small self-contained tickets (spec.md + eval.md) and a board for the yannn-agent router. Run with /yannn-agent:plan.
argument-hint: "[specs/plan/<slug>.md | idea | path/to/PRD.md]"
disable-model-invocation: true
---

# Goal
Every ticket must be buildable by a Sonnet worker that has never seen this conversation,
reading only its own folder. Short and exact beats long and vague: workers pay tokens for
every line they read. Small scope is the biggest saving of all.

Input: $ARGUMENTS
Templates: `${CLAUDE_PLUGIN_ROOT}/skills/plan/templates/`.
Talk to the user in ${user_config.language}. Write tickets in English (cheaper, and
workers follow it more exactly), except `Goal`, which stays in the user's language.

# 1. Intake
- A `specs/plan/*.md` file from brainstorm or grill: its Goal, Must have, Not doing and
  Decisions are settled. Do not re-ask them. Only ask what splitting into tickets needs.
- Anything else: map the project with `yannn-agent:scout`, never by reading files
  yourself (stack, test command, dev server command, what can be reused, files touched).
  If `ya-graph status` says fresh, tell scout to use the graph. Play the request back in
  max 5 lines and ask only questions whose answer changes what gets built: up to 4 per
  AskUserQuestion call, multiple choice, simplest option first and marked (Recommended).
  Many open questions: suggest `/yannn-agent:grill` first instead.

# 2. Cut scope (Ponytail)
For each requirement ask: does it need to exist in this batch? Already in the codebase?
A platform feature? Move everything not essential to "Not doing" with one line of reason.

# 3. Split
One ticket = one mergeable feature, ideally under ~300 changed lines. Bigger: split it.
Mode `solo` by default. `team` only when one feature has 2+ parts buildable in parallel
against a fixed interface (for example API + UI); team costs more tokens.

Board `UI` column:
- `no`       no visible change
- `yes`      app screens, forms, dashboards (worker builds it, following DESIGN.md if any)
- `landing`  landing, marketing, pricing or company pages (designer builds it with
             taste-skill). Keep the `taste` row in eval.md.

Set `tdd: yes` only for logic where a wrong answer is costly: money, permissions,
parsing, dates, data migration.

# 4. Write (goal-driven: every requirement gets a check that proves it)
For each ticket create `specs/tickets/<NN>-<slug>/`:
- `spec.md` from `templates/spec.md` (hard limit 80 lines)
- `eval.md` from `templates/eval.md` (hard limit 40 lines, every check runnable)
- `contract.md` from `templates/contract.md` only when Mode = team
Create or update `specs/tickets/board.md` from `templates/board.md`. New tickets start as
`draft`. Runs: `alongside` or `after:<IDs> (<reason>)`.

spec.md: "Touch" lists exact paths. "Do not touch" lists what must stay as is. "Reuse"
lists existing code to build on. "Decisions" copies every settled choice, so the worker
never re-asks. No essays, no restating the PRD.

# 5. Hand over
Run `ya-board show`. Ask with AskUserQuestion (multiSelect) which tickets to build now
and run `ya-board set <ID> ready` for each one picked. (`ya-board` is on Claude's PATH
only, not in the user's own terminal.) Then `ya-board commit`, and tell the user in 2
lines: start the router in a new terminal with
`claude --agent yannn-agent:router --permission-mode auto` and type "kerjakan tiket yang ready".
