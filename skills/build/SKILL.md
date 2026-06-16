---
name: build
description: Load the canonical patterns relevant to the task being planned, by routing through CANONIFY.md. Run when the user types /canonify:build or signals plan-time intent - phrases like "let's plan this", "I want to add X", "what's the right pattern for Y", "starting work on Z". Bookend partner to /canonify:commit. Routing logic is identical to commit-time, but pointed at the task description instead of the diff.
---

# Canonify: Build

The **Build** gate of **Canonify** - the middle of three lifecycle gates, all routing through `CANONIFY.md`:

- **Plan (`/canonify:plan`):** orient on the whole landscape (breadth, no sub-docs).
- **Build (this skill, `/canonify:build`):** task -> match `CANONIFY.md` summary lines -> load implicated docs -> use the canonical patterns when writing code (depth on the task's slice).
- **Commit (`/canonify:commit`):** diff -> match the same summary lines -> audit the code against the docs' rules.

This skill makes the plan-time read explicit and visible. Without it, the routing happens implicitly and silently - `CANONIFY.md` is just a markdown file someone is "supposed to" read. Running it as a skill makes the routing decision part of the planning conversation.

## How to run

1. **Read `CANONIFY.md` at the repo root.** Note every one-line summary - those are the routing signals. By convention the manifest is `CANONIFY.md` and the canon corpus it points to lives under `docs/`.

2. **Read the task.** Use the user's stated task description. If unclear, ask for one sentence ("what are we building?") before continuing. Don't guess.

3. **Walk every CANONIFY.md summary line and ask whether the task touches that area.** For **each** doc-summary line (including any project-wide rules entry):

   - Read the one-paragraph summary describing what that doc covers.
   - Evaluate against the task: does the task plausibly require work in this area? Consider:
     - Feature areas named in the task (e.g. "add a new `<feature>`" -> the `<feature>` canon).
     - Side effects implied by the task (e.g. "send the user a notification when X" -> the relevant integration canon; "show only to admins" -> the access-control canon).
     - The general code surface a change touches (any code change -> the project-wide rules entry, if one exists).
   - If yes -> mark that doc as **implicated**. If no -> record it as **skipped** with a one-line reason.

   Output a short routing table:

   ```
   Implicated: <doc-a>.md, <doc-b>.md, <project-wide-rules>.md
   Skipped:    <doc-c>.md (one-line reason it doesn't apply)
               <doc-d>.md (one-line reason)
               <doc-e>.md (one-line reason)
   ```

   The "Skipped" list proves the read covered every doc and consciously dropped it - not that the doc was forgotten.

4. **Load the implicated docs in parallel.** Read each one in full. Pay attention to:
   - Tables of rules
   - Checklists (multi-step procedures the doc spells out)
   - "Gotchas" sections
   - Any project-wide rules entry
   - Worked examples - when an existing implementation matches the task shape, name it as a reference

5. **Summarize the canonical pattern for the task.** A few short paragraphs covering:
   - The one or two reference implementations to copy from (file paths + line refs)
   - The non-obvious gotchas relevant to *this* task (don't paste every gotcha from every doc - only the ones the task will actually hit)
   - Any multi-step checklists or imports/declarations the task will need
   - The order to build in (call out which step has to come first and why)

6. **Hand off to coding.** Once the routing + canonical pattern is laid out, proceed to writing the plan or implementation. The audit ends when the user has enough context to either commission the work or refine the task.

## Don't do

- **Don't load every doc.** Walk the summaries and skip what doesn't apply. Reading docs that aren't implicated wastes context.
- **Don't write code yet.** This skill is about loading patterns, not implementing. Coding is the next step, post-routing.
- **Don't paraphrase rules** when listing them in the summary - quote from the docs directly so the user can verify.
- **Don't replace the CANONIFY.md file itself.** It's the routing manifest. This skill consults it; it doesn't supersede it.

## Output template

```
## /canonify:build - pattern routing

Task: <one-line restatement of what we're building>

### Routing - CANONIFY.md summary sweep

Implicated:
- <doc-a>.md - <one-line reason: which part of the task pulls this in>
- <doc-b>.md - <one-line reason>

Skipped:
- <doc-c>.md - <one-line reason it doesn't apply>
- <doc-d>.md - ...

### Canonical pattern for this task

**Reference implementations:**
- `path/to/file.ext:NN` - <what it does, why it's a good model>
- `path/to/other.ext:NN` - <what it does>

**Required imports / multi-step checklists:**
- ...

**Gotchas that will hit this task specifically:**
- <Gotcha 1> - <one-line from the doc>
- <Gotcha 2> - <one-line from the doc>

**Build order:**
1. <Step> - <why first>
2. <Step>
3. ...

Ready to write the implementation.
```
