---
name: update-canon
description: Capture a mid-coding discovery into an EXISTING canon - describe the update and Canonify routes to the right canon, checks whether it's already covered (and whether a revision is needed) or adds it in house-style, then keeps registration + the verified SHA honest. Run when the user types /canonify:update-canon, or says "add this to the canon", "the FormBuilder canon is missing/wrong about X", "I discovered Y, capture it", "update the Z canon", "this should be in the canon", "revise the canon for X".
---

# Update a canon

Canonify's **maintenance** gate. Where `/canonify:create-canon` authors a *new* canon from a
file/service/element, this gate **edits an existing one**: you discovered something while coding -
a footgun, a sharper rule, a pattern that drifted - and you want it captured *now*, in flow, without
hand-editing markdown. Describe the update; this gate finds the right canon, decides whether the
knowledge is already there, and either tells you it's covered or adds it cleanly.

It routes off `CANONIFY.md` exactly like build/commit (match the summary lines), and writes to the
same house-style as create-canon (`${CLAUDE_PLUGIN_ROOT}/templates/doc-style.md`): cite every claim
`file:line`, ASCII punctuation, terse and project-specific.

## How to run

1. **Get the update.** The user describes what they learned. If they didn't say it, ask once: "what
   did you discover, and which area does it touch?" Don't guess the content.

2. **Route to the canon.** Find which existing canon owns this. Same selector as build/commit:
   read `CANONIFY.md`, match the update against the one-line summaries. Usually it's inferable from
   the file in play (working in form code -> the FormBuilder canon) or named outright. Confirm the
   target in one line. If two canons could own it, ask which. **If NO canon covers the area at all,
   stop and hand off:** this gate edits existing canons - say so and suggest `/canonify:create-canon`.

3. **Load the canon + re-read the cited code.** Read the target canon in full, and re-read the
   specific code it cites for the affected section. You need both: the doc (to dedup against) and the
   live code (so any addition is cited to a real `file:line` and any "is it still true" check is
   against reality, not just the old prose).

4. **Triage the update against the canon - the dedup check.** This is the core. Decide which case:
   - **Already covered, still accurate** -> Don't edit. Tell the user where it's documented (section
     + the `file:line` it cites). Offer a *sharpen* only if the wording is genuinely weak (buried,
     vague, easy to miss); otherwise say it's covered and stop.
   - **Partially covered or stale** -> Propose a revision: tighten the rule, add the missed nuance,
     or correct what no longer matches the code. Show before -> after for the changed lines.
   - **Not covered** -> Draft the addition in house-style, cited to `file:line`, placed in the right
     section by kind: a footgun -> **Gotchas**; a new or sharper rule -> **The pattern**; a new call
     site -> **Recipes**; a config key -> **Configuration / API surface**.

5. **Classify the edit so `verified` stays honest** (the key mechanic - see `SPEC.md` 3a):
   - **Harden / append** - the canon *gains knowledge* but the code did not change (a newly noticed
     gotcha, a rule made clearer). **Do NOT bump `verified`.** You added to the doc; you did not
     re-confirm the whole canon against new code, so the staleness clock must not reset.
   - **Correct / re-sync** - the canon was *wrong vs current code* and you re-cited the affected part
     against HEAD. **Bump `verified` to HEAD** for that re-confirmation (shell to git per the repo's
     git rule; read-only `git rev-parse HEAD`).
   - If unsure which, ask one line. Default to **harden** (the safer choice - it never falsely clears
     a drift flag).

6. **Keep registration in sync.** A small gotcha rarely changes scope. But if the update *widens
   what the canon covers* (a new area, type, or surface), update the one-line summary in BOTH
   `CANONIFY.md` and the category `INDEX.md` so routing still matches - the summary is what the gates
   match on. Don't silently widen scope without touching the summary.

7. **Report.** State the triage outcome plainly: already-covered (and where) / revised (before->after)
   / added (the new lines + section). Note whether `verified` was bumped and why, and any
   registration-summary change.

## Don't do

- **Don't create a second canon.** If the area is genuinely uncovered, hand off to
  `/canonify:create-canon` - don't fork a near-duplicate or graft an unrelated section on.
- **Don't add an uncited claim.** Same rule as authoring: no `file:line`, no line. If you can't point
  at it in the code, ask or omit.
- **Don't bump `verified` on a harden/append edit.** Re-stamping without re-confirming against code
  falsely clears a staleness flag and defeats the Doctor gate.
- **Don't pad.** If it's already covered, say so and stop - don't reword a healthy section to look busy.
- **Don't widen scope silently.** New coverage means the `CANONIFY.md` + `INDEX` summary changes too.

## Output template

```
## /canonify:update-canon - <the discovery, one line>

Routed to: docs/<category>/<topic>.md  (matched CANONIFY.md summary: "<the line>")
Read: <canon> + <code re-read, file:line>

Triage: <already-covered | revise | add>

[already-covered]
Already documented: "<section>" -> cites `path:line`. No change needed.
(Optional sharpen offered: <one line>, declined unless you want it.)

[revise]
Revised "<section>":
  - before: <old line>
  + after:  <new line, cited path:line>

[add]
Added to "<section>":
  + <the new line(s), cited path:line>

verified: <unchanged (harden - code didn't change) | bumped to <sha> (re-synced against code)>
Registration: <unchanged | CANONIFY.md + INDEX summary updated - new scope: <what>>
```
