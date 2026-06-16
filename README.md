# Canonify

Canonify keeps every change true to a project's **canon** - the authoritative patterns a
codebase already uses - by routing work to the right reference doc at three lifecycle gates.

## The idea

One manifest (`CANONIFY.md`) at the repo root indexes a `docs/` corpus of canonical-pattern
docs. Three skills consult the manifest's one-line summaries:

- **Plan** (`/canonify-plan`) - orient on the whole landscape before planning (breadth, no sub-docs).
- **Build** (`/canonify-build`) - route a task to the docs it implicates, load them, follow the pattern (depth).
- **Commit** (`/canonify-commit`) - audit a diff against those same canons before shipping.

**Vocabulary:** a **canon** is one pattern doc (a `.md`); **the canon** is a repo's whole body of
them; **`CANONIFY.md`** indexes them; **`/create-canon`** authors a new one. The name is the verb:
each gate helps *canonify* a change - orient to the canon, apply it, verify against it.

**Language-agnostic by design:** the gates match prose summaries, not code, so the same skills
work on a .NET or a Python repo. Only each repo's `CANONIFY.md` + `docs/` differ (the *content*);
the skills + manifest convention + doc house-style are the portable *framework*.

## Quickstart

Drop the `canonify/` folder in your repo root and run **`/kickoff`** - it reads your project, asks two
questions (scan-and-seed? scheduled Doctor?), and builds the canon.

> *Drop it in. Kick it off.*

## Status: incubating

This folder is the **framework**, incubated inside the `ceosite` repo - which is its reference
implementation. The plan is to extract it to its own repo and publish it as a Claude Code plugin
once it is proven on a second project (ideally a different language). See [SPEC.md](SPEC.md) for
the feature checklist and the release definition-of-done.

`ceosite` itself keeps the *content* (`CANONIFY.md` + `docs/`) at its natural locations - that
does not move here.

## Layout (target)

```
canonify/
  SPEC.md            - feature checklist + release definition-of-done (the rubric)
  CHANGELOG.md       - semver history
  README.md          - this file
  templates/         - seed a new repo
    CANONIFY.md.template
    doc-style.md
  skills/            - plan, build, commit, create-canon, kickoff, doctor (all generic, done)
  LICENSE            - MIT
  .claude-plugin/    - plugin.json + marketplace.json (done)
```
