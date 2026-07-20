# Phase-preset optimization

The six phase presets in `index.html` were originally hand-picked. This document
records how they were re-derived by **running the simulation offline and
optimizing each preset against quantitative order parameters**, so that every
labelled phase actually exhibits the behavior it claims.

## Method

A headless Node port of the *exact* integrator in `index.html` (same Rayleigh
self-propulsion `(b−v²)v`, same Morse-like force, same cell-list, same
mulberry32 PRNG, same `DT`) was driven to steady state and measured. For each
candidate parameter set the run was equilibrated (~15–18k steps ≈ 20–25 s of
app time) and order parameters were averaged over a following window.

### Order parameters

Clusters are found by a periodic union-find over the link distance
`LINK = 1.8`; each cluster's coordinates are **unwrapped** so rotation and drift
are measured in a continuous frame.

| symbol | meaning | high when… |
|---|---|---|
| `rotOrder` | `|Σ rᵢ×vᵢ| / Σ |rᵢ||vᵢ|` per cluster, size-weighted | particles **orbit** the common centre (rigid rotation → 1) |
| `polar` | `|Σ vᵢ| / Σ|vᵢ|` per cluster | particles **translate together** (a flock) |
| `driftRatio` | cluster-COM speed / particle speed | the cluster itself moves fast |
| `fracClustered` | fraction of particles in clusters ≥ 8 | condensed |
| `largest` | largest cluster / N | drops have merged |
| `hexatic` | bond-orientational order ψ₆ | crystalline lattice |
| `turnover` | temporal CV of clustered fraction | clusters constantly form & break |

The **swirlon signature** is `rotOrder > polar` (spin beats translation) with
particles still running near `√b` while the cluster crawls. A **flock** is the
opposite: `polar > rotOrder` with high `driftRatio`.

### Search

Per phase: ~64 random samples over a phase-appropriate box in
`(b, D, Cₐ, lₐ, N)`, then two rounds of local refinement around the top 4, each
scored by a phase-specific objective (e.g. swirlons reward
`rotOrder · spin-dominance · (1−driftRatio) · condensed · large-drops`). Winners
were re-validated at longer equilibration across multiple RNG seeds. Scripts
live outside the repo (scratchpad); this file is the durable record.

## Results

Old vs new parameters:

| preset | old (b, D, Cₐ, lₐ, N) | **new (b, D, Cₐ, lₐ, N)** |
|---|---|---|
| Swirlons | 1.5, 0.02, 4, 2.0, 280 | **2.11, 0.007, 6.2, 0.95, 1000** |
| Churn | 2.0, 0.10, 4, 1.5, 420 | **2.15, 0.06, 3.1, 1.25, 800** |
| Gas | 2.0, 0.25, 1.0, 1.5, 420 | **1.5, 0.30, 1.3, 1.2, 500** |
| Liquid | 1.0, 0.03, 5, 2.0, 820 | **2.2, 0.007, 4.9, 3.05, 1100** |
| Solid | 0.05, 0, 7, 1.6, 620 | **0.15, 0.001, 7.3, 2.3, 900** |
| Flock | 2.0, 0, 4, 3.0, 420 | **3.0, 0.017, 5.7, 4.0, 300** |

Measured behavior (2 seeds, warmup 15k / window 4k):

| preset | rotOrder | polar | driftRatio | fracClustered | largest | hexatic | turnover |
|---|---|---|---|---|---|---|---|
| Swirlons | **0.50** | 0.43 | 0.46 | 1.00 | 0.12 | 0.86 | – |
| Churn | 0.23 | 0.21 | 0.20 | 0.85 | 0.10 | 0.59 | **0.037** |
| Gas | – | – | – | **0.00** | – | – | – |
| Liquid | 0.21 | 0.35 | 0.40 | 1.00 | **0.17** | 0.62 | – |
| Solid | 0.14 | 0.20 | 0.20 | 0.97 | – | **0.93** | – |
| Flock | 0.32 | **0.60** | 0.63 | 0.98 | – | 0.83 | – |

Baseline swirlons scored `rotOrder 0.315 ≈ polar 0.305` — spin and translation
were **tied**, which is why the coherent-swirl behavior looked weak. The tuned
preset reaches `rotOrder 0.50 > polar 0.43`: spin now clearly dominates, +58% in
absolute rotation. Churn's turnover roughly doubled while its condensate became
leaky (fracClustered 0.85 vs swirlons' 1.00), and Liquid's drops are 3.6× larger
and fewer.

## What the optimization taught us

- **Attraction *range* selects swirlon vs flock, not strength.** The original
  Swirlons preset used `lₐ = 2.0` (fairly long-range), which let clusters feel a
  smooth pull toward their centre and simply translate — so they flocked as much
  as they spun. Shortening to `lₐ ≈ 0.95` frustrates uniform flow and forces the
  circulation that defines a swirlon. Flock, conversely, wants the *longest*
  range (`lₐ = 4.0`).
- **Denser systems need shorter range to stay swirlons.** At `N = 600` the best
  spin came from `lₐ ≈ 1.15`; pushing to the requested `N = 1000` with that same
  range flipped the system flock-like (`polar 0.59 > rotOrder 0.42`). A focused
  search with `N` pinned at 1000 recovered clean spin (`rotOrder 0.67 ≫ polar
  0.26` at short equilibration) only by shortening range to `lₐ ≈ 0.95`. The
  shipped default (`N = 1000`, `lₐ = 0.95`) reflects this.
- **Churn is intrinsically the marginal phase.** Sustained form-and-break
  turnover is strongest early and settles over time; the noise `D` that
  maximizes it sits just below the gas boundary. It is genuinely a narrow
  window, not a hand-tuning failure.

## Reference

Brilliantov, Abutuqayqah, Tyukin & Matveev, "Swirlonic state of active matter,"
*Scientific Reports* **10**, 16783 (2020).
