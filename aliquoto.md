# aliquoto

**Additive dequantizer — the spectrum before the synth.**

A pure additive sine synthesizer built to push additive synthesis toward its monstrous
conclusion. Partials may sit at *any* ratio of the fundamental — integer, fractional,
irrational, prime, or **below** the fundamental (subharmonic). The spectrum is written
as **mathematics** (summations, envelopes, conditionals) rather than drawn or dialed,
and the math is kept visible, not hidden inside an engine.

*aliquoto* = **Aliquot** + **koto**: exact parts of a whole, made playable as a resonant spectral instrument.

*aliquot* = a part contained in a whole an exact number of times (a divisor/multiple);
also the *aliquot strings* of pianos and harps that ring in sympathetic overtones.

Sibling to **Cycla** (`../tabota/cycla_builder.html`): Cycla is a tuning of *meter*
(recursive subdivision); aliquoto is a tuning of *timbre* (the sum of sines). Same
counter-poetics — both work the interstitial positions a standard grid disallows
(12-TET for pitch, integer harmonics for timbre).

---

## Name and aesthetic direction

Working name: **aliquoto** — **Aliquot** plus **koto**. It should feel like an instrument that does not quite exist yet: spectral arithmetic as a plucked/resonant object, not a generic synth plugin. Keep **Aliquot** as the mathematical root, but let the final `o` make it sound playable, physical, and a little uncanny.

Current device identity: The skin should read as a cursed bathypelagic Mathematica notebook running on a TI calculator: passive-matrix, slightly transflective LCD green as the substrate; dark pixel-ink graphics and controls in the foreground; dense symbolic readouts; exacting firmware labels; notebook-like `In[]` / `Out[]` affordances where they help.

Design rules for the light skin:

- **LCD first.** The background is not “white mode”; it is green calculator substrate, sunlit, stained, low-contrast, and a little mineral.
- **Dark ink foreground.** Spectra, waveforms, controls, keys, and table glyphs should feel printed into or rising out of the LCD, not glowing neon.
- **High nerd density.** Prefer small labels, status chips, parser/compiler language, table readouts, exact units, and symbolic notation over decorative explanation.
- **Modern affordances, old device soul.** Keep it usable: clear buttons, hover/focus states, legible controls, responsive layout. The cursed part should come from the math/device logic, not from making it hard to use.
- **Pacific abyss, lightly.** Hint through bathymetric, oceanographic, pressure-depth, sonar, and firmware language. Avoid obvious horror decoration; keep the surface clinical, technical, and wrong by implication.


---

## Philosophy

The raw spectral material of additive synthesis is interesting *in its own right*,
before any synthesizer-ish processing. So:

- **The equation is the interface.** You type spectra; the tool renders the LaTeX,
  plots the envelope, and tabulates every computed partial (the `ƒ math` card).
- **The core stays pure.** No FM, no filters, no unison, no fx. Those belong downstream
  in an external chain. aliquoto only makes sines and sums them.
- **Dequantize.** Arbitrary ratios, subharmonics, inharmonic and irrational series,
  time-varying frequencies, and per-partial drift exist to escape the integer-harmonic lattice most additive
  synths are locked to.

Every voice evaluates live as a summation of dynamic, enveloped sines:

$$y(t) = \sum_{n} e_n(t) \cdot a(n) \cdot \sin\big(2\pi \cdot r_n(t) \cdot f_0 \cdot t + p(n)\big)$$

---

## Quick start

Open `index.html` in a modern browser (Chrome/Edge/Firefox). Click **▶ enable audio**,
then play with the on-screen keys, your QWERTY row (`a s d f …`, `z`/`x` shift octave),
or a MIDI controller. Type a spectrum in the **Y=** box and press **apply** (Ctrl+↵).

No build step, no dependencies to install — fonts and KaTeX load from CDN.

---

## The spectrum grammar

Each non-empty line is one statement. `#` starts a comment. `:` is the field
separator everywhere (hence `iff(...)`, never ternary `?:`).

### `partial` — one sine
```
r : a : p°                     # ratio, amp, phase (a=1, p=0 default)
r : a : p° : A : D : S : R     # + per-partial ADSR (7 fields exactly)
```
`r` may be any constant expression (`phi`, `pow(2,7/12)`) **or contain `t`**
(`t+1` = ratio rises from 1) — an `r(t)` partial is *dynamic* (see semantics).
`a` may NOT contain `t` (error says: use gain/env) — `a` is **a_max**.

### `sum` — a series
```
sum n=1..N : r(n[,t]) : a(n) : p(n) [: A : D : S : R] [where PRED]
```
`N=inf|∞|*` clamps to the **Σ ceiling** control (truncated infinite sum).
Multiple indices `sum n=1..N k=1..M : ...` = product/lattice spectrum.
`where PRED` filters index combinations. Optional 4 ADSR columns after phase.

### `env` — spectral envelope (r-domain)
```
env : f(r)          # multiplies every partial's amp; hz/f0 also in scope
env : f(t,...)      # if it contains t it becomes a gain line (time-domain)
```

### `gain` (aliases `shape`, `ampenv`) — time-domain amplitude
```
gain : f(t,r,hz,f0,a)     # replaces the voice master ADSR for these partials
```

### `adsr` — pitch-mapped envelope shapes
```
adsr : A(r) : D(r) : S(r) : R(r)    # per-partial ADSR as functions of ratio
```

### Variables in scope
`n,k,…` (indices) · `r` (this partial's ratio) · `hz` (= r·X) · `f0` (= X, the
`X pitch Hz` keyfollow reference) · `t` (seconds since note-on; only where noted).

### Functions / constants / operators
`prime fib sin cos tan exp log sqrt pow abs mod clip01 adsr(t,a,d,s,r)` ·
conditionals `iff(c,a,b) odd even step between clamp not` · comparators
`< > <= >= == != && ||` (1/0) · consts `pi e phi tau` · power `^`.

---

## Architecture map (for future sessions)

Single file, `index.html` (~1500 lines). Key state + seams:

- **`buildPartials(text,ceil,f0ref)`** — grammar → `{parts,meta}`. Each partial:
  `{pid, ratio, amp, phase, adsr?, dyn?{r:{expr,vars,vals}}, gains?[]}`.
  **`pid`** is the stable identity: `s{line}:{indexTuple}` (sum-born),
  `l{line}` (literal-born), `a{seq}` (graphically added).
- **`applyGrammar()`** — reparse → prune stale non-`add` OVR entries + SEL →
  `applyOverrides()` (mutates/filters/adds partials from `OVR`) → redraw/readouts.
- **`OVR` Map** (pid → `{add?, removed?, ratio?, amp?, phase?, adsr?, rExpr?, gainExpr?}`)
  — the live graphic-edit layer over the grammar. Survives re-apply; bake commits it.
- **Selection shell** — `TOOL` (`select`/`move`/`place`), `SELMODE`
  (`replace`/`+` include/`-` exclude, Shift/Alt override), `SEL` Set of pids, marquee,
  ring aura; `move` = select-then-drag, axis-locked (y = scale group a_max,
  x = transpose group ratios by common factor); `place` treats `+` as add and
  `-` as delete-one-point (`replace` is idle); Delete/Backspace = remove selection;
  `restore removed` chip undoes removals (they live only in OVR, outside text-undo).
- **`bakeSelectiveGraphic()`** — commits OVR to grammar text: literal-born
  override rewrites its line; removed literal deletes its line; sum-born
  override/removal appends `where !(n==k)` to its sum (composes with existing
  `where` via `&&`); modified/added partials append as literals under
  `# — baked overrides —`; `r(t)`/`gain(t)` overrides can't flatten → kept live
  ("N dynamic override(s) kept live"). No overrides → full flatten.
- **Undo/redo** — grammar-text snapshots only (preset/bake/normalize/import);
  typing uses native textarea undo. OVR state is NOT in the undo stack.
- **Audio** — `WORKLET_SRC` AudioWorklet per voice (phase-exact, per-partial ADSR
  fast path, shared gain exprs, drift, a-rate `f0` param = time-varying pitch);
  oscillator-bank fallback. Voice seam: `startNote(id,hz,vel)` / `bendNote` /
  `stopNote` — every input (piano/QWERTY/MIDI/hex/ribbon/tabota) just produces Hz.
- **Note model = Tabota.** `noteToEvent`/`eventToNote`/`performanceDoc` encode
  voices as realizable Tabota Events (`chronological · frequency`, pitch in Hz,
  glides as `from/to`); spectrum rides in `payload` (aliquoto's paddy). Doc shape:
  `{payload, score:[{frame,events}]}` — **`score`, not `events`, at root.**
- **`.tabota` import** — vendored `tabota-resolve.js` (classic script, stamped
  with source commit; strips the one `export`; loads from `file://`).
  `resolvedToNotes()` plays the determinate fragment; background hash check vs
  `../tabota/tabota-resolve.js` shows a "resolver ✱" chip (notify-only). No MIDI
  anywhere in the Tabota path.
- **Tuning** — `TUNING.edo` n-EDO, `stepToHz(step)=A4·2^((step+edo·oct)/edo)`.
- **Surfaces** — piano (retuned) / isomorphic hex (QWERTY rows z…/a…/q…/1… map
  isomorphically) / continuous ribbon (x=log-Hz drag-glide, no key/MIDI map);
  multitouch, pointerId-keyed.
- **ƒ math modal** — KaTeX live equation, env plot, TI-TABLE, live tabota doc.

Verify caveat: `preview_screenshot` can hang (CSS animation keeps renderer
non-idle) and canvas width can read 0 in headless eval — test via
`preview_eval` + injected `SM` dimensions where pixel-scaled.

---

## Roadmap

### Graphic spectrum editing (next arc — binlod-ported)
1. ~~**Selection shell**~~ — **done.** SPECTRUM toolbar: `select`/`move`/`place` tool chips,
   selMode `replace`/`+`/`-` (+Shift/Alt override), `all`/`none`, undo/redo
   (`↶`/`↷`, Ctrl+Z / Ctrl+Shift+Z outside the textarea — native textarea undo still
   handles free typing). Select tool: click a partial to select it (never moves it);
   click+drag empty space = marquee; selected partials get a ring aura. Move tool:
   click/drag a partial first applies the same selection mode, then immediately moves
   the selected partial/group; click+drag empty space = the same marquee behavior. Each
   partial carries a stable `pid` (grammar line + index tuple) so selection survives
   re-`apply` where the identity still resolves. Undo/redo covers program-driven
   rewrites (preset load, bake, normalize, `.tabota` spectrum-adopt) — plain typing
   uses the textarea's own native undo.
2. ~~**Override panel**~~ — **done.** A card docked under the global Voice/ADSR
   sliders and above drift, shown only
   when something's selected ("nothing selected edits globals" — the sidebar's grammar
   text + Voice/ADSR/drift controls). Fields: `ratio`, `a_max`, `phase°`, ADSR
   `A`/`D`/`S`/`R` (+ a mini envelope graph of the first selected partial's shape),
   `r(t)` override, `gain(t)` override. Multiple selected → mixed values show blank
   (tabota contextual-editor pattern); editing writes to every selected partial.
   Blank a field + commit (Enter/blur) clears just that override, falling back to
   the global formula on the next apply. Editing one ADSR field freezes the other
   three at their *current effective* values (global or already-overridden) rather
   than resetting them. Overridden partials get a small magenta square badge over
   their spectrum dot; the readout reports live override state as added / modified /
   removed counts. Overrides are
   keyed by each partial's `pid` (grammar line + index tuple) in an `OVR` map,
   reapplied after every grammar re-`apply` (and pruned when a pid no longer
   resolves) — so they survive re-eval, and flow straight into today's bake/`partialLine`
   (a fully-overridden partial already bakes as a 7-field literal `r:a:p:A:D:S:R` line).
3. ~~**Bake v2**~~ — **done.** `⤓ bake` is now *selective*: a partial carrying an
   override becomes a literal line; untouched sums stay sums. A literal-sourced
   override rewrites its own line; a sum-sourced override adds a `where !(n==k)`
   exclusion to that sum (composing with any existing `where`) and appends the
   partial as a literal under a `# — baked overrides —` marker (placed after any
   `env`/`adsr`/`gain` lines so those don't re-touch it). With no overrides at all,
   bake still full-flattens as before. Overrides carrying `r(t)`/`gain(t)` can't
   become a static literal, so they're **kept live** and the status notes "N dynamic
   override(s) kept live" — this is where the per-partial-`gain` literal syntax
   decision (8th field vs `gain@id`) still waits.
   - **Override editing got richer too:** the panel's numeric fields are now number
     spinners with arrows + wheel-nudge (Shift = ×10 step). For a **multi-selection**,
     `ratio` and `a_max` disable (an absolute number is ambiguous across a group) —
     instead, with the `move` tool you **drag the group in the spectrum**:
     vertical = scale all `a_max` together (relative spread preserved), horizontal =
     transpose all ratios by a common factor (intervals preserved). The gesture is
     axis-locked (first move decides x or y). Drags write to `OVR`, so they carry the
     "modified" badge and bake like any override.
4. ~~**Graphic add/remove**~~ — **done.** `place` + click places a live synthetic
   partial (x→ratio, y→a_max), selects it, and stores it as an override entry until
   bake. `place` - click on a partial removes that one point. Delete/Backspace
   outside text fields still removes the current selection as live override state:
   existing grammar partials become invisible `{removed:true}` entries, while
   newly-added unbaked partials are simply un-added. Readout names invisible state
   explicitly (`1 removed`, `2 added`) so removal has feedback even though the partial
   disappears. Bake commits adds as literals, literal removals as deleted lines, and
   sum removals as `where !(...)` exclusions.
   *(Post-review fixes 2026-07-02: plural labels no longer produce "removeds";
   dead pre-graphic `bakeSelective()` deleted — `bakeSelectiveGraphic()` is the
   only bake; added a `restore removed` toolbar chip, since removed partials are
   unselectable and live outside the text-scoped undo — previously irreversible.)*
5. **Move snapping** — snap modes for move drags (free / harmonic / EDO / prime).
   (Single + group x/y drag already live; this adds quantized targets.)
6. **Extras** — solo/mute selection audition · group ops (scale amps, transpose ·k,
   stretch ^k, scatter ±¢) · copy-selection-as-grammar · ADSR hover ghost.

### Then (carried forward)
Scala `.scl/.kbm` import · quartertone split-key piano · performance presets
{surface + tuning + range} · per-note X evaluation (timbre as function of pitch) ·
BroadcastChannel live bridge to TaBoTa Roll · resynthesis (FFT → grammar) ·
VST via iPlug2/JUCE + WebView.

### Language notes (current semantics)
Multiply order per partial: `a (a_max) · env(r) · adsr(t) · gain(t)`.
`r(t)` present → per-partial drift off (explicit motion supersedes random).
`gain`/`a(t)` present → voice master ADSR off (no double-envelope).
Open decision (still parked): literal syntax for a per-partial `gain(t)` so
dynamic overrides can bake (8th field vs `gain@id : f(t)`).

---

## Files

- `index.html` — the whole instrument (single file; the dev artifact).
- `index dark version.html` — the earlier dark spectral-bloom skin, kept as backup.
- `tabota-resolve.js` — vendored Tabota resolver (classic-script build; header
  stamp says which `../tabota/` commit it was copied from and how to re-vendor).
- `IntraNet.otf`, `Seona-DEMO.otf` — local display faces for the LCD skin.
- `aliquoto.md` — this file: roadmap, spec, and private LLM-facing documentation.

## Session state (2026-07-02)

Instrument is **solid and feature-complete for this arc**: grammar (t-dynamics,
per-partial ADSR, gain, keyfollow) · selection/override/bake graphic editing
(phases A–D) · n-EDO tuning + piano/hex/ribbon surfaces · Tabota-native note
model + `.tabota` import · naked-math card. Development wrapped here by choice;
next arcs live in the roadmap above (move snapping + extras 5–6, then the
carried-forward list).
