# Yannn Agent

A Claude Code plugin for a think-then-build workflow that spends as few tokens as
possible.

You think in your own session: **brainstorm** a vague idea, **grill** the plan until
nothing is left assumed, and **plan** it into small tickets. Then a router on Opus hands
each ticket to a Sonnet worker in its own git worktree, Haiku searches the code and runs
the tests, and shell scripts do the mechanical parts for zero tokens. Ponytail keeps the
code small, which is the biggest token saving of all.

## The flow

```
/yannn-agent:brainstorm <idea>      vague idea  ->  specs/plan/<slug>.md   (approaches, Not doing)
/clear
/yannn-agent:grill specs/plan/x.md  stress-test ->  same file + Decisions  (rounds of questions)
/clear
/yannn-agent:plan specs/plan/x.md   plan        ->  specs/tickets/*        (spec + eval + board)

claude --agent yannn-agent:router --permission-mode auto
> kerjakan tiket yang ready         build       ->  merged branches, one line per ticket
```

Every step writes to a file, so you can `/clear` between steps and start the next one
with an empty context. Skip what you don't need: a clear request can go straight to
`/yannn-agent:plan`.

## What's inside

| Part | Runs on | Job |
|---|---|---|
| `/yannn-agent:brainstorm` | your session | Turns an idea into a short design: goal, 2 or 3 approaches, Must have, **Not doing** |
| `/yannn-agent:grill` | your session | Interviews you in rounds of up to 5 numbered questions, each with a recommended answer, until every decision is settled |
| `/yannn-agent:plan` | your session | Cuts scope, splits into tickets of ~300 lines max, writes `spec.md` + `eval.md` + board |
| `/yannn-agent:setup` | no LLM | One-time project settings, and a check of companion tools |
| `yannn-agent:router` | Opus, medium | Talks to you, dispatches tickets, reads eval results, merges. Never reads code |
| `yannn-agent:worker` | Sonnet, medium | Builds one ticket in its own worktree |
| `yannn-agent:designer` | Sonnet, medium | Builds landing pages with taste-skill |
| `yannn-agent:reproducer` | Sonnet, medium | Reproduces a bug, finds the root cause, fixes it if small |
| `yannn-agent:scout` | Haiku, high | Code search: graph first, then `rg` |
| `yannn-agent:tester` | Haiku, high | Runs the checks in `eval.md`, reports only failures |
| `ya-board`, `ya-watchdog`, `ya-merge`, `ya-graph`, `ya-setup` | no LLM | Scripts in `bin/`, on Claude's PATH while the plugin is on |

## Install

```bash
claude plugin marketplace add mdchrstian/yannn-agent
claude plugin install yannn-agent@yannn-agent             # also installs ponytail
claude plugin install taste-skill@yannn-agent             # optional, for landing pages
```

Claude Code asks for your reply language when the plugin is enabled (default Bahasa
Indonesia). Change it any time in `/config`.

In each project (a git repo with at least one commit), once:
```
/yannn-agent:setup
```
A plugin cannot set env vars or permissions, so setup merges them into the project's
`.claude/settings.json` (your existing keys stay), adds `graphify-out/` and
`.taste-skill/` to `.gitignore`, and reports which companion tools are missing.

**Upgrading from the old copy-the-`.claude/`-folder version:** delete `router.md`,
`worker.md`, `tester.md`, `reproducer.md` and `Explore.md` from `.claude/agents/`, plus
`.claude/skills/ticket/` and `.claude/scripts/`. Your `specs/` folder keeps working.

## Thinking before building

**Brainstorm** (from Superpowers and Karpathy's "think before coding"): looks at your
code before asking anything, plays your idea back in your own words, asks one question
at a time with its recommendation, states its assumptions out loud, and says so when a
simpler approach exists. It ends with 2 or 3 approaches and you pick one.

**Grill** (from Matt Pocock's grilling skill): maps your plan as a tree of decisions and
asks every question that can be answered now in one round, each worded so "yes" accepts
its recommendation. Facts it looks up itself through scout; decisions are always yours.
It starts with the hard parts (lost data, money, permissions, errors, what "done" means)
and pushes back on anything that adds work without serving the goal. It's done when no
question is left open.

**Plan**: reads the plan file without re-asking what is settled, moves anything
non-essential to **Not doing**, and gives every requirement a runnable check in
`eval.md` (goal-driven, so workers know exactly when they are done).

## Simplicity: Ponytail, fully installed

[Ponytail](https://github.com/DietrichGebert/ponytail) is installed as a dependency.
Its own benchmark reports 53% less code and 45% fewer tokens. It adds about 900 tokens of
rules to each session (cached after the first turn), and setup tunes it for this
workflow:

- `PONYTAIL_SUBAGENT_MATCHER=worker|designer|reproducer`: rules go only to the agents
  that write code, not to scout and tester on Haiku.
- `PONYTAIL_MAP=0`: Ponytail's 2,000-character repo map is off, since scout and the
  graph already cover search.

You also get its commands: `/ponytail lite|full|ultra`, `/ponytail-review`,
`/ponytail-audit`, `/ponytail-debt`.

Workers add Karpathy's **surgical changes** rule: every changed line traces to the
ticket, no "improving" nearby code, and only their own leftovers are cleaned up.

## Code graph (graphify, optional)

[graphify](https://github.com/safishamsi/graphify) builds a graph of your code with
tree-sitter. No LLM is involved, so building it costs zero tokens. Scout asks the graph
first (`ya-graph q`, answers capped at 600 tokens), workers check what a change breaks
(`ya-graph affected`), and `ya-merge` refreshes the graph after every merge.

```bash
uv tool install graphifyy
```
Then ask Claude to run `ya-graph update`. Worth it from about 200 source files; on a
small repo `rg` is cheaper. Independent tests put the saving at about 5 to 10 times per
search, and more for reading code than writing it. Don't run `graphify claude install`:
its hooks would also nudge the tester and the router. The graph lives in the main
checkout, so every worktree shares it.

## Design (taste-skill, optional)

Tickets marked `UI=landing` go to the designer, which loads
[taste-skill](https://github.com/Leonxlnx/taste-skill). It needs Node 22+ and Chrome,
Edge or Chromium. On top of taste-skill, the designer follows fixed rules:

- A project `DESIGN.md` wins: its colors, fonts, spacing and voice stay fixed, and
  taste-skill only picks layout, sections, imagery and motion.
- No `DESIGN.md` yet: the designer writes one (max 30 lines) from what taste-skill chose,
  so later pages match.
- Banned: purple gradients, gradient text, emoji as icons, empty glass cards, "New"
  badges, lorem ipsum.
- Checked at 1440, 768 and 390 px wide. taste-skill's finish gate is a row in
  `eval.md` that the tester runs.

Normal app screens (`UI=yes`) go to the worker, following `DESIGN.md`. Use only one
design skill for landing pages, so their rules don't clash.

## How it saves tokens

1. You think in short steps, each written to a file, with `/clear` in between.
2. Grill asks a whole round of questions at once: fewer turns, each with a default you
   can accept with "yes".
3. Scope is cut before code exists: every ticket has a Not doing list.
4. The router never reads code. Opus sees only `ya-board` output and the eval table.
5. Board, watchdog, merge and graph are shell scripts: zero tokens.
6. Reports have a fixed, short format: worker 7 lines, tester 10, scout 15.
7. Wide searches go to Haiku, and graph answers are capped at 600 tokens.
8. Fixes go back to the same worker through SendMessage, while its context and cache
   are still warm.
9. Ponytail: less code means fewer tokens to write, test and fix.
10. Hard limits: 3 agents in parallel, 2 fix rounds, 3 tester runs, `maxTurns`,
    `autoCompactWindow`, `omitClaudeMd` on Haiku agents, and a 1-hour cache for the router.
11. The built-in Explore and Plan agents are turned off and `general-purpose` is denied,
    so nothing quietly runs on Opus.

## Good to know

- Tested: the manifest passes `claude plugin validate`; every agent and skill loads; a
  clean install pulls in Ponytail; taste-skill installs from this marketplace; all five
  scripts pass on macOS's built-in bash 3.2. **Not tested yet: a full batch with real
  agents.**
- `ya-board` and the other scripts are on Claude's PATH, not in your own terminal. Ask
  Claude, for example "mark ticket 02 ready".
- Plugins can't set `permissionMode`, so start the router with
  `--permission-mode auto` (or `acceptEdits`).
- The router replies in your chosen language; tickets are written in English, which is
  cheaper and followed more exactly.
- Watchdog thresholds can be changed through env vars: `STALE_MIN` (15), `LONG_MIN`
  (45), `INTERVAL` (600 seconds).

## Credits

Adapted from Superpowers, Matt Pocock's skills and andrej-karpathy-skills; works with
Ponytail, taste-skill and graphify. All MIT. Details in [NOTICE](NOTICE).
