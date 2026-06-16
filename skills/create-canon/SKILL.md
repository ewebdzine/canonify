---
name: create-canon
description: Author a new "canon" - one canonical-pattern doc (a .md) - for a file, service, or design element, then register it in CANONIFY.md + the category INDEX. The command name teaches the vocabulary: each pattern doc is a "canon." Run when the user types /canonify:create-canon, or says "create a canon for X", "document this file/service", "I need to document X", "write a canon for the Y service", "add a doc for this design element".
---

# Create a canon

Canonify's **authoring** gate. A **canon** is one canonical-pattern doc (a `.md`); the root manifest
`CANONIFY.md` indexes every canon by a one-line summary, and the lifecycle gates
(plan / build / commit) route off those summaries. Where they *consume* the canon, this gate
*grows* it: point it at a file, service, or design element and it writes a new canon and wires it
into `CANONIFY.md` so the other gates can immediately route to it.

It codifies the workflow for authoring a canon: read the real code, cite `file:line`, draft from
evidence, and ask only for what the code can't tell you.

## How to run

1. **Identify the target.** The user points at one of:
   - a **file / class** (a path or a type name),
   - a **service / integration** (e.g. a third-party API client, a `*Service` wrapper),
   - a **design element** (a UI pattern + its CSS/JS source).
   If nothing is named, ask once: "what file, service, or element should I turn into a canon?" Don't guess.

2. **Place it.** Decide the category + destination from `CANONIFY.md`'s blocks and the existing
   `docs/` folders - the repo defines its own categories. Don't impose a fixed taxonomy; read what
   categories already exist (the blocks in `CANONIFY.md`, the subfolders under `docs/`) and slot the
   canon into the matching one, creating a new category only when none fits. The destination is
   `docs/<category>/<topic>.md`. Infer the obvious case; confirm the destination path in one line.

3. **Index the target (the evidence pass).** Read the target in full and gather the proof every
   claim will cite:
   - the wrapper / class and its public surface,
   - how it's wired up / registered / constructed (DI, factory, entry point),
   - config keys / settings and where they're read (`file:line`),
   - real call sites (grep the repo),
   - related types / canons it overlaps with.
   For a **design element** also read its source partial(s), any vendor library it depends on, and a
   reference template / view that uses it.

4. **Ask only the gaps.** Draft from the code first, then ask a SHORT set of questions for what the
   code cannot reveal:
   - the one-line scope / title (confirm),
   - gotchas / footguns the author knows that aren't visible in code,
   - anything to deliberately defer to another canon (overlap),
   - (design) a production screen or tokens to match.
   Keep it to a few questions. Don't interrogate - prefer reasonable defaults and confirm.

5. **Write the canon** following the doc-style template that ships with Canonify
   (`${CLAUDE_PLUGIN_ROOT}/templates/doc-style.md`). The core rules, inlined so this gate is self-contained:
   - **Cite every factual claim with `file_path:line_number`.** If you can't point at it in the
     code, don't write it. Never invent a method name, config key, or behavior.
   - **ASCII punctuation only** (`-` not em-dash, straight quotes, `->` not arrow, `...` not ellipsis).
   - **Terse and project-specific.** Document THIS codebase's usage, not the library's general theory.
   - **Section skeleton, in order:** title + one-line intro; a living-standard + routing note;
     a reference-implementation table (the files to copy from); the pattern (a real excerpt with
     `file:line`); config / API surface; recipes (real call sites); gotchas; references.
   - **`verified:` frontmatter marker.** Open the canon with a YAML frontmatter line
     `verified: <git-sha>` recording the commit at which the canon was confirmed against the code.
     Set it here at authoring time; the Doctor gate reads it to flag drift. One SHA, set by the
     gates - never hand-maintained.
   - For a **UI / design canon**, add a **Mockup recipe** section: a static, copy-pasteable snippet
     that reproduces the element so a mockup matches production. If the element needs a JS library a
     static mockup can't run, give a representative static approximation.

6. **Register the canon (required).** An unregistered canon is invisible to routing, so do both:
   - add a one-line summary to the category's `docs/<category>/INDEX.md`,
   - add the same summary - as a `### docs/<category>/<topic>.md` block - to **`CANONIFY.md`** under
     the right category.
   The summary is what the gates match on: name the key types / area so a future task or diff that
   touches it matches here.

7. **Report.** The new canon's path, what was indexed, the INDEX + `CANONIFY.md` edits, and any gaps
   the author still needs to fill (e.g. a gotcha you couldn't verify from code).

## Don't do

- **Don't skip registration.** Writing the canon without updating `CANONIFY.md` + the `INDEX`
  leaves it un-routable - the most common and most damaging miss.
- **Don't invent.** No claim without a `file:line`. If you can't verify it, ask or omit it.
- **Don't over-ask.** Draft from the code; ask only what the code can't tell you.
- **Don't paste vendor theory.** Document THIS codebase's usage, not the library's general docs.
- **Don't duplicate.** If a canon already covers the area, update it instead of writing a second one.

## Output template

```
## /canonify:create-canon - <target>

Indexed: <files read + greps run>
Destination: docs/<category>/<topic>.md  (category: <chosen from CANONIFY.md / existing docs/ folders>)

Questions (gaps only):
1. <question the code can't answer>
2. ...

[after answers -> write the canon, then:]

Created: docs/<category>/<topic>.md  (the new canon, <N> lines, every claim cited file:line)
Registered:
- docs/<category>/INDEX.md   (+1 summary line)
- CANONIFY.md                (+1 block under "<Category>")

Gaps to fill: <anything unverified the author should confirm>
```
