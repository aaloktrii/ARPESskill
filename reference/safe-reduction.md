# Safe reduction checklist

**Reduction** here means producing analysis products from a loaded scan — cuts,
Fermi maps, EDC, MDC — not compressing the file.

Follow this numbered checklist in order. See also `SKILL.md` workflow step 5
and `reference/formats-and-axes.md` for load/inspect details.

## Checklist

1. **Confirm file path and loader**
   - State the absolute or workspace path being analyzed.
   - State which loader was used (`arpes.io.load_data`, project loader, or
     xarray/h5py fallback).
   - If PyARPES is missing, say so and limit scope to inspect-only.
   - **Folder scope:** if analyzing many files, ensure `analysis/manifest.json`
     exists/refreshed first (`reference/folder-manifest.md`); pick the file
     from the manifest when possible.

2. **Print coords + units**
   - Run the sanity print from `reference/formats-and-axes.md` (shape, coord
     names, min/max, units).
   - Confirm **binding vs kinetic** energy; never swap silently.

3. **Select ROI / energy window with explicit values**
   - Define angle, spatial, or momentum bounds with numeric limits (e.g.
     `phi ∈ [−15°, 15°]`, `eV ∈ [−0.5, 0.1] eV binding).
   - Do not use vague ranges ("around EF", "middle of scan") without numbers.

4. **Extract cut OR Fermi map**
   - **Cut:** E–angle or E–k slice at a stated fixed value (e.g. fixed `phi`,
     or fixed `k` after conversion if already in k-space).
   - **Fermi map:** integrate over a near-EF window; **state the window in meV**
     (e.g. `±25 meV` binding relative to EF).
   - Pick one primary product for the current task; name the selection method
     (`sel`, `isel`, PyARPES helper).

5. **Extract EDC or MDC with window stated**
   - **EDC** (energy distribution curve): fixed k or fixed angle; state the
     fixed value and any averaging window (e.g. `±0.02 Å⁻¹` or `±0.5°`).
   - **MDC** (momentum distribution curve): fixed energy; state E and any
     averaging window in eV or meV.
   - Do not extract EDC/MDC before steps 1–3 are satisfied.

6. **Plot with axis labels + units**
   - Label axes with physical names and units (eV, °, Å⁻¹ as appropriate).
   - Title or caption should state binding vs kinetic and key selections.
   - **Default overview** (catalog / first look): follow
     `reference/default-overview-plots.md` —
     **valence cut** → full detector×energy;
     **suspected core-as-2D** (swept + deep/core clues; soft: span ≳10 eV or
     deepest ≳5 eV below EF) → detector×energy **and** angle-integrated EDC;
     **Fermi map** → trio; **hv/kz** → trio.
   - After step 2: if cut-shaped but core-as-2D heuristics fire, **tell the user**
     and do not treat as valence dispersion for k conversion by default.

7. **Only then fit or convert to k/kz**
   - Fitting: see `reference/edc-mdc-fitting.md` (**core** then EDC/MDC);
     name lineshape and background; PyARPES only.
   - k conversion: see `reference/k-and-kz-conversion.md`; state geometry.
   - kz from hv: state inner potential V₀ (or mark as unknown/relative).
   - Near-EF / gap / pseudogap (user-asked): `reference/near-ef-gap.md` — metal
     EF, resolution-broadened FD divide, symmetrize when gap/pseudogap.

Do not skip ahead to fit or k/kz on raw angle–energy data without completing
inspection and at least one reduction product (cut, Fermi map, EDC, or MDC).

## Skip with reason

If a checklist step cannot be completed, **do not silently skip it**. The agent
must state:

- **Which step** is skipped (by number and name).
- **Why** (e.g. missing PyARPES, ambiguous energy convention, file truncated,
  user asked for inspect-only).

Example: *"Skipping step 4 (Fermi map): energy axis is kinetic and `hv` is
unknown; cannot define EF window — ask user for hv or binding convention."*

Partial progress is acceptable when reported honestly; invented axes or units
are not.
