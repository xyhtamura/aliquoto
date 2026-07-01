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
  and per-partial drift exist to escape the integer-harmonic lattice most additive
  synths are locked to.

---

## Quick start

Open `index.html` in a modern browser (Chrome/Edge/Firefox). Click **▶ enable audio**,
then play with the on-screen keys, your QWERTY row (`a s d f …`, `z`/`x` shift octave),
or a MIDI controller. Type a spectrum in the **Y=** box and press **apply** (Ctrl+↵).

No build step, no dependencies to install — fonts and KaTeX load from CDN.

---

## The spectrum grammar

Each non-empty line is one statement. `#` starts a comment.

### `partial` — one sine
```
ratio : amp : phase°
```
`ratio` is a multiple of the fundamental (`2` = octave, `1.5` = fifth, `0.5` =
sub-octave, `2.3` = inharmonic). `amp` defaults to 1, `phase` (degrees) to 0.
Any of the three may be a constant expression (`phi : 1`, `pow(2,7/12) : 0.5`).

### `sum` — a series
```
sum n=1..N : r(n) : a(n) : p(n)
```
Generates one partial per index value. `N` may be `inf` (or `∞`/`*`), which clamps to
the **Σ ceiling** control — the truncated infinite sum.

Multiple indices multiply (a product / lattice spectrum):
```
sum n=1..6 k=1..3 : n*k : 1/(n*k)
```

Filter with a **`where`** predicate:
```
sum n=1..48 : n : 1/n where odd(n)
```

### `env` — spectral envelope
```
env : f(r)
```
Multiplies every partial's amplitude by a function of its ratio `r` — the bijective
pitch→amp map. Compose several; they multiply. Example: low-pass a saw with
`env : exp(-r/12)`.

---

## Variables and keyfollow (`X`)

Inside `a(n)`, `p(n)`, `env`, and `where` you can use:

| var  | meaning |
|------|---------|
| the index(es) (`n`, `k`, …) | current index value(s) |
| `r`  | this partial's ratio |
| `hz` | this partial's frequency in Hz (= `r · X`) |
| `f0` | the reference fundamental (= `X`) |

**`X` (the pitch reference)** is set by the `X pitch Hz` control. It is the pitch the
`hz`/`f0` variables resolve against — this is **keyfollow**. It lets you write spectra
that depend on absolute frequency, not just partial index:

```
# below 800 Hz: full harmonics; above: odd only
sum n=1..48 : n : iff(hz<800, 1/n, iff(odd(n),1/n,0))

# a low-pass that tracks the pitch, in Hz
sum n=1..64 : n : 1/n
env : iff(hz<2500, 1, 2500/hz)

# a fixed-Hz resonance peak (vowel-ish formant)
sum n=1..48 : n : 1/n
env : 1/(1+((hz-1200)/400)^2)
```

> **Today `X` is a single fixed reference.** The planned next step is to bind `X` to the
> *played note*, so `hz`/`f0` follow the keyboard and the timbre becomes a function of
> pitch — effectively a multi-instrument spread across the keys (formants that stay put,
> brightness that scales, region splits per octave). See Roadmap.

---

## Functions, constants, operators

- **Functions:** `prime(n)` `fib(n)` `sin cos tan exp log sqrt pow abs mod(a,b)`
- **Conditionals:** `iff(cond, a, b)` · `odd(n)` `even(n)` (→1/0) · `step(edge,x)` ·
  `between(x,lo,hi)` · `clamp(x,lo,hi)` · `not(x)`
- **Comparators / logic:** `< > <= >= == != && ||` (true = 1, false = 0). No ternary
  `?:` — the `:` is the field separator; use `iff(...)` instead.
- **Constants:** `pi e phi tau`
- **Power:** `^` (or `**`)

---

## Interaction

- **Drag a partial** in the SPECTRUM view to reshape its amplitude (a drawbar gesture).
  This marks the spectrum `edited`.
- **bake** rewrites the current partials back into explicit `ratio : amp : phase` lines,
  so hand-edits become honest math again.
- **normalize** rescales amplitudes so the loudest partial is 1.
- **ƒ math** opens the naked-math card: the general equation, your spectrum as live
  LaTeX, the envelope plot, and a TI-TABLE of every computed partial.

---

## Engine

- **Sound:** one `AudioWorkletProcessor` per voice, summing sines with true per-partial
  phase (a plain oscillator bank can't honor phase — that's the fallback if AudioWorklet
  is unavailable). Amplitude is normalized by Σ|aₙ|; a compressor guards the master.
- **Pitch is an automatable param:** the fundamental `f0` is an a-rate `AudioParam`, so a
  note's pitch can move during its life — glides, ribbon sweeps, MPE bend all schedule
  onto it. Discrete keys just set it once.
- **Per-partial drift:** `spread¢` = static random detune per partial; `rate` = each
  partial slowly wanders. The current organicizer; per-partial/multi-target jitter is
  planned (see Roadmap).
- **Partial cap:** 1024. Lower the Σ ceiling if you hit it.

---

## Note model — Aliquoto speaks Tabota

Aliquoto's internal note **is** a realizable [Tabota](../tabota/) Event. Every input —
on-screen keys, QWERTY, MIDI, and (soon) a `.tabota` file or a continuous surface —
produces the same thing, and the engine realizes it. Tabota is the lingua franca; the
synth is a *realizer*.

**The seam.** All sources funnel through two source-agnostic calls:
`startNote(id, hz, vel)` / `stopNote(id)`. A source only has to produce a **frequency in
Hz** (plus gate + velocity). The keyboard/MIDI wrappers map a MIDI note through the
`TUNING` layer (`TUNING.toHz`, 12-TET today, pluggable) — but a hex grid, ribbon, or
Tabota importer would call `startNote` with a Hz it computed itself.

**The note.** A note carries a pitch *trajectory* `pts[]{t, hz, curve}` (seconds from
onset): one point = a held pitch, more = a glide. `bendNote(note, hz, glideSec, curve)`
appends a breakpoint live and schedules it onto the `f0` param — the seam for
ribbon/MPE/glide.

**The encoding** (`noteToEvent` / `eventToNote`): the note maps to a Tabota `Event` on a
`chronological · frequency` frame — `value.pitch.hz` (or `.from/.to/.interpolation` for a
glide, or nested per-segment glide children for a multi-point trajectory),
`value.velocity`, `position.at` (seconds), `extent.duration` (or `end:"open"` while held).
`performanceDoc()` wraps the sounding voices into a full doc; the **spectrum rides in
`payload`** — Aliquoto reads and writes only its own paddy of the shared file. Watch it
live in the **tabota** tab of the `ƒ math` card.

Because the note is Hz-native and the pitch is a param, the `.tabota` import path (next)
is small: run `tabota-resolve.js` `resolve(doc)`, turn each resolved node into a note,
schedule. **No MIDI in the middle** — MIDI's 12-TET + power-of-2 bend is the exact
quantization Tabota refuses.

---

## Interfaces & tuning

The bottom panel is a **switchable surface** (the `// interface` chip row) — pick how you
play. All surfaces are multitouch and feed the same engine via `startNote(id, hz, vel)` /
`bendNote` / `stopNote`; a surface just computes Hz from a gesture.

- **piano** — the standard keyboard (click · QWERTY `a s d f…`, `z`/`x` octave · MIDI),
  retuned through the tuning layer.
- **hex** — an **isomorphic** grid (Lumatone lineage). Each cell is a pitch; moving right
  adds `→ steps`, up-right adds `↗ steps` (in EDO steps, editable) — so every interval is a
  constant shape you can transpose anywhere. Cell numbers are pitch-classes; the tonic (0)
  is highlighted (and repeats as a lattice — your orientation anchor). Drag across cells to
  slide. Also playable from the **computer keyboard**: the four physical rows map
  isomorphically onto the hex (`z…` / `a…` / `q…` / `1…` bottom-to-top).
- **ribbon** — a **continuous** strip (stylophone / Continuum): x = pitch (log-Hz over
  `octaves`), **drag = glide** (rides the `f0` param), Y = velocity, multiple fingers =
  chords. Optional **snap to EDO** quantizes to scale degrees; off = true glissando.
  Continuous, so it maps to neither computer keyboard nor MIDI.

The panel resizes to the selected interface; switching surfaces releases any held notes.

**Tuning** is **n-EDO**: set `EDO` (divisions per octave — 12, 24 quartertone, 19, 31,
22…). Reference is the `A4 Hz` control (step 0 = A4); `oct` shifts by octaves. Every surface
and MIDI/QWERTY resolve their step index through `stepToHz(step) = A4·2^(step/EDO)`. Scala
`.scl/.kbm` import is the next tuning source; MPE and more surfaces after.

---

## Importing a `.tabota` score

Drag a `.tabota`/`.tab`/`.json` file onto the window, or use **⤒ .tabota** in the header.
Aliquoto parses it, runs the **vendored resolver** (`tabota-resolve.js`, a classic-script
copy that self-registers `window.TabotaResolve`), and realizes the **determinate fragment**
— every node with a grounded time (`start.sec`) and a usable pitch (`hz`, or a `from/to`
glide). Indeterminate / relational / unplaced / banded pitches are skipped (counted in the
status line). **▶ score** plays it through the current spectrum; each note is a scheduled
voice with its pitch on the `f0` param (so glides sound). If the file's `payload` carries an
`aliquoto` spectrum (as Aliquoto's own exports do), that spectrum is adopted on load.

No MIDI anywhere in this path — the score's Hz go straight to the engine.

**Resolver freshness.** The local copy is pinned and stamped with its source commit. When
served over http(s), Aliquoto does a background hash check against the canonical
`tabota-resolve.js`; if it differs, the **resolver ✱** chip appears — a nudge to re-vendor
(it never auto-swaps). Offline / `file://` skips the check and just uses the local copy.

---

## Roadmap

1. **More surfaces & tuning** — quartertone (24-EDO) split-key piano; **Scala `.scl/.kbm`**
   import (arbitrary scales / JI); saved **performance presets** `{surface + tuning + range}`.
   (Hex, ribbon, n-EDO — done.)
2. **Live bridge** — `BroadcastChannel` link to TaBoTa Roll (Roll = sequencer, Aliquoto =
   instrument), real-time. (File import — done.)
2. **Alt-tuning surfaces** — a `TUNING` layer (Scala `.scl/.kbm`, n-EDO) + on-screen
   surfaces (hex isomorphic, quartertone, continuous ribbon) that emit Hz-notes directly;
   multitouch for phone/tablet. Saved as **performance presets** `{surface + tuning + wiring}`.
3. **Per-note evaluation** — bind `X`/`f0` to the played note so spectra are computed
   per key. Unlocks tracking filters, fixed-Hz formants, key-scaled brightness,
   octave/region splits — timbre as a *function of pitch*. (Architectural: split the
   parsed *spec* from the per-note *realized* partials.)
2. **Per-partial jitter, multi-target** — settable pitch/amp/phase jitter as a function
   of `n` or `r`, smoothed random-walk (life, not vibrato). Rides on #1's eval machinery.
3. **Resynthesis** — audio in → FFT → peak-pick → export as a partial list / `sum`.
   (Longer term: sinusoidal modelling for time-varying partial tracks.)
4. **Never in the core:** filters, unison, fx — those live in an external chain.

### As a real plugin
The DSP is a near 1:1 port of the worklet to C++. Best route to a VST is **iPlug2 or
JUCE with a WebView UI**, reusing this HTML front-end and bridging params to a C++
additive voice engine. Arbitrary ratios pair naturally with MPE + MTS microtuning.

---

## Files

- `index.html` — the whole instrument (single file).
- `README.md` — this document.
