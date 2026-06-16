# Changelog

All notable changes to Canonify. Format follows Keep a Changelog; versioning is semver.
Canonify is pre-1.0 and incubating inside the `ceosite` repo (its reference implementation).

## [Unreleased]

### Added
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
