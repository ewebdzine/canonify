---
name: doctor
description: Health-check the canon - verify every canon's references still exist and are registered, and flag canons whose referenced source files have changed since they were last verified (git-derived staleness). Reports a worklist; does not auto-fix. Run when the user types /canonify:doctor, or says "check the canon", "are the docs stale", "audit the canons", "canonify health check", "lint the canon".
---

# Doctor - keep the canon honest

The health gate. It runs two checks over every canon and reports a worklist - it does not auto-fix.
Run it as a full-repo sweep (manually, or on a schedule set by `/canonify:kickoff`); the incremental
version of the staleness check also runs inside `/canonify:commit` on the diff.

> A **canon** is one canonical-pattern doc; `CANONIFY.md` indexes them. This gate checks that the
> canons still point at real code (reference integrity) and still match it (staleness).

## How to run

1. **Load the manifest.** Read `CANONIFY.md` and enumerate every canon it lists, plus each category
   `docs/<area>/INDEX.md`.

2. **Check 1 - reference integrity (structural).** For each canon:
   - Every `file:line` path it cites still exists in the repo (flag broken paths).
   - It is **registered**: a one-line block in `CANONIFY.md` AND a row in its category `INDEX.md`.
   - No **orphans** (a canon file with no manifest entry) and no **dangling entries** (a manifest line
     pointing at a canon file that no longer exists).

3. **Check 2 - staleness (temporal).** Git holds the dates - do not hand-log them. For each canon:
   - `baseline` = the canon's `verified: <git-sha>` frontmatter marker (the commit at which it was
     last confirmed against the code). If absent, fall back to the canon's own last-commit SHA and
     flag it as unmarked.
   - `refs` = the source files the canon cites (its `file:line` references + its References section).
   - For each `ref`: run `git log <baseline>..HEAD -- <ref>`. If non-empty, **flag** - the canon
     references code that moved since it was verified.
   - Use **git, never file mtime** (checkout resets mtime). Use the platform git the repo requires
     (read-only `git log`); on a Windows working tree, the Windows git, not a WSL git.

4. **Report a worklist.** Group by canon. For each finding: the canon, the issue (broken ref /
   unregistered / orphan / dangling / stale ref), the `file:line` or commit evidence, and a one-line
   suggested action. End with a summary line: N canons checked, B reference issues, S staleness flags.

## Don't do

- **Don't auto-fix.** Report only; the human (or `/canonify:commit`) decides, updates the canon, and
  bumps its `verified` SHA.
- **Don't hand-maintain dates.** The `verified` SHA is the only stored state; everything else comes
  from git at runtime.
- **Don't fail on a staleness flag.** A referenced file changing does not prove the canon is wrong -
  it is a "go look" list. (v1 is file-level: a change far from the cited lines still flags.)

## Output template

```
## /canonify:doctor - <repo>

Canons checked: <N>

### Reference integrity
- [OK] all <N> canons registered and cited paths exist
  (or, per issue:)
- [BROKEN]   <canon>.md cites `path:line` - file not found
- [ORPHAN]   docs/<area>/<canon>.md - no CANONIFY.md entry
- [DANGLING] CANONIFY.md lists <canon>.md - file missing

### Staleness (verified..HEAD)
- [STALE]     <canon>.md - references `path` (changed in K commits since verified <date>) - review
- [NO-MARKER] <canon>.md - no `verified` SHA; used last-commit fallback

### Summary
<N> checked, <B> reference issues, <S> staleness flags. <Clean | Address the above>.
```
