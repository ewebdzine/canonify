# Canonify doc house-style

How to write a canonical-pattern doc so the gates can route to it and a developer can copy from it.
Distilled from the reference exemplars: a service doc (`docs/services/activecampaign.md`) and a
design doc (`docs/design/charts.md`). Keep new docs in this shape.

## The rules (non-negotiable)

1. **Back every factual claim with `file_path:line_number`.** If you cannot point at it in the
   code, do not write it. Never invent method names, config keys, or behavior.
2. **One topic per file.** Small, coherent docs route better than monoliths.
3. **Terse and practical.** Document *this codebase's* usage, not the vendor's general theory.
   Every line should be specific to how the project actually does it.
4. **ASCII punctuation only** in anything that may be rendered (`-` not em-dash, straight quotes,
   `->` not arrow, `...` not ellipsis). Some renderers mojibake Unicode.
5. **Quote rules, don't paraphrase**, so a reader (or the commit audit) can verify against source.
6. **It is a living standard.** Update the doc in the same change that alters the pattern.
7. **Carry a `verified` marker.** A YAML frontmatter line `verified: <git-sha>` records the commit
   at which the canon was last confirmed against the code. `/create-canon` stamps it, `/canonify-commit`
   refreshes it, and `/canonify-doctor` reads it to flag drift (see the Doctor design in `SPEC.md`).
   One SHA, set by the gates - never hand-maintained, never a per-file date log (git holds the dates).

## Section skeleton

Adapt to the topic, but keep this order:

```
---
verified: <git-sha>   # commit where this canon was last confirmed against the code (set by /create-canon, refreshed by /canonify-commit)
---

# <Topic> - <the tool / pattern in one phrase>

<One-line intro: what it is and why this codebase uses it.>

> Living standard + how it routes (read by the Canonify gates via CANONIFY.md).

## Reference implementation (copy this)
| Layer | File |
... the canonical files + file:line to copy from ...

## The pattern (non-negotiable)
<the canonical markup / call sequence, with a real excerpt and file:line>

## Configuration / API surface
<config keys table (key | holds | read at file:line); or the public methods, grouped>

## Recipes
<real call sites with file:line and short excerpts>

## Gotchas
<honest footguns: tenant-specific values, async traps, expiry, ordering, dead code>

## References
<file:line list of the key files + vendor doc URL + links to related docs>
```

## Design-doc extra: the Mockup recipe

A doc describing a UI element adds a **Mockup recipe** section: a copy-pasteable, token-accurate
HTML/CSS snippet that reproduces the element in a static mockup so it matches production. If the
element needs a JS library that a static mockup cannot run, provide a representative static
approximation (see how `charts.md` draws a Morris-style chart as inline SVG). All element docs
assume the project's `foundation.md` token wrapper is around the snippet.

## Index every doc

Add each new doc to its category's `INDEX.md` and to the `CANONIFY.md` manifest with a one-line
summary - that summary is what the gates match against. An un-indexed doc is invisible to routing.
