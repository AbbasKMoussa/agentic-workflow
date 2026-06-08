# Agentic Kit

A personal Claude Code toolkit, assembled by working through Claude Code's
features topic by topic and keeping only what earns its place — skills,
commands, hooks, and agents that improve the daily workflow, not just demos.

## What's built

- **[`skills/scaffold-context`](skills/scaffold-context/)** — generates a
  tailored CLAUDE.md for a project. Auto-detects what it can from the codebase
  (stack, scripts, layout), interviews for what it can't (risk tolerance,
  architecture, anti-patterns), and routes the result to the right layer:
  project `CLAUDE.md`, personal `~/.claude/CLAUDE.md`, or path-scoped
  `.claude/rules/`.
- **[`agents/codebase-mapper`](agents/codebase-mapper.md)** — maps a specific
  subsystem before you plan changes to it, so the plan rests on what the code
  actually does instead of what it looks like it does. Read-only, traces real
  call paths, cites `file:line` for every claim (no "probably"/"typically"),
  prefers whatever semantic-search/LSP/indexing MCP tools happen to be
  available and falls back to grep when there aren't any, and persists what it
  learns to project memory so the same subsystem never gets re-explored from
  zero. Invoke with a scoped brief — "map the notification subsystem" — not
  "understand everything."
- **[`skills/plan-feature`](skills/plan-feature/)** — turns "I want to build X"
  into a written spec an agent can execute against without drifting. Grounds
  brownfield work in `codebase-mapper`'s cited output (and reaches for
  LSP/semantic-indexing MCP tools directly for its own spot-checks), surfaces
  its assumptions before drafting so wrong premises get caught at the cheapest
  moment, drafts against a baked-in six-section template with checkable
  Given/When/Then acceptance criteria (numbers, not adjectives), breaks work
  into small dependency-ordered tasks, and gates everything behind explicit
  human approval before implementation starts.

This list grows as each topic produces something worth keeping.
