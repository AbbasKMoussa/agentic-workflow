---
name: scaffold-context
description: Generate a tailored CLAUDE.md for a project (or improve an existing one) by combining automatic codebase detection with a short interview for what can't be inferred — risk tolerance, architectural conventions, anti-patterns, banned approaches. Use whenever the user wants to create, bootstrap, write, scaffold, or improve a project's CLAUDE.md or .claude/rules, set up Claude Code for a new repo, generate project memory/context files, or asks for help getting Claude up to speed on a codebase. Also routes durable personal preferences to the user's ~/.claude/CLAUDE.md and can split path-specific conventions into scoped rules files.
---

# Scaffold Context

Generate a CLAUDE.md that earns its place in the context window — grounded in what
the codebase actually does, not generic boilerplate, and routed to the right layer
(project vs. personal vs. path-scoped).

## Why this approach

CLAUDE.md is **context, not enforcement** — it competes for a limited, always-on
budget, and a wrong fact in it is worse than a missing one (Claude will follow it
confidently into a wall). So this skill is built around three commitments:

1. **Detect before asking.** Most of what belongs in a CLAUDE.md (stack, scripts,
   layout, conventions) is already sitting in the repo. Read it first — asking the
   user to repeat what `package.json` already says wastes their time and yours.
2. **Interview only what code can't tell you.** Risk tolerance, architectural
   judgment calls, and "things that bit us before" live in the user's head, not
   the filesystem.
3. **Never invent facts.** Where detection and interview both come up empty, write
   `TODO: <gap>` rather than a plausible-sounding guess.

## Step 1 — Detect (dynamic context injection)

Before asking the user anything, build a real picture of the project by reading it
directly. This "injects" project-specific context fresh on every run, instead of
leaning on generic assumptions about what a project like this "probably" looks like:

- **Stack & scripts**: `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod`
  / `Gemfile` / `composer.json` — dependencies, test/build/lint commands
- **Layout**: top-level directories, monorepo structure, where source vs. tests
  vs. config live
- **Existing docs**: `README.md`, `CONTRIBUTING.md`, any existing `CLAUDE.md` or
  `.claude/` directory — don't duplicate or contradict what's already there, improve it
- **Living conventions**: recent `git log` for commit style and branch naming;
  skim a few recently-touched files for naming and formatting patterns

Summarize what you found back to the user in a few lines before moving on. This
catches misdetections early (e.g., a vendored subproject that isn't the real stack)
and shows them you did the homework instead of generating something generic.

## Step 2 — Interview what detection can't reach

Ask focused, conversational questions — batch them, and skip anything detection
already answered. Use `AskUserQuestion` where a multiple-choice framing fits;
let open-ended things stay freeform. Roughly in priority order:

1. **Risk & blast radius** — which actions in this repo need confirmation before
   Claude takes them (deploys, migrations, force-pushes, schema changes)? What's
   reversible vs. not?
2. **Architecture & conventions** — patterns to follow or avoid, where new code of
   a given kind belongs, abstractions the team has strong opinions about
3. **Anti-patterns** — specific things Claude should never do here: libraries to
   avoid, approaches that caused past incidents, "we tried that, don't"
4. **Workflow norms** — PR/commit conventions, review expectations (skip if
   `git log` already made this obvious)

Don't turn this into a questionnaire dump. Two or three short rounds that adapt to
what they say beats one long form people rush through.

## Step 3 — Draft

Write the draft to fit constraints that come from hard-won lessons about what
actually survives in a CLAUDE.md that gets used for months, not just written once:

- **Budget: ~150 lines, max.** This file sits in context on every single turn —
  padding it to "look thorough" is a tax on every future conversation.
- **Section order matters**: lead with the concrete and load-bearing (stack,
  commands, layout), then conventions and workflow norms, and put **anti-patterns
  last** — they read as corrections, and land better once the model already has
  the positive picture of how things should go.
- **No invented facts.** Anything detection plus interview didn't pin down becomes
  `TODO: <specific gap>` — visible, not buried, so the user can fill it in later.
- **Concrete beats vague.** "`npm run test:unit` for unit tests; `npm run test:e2e`
  needs the dev server running first" beats "make sure to write tests."

## Step 4 — Route to the right layer

Not everything that comes up belongs in the project's `CLAUDE.md`. Sort it:

| Goes in | What | How to handle |
|---|---|---|
| `./CLAUDE.md` (committed) | Facts true of *this* project for *anyone* working in it: stack, commands, architecture, this-repo's anti-patterns | Write directly, after confirmation (Step 5) |
| `~/.claude/CLAUDE.md` (user-level) | Durable *personal* preferences that hold across all the user's projects (communication style, standing habits) | **Never edit directly** — hand the user a snippet to merge themselves; it's their file across every project, not this project's to own |
| `.claude/rules/<name>.md` (path-scoped) | Conventions that only apply to part of the tree (e.g., "frontend/ lints differently than backend/") | Offer to split out, with `paths:` frontmatter glob-matching the relevant files, so the instructions load only when relevant — keeps the always-on budget small |

If everything cleanly fits the main CLAUDE.md, don't manufacture rules files just
to look sophisticated — only split out when there's a genuine path-specific divergence.

## Step 5 — Confirm, then write

Show the complete draft before writing anything. Call out every `TODO` explicitly
and ask whether the user can fill any of them in now. Once they approve:

- Write `./CLAUDE.md` (and any `.claude/rules/*.md` files) directly
- For `~/.claude/CLAUDE.md`, hand over the snippet — let them place it themselves
- Briefly summarize what landed where and why, so the routing decisions are visible
  and easy to undo if they disagree with one

## Anti-patterns (for this skill itself)

- **Generic output** — if the draft could be pasted into any other project
  unchanged, it has failed; every section should reflect something specifically
  true about *this* codebase
- **Padding to hit a length** — an honest 80 lines beats a padded 150
- **Silently writing outside the project** — `~/.claude/CLAUDE.md` belongs to the
  user across every project they have; always show and ask, never auto-edit it
- **Skipping the interview because detection looked thorough** — risk tolerance
  and anti-patterns live in people's memories of past incidents, not in the code
