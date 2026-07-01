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

Each non-empty line is one statement. `#` starts a comment.

### `partial` — one sine