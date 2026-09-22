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
2. If PyARPES missing: **xarray + h5py** for load/inspect only; say so.
3. Always state which path was used.
4. TensorSpec / TensorSpec_GUI: out of v1 — if asked, say deferred.

## Hard rules

- Never invent axis names or units.
- Never treat detector angle as momentum without conversion + stated assumptions.
- Never hv→kz without stating inner potential V₀ (or that it is unknown).
- Never claim Γ found without method (manual / fit / model).
- Never report fits without naming lineshape (+ background if used).
- Prefer scripted PyARPES + matplotlib over launching Qt/Bokeh GUIs.
- Prefer existing project loaders before writing new ones.

## Error handling

- Missing PyARPES — suggest `pip install arpes`; limit to load/inspect via xarray.
- Ambiguous axes — stop and ask one sharp question.
- Ambiguous V₀ — ask or mark kz as relative/uncertain.
- Corrupt/partial file — report readable parts only.

## Workflow

1. Identify artifact (file type, shape, existing loaders).
2. Choose stack (PyARPES vs xarray); state which.
3. Lock coordinates — names + units (° vs Å⁻¹, eV, hν; binding vs kinetic).
4. Sanity print — shape, ranges, one mid-cut summary.
5. Reduce — cut / FS / EDC / MDC (see `reference/safe-reduction.md`).
6. If reporting momentum — convert to k (`reference/k-and-kz-conversion.md`).
7. If hv-dependent — convert to kz; state V₀.
8. If line analysis — fit + optional broadcast (`reference/edc-mdc-fitting.md`).
9. Plot/report with labeled units; state assumptions.

If unsure: read the matching `reference/` file; ask the user one sharp question.

## References

- `reference/formats-and-axes.md`
- `reference/safe-reduction.md`
- `reference/edc-mdc-fitting.md`
- `reference/k-and-kz-conversion.md`
- `reference/failure-modes.md`

## Examples

- `examples/maestro_pyarpes.md`
- `examples/fit_edc_mdc.md`
- `examples/convert_k_kz.md`

## Requires (full analysis)

- Python 3.8+
- `pip install arpes` (PyARPES) plus its usual scientific stack
- For load/inspect fallback only: `xarray`, `h5py`

## Key PyARPES paths

See `reference/` for recipes. Common entry points:

- Load: `arpes.io.load_data` or project loaders
- k-space: `convert_to_kspace` on cuts / Fermi maps (state geometry)
- kz: hv/Eph scans with stated inner potential V₀
- Fit: EDC/MDC with Gaussian, Lorentzian, or Voigt; `broadcast_model` for
  width vs E, width vs k, or E vs k plots
