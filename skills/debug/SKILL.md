---
name: debug
description: Start a debug session - load the CANONIFY.md index, route the error you paste (text, screenshot, or a log-file reference) to the canon for that area so Claude debugs with the right context, then after the fix suggest capturing the footgun via /canonify:update-canon or /canonify:create-canon. Run when the user types /canonify:debug, or says "help me debug this error", "I'm getting this exception", "here's an error log / stack trace", "why is this failing".
---

# Debug - route an error to its canon, then feed the fix back

Canonify's **reactive** gate - the twin of `/canonify:build`. Build routes a *task* forward to its
canons; Debug routes a *failure* to the canon for the area it lives in, so Claude diagnoses with the
documented patterns and gotchas already in context instead of grepping the project cold. Then, once
the bug is fixed, it closes the loop: the lesson goes back into the canon so the same class of bug
cannot recur.

**Canonify does not do the debugging.** This gate front-loads the right context and, at the end,
points you at the capture command - the diagnosis and fix in between are normal work.

## How to run

1. **Load the index, set intent.** Read `CANONIFY.md` (the manifest of one-line canon summaries). If
   the user typed `/canonify:debug` alone, acknowledge and wait for the error; if they included it,
   go on.

2. **Take the error in.** Accept any form: pasted text, a **screenshot** (read it via vision and
   transcribe the message + stack), or a referenced **log file** (read it). Pull out the routing
   signal: the **file paths + line numbers** in the stack frames, the **exception / error type**, and
   the **symbols / methods** named.

3. **Route the error against the index.** Walk the `CANONIFY.md` summaries and ask which canon's area
   the error falls in. A stack trace is a *precise* routing signal - it names exact files and symbols,
   a stronger match than a prose task. If a canon's area matches, **load it in full**, paying special
   attention to its **Gotchas** section (the known footguns for that area). If nothing matches,
   proceed on breadth alone and say so - do not shoehorn the error into an unrelated canon.

4. **Debug normally.** With the right canon (and its gotchas) in context, find the cause and
   propose/apply the fix in the user's usual flow. This step is ordinary debugging - the gate's value
   is the context it loaded around it.

5. **Close the loop - suggest, never write.** Judge whether the bug carries a *durable, reusable*
   lesson (a footgun worth documenting), then point at the right command:
   - **a reusable footgun + a canon was loaded** -> "worth a gotcha in `<canon>` - run
     `/canonify:update-canon`", and state the rule/gotcha you would add.
   - **a reusable footgun + no canon covers the area** -> "this area is undocumented - run
     `/canonify:create-canon`".
   - **the loaded canon ALREADY warns about this** -> surface it: the gotcha exists but the bug
     happened anyway, so either it needs to be sharper (`/canonify:update-canon` to tighten it) or
     `/canonify:build` never loaded that canon for the original task. This is feedback on the canon,
     not a new entry.
   - **a one-off slip with no general lesson** -> say so and suggest nothing. Don't nag.

   The user confirms by running the suggested command. The capture itself is owned by
   `/canonify:update-canon` and `/canonify:create-canon` - this gate never writes a canon.

## Don't do

- **Don't write or edit a canon here.** Debug routes and suggests; `/canonify:update-canon` (add /
  revise) and `/canonify:create-canon` (new) own the capture. Zero duplication.
- **Don't claim Canonify fixed the bug.** It front-loads context and closes the loop; the diagnosis
  and fix are normal work.
- **Don't capture noise.** No durable, reusable lesson -> no suggestion. A typo teaches the canon
  nothing.
- **Don't force a match.** If the error's area isn't in any canon, debug on breadth and say nothing
  matched - never route it into an unrelated canon.

## Output template

```
## /canonify:debug

Index loaded: CANONIFY.md (<N> canons)
Error: <the exception / message + where it surfaced>
Routing signal: <files / symbols / type pulled from the error>

Routed to: <canon>.md  (matched summary: "<line>")
           (or) no canon matches this area - debugging on breadth only

[diagnosis + fix - normal work]

Fix: <one-line what changed> @ `path:line`

Capture: <update | create | sharpen | none>
- update   Worth a gotcha in <canon>.md: "<the rule>". Run /canonify:update-canon.
- create   This area has no canon. Run /canonify:create-canon.
- sharpen  <canon>.md already warns about this - it happened anyway. Sharpen it
           (/canonify:update-canon), or it wasn't loaded at build time.
- none     One-off slip, no reusable lesson - nothing to capture.
```
