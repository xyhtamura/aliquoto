# The math-synth suite

*A living overview of the three digital, mathematics-first synthesizers built so
far, and where the family can go. The collection isn't formally named or grouped
yet — this doc is the "as a whole" record until that decision is made. Per-tool
detail lives in each project's own `.md` (`aliquoto.md`, `../cella/cella.md`,
`../moire/moire.md`).*

Shipped (2026-07): **Aliquoto**, **Cella**, **Moire** — each a single self-contained
`index.html`, each in its own git repo.

---

## The thesis

**Pure mathematics, by itself, produces "organic," irregular, unpredictable sound.**

The received idea is that the digital/machinic is inherently cold, rigid, robotic —
and that warmth, life, and irregularity are things you claw back out of it with
great effort (analog gear, tape, hand-dithered noise, careful humanizing). These
tools argue the opposite: the machinic has its own native wildness, and "robotic"
is an artifact of *how we cage it* — quantizing, looping, expecting grids. Hand the
math its head and the irregularity comes back on its own.

Here, **irregularity / unpredictability / indiscretizability are the easy, natural
friends** — not the hard-won exceptions. Precision and griddedness remain fully
available for anyone who wants them (integer ratios, equal divisions, clean
envelopes); they're just not the only or default place the instrument lives. The
kinship is 1950s–70s avant-garde (Xenakis, GENDYN, the Barrons) — organic-from-
formal — but achieved with the *purity of the digital*: the irregularity lives in
the numbers, not in analog hardware grit.

## The three, and the taxonomy

The family is sorted by **how a spectrum comes to be**:

- **Aliquoto** — *written*. Additive: a linear sum of sines, every partial specified.
  The **string**. Grammar: `sum n=1..N : r(n) : a(n) : p(n)`, envelope laws, keyfollow.
- **Cella** — *answered*. A noise/impulse drive through a resonator bank; lines
  specified plus a width (Q). The **room**. Voigt lineshapes, release-is-physics.
- **Moire** — *woven*. Phase modulation: pure lines written into each other's phase;
  the spectrum is the interference, containing frequencies present nowhere in the
  source text. The **cloth**. First *nonlinear* member — "FM with the algorithm chart
  dissolved into an equation."

Reserved / adjacent members:

- **Horn of Plenty** — *harvested*. **Exists, but as a tool, not a synth:**
  `../hindcasts/horn-of-plenty/`, an offline stationarizer (scrap → yardage,
  *felt not weave*). The suite-member idea survives as an *ingredient*: borrow its
  engine to stationarize samples into playback substrate (one candidate idea for
  member 4, the mellotron-like synth — see Future directions).
- **Fano** — the **chimera**: a source–filter marriage (a broadband substrate
  feeding a Cella-style driven resonator — discrete lines interfering with
  a driven continuum). Cella already exposes an unfilled drive-buffer input port for
  exactly this.

## The shared engine

The load-bearing common substrate is **Aliquoto's engine of mathematical
definitions**: a small expression DSL (grammar → per-voice generated JS function →
AudioWorklet, with a ScriptProcessor fallback), a source-agnostic voice seam
(`startNote(id,hz,vel)` / `bendNote` / `stopNote` — every input just produces Hz),
n-EDO tuning, on-screen piano/hex/ribbon surfaces, live Web MIDI, and MIDI-file →
offline WAV export. Cella and Moire are hand-synced copies of this DSL + MIDI/WAV
path (tracked in `../DEPENDENCIES.md`).

**Consequence for the future:** later instruments don't need a new engine. They are
new *definitions* layered on this one. Whatever the sound source, if it can be
expressed as "a (possibly continuous) frequency/definition over time," it plugs into
the same voice seam, surfaces, tuning, and export.

---

## Future directions

### Shared, first pass shipped — `rnd()` (important for all three)
A first-class **random function in the grammar** is now available in every tool's
expression scope. `rnd()` returns 0..1; `rnd(max)` and `rnd(lo,hi)` scale the draw,
so jitter can live directly in `r(n)`, `a(n)`, `p(n)`, `q(n)`, `gain(t)`, or Moire
operator equations. This first pass is intentionally simple and unseeded: it uses
the browser's random source wherever the expression evaluates.

Next refinement: make the primitive seedable/deterministic per voice and add a
smoothable/band-limited companion, so a patch is reproducible but can still wander
as *life* rather than hash.

### The 1.x arc order (set 2026-07-10; shipped trio = 1.0)

1. **1.1 — signal-source seam + seeded/smooth rnd.** One registry: name → f(t),
   evaluated per block. Seeded rnd, smooth/band-limited rnd, file follower, and
   band-energy lookup all register into the same seam — build the plumbing once.
   Patches become reproducible *before* they become file-dependent.
2. **1.2 — file drop.** Aliquoto first (richest gain infra; keyfollow vocoder is
   the flagship claim); cella's drive port in the same arc (seam already designed —
   a dropped file is the first guest, no need to wait for a harvested member);
   moire third (established copy pattern). The infra (decode/resample/analysis)
   doubles as groundwork for member 4.
3. **1.3 — negative lines.** Prototype in cella (physics home), port the series
   zero to moire; aliquoto needs nothing (`env` dip already is it).

One dropped file across all three tools = the taxonomy demo: aliquoto
*dereferences* it, cella is *rung* by it, moire *weaves* with it.

### Shared — dropped sound as f(t) (Aliquoto, Moire; ideation 2026-07-10)
Let a soundfile (or live noise source) be dropped into the instrument — used not
as audio but as a **general source of values over time**. This gives real sound a
*third role* in the family, now a clean taxonomy:

- **substrate** — Horn of Plenty: the sample's *statistics* become the sound.
- **excitation** — Cella's drive-buffer port / Fano: the sample *enters the audio
  path* and rings the resonators.
- **modulator** — this idea: the sample **never enters the audio path**; it only
  supplies numbers. Per-partial `amp(t)` from an envelope follower (each partial
  can get its own dropped file), pitch-waver depth, modulation amount — anything
  the grammar can read as `f(t)`.

The point: a "real," "physical" sound manipulates something digitally pure
*without wrecking its digital identity* — the sines stay pure sines by
construction, because the file is dereferenced, not mixed.

Standout mode: the **keyfollow vocoder**. A partial's amp reads the band energy
at *that partial's own hz*; since partials keyfollow, the analysis bands move
with the played note. A ratio-defined vocoder — no fixed-band vocoder does this,
and only an additive synth can (subtractive synths can merely filter the file;
additive can dereference it per-partial). In Moire: a follower on modulation
index = the real sound *weaves the interference*; audio-rate file as a phase
input = PM cross-synthesis.

Rate question to settle at build time: control-rate follower (per-block, the
existing cella-dynamics grain, cheap) vs audio-rate reads. Per-tool notes in
`aliquoto.md` / `../moire/moire.md`.

### Cella — negative filter (anti-resonance) *(expanded 2026-07-10)*
A **negative/subtractive Cella**: instead of resonating frequencies *up*, scoop them
*out* — notches, anti-resonances, spectral holes. Worthwhile as a standalone Cella
feature, and a natural ingredient when building **Fano** (the chimera wants both
additive driven peaks and subtractive scoops acting on a substrate).

Ideation findings (detail in `../cella/cella.md`):

- Physics name: a **zero**, not a pole. Fundamentally asymmetric to the normal
  cella line — it requires sound present to be heard at all. A different entity,
  not a parameter setting.
- Because its center is **ratio-defined**, it keyfollows at baseline — and it can
  carry `q(t)`, `rnd()`, drift like any line: breathing, wandering spectral holes.
  A static notch filter is the degenerate case, not the design.
- Entity framing: it is closer to the **env() family** than to the pole family —
  spectral shaping that keyfollows by design. But it is *more* than env: `env(r)`
  scales discrete line amps at build time; a zero carves the *realized continuous
  spectrum* (Lorentzian tails, ensemble bloom, ring-out), which env can't touch.
- Per-sibling meaning differs: **aliquoto** already has it as sugar (`env : f(r)`
  with a dip — lines are discrete, so scaling amps *is* the notch); **cella** gets
  new physics; **moire** is where it's *irreplaceable* — sidebands are emergent,
  no env(r) can reach them, so a series zero after the sum is the only way to
  sculpt woven lines. Same entity, three different depths of necessity.

### Members 4–5 — the next two synths (unnamed, ideation 2026-07-10)
A different register from the digital-first trio; same engine, new definitions:

4. **Mellotron-like** (no name yet). Mathematical definitions operating over
   sampled material rather than pure sines — pushing what a sampler/mellotron is,
   the way the trio pushes additive/resonant/FM. One candidate ingredient (of
   several): Horn of Plenty's stationarizer engine
   (`../hindcasts/horn-of-plenty/`) to turn a sample into endless stationary
   playback substrate.
5. **Vintage synth simulator** (no name yet). *Not* faithful recreation — u-he
   Diva, PSpice-grade modelling, the Novachord/Solovox VSTs already own that
   ground. Instead: import the **physics of circuits** as the thing that
   determines sound, then let it do what hardware physically can't — component
   values, topologies, operating points no real circuit could hold.

Parked beyond those: a physical-modelling member (string/tube/membrane in the
grammar idiom), and the deferred summit — the **nonlinear-ODE synth**, "spectrum
*grown*," the family's hardest, last sibling.

Recognized (2026-07-10): member 5 and the summit are **the same animal in
different notation** — circuit physics *is* nonlinear ODEs; circuit topology
(VCO, ladder, feedback path) is just a familiar vocabulary for writing an ODE
system. So #5 is the *on-ramp*, not a detour: build the circuit-notation synth,
let "physically impossible values" loosen gradually, and the summit arrives by
erosion rather than assault.

Framing that ties it together: each tool takes one axis a standard imposes as a grid
and lets you work the interstitial positions the grid forbids — Aliquoto the spectrum
grid, Cella the resonance grid, Moire the interference grid, and (via the shared
tuning) the pitch grid. New members add new axes, not new engines.

---

*Status: three shipped and stable (2026-07) — the 1.0 trio. Collection
name/grouping undecided. Next: arc 1.1 (signal-source seam + seeded/smooth rnd).*
