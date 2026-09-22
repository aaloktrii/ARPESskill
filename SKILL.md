---
name: arpes
description: >
  Load and analyze ARPES photoemission data with correct axes, units,
  EDC/MDC extraction, Gaussian/Lorentzian/Voigt peak fitting, k-space
  conversion, and photon-energy to kz conversion via PyARPES. Use when
  working with ARPES spectra, Fermi surfaces, EDC, MDC, MAESTRO or
  NeXus/HDF5 ARPES files, angle-to-momentum conversion, hv/kz scans,
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
   - If **no**: ask whether to continue with **xarray + h5py load/inspect only**
     (no fit / k / kz). Only then use that fallback; say so explicitly.
4. After setup, run analysis with `.venv-arpes/bin/python` (state that path).
5. Always state which path was used (PyARPES 3.8 venv vs inspect-only).
6. TensorSpec / TensorSpec_GUI: out of v1 — if asked, say deferred.

## Hard rules

- Never invent axis names or units.
- Never treat detector angle as momentum without conversion + stated assumptions.
- Never hv→kz without stating inner potential V₀ (or that it is unknown).
- Never claim Γ found without method (manual / fit / model).
- Never report fits without naming lineshape (+ background if used).
- Prefer scripted **calls to PyARPES** (+ matplotlib) over launching Qt/Bokeh GUIs.
- **Package-first:** use PyARPES / existing project APIs; do **not** write a new
  loader or reimplement package features without asking (see
  `reference/package-first.md`).
- Prefer existing project loaders before writing new ones — and **ask** before
  any new loader.
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
- **Γ for k conversion:** provisional heuristic OK if labeled; **user offset
  always wins**; never claim Γ without method.
- After k/kz conversion: save `analysis/kspace/*.npz` with required meta;
  prefer reload from cache when meta still matches.

## Package-first (important)

Default = **call code that already exists** (PyARPES or the project).
If a package path fails or is missing a feature, **tell the user** and ask
before writing a new custom loader/implementation. Details:
`reference/package-first.md`.

## Token awareness

Before steps that will burn a lot of context (full-folder catalogs, pasting
arrays, broadcast fits over whole maps, many figures in-chat), give a short
**Token note** and offer a cheaper option (script → `analysis/`, summarize in
chat). Details: `reference/token-usage.md`.

## Error handling

- Missing PyARPES / wrong Python — **ask to create `.venv-arpes` (Python 3.8)
  and install** (`reference/pyarpes-env.md`); only after user declines, offer
  xarray/h5py inspect-only.
- Package load fails / feature missing — quote error; ask before custom code
  (`reference/package-first.md`).
- Ambiguous axes — stop and ask one sharp question.
- Ambiguous V₀ — ask or mark kz as relative/uncertain.
- Corrupt/partial file — report readable parts only.

## Workflow

1. Identify artifact (file type, shape, **existing** package/project loaders).
2. Check PyARPES + Python 3.8; if missing/wrong, ask for dedicated venv
   (see Stack policy / `reference/pyarpes-env.md`) before continuing.
3. Try package load/analysis first (`reference/package-first.md`). For
   MAESTRO: prefer sibling **`.fits`** + `location='MAESTRO'` before MH1
   `.h5`; pick main spectrum carefully (`formats-and-axes.md`). If load
   fails or needs new code — **ask** before a custom loader.
4. Lock coordinates — names + units (° vs Å⁻¹, eV, hν; binding vs kinetic).
5. Sanity print — shape, ranges, one mid-cut summary.
6. Reduce — cut / FS / EDC / MDC (see `reference/safe-reduction.md`).
7. **Quick report:** stop at angle-space overviews
   (`default-overview-plots.md`). **Do not** convert to k/kz here.
8. **Analysis / user-requested momentum:** convert to k / kz
   (`reference/k-and-kz-conversion.md`) — EF-aligned energy, provisional or
   user Γ, stated V₀ for kz; save `analysis/kspace/*.npz`; prefer cache reload.
9. If line analysis — fit + optional broadcast (`reference/edc-mdc-fitting.md`).
10. Plot/report with labeled units; **list overview / conversion assumptions**.
   Default overviews: `reference/default-overview-plots.md`
   (cut → 1 dispersion; Fermi / hv–kz → trio; dims win over log for kind).
11. Before expensive batch work — token note (`reference/token-usage.md`).

If unsure: read the matching `reference/` file; ask the user one sharp question.

## References

- `reference/formats-and-axes.md`
- `reference/safe-reduction.md`
- `reference/edc-mdc-fitting.md`
- `reference/k-and-kz-conversion.md` — analysis-mode k/kz + Γ + npz cache
- `reference/failure-modes.md`
- `reference/pyarpes-env.md` — Python 3.8 dedicated venv + install
- `reference/token-usage.md` — when to warn about token cost
- `reference/default-overview-plots.md` — cut / Fermi trio / hv–kz trio + assumptions
- `reference/package-first.md` — use package APIs; ask before new code

## Examples

- `examples/maestro_pyarpes.md`
- `examples/fit_edc_mdc.md`
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
- Fit: EDC/MDC with Gaussian, Lorentzian, or Voigt; `broadcast_model` for
  width vs E, width vs k, or E vs k plots
