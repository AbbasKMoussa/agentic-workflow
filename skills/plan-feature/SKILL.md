---
name: plan-feature
description: Turn an idea for a feature, story, or epic into a written spec an implementing agent can execute against without drifting — grounded in what the codebase actually does (via the codebase-mapper agent for brownfield work and live LSP/indexing tools for spot-checks), built on checkable acceptance criteria, broken into small dependency-ordered tasks, and gated behind explicit human approval before any code gets written. Use whenever the user wants to plan, spec, scope, or write a PRD/proposal for a feature or epic, break work into stories or tasks, or is about to start any non-trivial implementation where the requirements could be read more than one way.
---

# Plan Feature

Turn "I want to build X" into a spec an agent can execute against without
guessing — grounded in what the code actually does, checkable down to its
acceptance criteria, and never starting work without a human's sign-off.

## Why this approach

The single biggest source of agents building the wrong thing isn't a weak
model — it's an underspecified ask. **Ambiguity isn't neutral**: humans fill
gaps from shared context, agents fill them from statistical likelihood, and
every vague phrase in a spec is a coin-flip the agent will resolve confidently
and wrong. So this skill rests on three commitments:

1. **Ground claims in reality, not assumption** — especially in brownfield
   work, where "prior decisions" and "existing patterns" are facts to verify,
   not things to infer from folder names
2. **Surface what you're assuming before you draft** — catching a wrong premise
   costs one message before the spec exists, a redraft after, a rewrite once
   code is built on top of it
3. **Make every criterion checkable** — numbers, not adjectives — so it can
   double as the verification phase's test plan later, instead of becoming a
   second vague document to argue about

## Step 1 — Ground the spec in what's actually there

**Brownfield**: before drafting anything, delegate to the `codebase-mapper`
agent, scoped to the subsystem this feature touches — its entire job is
producing a cited, traced picture of the relevant code instead of a guess. Its
output becomes this spec's "Grounding" and "Constraints & prior decisions"
sections directly. Don't re-derive what it already produced.

**Use indexing/LSP tools yourself for anything you spot-check.** When something
comes up mid-conversation that needs a quick verification — a claim the user
makes, a pattern you want to confirm before writing it down as a "prior
decision" — check your own tool list *first* for semantic-search /
symbol-lookup / codebase-indexing MCP tools (names like `codebase-retrieval`,
`find_symbol`, `find_referencing_symbols`, `search_code` — they surface as
`mcp__<server>__*` regardless of which product provides them: Auggie, or
whatever else is wired up) and for native code-intelligence tools
(jump-to-definition, find-references, diagnostics — present when LSP support is
enabled). These return one precise, structured answer where grep takes many
exploratory passes. Reach for them first; fall back to grep/read only when
neither is available — and say so, so the user knows installing one would help.

**Greenfield**: there's nothing to map — this step becomes "establish the
vision." Interview for what currently exists only in the user's head: who's it
for, what does it replace or improve, what does success look like. Same
instinct as brownfield grounding (don't let the agent invent the vision either),
just aimed at intent instead of legacy code.

## Step 2 — Surface assumptions before drafting anything

Before writing the spec itself, write down — and show the user — what you're
about to assume:

```
ASSUMPTIONS I'M MAKING:
1. [technology / architecture / scope assumption]
2. [...]
→ Correct me now, or I'll draft the spec around these.
```

Wait for the correction. This is the cheapest possible moment to catch a wrong
premise — don't skip it because "it's obviously what they meant."

## Step 3 — Draft against this template

````markdown
# Spec: <Feature / Story / Epic name>

## Objective
What are we building, for whom, and why does it matter? One or two concrete
sentences — if you can't state success in a sentence, you're not ready to
draft the rest.

## Scope
**In scope:**
- ...
**Explicitly out of scope:**
- ... (this list matters as much as the one above — agents over-build by default)

## Grounding — what's actually there
<!-- brownfield only; delete this section entirely for greenfield work -->
From `codebase-mapper`'s traced output for this subsystem — every line below
carries a `file:line`, not an assumption:
- Existing pattern: ...
- Constraint this creates for the new work: ...

## Constraints & prior decisions
- Architectural choices already made, and why (link the decision if one exists)
- Hard limits: performance budgets, compliance requirements, libraries to use
  or avoid
- Things that bit us before — name them, so nobody repeats them

## Code style
<!-- one real snippet from this codebase — not a description of "clean code" -->
```<lang>
...
```

## Acceptance criteria
Checkable, not descriptive — numbers, not adjectives; happy *and* unhappy paths:
- Given <state>, When <action>, Then <observable outcome> [and <measurable bound>]
- ...

## Task breakdown
Dependency-ordered; small enough each is independently checkable (~5 files):
1. [ ] <task> — touches: <paths> — proves: <which acceptance criterion above>
2. ...

## Open questions
Anything genuinely unresolved that needs a human call before or during the work.
````

## Step 4 — Make every acceptance criterion checkable

This is the highest-leverage habit in the whole skill — it's what later lets a
*separate* verification pass check the work against the spec instead of
against vibes:

- **Numbers, not adjectives.** "Fast" → "p95 under 200ms." "Handles large
  files" → "Given a 100K-row CSV, When export is triggered, Then the file
  streams to download, completes within 30s, and peak memory stays under
  512MB."
- **Given/When/Then** for anything with a trigger and a response — it forces
  the trigger, the condition, and the observable result into the open, where
  an agent (or a test) can check each one independently.
- **State the unhappy paths on purpose** — empty input, duplicates, timeouts,
  permission errors. Skip them and the agent invents its own behavior for
  them, silently, and you find out in production.

## Step 5 — Break into small, checkpointable tasks

Order by dependency, and size each one so it's independently verifiable —
roughly five files or fewer. Each task should name which acceptance criterion
it proves. This is what turns "review one giant plan and hope it holds
together" into "checkpoint after each small piece, catch drift while it's
still cheap to redirect."

## Step 6 — Approval gate: nothing proceeds without sign-off

Show the complete draft. Walk through it section by section if it's long.
Refine based on feedback — possibly more than once. Implementation does not
start until the human explicitly approves: this mirrors Plan Mode's
read-propose-approve-execute loop, and it's the gate that makes every other
step in this skill matter — a perfectly-specified plan nobody reviewed is just
a more confident way to build the wrong thing.

## Anti-patterns (for this skill itself)

- **Drafting from memory instead of grounding** — writing "prior decisions"
  from a guess about the codebase when `codebase-mapper` (or a quick
  semantic/LSP lookup) could have given you a cited fact
- **Descriptive acceptance criteria** — "works well," "handles errors
  gracefully," anything that can't become a test case as written
- **Skipping the assumptions step** because the ask "seems obvious" — that's
  exactly when an agent's silent guess differs most from what the user meant
- **One giant task** instead of several small, checkpointable ones — it turns
  "catch drift early" back into "discover drift at the end"
- **Treating the approval gate as a formality** — rubber-stamping your own
  draft defeats the entire purpose of having a human in the loop at the
  cheapest possible moment to redirect it
