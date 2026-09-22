# Example: Convert angle cut to k and hv scan to kz

**Goal:** Convert an angle-space cut to in-plane momentum k, then convert a
photon-energy scan to kz with a stated inner potential V₀. Follow
`reference/k-and-kz-conversion.md` and `reference/safe-reduction.md` steps 1–7.

## 1. Angle cut → in-plane k

Use tutorial data when no user file is available:

```python
from arpes.io import example_data
from arpes.utilities.conversion import convert_to_kspace

cut = example_data.cut.spectrum  # or example_data.map.spectrum

# State axes + units before conversion (angles in degrees, not Å⁻¹)
print(cut.dims, list(cut.coords))
for name in cut.dims:
    c = cut.coords[name]
    print(f"  {name}: [{float(c.min()):.4g}, {float(c.max()):.4g}] "
          f"{c.attrs.get('units', '?')}")

kdata = convert_to_kspace(cut)
# Report: geometry, which angles mapped, output coord names (kp/kx/ky) and units (Å⁻¹)
kdata.S.plot()
```

**Agent narrative:** State sample geometry (normal emission, manipulator settings),
which angle coordinates were mapped, and EF alignment if relevant. Never label plot
axes as Å⁻¹ until `convert_to_kspace` has run.

## 2. hv scan → kz (state V₀)

Photon-energy scans require an **inner potential** V₀ (eV) for absolute kz.
**Absolute kz depends on V₀** — if V₀ is unknown, ask the user or mark kz as
relative/uncertain.

```python
from arpes.io import example_data
from arpes.utilities.conversion import convert_to_kspace
import numpy as np

spectrum = example_data.photon_energy

# MUST state V₀ to user before trusting absolute kz values
spectrum.attrs["inner_potential"] = 10.0  # eV — ask user if unknown; do not assume silently

kz_data = convert_to_kspace(
    spectrum.S.fermi_surface,  # or appropriate hv-dependent array
    kp=np.linspace(-2, 2, 500),
    kz=np.linspace(3.5, 5.2, 400),
)
```

**Agent narrative:**

- Print/state **V₀ = 10.0 eV** (or user-provided value) and its source (literature,
  user input, periodicity fit).
- Explain that **absolute kz scales with V₀** — changing V₀ shifts kz without
  changing in-plane k.
- Typical V₀ for metals/semiconductors: ~5–15 eV; material- and surface-dependent.
- When possible, cross-check kz by verifying **periodicity** (repeated band features
  vs hv) rather than relying on V₀ alone.

## Rules summary

| Rule | Detail |
|------|--------|
| **State V₀** | Print inner potential before reporting absolute kz |
| **No fake Å⁻¹** | Angle axes stay in degrees until conversion runs |
| **State geometry** | Normal emission, sample orientation, which angles mapped |
| **Prefer periodicity** | Compare band positions across hv when data allow |

See `reference/failure-modes.md` for common mistakes (angle labeled as k, hv scan
plotted as kz without V₀).
