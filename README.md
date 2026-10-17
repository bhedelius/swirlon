# Swirlons — Active Matter Playground

An interactive, in-browser simulation of the **swirlonic state of active matter**:
self-propelled particles that condense into clusters which, instead of flocking
off in a straight line, settle into coherent **orbital rotation** about their own
centre of mass. No fluid, no alignment rule — just local attraction frustrating a
fixed self-propulsion speed.

**▶ Live demo: https://bhedelius.github.io/swirlon/**

Everything runs client-side in a single `index.html` (canvas + vanilla JS); there
is no build step and no backend.

## What you're looking at

Each particle obeys a Langevin equation `v̇ = f + (b − v²)v + √(2D)·ξ`:

- `(b − v²)v` is **self-propulsion** — it drives every particle toward speed `√b`
  in whatever direction it happens to point. There is *no* alignment term, so
  neighbours never copy each other's heading.
- `f` is a pairwise interaction with a hard core and a **short-range** attraction.
  That short range is the whole trick: a bound particle must still move at `√b`
  but is hemmed in by neighbours, so a cluster resolves the conflict by
  circulating around its centre — a **swirlon**. Long-range attraction instead
  lets clusters translate together (a flock).
- `√(2D)·ξ` is thermal noise, trading clean rotation against a steadier
  form-and-break population.

The counter-intuitive payoff: a swirlon pushed by a steady force drifts at
**constant velocity** (not constant acceleration), as if immersed in a viscous
fluid — the active-matter signature the readout highlights.

## Using it

- **Phase presets** — one-click jumps to Swirlons, Churn, Gas, Liquid, Solid, and
  Flock. These were tuned quantitatively against order parameters (see below).
- **Sliders** — self-propulsion (`√b`), noise `D`, attraction strength and range,
  particle count, and a steady external nudge.
- **Drag on the field** to stir the particles.
- Coherent rotation takes ~20–30 s to build from a fresh disordered start.

## Phase presets

The six presets aren't hand-picked. They were re-derived by running a headless
port of the exact integrator offline and optimizing each one against quantitative
order parameters (rotational order, polar order, drift ratio, clustered fraction,
hexatic order, turnover), so that every labelled phase actually exhibits the
behavior it claims. The method, search procedure, and measured results are
documented in [`doc/optimization.md`](doc/optimization.md).

## Reference

Brilliantov, Abutuqayqah, Tyukin & Matveev, "Swirlonic state of active matter,"
*Scientific Reports* **10**, 16783 (2020).
