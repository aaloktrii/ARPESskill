# k and kz conversion

Reference for converting angle-space ARPES data to **in-plane momentum** (k, kx, ky)
and **out-of-plane momentum** (kz) via PyARPES. Requires completed load/inspect and
reduction steps (`reference/safe-reduction.md`); do not convert raw scans without
stating geometry and energy convention.

**PyARPES docs:** [Converting to k-space](https://arpes.readthedocs.io/en/latest/notebooks/converting-to-kspace.html)

## Prerequisites

Before calling `convert_to_kspace`:

1. Confirm **binding vs kinetic** energy (see `reference/formats-and-axes.md`).
2. Identify which **angles** map to in-plane momentum (e.g. `phi`, `theta`, manipulator
   vs analyzer — read from `.coords`, do not invent).
3. State **sample geometry** (normal emission, fixed `hv`, manipulator settings) in
   the analysis log.
4. Align **EF** if relevant (Fermi map or cut near EF should use a stated EF reference).

Never label plot axes as Å⁻¹ unless conversion has actually been run.

## In-plane k (cut / Fermi map)

Use `convert_to_kspace` on an angle-space `DataArray` — a cut, map, or Fermi surface
slice — not on raw multidimensional scans without reduction.

```python
from arpes.utilities.conversion import convert_to_kspace
import numpy as np

kdata = convert_to_kspace(
    cut_or_fs,  # angle-space DataArray
    # pass kp / kx / ky grids as appropriate for the scan type
)
# State: geometry, which angles mapped, EF alignment if relevant
```

**After conversion, report:**

- Which input array was converted (cut, Fermi map, etc.).
- Geometry and angle coordinates used.
- Output coordinate names (`kp`, `kx`, `ky`, …) and units (Å⁻¹).
- EF alignment method if the conversion or plot depends on it.

For Fermi maps, convert the near-EF integrated map or the stated energy slice — not
the full 3D volume unless the task requires it and geometry is locked.

## hv → kz

Photon-energy scans require an **inner potential** V₀ (eV) to convert to absolute kz.
Absolute kz depends on V₀; if V₀ is unknown, ask the user or mark kz as
**relative/uncertain**.

**Always print/state V₀ before trusting absolute kz values.**

```python
spectrum.attrs["inner_potential"] = 10.0  # eV — MUST state to user; ask if unknown
kz_data = convert_to_kspace(
    hv_scan.S.fermi_surface,  # or appropriate hv-dependent array
    kp=np.linspace(-2, 2, 500),
    kz=np.linspace(3.5, 5.2, 400),
)
```

Typical V₀ values are material- and surface-dependent (often ~5–15 eV for metals and
semiconductors). Do not silently assume 10 eV — state the value used and its source
(literature, user input, fit to known kz periodicity).

**When possible**, cross-check kz by verifying **periodicity** (e.g. repeated band
features vs hv) rather than relying on V₀ alone.

## Rules summary

| Rule | Detail |
|------|--------|
| **State V₀** | Print inner potential before reporting absolute kz; ask if unknown |
| **Prefer periodicity check** | Compare band positions across hv when data allow |
| **No fake Å⁻¹ labels** | Angle axes stay in degrees until `convert_to_kspace` runs |
| **State geometry** | Normal emission, sample orientation, which angles mapped |
| **No silent angle→k** | Never treat detector angle as momentum without conversion |

## Common mistakes

See `reference/failure-modes.md` for agent anti-patterns (angle labeled as k, hv scan
plotted as kz without V₀, etc.).
