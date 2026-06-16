---
name: commit
description: Audit the current diff against the canonical patterns in CANONIFY.md and the docs it points to. Run when the user types /canonify:commit or asks to check before committing/pushing/deploying/shipping, verify the changes, or perform similar pre-commit verification. Mirrors the plan-time CANON routing - same selector, opposite direction.
---

# Canonify: Commit

The **Commit** gate of **Canonify** - the last of three lifecycle gates, all routing through `CANONIFY.md`. Same routing logic as `/canonify:build`, opposite direction:

- **Build (`/canonify:build`):** task description -> match `CANONIFY.md` summary lines -> load implicated docs -> write code following the canonical patterns.
- **Commit (this skill, `/canonify:commit`):** diff -> match the same summary lines -> load the same implicated docs -> check the diff against the rules in those docs.

This skill does **pattern adherence** only. It does NOT check scope creep, intent, or "did we build the right thing." It checks "did we follow the canonical patterns the docs document."

## How to run

1. **Refresh the routing map.** Read `CANONIFY.md` at the repo root. Note the one-line summaries for every doc - those are the routing signals.

2. **Get the diff.** In parallel:
   - `git diff` (unstaged changes)
   - `git diff --cached` (staged changes)
   - `git status` (to see new files not yet diffed)

   Combine into a working set of touched files and the actual line changes.

3. **Walk every CANONIFY.md summary line and ask whether the diff touches that area.** This is the inverse of the plan-time read: the summaries are the complete checklist of doc-covered patterns; iterate over them so nothing slips by.

   For **each** doc-summary line in `CANONIFY.md`:

   - Read the one-paragraph summary describing what that doc covers.
   - Evaluate against the diff: does the diff touch anything described in that summary? Consider:
     - File paths touched (a new file under a path the summary names clearly implicates that doc).
     - Types, methods, or facades named in the change (a call to a facade the summary describes implicates that doc even if the file lives elsewhere).
     - Patterns introduced (a new class implementing an interface a summary covers implicates that doc regardless of folder).
     - File-type or surface rules (any file of a type a summary governs implicates that doc).
     - Cross-cutting rules (any change in a language or area a project-wide rules doc covers implicates that doc).
   - If yes -> mark that doc as **implicated**. If no -> record it as **skipped** with a one-line reason.

   Output a short routing table before continuing to step 4:

   ```
   Implicated: <doc-a>.md, <doc-b>.md, <project-wide-rules>.md
   Skipped:    <doc-c>.md (no matching surface in the diff)
               <doc-d>.md (no matching surface in the diff)
               <doc-e>.md (no matching surface in the diff)
   ```

   The "Skipped" list proves the audit considered every doc and consciously dropped it - not that the doc was forgotten.

4. **Load the implicated docs in parallel.** Read each one. Pay attention to:
   - Tables of rules (often under "Conventions" or "Gotchas" or similar)
   - Checklists (e.g. multi-place edit lists that must all be present)
   - "Gotchas" sections
   - Any project-wide rules doc the manifest points to
   - Any rule with a "**How to apply:**" or "**Rule:**" marker

5. **Check the diff against each rule.** For each rule the implicated canons document, evaluate whether the diff complies. The canons hold the authoritative rules - re-read them at runtime; do not carry a hardcoded rule list in this skill.

6. **Report findings.** Group output by doc, then by rule. For each finding:
   - **Rule:** one-line summary of what the doc says
   - **Status:** pass, uncertain (couldn't verify cleanly), or violation
   - **Location:** `file:line` reference into the diff
   - **Fix:** one-line description of what to change. Don't auto-fix - the user decides.

   End with a one-line summary: how many docs were checked, how many rules, how many findings.

7. **Offer to capture canon drift (the loop's feedback edge).** Sometimes the diff doesn't violate
   the canon - it reveals the *canon* is behind: the change intentionally and consistently diverges
   from a documented rule (the pattern moved on), or it surfaced a real footgun the canon never
   mentions. When you see that, don't file it as a violation - note it and offer one line:
   "`<canon>` looks behind the code here (<what>). Capture it with `/canonify:update-canon`?" An
   offer, never an auto-edit - the user decides whether the canon or the diff is the source of truth.

## Don't do

- **Don't auto-fix.** Report only - including the canon-drift offer (step 7): suggest, never edit the canon here.
- **Don't run the full test suite or build.** Pattern audit only.
- **Don't check scope creep** ("did we add features beyond what was asked"). That's a separate concern not in v1.
- **Don't recommend refactors.** If the diff complies with the docs, it passes - even if you'd structure it differently.
- **Don't be exhaustive on unmatched docs.** If the diff doesn't touch a doc's surface, don't waste tokens reading it.

Note: in some repos, `git push` triggers a deploy (e.g. pushing to the default branch). Treat this gate as a real safety check, not a soft suggestion - it may be the last review before changes reach production.

## Output template

```
## /canonify:commit - pattern adherence audit

Files touched: <N>

### Routing - CANONIFY.md summary sweep

Implicated:
- <doc-a>.md - <one-line reason: which area in the diff touched this doc>
- <doc-b>.md - <one-line reason>

Skipped:
- <doc-c>.md - <one-line reason it doesn't apply>
- <doc-d>.md - ...

### Findings

#### <doc-a>.md

- pass <Rule>: <one-line evidence>
- violation <Rule>: <what's wrong> @ `path/to/file:42`
  - Fix: <one-line>
- uncertain <Rule>: <why uncertain> @ `path/to/file:N`

#### <doc-b>.md
...

### Summary

<N> docs swept, <M> implicated. <K> rules checked. <V> violations, <U> uncertain. <Free to commit | Address findings above before committing>.

<optional, only if a canon looks behind the code:>
Canon drift: <canon>.md may be behind the diff (<what changed>). Capture with /canonify:update-canon?
```
