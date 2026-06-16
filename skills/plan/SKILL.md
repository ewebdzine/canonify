---
name: plan
description: Orient on the whole codebase landscape before planning - load CANONIFY.md for a full-breadth, summary-level read of every subsystem, integration, and surface the project has documented, then triage the rough idea into reuse-vs-build. Run when the user types /canonify:plan, or signals pre-task / scoping intent - fuzzy phrases like "I have an idea for X", "what would it take to build Y", "I'm thinking about Z", "before we plan this", "let's scope out W". The breadth-first bookend that runs *before* /canonify:build.
---

# Canonify: Plan

The **Plan** gate of **Canonify** - the first of three lifecycle gates, all routing through `CANONIFY.md`. It reads the breadth manifest so the plan is grounded in what the codebase already contains, at a high level, before any architecture is committed.

The three gates are different altitudes, run in order:

- **Plan (this skill, `/canonify:plan`):** the whole landscape -> read every `CANONIFY.md` summary line -> triage the idea into "what already exists I can reuse" vs. "what's net-new." **Breadth. No sub-docs opened.** Raises *plan quality* (architecture).
- **Build (`/canonify:build`):** a concrete task -> route to the implicated docs -> load them in full -> canonical pattern + file:line refs. **Depth on a slice.** Raises *implementation fidelity*.
- **Commit (`/canonify:commit`):** the diff -> route to the same docs -> audit the code against their rules. Raises *correctness at the gate*.

The Plan gate exists because Build's routing is only as good as the task framing - if the planner doesn't know a capability exists, the task won't mention it, and the route never fires. Plan removes that dependency: it puts the full menu in front of the plan first, so the architecture reuses what's there (an existing service / client / helper / subsystem the idea can stand on) instead of reinventing it.

## How to run

1. **Read `CANONIFY.md` at the repo root, in full.** It is the **complete breadth map** - one one-line summary per canon across every category the project defines for itself (`CANONIFY.md` declares its own category structure; each category points at a block of docs under `docs/`). Read every summary line; the point of the Plan gate is total coverage of the landscape, not a slice. (`CANONIFY.md` is authoritative - you should not need to open the per-category `INDEX.md` files to get breadth.)

2. **Read the idea.** It is allowed to be fuzzy - that is what this skill is *for*. If there is no idea stated at all, ask for one or two sentences of intent ("what are we thinking about building?") before continuing. Don't invent one.

3. **Stay at summary altitude - do NOT open any `docs/` sub-doc.** Opening and deep-reading the implicated docs is Build's job, once the task firms up. Plan works only from the one-paragraph summaries in `CANONIFY.md`. Loading sub-docs here wastes context on a still-fuzzy idea.

4. **Cross-reference project memory, if any.** If the project carries a memory file (e.g. `MEMORY.md`) of in-flight and planned work, it is the other half of orientation - it tells you what overlaps, what's already underway, and what constraints are live. Note any project the idea touches or duplicates. Skip this step if no such memory exists.

5. **Produce a reuse-vs-build triage.** A few short groupings:
   - **Lean on (already exists):** the existing services / data patterns / subsystems the idea can stand on, named one line each, drawn from the `CANONIFY.md` summaries.
   - **Net-new:** the parts no existing doc or pattern covers - the genuinely new work.
   - **Overlaps in flight:** any active/planned project from memory that intersects the idea.
   - **Build will likely need:** name (don't open) the handful of docs Build will route to once the task is concrete. This is a loose pointer, not a routing decision.

6. **Hand off.** Plan ends when the idea is oriented. When it firms into a concrete task, run `/canonify:build` to route to the specific docs and load the canonical pattern. Plan does not plan and does not write code.

## Don't do

- **Don't open sub-docs.** Summary altitude only. The moment you open any doc under `docs/`, you've crossed into Build.
- **Don't produce an Implicated / Skipped routing table.** That precise sweep is Build's deliverable. Plan names *likely* docs loosely and moves on - it doesn't commit a route.
- **Don't write a plan or code.** This is orientation. Planning and implementation come after, post-handoff.
- **Don't skip the memory cross-check** (when the project has a memory file). In-flight projects are half the picture; the code summaries are the other half.
- **Don't replace Build or Commit.** Plan precedes them. It is additive - orient, then route, then audit.

## Output template

```
## /canonify:plan - landscape orientation

Idea: <one-line restatement of the fuzzy idea / area we're scoping>

### Lean on (already exists)
- <subsystem/service> - <one line: what it gives the idea, from the CANONIFY.md summary>
- <subsystem/service> - <one line>

### Net-new (no existing pattern)
- <area> - <one line: why nothing in the docs covers it>

### Overlaps in flight (from memory)
- <project> - <one line: how it intersects / whether it duplicates>

### Build will likely route to (not loaded yet)
- <doc-a.md>, <doc-b.md>, <doc-c.md>

Oriented. When the task firms up, run /canonify:build to route and load the canonical patterns.
```
