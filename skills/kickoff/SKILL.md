---
name: kickoff
description: One-time onboarding wizard that sets Canonify up in a fresh repo - a short survey (scan-and-seed? scheduled doctor?), then write CANONIFY.md + the docs/ skeleton and optionally bulk-seed canons by scanning the codebase. Run when the user types /canonify:kickoff, or says "set up Canonify", "kick off Canonify", "onboard this repo", "initialize the canon", "start canonify here".
---

# Kickoff - set Canonify up in a repo

The onboarding gate. Run it ONCE in a fresh repo to stand up Canonify: a routing manifest
(`CANONIFY.md`), a `docs/` corpus of canons, and - if you want - an initial set of canons drafted
from the codebase. It is an interactive wizard, not a silent file-creator.

> Vocabulary: a **canon** is one canonical-pattern doc; `CANONIFY.md` is the manifest the gates
> (`/canonify:plan` / `:build` / `:commit`) route through. This gate creates both. The templates ship
> with the plugin at `${CLAUDE_PLUGIN_ROOT}/templates/`.

## How to run

1. **Detect.**
   - Confirm this is a git repo (Canonify's staleness checks rely on git).
   - **Already set up?** If `CANONIFY.md` exists at the repo root, do NOT re-run the survey - report
     what is there and offer to *reconfigure* (add categories, seed more canons) instead.
   - Detect the stack to pre-fill sensible category names: `*.csproj`/`*.sln` -> .NET; `pyproject.toml`/
     `requirements.txt` -> Python; `package.json` -> Node; `go.mod` -> Go; and so on. Categories are
     the `docs/<area>/` folders (e.g. integrations, services, ui) - the project's own, not a fixed set.

2. **Survey** (ask a few; auto-detect the rest - keep it short):
   - **Auto-seed by scanning?**
     - *Scan and propose* (recommended): discover the surface (third-party integrations, internal
       services, key subsystems, UI patterns) and **propose a list of canons to confirm** before
       writing any.
     - *Start empty*: just the skeleton; author canons later with `/canonify:create-canon`.
   - **Scheduled doctor backstop?** *Yes (weekly)* / *no, run it manually* / *later*. A periodic
     `/canonify:doctor` sweep catches drift that accumulates between commits.
   - **Confirm the stack + category folders** you detected.

3. **Act.**
   - Write `CANONIFY.md` at the repo root from `${CLAUDE_PLUGIN_ROOT}/templates/CANONIFY.md.template`,
     filling in the detected categories.
   - Create the `docs/<category>/` skeleton with a short `INDEX.md` per category.
   - Optionally copy `${CLAUDE_PLUGIN_ROOT}/templates/doc-style.md` into the repo (e.g.
     `docs/doc-style.md`) so authors have the house-style locally.
   - **If scanning:** discover -> propose the canon list -> on confirm, run `/canonify:create-canon`
     in bulk over each target (index the code, draft in the house-style, register in `CANONIFY.md` +
     the category `INDEX`, stamp `verified`). Show progress; never silently generate a large batch.
   - **If scheduled doctor:** set up a periodic `/canonify:doctor` run via the host's scheduling
     (a scheduled agent, or a repo cron) at the chosen cadence.

4. **Report.** What was created (`CANONIFY.md`, the category skeleton, any seeded canons + their
   `verified` markers), and the next steps: author more with `/canonify:create-canon`, orient before
   planning with `/canonify:plan`.

## Don't do

- **Don't re-run on an already-set-up repo.** Detect `CANONIFY.md` and offer reconfigure instead.
- **Don't silently mass-generate canons.** Propose the list, get confirmation, then write + register.
- **Don't impose a fixed category set.** Use the categories that fit THIS repo's stack and domain.
- **Don't invent canon content during the scan.** Each seeded canon goes through `/canonify:create-canon`
  (evidence-based, every claim `file:line`-cited), not a guess.

## Output template

```
## /canonify:kickoff - <repo>

Detected: <stack>, git repo. Categories: <area-a>, <area-b>, ...

Survey:
- Auto-seed: <scan-and-propose | empty>
- Scheduled doctor: <weekly | manual | later>

Created:
- CANONIFY.md (manifest, <N> category blocks)
- docs/<area>/INDEX.md  (x N)
[- seeded canons: <doc-a>.md, <doc-b>.md, ... (each registered + verified)]

Next: /canonify:create-canon to add canons; /canonify:plan to orient before planning.
```
