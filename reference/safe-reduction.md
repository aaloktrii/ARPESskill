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

7. **Only then fit or convert to k/kz**
   - Fitting: see `reference/edc-mdc-fitting.md`; name lineshape and background.
   - k conversion: see `reference/k-and-kz-conversion.md`; state geometry.
   - kz from hv: state inner potential V₀ (or mark as unknown/relative).

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
