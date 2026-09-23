---
name: arpes
description: >
  Load and analyze ARPES photoemission data with correct axes, units,
  EDC/MDC extraction, Gaussian/Lorentzian/Voigt peak fitting, k-space
  conversion, photon-energy to kz conversion, and near-EF gap/pseudogap
  analysis (metal EF, resolution-broadened FD divide, symmetrize) via
  PyARPES or a confirmed user-project capability map. Use when working
  with ARPES spectra, Fermi surfaces, EDC, MDC, MAESTRO or NeXus/HDF5
  ARPES files, angle-to-momentum conversion, hv/kz scans, pseudogap,
  or PyARPES.
---

# ARPES

## When to use

User or task involves ARPES spectra, Fermi maps, EDC/MDC, peak fitting,
k or kz conversion, MAESTRO/NeXus/HDF5/Igor ARPES files, or PyARPES.

## What reduction means

In ARPES, **reduction** does not mean compress the file. It means turning a
raw multidimensional scan into analysis products: energy/momentum cuts,
EDC/MDC, Fermi-surface maps, fitted dispersions, k- and kz-converted
volumes, etc.

## Stack policy

1. Prefer **PyARPES** for analysis (fit, k, kz).
2. **At session start (and before any analysis):** check whether PyARPES
   imports **and** that the interpreter is **Python 3.8.x**.
   (`python -c "import arpes, sys; print(sys.version_info[:2])"`)
   PyARPES (`arpes` on PyPI) requires `>=3.8,<3.9` — not 3.9+.
3. **If PyARPES is missing or Python is not 3.8 — STOP and ask**
   (do not silently fall back; do not `pip install arpes` into the current env):
   - Explain: PyARPES needs a **dedicated Python 3.8 venv** so it does not
     break the user’s normal Python / other projects.
   - Ask: *Create a new `.venv-arpes` with Python 3.8 and install PyARPES there?*
   - Wait for yes/no.
   - If **yes**: follow `reference/pyarpes-env.md` (find `python3.8` →
     `python3.8 -m venv .venv-arpes` → `pip install arpes` in that venv →
     re-check import). If `python3.8` is missing, ask Homebrew vs conda
     (or a user-provided 3.8 path) before inventing installs.
   - If **no**: enter **user-map** path — search project → propose capability →
     callable map → **confirm** → use (`reference/backend-capability-map.md`).
     Only if no mappable `load_spectrum` (etc.) → ask for **xarray + h5py
     load/inspect only** (no fit / k / kz / gap). State path explicitly.
4. After PyARPES setup, run with `.venv-arpes/bin/python` (state that path).
5. Always state backend: `pyarpes` | `user-map` | `inspect-only`.
6. TensorSpec / TensorSpec_GUI: deferred named backend later (not wired).
7. **Living list:** every new skill workflow ships PyARPES-first docs **and**
   new/updated rows in `reference/backend-capability-map.md` in the same change.

## Hard rules

- Never invent axis names or units.
- Never treat detector angle as momentum without conversion + stated assumptions.
- Never hv→kz without stating inner potential V₀ (or that it is unknown).
- Never claim Γ found without method (manual / fit / model).
- Never report fits without naming lineshape (+ background if used).
- **Peak fitting:** backend models only (PyARPES or confirmed user-map; core →
  then EDC/MDC); ask before any new lineshape (`reference/edc-mdc-fitting.md`).
  After valence broadcast: default E vs k / width plots; linear or parabolic on
  E(k) → report vF / m* when asked by that workflow (prefer k-space).
- **Near-EF / gap / pseudogap (cuts, user-asked only):** metal-ref EF → shift
  sample; optional divide by **resolution-broadened** FD (state T + resolution);
  if gap/pseudogap → symmetrize EDC about E=0 (`arpes.analysis.gap.symmetrize`);
  state p–h symmetry; ask before inventing a Δ fitter
  (`reference/near-ef-gap.md`).
- Prefer scripted **calls to the active backend** (PyARPES or confirmed user-map)
  + matplotlib over launching Qt/Bokeh GUIs.
- **Package-first:** use PyARPES / confirmed user-map callables / existing
  project APIs; do **not** write a new loader or reimplement package features
  without asking (see `reference/package-first.md`,
  `reference/backend-capability-map.md`).
- Prefer existing project loaders before writing new ones — and **ask** before
  any new loader.
- **Capability map living list:** when inserting a new analysis workflow, add
  PyARPES-default row(s) to `reference/backend-capability-map.md` in the same
  change so later sessions can map user replacements.
- **Never skip the PyARPES / Python 3.8 venv question** when `import arpes`
  fails or `sys.version_info` is not `(3, 8)`.
- **Never `pip install arpes` into Python 3.9+** or into the user’s default env
  without asking first.
- **Never silently fall back** to xarray/h5py without the user declining the
  dedicated venv (or explicitly choosing inspect-only).
- **Warn before high-token steps** (see `reference/token-usage.md`) — inform,
  do not discourage; offer a lighter path when useful.
- **Fermi map / hv–kz reports:** include the **overview trio** — see
  `reference/default-overview-plots.md`.
- **Echo overview assumptions** in reports (kind from dims; center slices;
  no silent EF recal; anti-claims on Γ / k / EF / kz) — see
  `reference/default-overview-plots.md` § Default overview assumptions.
- **No k/kz in quick report** — overview trios stay angle-space; convert only
  in analysis / when user asks (`reference/k-and-kz-conversion.md`).
- **State energy axis** on load (Ek / Eb / E−EF / ambiguous).
- **Before cut or Fermi map → k:** PyARPES EF finder; always report EF_fit +
  deviation from 0 eV; charging warn if claimed E−EF/Eb and `|EF_fit| > 50 meV`;
  then shift EF→0. Package `convert_to_kspace` only — no invent formulas.
- **Γ for k conversion:** user offset wins. Cuts may use labeled provisional
  nearest-0°. **Fermi maps:** package `S.offsets` / optional `pocket_parameters`
  / optional `ktool` only — else **ask**; never invent a center finder
  (`reference/k-and-kz-conversion.md`).
- **hv → kz:** EF-align **per hv** (PyARPES broadcast on `hv`); slit offset from
  **lowest-hv** slice; state V₀; soft X-ray → `reference/beamline-geometry.md`
  (MAESTRO 55°; ALBA LOREA 55° — ask; SLS soft X-ray postponed) + **ask**
  about photon momentum / incidence.
- After k/kz conversion: save `analysis/kspace/*.npz` with required meta;
  prefer reload from cache when meta still matches.
- **Core-as-2D:** cut-shaped file with swept + deep/core clues (soft: span ≳10 eV
  or deepest ≳5 eV below EF) → suspect core level saved as 2D image; quick
  report = detector×energy **and** angle-integrated EDC; no default valence k
  (`reference/default-overview-plots.md`).
- **Folder first:** for multi-file folders, build/refresh `analysis/manifest.json`
  before deep analysis; later **recall** from it (`reference/folder-manifest.md`).

## Package-first (important)

Default = **call code that already exists** (PyARPES, or **confirmed** user-map
callables, or the project). If a path fails or is missing a feature, **tell the
user** and ask before writing a new custom loader/implementation. Details:
`reference/package-first.md`, `reference/backend-capability-map.md`.

## Token awareness

Before steps that will burn a lot of context (full-folder catalogs, pasting
arrays, broadcast fits over whole maps, many figures in-chat), give a short
**Token note** and offer a cheaper option (script → `analysis/`, summarize in
chat). Details: `reference/token-usage.md`.

## Error handling

- Missing PyARPES / wrong Python — **ask to create `.venv-arpes` (Python 3.8)
  and install** (`reference/pyarpes-env.md`); if declined → user-map path
  (`backend-capability-map.md`); only then inspect-only xarray/h5py.
- Package load fails / feature missing — quote error; ask before custom code
  (`reference/package-first.md`).
- Ambiguous axes — stop and ask one sharp question.
- Ambiguous V₀ — ask or mark kz as relative/uncertain.
- Corrupt/partial file — report readable parts only.

## Workflow

1. If the user points at a **folder** / many files: build or refresh
   `analysis/manifest.json` (+ optional `manifest.md`) **first** — see
   `reference/folder-manifest.md`. Later turns **recall** from the manifest;
   do not re-walk the folder into chat.
2. Identify artifact (file type, shape, **existing** package/project loaders);
   prefer rows from the manifest when present.
3. Check PyARPES + Python 3.8; if missing/wrong, ask for dedicated venv
   (see Stack policy / `reference/pyarpes-env.md`). If declined → propose
   user capability map (`backend-capability-map.md`) before inspect-only.
4. Try package load/analysis first (`reference/package-first.md`). For
   MAESTRO: prefer sibling **`.fits`** + `location='MAESTRO'` before MH1
   `.h5`; pick main spectrum carefully (`formats-and-axes.md`). If load
   fails or needs new code — **ask** before a custom loader.
5. Lock coordinates — names + units (° vs Å⁻¹, eV, hν; binding vs kinetic).
6. Sanity print — shape, ranges, one mid-cut summary.
7. Reduce — cut / FS / EDC / MDC (see `reference/safe-reduction.md`).
8. **Quick report:** stop at angle-space overviews
   (`default-overview-plots.md`). **Do not** convert to k/kz here.
   Update manifest `overview_paths` when PNGs are written.
9. **Analysis / user-requested momentum:** convert to k / kz
   (`reference/k-and-kz-conversion.md`) — state energy axis; EF finder +
   report EF_fit/deviation (charging warn if >50 meV on E−EF/Eb); Γ per
   cut vs Fermi rules; stated V₀ for kz; save `analysis/kspace/*.npz`; link
   `product_paths` on the manifest row.
10. If line / core analysis — fit (`reference/edc-mdc-fitting.md`: **core first**,
   then EDC/MDC; package models only).
11. If user asks near-EF / metal EF / FD / **gap** / **pseudogap** on a cut —
    `reference/near-ef-gap.md` (metal fit → shift; optional resolution-broadened
    FD divide; symmetrize when gap/pseudogap).
12. Plot/report with labeled units; **list overview / conversion assumptions**.
13. Before expensive batch work — token note (`reference/token-usage.md`).

If unsure: read the matching `reference/` file; ask the user one sharp question.

## References

- `reference/formats-and-axes.md`
- `reference/safe-reduction.md`
- `reference/edc-mdc-fitting.md` — peak fitting: core, then EDC/MDC (PyARPES only)
- `reference/near-ef-gap.md` — metal EF, resolution-broadened FD divide, symmetrize (gap/pseudogap)
- `reference/k-and-kz-conversion.md` — analysis-mode k/kz + Γ + npz cache
- `reference/beamline-geometry.md` — MAESTRO / ALBA LOREA 55° defaults; SLS soft X-ray postponed
- `reference/failure-modes.md`
- `reference/pyarpes-env.md` — Python 3.8 dedicated venv + install
- `reference/token-usage.md` — when to warn about token cost
- `reference/default-overview-plots.md` — cut / Fermi trio / hv–kz trio + assumptions
- `reference/package-first.md` — use package APIs; ask before new code
- `reference/backend-capability-map.md` — capability IDs; PyARPES defaults; user-map; living list
- `reference/folder-manifest.md` — folder inventory before analysis; recall later

## Examples

- `examples/maestro_pyarpes.md`
- `examples/fit_edc_mdc.md`
- `examples/near_ef_gap.md`
- `examples/backend_user_map.md`
- `examples/convert_k_kz.md`

## Requires (full analysis)

- **Python 3.8.x only** for PyARPES (`>=3.8,<3.9` on PyPI)
- Dedicated venv (recommended name: `.venv-arpes`) — see `reference/pyarpes-env.md`
- `pip install arpes` **inside that venv**
- For load/inspect fallback only: `xarray`, `h5py` (any modern Python OK)

## Key PyARPES paths

See `reference/` for recipes. Common entry points:

- Load: `arpes.io.load_data` or project loaders
- k-space: `convert_to_kspace` + `S.apply_offsets` (analysis mode; state Γ method)
- kz: hv scans with stated `inner_potential` V₀; cache under `analysis/kspace/`
- Fit: core (Shirley + multi-peak) then EDC/MDC; `broadcast_model` for maps /
  dispersion parameter plots
- Near-EF / gap: metal `AffineBroadenedFD` → shift; optional broadened FD divide;
  `arpes.analysis.gap.symmetrize` for gap/pseudogap (`near-ef-gap.md`)
