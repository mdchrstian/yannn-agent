---
name: setup
description: One-time yannn-agent setup for the current git project (settings, permissions, gitignore, .worktreeinclude) and a check of companion tools (ponytail, graphify, taste-skill, node).
disable-model-invocation: true
allowed-tools: Bash(ya-setup) Bash(ya-setup --check) Bash(ya-graph *)
---

1. Run `ya-setup`. It prints `ok` / `warn` lines. Do not read any other file.
2. For each `warn` line, give the user the one fix command from that line, in
   ${user_config.language}. Nothing else.
3. If graphify is installed and the repo has more than ~200 source files
   (`git ls-files | wc -l`), offer to run `ya-graph update` now (no LLM, zero tokens).
   Smaller repo: say the graph is not worth it yet; `rg` is cheaper.
4. If `.claude/agents/` in this project contains router.md, worker.md, tester.md,
   reproducer.md or Explore.md, or `.claude/skills/ticket/` exists (old non-plugin
   version), tell the user to delete them, since they now come from the plugin.
5. End with: next step is `/yannn-agent:brainstorm <idea>`, or `/yannn-agent:plan <idea>`
   when the idea is already clear.
