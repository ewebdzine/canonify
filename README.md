# Canonify

> Canonical-pattern docs your AI assistant routes to at every lifecycle gate.

![Canonify gate flow](assets/canonify-flow.svg)

**Canonify** gives an AI coding assistant the one thing it lacks in a real codebase: a memory of how
*this* project already does things. You capture each pattern as a **canon** - a short markdown doc
with `file:line` references - a `CANONIFY.md` manifest indexes them, and six skills route to the
right canon at the right moment. The assistant follows your conventions instead of reinventing them.

It is a [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) plugin. **Language-agnostic:**
the gates match prose, not code, so the same skills work on a .NET, Python, Node, or Go repo - only
each repo's canons differ.

## The problem

Drop an AI assistant into a mature codebase and it confidently rebuilds what already exists: a second
HTTP client, a config pattern you abandoned, a form that ignores your validation conventions. Not
because it is wrong - because it cannot see what is *canonical*. Canonify makes that knowledge
explicit and routable.

## Concepts

- **A canon** - one pattern doc (`.md`), every claim cited to `file:line` (e.g. how you call Twilio,
  how you build a data table, how a particular feature works).
- **`CANONIFY.md`** - the manifest: a one-line summary of every canon. The breadth map the gates
  route through.
- **The gates** - six skills that read or maintain the canon at each point in the lifecycle.

## The gates

| Gate | Command | What it does |
|---|---|---|
| Kickoff | `/canonify:kickoff` | onboard a fresh repo: survey -> write `CANONIFY.md` + a `docs/` skeleton -> optionally scan-and-seed canons |
| Create-canon | `/canonify:create-canon` | author a new canon from a file / service / element and register it |
| Plan | `/canonify:plan` | breadth orientation before planning (reuse-vs-build triage) |
| Build | `/canonify:build` | route a task to the canons it implicates, load them, follow the pattern |
| Commit | `/canonify:commit` | audit a diff against the rules the canons document |
| Doctor | `/canonify:doctor` | health check: reference integrity + git-derived staleness |

## Quickstart

```sh
# add this repo as a plugin marketplace (a Git URL, or a local path)
/plugin marketplace add <repo-url>
/plugin install canonify@canonify

# stand Canonify up in your project
/canonify:kickoff
```

`kickoff` asks a couple of questions, writes your `CANONIFY.md` + `docs/` skeleton, and (optionally)
scans the codebase to draft a first set of canons. From there, `/canonify:create-canon` adds more, and
the lifecycle gates route to them.

## How staleness stays honest

Each canon carries a `verified: <git-sha>` marker - the commit at which it was last confirmed against
the code. `/canonify:doctor` runs `git log <verified>..HEAD` over every file a canon references;
anything that moved since gets flagged for review. No hand-maintained date logs - git holds the dates,
and `/canonify:commit` refreshes the marker when it re-confirms a canon.

## Status

Pre-1.0. All six gates are built and project-agnostic. The reference implementation is a large
production .NET / Composite C1 codebase (~30 canons across platform, integrations, services, and UI).
Next: prove it on a second stack, then a v1.0 release. See [SPEC.md](SPEC.md) for the feature and
release checklist, and [CHANGELOG.md](CHANGELOG.md) for history.

## License

[MIT](LICENSE).
