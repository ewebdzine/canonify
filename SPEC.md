# Canonify - SPEC (feature checklist + release definition-of-done)

The rubric Canonify is checked against before it is extracted to its own repo and published.
Update status as pieces land. Status legend: **Done** / **Partial** / **Planned**.

> Context: Canonify generalizes ceosite's three "before-you-*" skills into a portable system.
> `ceosite` is the reference implementation; this spec is what "complete and publishable" means.

---

## 1. Feature checklist (what "complete Canonify" is)

| Capability | What it is | Status |
|---|---|---|
| **Plan gate** | `/canonify-plan` - breadth orientation, reuse-vs-build triage | Done (ceosite) |
| **Build gate** | `/canonify-build` - route task -> implicated docs -> canonical pattern | Done (ceosite) |
| **Commit gate** | `/canonify-commit` - audit diff against the same canons; offer `/canonify:update-canon` when a diff looks to have outrun a canon (the feedback edge); also (planned, with Doctor) refresh `verified` SHAs and flag canons whose referenced files the diff touched (incremental staleness) | Audit Done (ceosite); drift-offer Done; staleness duty Planned |
| **Create-canon gate** | `/create-canon` - author a new canon (one pattern doc) from a file/service/element: index the code, ask only the gaps, write house-style, register in `CANONIFY.md` + the category `INDEX`. The command name teaches the "canon" vocabulary | Done (ceosite) |
| **Update-canon gate** | `/canonify:update-canon` - capture a mid-coding discovery into an *existing* canon: route to it, dedup-check (already covered -> say where; partial/stale -> revise; uncovered -> add), then keep registration + the `verified` SHA honest (harden = don't bump; correct/re-sync = bump). Complements create-canon (new canon) and is offered by the commit gate when a diff looks to have outrun a canon | Done (generic in `canonify/skills/update-canon`) |
| **Import-canon gate** | `/canonify:import-canon` - bulk-seed THIS repo's canons from another project's list: use each source canon as a skeleton, re-prove every claim against this repo's code (flag divergence where it has no equivalent), dedup against existing canons, register, and stamp `verified` to this repo's HEAD. It is create-canon over a list, not a copy - for sharing platform canons (e.g. Composite) across several repos | Done (generic in `canonify/skills/import-canon`) |
| **Kickoff gate** | `/kickoff` - one-time onboarding wizard for a fresh repo: a short survey (scan-and-seed? scheduled Doctor?), then write `CANONIFY.md` + the `docs/` skeleton and optionally bulk-seed canons by scanning. Design in section 3b | Done (generic in `canonify/skills/kickoff`) |
| **Doctor / Lint gate** | `/canonify-doctor` - two checks per canon: **reference integrity** (every cited path exists, every canon registered in `CANONIFY.md` + `INDEX`) and **staleness** (git-derived: flag any referenced file changed since the canon's `verified` SHA). Design + triggers in section 3a | Done (generic in `canonify/skills/doctor`) |
| **Manifest convention** | `CANONIFY.md` at root + per-domain `INDEX.md` | Done (ceosite) |
| **Doc house-style** | the section skeleton + rules (cite file:line, ASCII, terse, no theory padding) | Done (exemplars: `docs/services/activecampaign.md`, `docs/design/charts.md`); captured in `templates/doc-style.md` |
| **Routing logic** | semantic summary matching + Implicated/Skipped table (not hardcoded conditionals) | Done (ceosite) |
| **Mockup-recipe convention** | design docs carry a token-accurate static snippet so mockups match production | Done (ceosite `docs/design/`) |
| **Framework <-> content split** | portable skills vs per-repo `CANONIFY.md` + docs | Done - all 8 gates generic + namespaced in `canonify/skills/`; content (`CANONIFY.md` + docs) stays per-repo |
| **Back-compat aliases** | BYP / BYB / BYC still trigger the gates | Done |
| **Language-agnostic** | proven on more than one language/stack | Partial - .NET / Composite C1 only so far |

## 2. OSS-readiness checklist (what GitHub needs)

| Item | Status |
|---|---|
| `README.md` (pitch + quickstart) | Partial - placeholder exists |
| `LICENSE` | Done (MIT) |
| `CONTRIBUTING.md` | Planned |
| `CHANGELOG.md` + semver | Started |
| Generic skills lifted into `canonify/skills/` (no project specifics) | Done - all 8 gates (plan/build/commit/create-canon/update-canon/import-canon/kickoff/doctor) generic + namespaced |
| `templates/` (manifest + doc-style) usable to seed a new repo | Done (initial) |
| **>= 2 example repos** - one .NET (ceosite-derived), one Python | Planned (1 of 2: ceosite) |
| Plugin packaging: `plugin.json` + marketplace metadata | Partial - `.claude-plugin/{plugin,marketplace}.json` done; gates namespaced `canonify:plan/build/commit/create-canon`; needs extraction to own repo + GitHub URL |
| Quickstart that takes a cold repo to a working canon in <10 min | Planned |

## 3. Confidence - how we know it works

- **Doctor conformance check** (`/canonify-doctor`): the same tool that gates a release runs ongoing
  in each adopting repo - reference integrity + the git-derived **staleness guard**. Full design in
  section 3a below.
- **Eval set**: a table of `sample task -> expected implicated docs`, run through the
  `skill-creator` eval harness so a model update cannot silently break routing. Start with ~10
  cases drawn from real ceosite tasks. Status: Planned.

## 3a. Doctor gate - design (the two checks)

`/canonify-doctor` runs two checks over every canon and reports a worklist - it does not auto-fix.

**1. Reference integrity (structural).**
- Every `file:line` path a canon cites still exists.
- The canon is registered: a one-line block in `CANONIFY.md` AND a row in its category `INDEX.md`
  (no orphan canons, no dangling manifest entries).

**2. Staleness (temporal) - the key check.**
Each canon carries one `verified: <git-sha>` frontmatter marker: the commit at which a human last
confirmed the canon matches the code. Doctor never hand-logs dates - it asks git:

```
refs     = file paths the canon cites (file:line cites + the References section)
baseline = the canon's `verified:` SHA      (fall back to the canon's own last-commit SHA if absent)
for each ref:
    if `git log <baseline>..HEAD -- <ref>` is non-empty:
        FLAG: "<canon> references <ref>, changed in N commits since verified (<dates>) - review"
```

Maintenance is **one SHA, set by the gates**: `/create-canon` stamps `verified` at authoring;
`/canonify-commit` bumps it to HEAD when it re-audits a canon's area and confirms it still holds.
Doctor only reports.

Why this beats a hand-written per-file date log: git already holds every file's edit date (and the
canon's), so a date log would be redundant and would itself drift. The single `verified` SHA also
distinguishes "canon edited" (e.g. a typo) from "canon re-confirmed against code" - only the latter
clears a staleness flag.

Notes:
- **File-level for v1.** A change far from the cited lines still flags (a benign false positive) -
  it is a "go look" list, not a failure. Later refinement: check whether the cited line ranges moved.
- **Git for all dates, never file mtime** (checkout resets mtime; unreliable on `/mnt/c`). On this
  repo, shell to `git.exe` (the `/mnt/c` git rule in `CLAUDE.md`), read-only `git log`.
- This makes Doctor partly deterministic (git) - a candidate for the bundled script/MCP bit (see
  Open questions), with the prose/house-style checks staying in the skill.
- **Backfill:** existing canons predate the `verified` field; stamp each at its current last-commit
  SHA when Doctor is built (or on the next `/canonify-commit` that touches it).

### Triggers

Where the checks fire, matched to where staleness is introduced vs. accumulates:

- **At commit (incremental) - owned by `/canonify-commit`.** The commit gate already routes the diff
  to implicated canons; it gains two duties: (a) if the diff touches a file a canon references but
  does not update that canon, **flag it** for review; (b) when a canon's area is confirmed still
  accurate, **bump its `verified` SHA to HEAD**. So freshness is maintained in the same commit that
  could have staled it. Rides the existing flow - `/ship` already offers `/canonify-commit` as step 1.
  (Semi-automatic: runs when the gate runs.)
- **Periodic full sweep (accumulated) - a scheduled task.** A weekly `/canonify-doctor` full run over
  the whole repo catches drift that slips past the gate (merges, hotfixes, commits made without the
  gate). Reports the stale-canon worklist; does not auto-fix. Set up as a scheduled cloud agent (or a
  repo cron) once Doctor exists.
- **Git hook (optional backstop) - later.** The staleness half is pure git, so it can run warn-only in
  a `pre-commit` / `pre-push` hook with no LLM. Deferred to avoid friction with this repo's existing
  `pre-push` hook; not v1.

Dependency: the commit-gate duty and the scheduled sweep both need the `verified` markers to exist, so
they are wired up when Doctor (and the marker backfill) land - not before.

## 3b. Kickoff gate - design (onboarding wizard)

`/kickoff` sets Canonify up in a fresh repo - run once: drop the `canonify/` folder in the repo root
and run it. It is an interactive wizard, not a silent file-creator.

Flow:
1. **Detect.** Confirm it's a git repo; detect the stack (`.csproj`/`.sln` -> .NET, `pyproject.toml`/
   `requirements.txt` -> Python, `package.json` -> Node, ...) to pre-fill sensible category folders.
   If Canonify is already set up here, do not re-survey - offer reconfigure instead.
2. **Survey** (a few questions; the rest auto-detected):
   - **Auto-seed by scanning?** *Scan & propose* (recommended) - discover the surface (services,
     integrations, key subsystems) and **propose a canon list to confirm** before generating; or
     *start empty*. Never silently generate N canons - propose, confirm, then write + register.
   - **Scheduled Doctor backstop?** *Yes, weekly* (scheduled cloud agent / repo cron) / *no, manual* /
     *later*. (Only acts once Doctor exists; until then, record the preference.)
   - **Confirm stack + category folders** (auto-detected) - a Python repo gets Python-shaped
     categories, not `composite/`. This is what makes Canonify language-agnostic in practice.
3. **Act.** Write `CANONIFY.md` + the `docs/` skeleton + templates. If scan: discover -> propose ->
   confirm -> bulk `/create-canon` over the surface, registering each + stamping `verified`. If
   scheduled: set up the Doctor task.

The scan reuses `/create-canon`'s machinery (index code -> draft house-style -> register), run in bulk
- exactly how ceosite's services + design canons were seeded.

Sequencing: the skeleton + scan can ship as soon as `/kickoff` is built (it leans on `/create-canon`,
which is done); the scheduled-Doctor step lights up when Doctor lands.

## 4. Release gate (the real test)

**Canonify is not "ready to publish" until it has run cleanly in a second, different project -
ideally a different language.** A checklist passing in ceosite alone only proves it works where
it was born. Project #2 is what surfaces the ceosite-specific assumptions still hiding in the
skills. Capture those learnings here before extraction.

- [ ] Proven in ceosite (.NET / Composite C1) - in progress
- [ ] Proven in a second project (different stack)
- [ ] Genericization pass: no ceosite identifiers in `canonify/skills/`
- [ ] Doctor passes clean in both example repos
- [ ] Eval set green

## 5. v1.0 definition-of-done

Canonify ships v1.0 when: all gates exist (plan/build/commit, create-canon, update-canon, import-canon, kickoff, doctor); the OSS
checklist is complete; it is proven on two stacks; the eval set is green; and a new repo can go
from zero to a working canon via `/kickoff` + the templates.

## 6. Open questions

- Distribution: plugin-only, or also a thin MCP server for the deterministic bits (kickoff scan,
  doctor)? (Leaning: plugin primary; MCP only if the deterministic ops benefit from real code.)
- Does `doctor` live as a skill (prompt-driven) or a script/MCP tool (deterministic)?
- How opinionated should the doc house-style be enforced vs advised?
