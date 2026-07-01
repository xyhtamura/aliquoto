# aliquoto

**Additive dequantizer — the spectrum before the synth.**

A pure additive sine synthesizer built to push additive synthesis toward its monstrous
conclusion. Partials may sit at *any* ratio of the fundamental — integer, fractional,
irrational, prime, or **below** the fundamental (subharmonic). The spectrum is written
as **mathematics** (summations, envelopes, conditionals) rather than drawn or dialed,
and the math is kept visible, not hidden inside an engine.

*aliquoto* = a part contained in a whole an exact number of times (a divisor/multiple);
also the *aliquoto strings* of pianos and harps that ring in sympathetic overtones.

Sibling to **Cycla** (`../tabota/cycla_builder.html`): Cycla is a tuning of *meter*
(recursive subdivision); aliquoto is a tuning of *timbre* (the sum of sines). Same
counter-poetics — both work the interstitial positions a standard grid disallows
(12-TET for pitch, integer harmonics for timbre).

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
- **Per-partial drift:** `spread¢` = static random detune per partial; `rate` = each
  partial slowly wanders. The current organicizer; per-partial/multi-target jitter is
  planned (see Roadmap).
- **Partial cap:** 1024. Lower the Σ ceiling if you hit it.

---

## Roadmap

1. **Per-note evaluation** — bind `X`/`f0` to the played note so spectra are computed
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
