# Changelog

All notable changes to Canonify. Format follows Keep a Changelog; versioning is semver.
Canonify is pre-1.0 and incubating inside the `ceosite` repo (its reference implementation).

## [Unreleased]

## [0.2.0] - 2026-06-16

### Added
- **Doctor plugin-freshness check** - `/canonify:doctor` gains an optional, best-effort Check 3:
  compare the *installed* Canonify plugin's effective version (from `~/.claude/plugins/installed_plugins.json`)
  against the source's effective version (the `version` in `.claude-plugin/plugin.json` on the
  default branch, falling back to the commit SHA for a SHA-tracked plugin) and flag when the project
  is behind, with the `/plugin update ... + /reload-plugins` remedy. Compares versions, not raw
  commits, so an unreleased commit does not false-flag. Skips cleanly for a local/dev checkout (no
  install record) or when offline; report-only, never self-updates. Lets a scheduled sweep also
  catch an out-of-date toolchain, not just canon drift.
- **Self-identifying Doctor sweeps** - `/canonify:doctor` now opens its report with a
  `CANONIFY DOCTOR - found N items to review` (or `- clean`) banner, and any follow-up task it spins
  off is prefixed `Canonify:` and opens with `CANONIFY DOCTOR flagged this:`. A scheduled sweep lands
  as a standalone message/task, so the first line names Canonify and the outcome at a glance.
- **`/canonify:update-canon` gate** - the maintenance gate: capture a mid-coding discovery into an
  *existing* canon. Routes off `CANONIFY.md` (same selector as build/commit), dedup-checks the update
  against the canon (already covered -> say where + offer a sharpen; partial/stale -> revise;
  uncovered -> add in house-style, cited `file:line`), and keeps `verified` honest via the
  harden-vs-correct fork (append knowledge -> don't bump; re-sync against code -> bump to HEAD).
  Hands off to `/canonify:create-canon` when no canon owns the area. Brings the gate count to seven.
- **Commit-gate feedback edge** - `/canonify:commit` now closes with an offer to capture canon drift
  via `/canonify:update-canon` when a diff looks to have outrun a canon (intentional divergence or a
  surfaced footgun), rather than filing it as a violation. An offer, never an auto-edit.
- `/canonify` incubator folder: `SPEC.md` (feature + release rubric), `README.md`, this changelog,
  and `templates/` (`CANONIFY.md.template`, `doc-style.md`).
- `/create-canon` gate (in ceosite): author a new **canon** (one canonical-pattern `.md`) from a
  file/service/design element - index the code, ask only the gaps, write house-style, and register
  it in `CANONIFY.md` + the category `INDEX`. The command name is the vocabulary: each pattern doc
  is a "canon." Codifies the workflow used to author the existing service + design canons.
- **Packaged `canonify/` as a Claude Code plugin** (`.claude-plugin/plugin.json` + `marketplace.json`,
  `LICENSE`): genericized all six gate skills into `canonify/skills/` (ceosite specifics stripped),
  namespaced as `canonify:plan` / `build` / `commit` / `create-canon` / `kickoff` / `doctor`, and
  authored the generic `kickoff` (onboarding wizard) and `doctor` (reference-integrity + git-derived
  staleness) gates. Templates referenced via `${CLAUDE_PLUGIN_ROOT}`.

### Context (in ceosite, the reference implementation)
- Three gates live and working: `/canonify-plan`, `/canonify-build`, `/canonify-commit`,
  routing through a root `CANONIFY.md` manifest over a `docs/` corpus
  (`composite/`, `services/`, `design/`, `frontend/`).
- Renamed from the working name "Canon" to **Canonify** (the verb) to avoid the camera-brand and
  "cannon" collisions. Manifest is `CANONIFY.md`; gates are `canonify-*`.

### Planned (see SPEC.md)
- Extract `canonify/` to its own git repo + push to GitHub; add the repo URL to `plugin.json` /
  `marketplace.json`.
- Prove it on a second project (different stack) - the release gate.
- The eval set (sample task -> expected implicated canons) via the skill-creator harness.
