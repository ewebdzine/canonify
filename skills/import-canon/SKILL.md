---
name: import-canon
description: Seed new canons in THIS repo from a list of existing canons in another project - using the source canons as the structural template, then re-indexing this repo's code so every citation and the verified SHA are correct here. It is create-canon driven by a source list, not a file copy. Run when the user types /canonify:import-canon, or says "import the canons from project A", "replicate the composite canons here", "bring in the canons from <path/repo>", "seed this repo's canons from <project>", "copy canons from <project> and re-verify".
---

# Import canons (from another project)

Canonify's **bulk seeding from a sibling project** gate. Several repos that share a platform
(e.g. Composite C1) each re-document the same patterns; this gate lets you take the canons one
project already wrote and stand them up in THIS repo without re-authoring from scratch - and without
the trap of a blind copy.

**It is `/canonify:create-canon` run over a list, seeded from existing canons instead of a blank
read of the code.** The crucial difference from `cp -r`: every canon cites `file:line` into a
*specific* repo and carries a `verified: <sha>` for *that* repo's HEAD. Copy the source files in and
those citations point at the OTHER project's paths and a foreign commit - they are wrong, and
`/canonify:doctor` will flag the whole batch. So each source canon is used only as a **skeleton +
checklist** (its topic, sections, the patterns/gotchas it documents); the claims are re-proved
against this repo's code. Sibling of `/canonify:kickoff`'s scan-and-seed, which seeds from a code
scan instead of from another project's canons.

## How to run

1. **Get the source list.** The user points at the source canons - a folder
   (`<projectA>/docs/<area>/`), a single canon file, or an explicit list of paths. Enumerate the
   concrete set of source `.md` canons (read the folder's `INDEX.md` if present). Confirm the list
   in one line. If nothing is named, ask which folder or canons to import.

2. **Place each into THIS repo's taxonomy.** For each source canon, decide its destination category
   from this repo's `CANONIFY.md` blocks + existing `docs/` folders - the same placement logic as
   `create-canon`. Do NOT mirror the source project's folder layout blindly; map into the categories
   this repo already uses (create a category only when none fits). Destination is
   `docs/<category>/<topic>.md`.

3. **Dedup before importing.** For each source canon, check whether this repo already has a canon
   covering that area (route via `CANONIFY.md` summaries, like build/commit/update-canon):
   - **Already covered here** -> SKIP the import; note it. If the source canon clearly carries
     something this repo's version lacks, suggest `/canonify:update-canon` instead of a second doc.
   - **Not covered** -> import it (steps 4-6).

4. **Import = create-from-template, re-proved against THIS repo's code.** Treat the source canon as
   the skeleton and the list of things to look for - never as ground truth to paste. For each:
   - Run `create-canon`'s evidence pass **against this repo**: find this repo's equivalent of each
     cited wrapper/class/config-key/call-site (grep this repo's code), and re-cite every claim to
     **this repo's** real `file:line`.
   - **Any claim with no equivalent here -> flag it, never fabricate.** Drop it (the projects
     genuinely differ) or leave a `TODO:` for the author. This is exactly where project A and B
     diverge, and surfacing it is half the value.
   - Carry over the source's prose and gotchas only where this repo's code confirms them. House-style
     applies: ASCII punctuation, terse, project-specific, every factual claim cited `file:line`.
   - For a UI/design canon, redo the **Mockup recipe** against this repo's tokens/partials.

5. **Register each (required).** Add the one-line summary to this repo's `docs/<category>/INDEX.md`
   AND a block to this repo's `CANONIFY.md`. An unregistered canon is unroutable - the most common
   miss when importing in bulk.

6. **Stamp `verified` to THIS repo's HEAD.** Each imported canon was just confirmed against this
   repo's code, so stamp this repo's current commit (read-only `git rev-parse HEAD`; use the platform
   git the repo requires). Never carry over the source canon's `verified` SHA - it is another repo's
   commit.

7. **Report.** Per source canon: imported (path, what was re-cited, what was flagged as divergent or
   left as TODO) | skipped (already covered) | partial (imported with N unresolved citations to
   fill). End with a summary: M sources, X imported, Y skipped, Z needing author follow-up.

## Don't do

- **Don't copy citations.** The source's `file:line` describe the source repo. Every cite must be
  re-resolved against this repo or dropped - never carry one over unverified.
- **Don't fabricate an equivalent.** If this repo has no counterpart for a pattern the source
  documents, flag it; do not invent a path, method, or line number to make the import "complete."
- **Don't carry over the source `verified` SHA.** Always stamp this repo's HEAD, set after re-proving.
- **Don't duplicate.** If this repo already covers the area, skip (or hand to `/canonify:update-canon`).
  Don't add a second canon or graft the source's structure onto an existing one.
- **Don't mirror the source's folders.** Map into this repo's categories; one topic per canon.
- **Don't skip registration.** Bulk imports make this easy to forget; an unregistered canon is invisible.

## Output template

```
## /canonify:import-canon - <source> -> this repo

Source canons (<N>): <folder or list>

Routing:
- <projectA>/docs/<area>/foo.md -> docs/<category>/foo.md            [import]
- <projectA>/docs/<area>/bar.md -> covered by docs/<category>/bar.md [skip]

[per imported canon:]
Imported: docs/<category>/foo.md  (re-cited against this repo: <files / greps>)
- diverged / unresolved (author to fill): <claims with no equivalent here>
Registered: docs/<category>/INDEX.md (+1), CANONIFY.md (+1)
verified: <this-repo HEAD sha>

### Summary
<M> sources -> <X> imported, <Y> skipped (already covered), <Z> with TODO citations to fill.
```
