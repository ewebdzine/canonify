---
name: doctor
description: Health-check the canon - verify every canon's references still exist and are registered, flag canons whose referenced source files have changed since they were last verified (git-derived staleness), and best-effort check whether the installed Canonify plugin is behind its GitHub source. Reports a worklist; does not auto-fix. Run when the user types /canonify:doctor, or says "check the canon", "are the docs stale", "audit the canons", "canonify health check", "lint the canon", "is canonify up to date".
---

# Doctor - keep the canon honest

The health gate. It runs two checks over every canon - plus an optional third check on whether the
installed Canonify plugin itself is behind its GitHub source - and reports a worklist; it does not
auto-fix. Run it as a full-repo sweep (manually, or on a schedule set by `/canonify:kickoff`); the
incremental version of the staleness check also runs inside `/canonify:commit` on the diff.

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

4. **Check 3 - plugin freshness (optional, best-effort - about the tooling, not the canons).** Also
   surface when the *installed* Canonify plugin is behind its GitHub source, so a scheduled sweep
   catches an out-of-date toolchain too. Skip cleanly when it does not apply:
   - **Installed version:** read Claude Code's plugin state (`~/.claude/plugins/installed_plugins.json`)
     for this plugin's entry (`canonify@<marketplace>`) and note its recorded `version` (and
     `gitCommitSha`). If there is **no install record** - the plugin is loaded locally for development
     (e.g. you are running inside its own source repo) - **SKIP**; a local checkout is its own latest.
   - **Source version (resolve it the way Claude Code does):** a plugin's effective version is the
     `version` in its `.claude-plugin/plugin.json`, else the marketplace entry's `version`, else the
     commit SHA. Resolve the marketplace's git URL (from `~/.claude/plugins/known_marketplaces.json`
     or the plugin's `marketplace.json` source - do not hardcode a repo), then read the source's
     current effective version: fetch `.claude-plugin/plugin.json` `version` from the default branch
     (raw file fetch, or `git ls-remote <url> HEAD` for the tip SHA when no `version` is published).
     If offline or unresolvable, **SKIP** and say so.
   - **Compare effective versions, not raw commits:** flag only when the source's effective version
     differs from the installed one -
     `Canonify plugin behind source: installed v<installed>, latest v<latest> - run /plugin update canonify@<marketplace>, then /reload-plugins`.
     Comparing versions (not raw `HEAD`) means an unreleased commit - a doc tweak with no version
     bump - does NOT false-flag.
   - **Caveat to state:** `/plugin update` only pulls when the published `version` moves (or, for a
     SHA-tracked plugin with no `version` field, on any new commit). If a maintainer ships changes
     without bumping the published `version`, neither this check nor `/plugin update` will see them -
     that is maintainer discipline, not a consumer problem. Never run the update yourself - report only.

5. **Open with a self-identifying banner, then report the worklist.** A scheduled sweep arrives as a
   standalone message with no surrounding context, so the FIRST line must name Canonify and say
   whether anything was found - e.g. `CANONIFY DOCTOR - found 2 items to review` (or, when clean,
   `CANONIFY DOCTOR - clean, nothing to review`). Then group findings by canon: the canon, the issue
   (broken ref / unregistered / orphan / dangling / stale ref), the `file:line` or commit evidence,
   and a one-line suggested action. End with a summary line: N canons checked, B reference issues,
   S staleness flags, plus the plugin-freshness verdict (current / behind / n-a).

6. **Self-identify any follow-up task too.** If a finding warrants its own working session (a real
   fix, not just a "go look"), and you spin it off as a task, prefix the task title with `Canonify:`
   and open the task prompt with the same `CANONIFY DOCTOR flagged this:` line - so a task that lands
   out of context (e.g. the Facebook tokens-in-logs case) is recognizable at a glance as a Canonify
   finding, exactly like the sweep message itself.

## Don't do

- **Don't auto-fix.** Report only; the human (or `/canonify:commit`) decides, updates the canon, and
  bumps its `verified` SHA.
- **Don't hand-maintain dates.** The `verified` SHA is the only stored state; everything else comes
  from git at runtime.
- **Don't fail on a staleness flag.** A referenced file changing does not prove the canon is wrong -
  it is a "go look" list. (v1 is file-level: a change far from the cited lines still flags.)

## Output template

```
CANONIFY DOCTOR - <found <N> item(s) to review | clean, nothing to review>

/canonify:doctor - <repo>, <N> canons checked

### Reference integrity
- [OK] all <N> canons registered and cited paths exist
  (or, per issue:)
- [BROKEN]   <canon>.md cites `path:line` - file not found
- [ORPHAN]   docs/<area>/<canon>.md - no CANONIFY.md entry
- [DANGLING] CANONIFY.md lists <canon>.md - file missing

### Staleness (verified..HEAD)
- [STALE]     <canon>.md - references `path` (changed in K commits since verified <date>) - review
- [NO-MARKER] <canon>.md - no `verified` SHA; used last-commit fallback

### Plugin freshness (installed vs source)
- [OK]      canonify@<marketplace> v<installed> matches source
- [BEHIND]  canonify@<marketplace> installed v<installed>, source v<latest> - /plugin update canonify@<marketplace>, then /reload-plugins
- [N/A]     loaded locally (dev) | offline - skipped

### Summary
<N> checked, <B> reference issues, <S> staleness flags, plugin <current | behind | n-a>. <Clean | Address the above>.
```
