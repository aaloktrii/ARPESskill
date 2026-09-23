# Package-first policy (do not invent loaders)

This skill drives **existing callables**: **PyARPES** by default, or a
**confirmed user-map** (`backend-capability-map.md`) after the venv offer is
declined. It is not a license to rewrite ARPES infrastructure in `analysis/`.

## Order of preference

1. **PyARPES** public API — `arpes.io.load_data`, endstation plugins,
   `convert_to_kspace`, `broadcast_model` / fit models, etc.
2. **Confirmed user-map** — project functions mapped to capability IDs after
   search + user confirm (`backend-capability-map.md`). Optional persist:
   project `analysis/backend_map.json`.
3. **Already in the user’s project** (PyARPES present) — import and call
   existing loaders/scripts when they are the better path (do not duplicate).
4. **Thin glue only** — short scripts that *call* those APIs and save plots under
   `analysis/` (orchestration, not a new library).
5. **New custom loader / reimplementation** — **only after asking the user**.

When adding a **new skill workflow**, document PyARPES first and **update the
capability inventory** in the same change (living-list rule).

## Before writing new code

**STOP and ask** if you are about to:

- Write a new HDF5/FITS/NeXus loader instead of `arpes.io.load_data` / a plugin
- Reimplement k-conversion, EDC/MDC extract, or peak fitting by hand
- Invent **Doniach–Šunjić** or other XPS lineshapes not in installed
  `arpes.fits.fit_models` (check first; then ask)
- Invent a **Fermi-surface / Γ center finder** (centroid, argmax, custom symmetry)
  instead of `S.apply_offsets`, user input, `pocket_parameters`, or `ktool`
- Invent **photon-momentum** or beamline **incidence** formulas / angles not in
  `beamline-geometry.md` or user/staff input
- Invent **symmetrize**, bare-FD divide, or a **gap/Δ fitter** instead of
  `arpes.analysis.gap` / edge models (`near-ef-gap.md`)
- Copy large chunks of package logic into `analysis/`
- Bypass PyARPES because the first plugin attempt failed

Ask in this shape (PyARPES missing / incomplete):

> PyARPES / package path failed or is incomplete: [exact error / missing
> feature]. I can (A) retry with another official entry point (`location=…`,
> different plugin / `pocket_parameters` / `ktool`), (B) **map helpers already
> in your project** to capability IDs (`backend-capability-map.md`), or (C)
> write a **new** custom helper under `analysis/` (not ideal). Which do you want?

If the user already declined `.venv-arpes`, prefer proposing **(B)** before
inspect-only xarray.

Do **not** start (C) until the user clearly chooses it.
Do **not** call (B) mappings until the user confirms the proposed map.

## Allowed without asking

- Small analysis scripts that **import** package functions and write figures/reports
- One-off `sel` / `isel` / plot after a successful package load
- Sanity prints of `.dims` / `.coords` / `.attrs`

## Not allowed silently

| Bad habit | Correct |
|-----------|---------|
| Custom `maestro_*.py` loader without asking | Report plugin failure; ask A/B/C |
| Hand-rolled Voigt fit when `arpes.fits` exists | Use package fit models |
| DIY angle→k with ad-hoc formulas | Use `convert_to_kspace`; state assumptions |
| DIY FS center / Γ from invent centroid code | Offsets / user / `pocket_parameters` / `ktool` / **ask** |
| DIY symmetrize / bare FD / custom gap Δ | `gap.symmetrize` + resolution-broadened FD; ask if missing |
| Call user project code without confirm | Propose capability map; wait (`backend-capability-map.md`) |
| New workflow with no capability row | Living-list: update inventory same change |
| “PyARPES can’t do MH1” → immediately rewrite | Document limitation; ask before new loader |

## MAESTRO decision tree

1. Look for a **`.fits`** (or `.fit`) for the scan. If present →
   `load_data(path, location="MAESTRO")` (or micro/nano location if known).
2. Pick the main spectrum variable carefully (largest `spectrum*`; skip
   `*_num_*` / monitor-like names) — see `formats-and-axes.md`.
3. If only **MH1 `.h5`** exists (or FITS load fails): quote the error; try
   another official `location=` / plugin entry if documented.
4. Check whether the **user’s project** already has an MH1 / custom helper —
   **use that** before writing anything new.
5. If still stuck → **ask** A/B/C above. Do not silently invent an MH1 loader
   or invent `rot90` / axis renames to “fix” the plot.

## After user approves custom code

- Keep it **minimal**, under the user’s analysis folder.
- Document that it is a **workaround**, not a replacement for the skill’s
  package path.
- Still never invent axis names/units — read from file metadata.
