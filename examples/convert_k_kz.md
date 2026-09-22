# Example: Convert angle cut to k and hv scan to kz

**Goal:** Analysis-mode conversion only (not quick report). Follow
`reference/k-and-kz-conversion.md`: EF-aligned energy, provisional or user Γ,
stated V₀ for kz, save `analysis/kspace/*.npz`.

## 0. Mode check

If the task is a **quick catalog / overview trio** — skip this example; stay in
angle space (`default-overview-plots.md`).

## 1. Angle cut → in-plane k

```python
from arpes.io import example_data
from arpes.utilities.conversion import convert_to_kspace
from pathlib import Path
import numpy as np
from datetime import datetime, timezone

cut = example_data.cut.spectrum  # or user-loaded cut

print(cut.dims, list(cut.coords))  # angles in degrees — not Å⁻¹ yet

# Provisional Γ (label method). User-supplied offsets always override.
# Keys must match motors present on the data.
phi0 = 0.0  # provisional: nearest-zero / mid — replace with user value if given
gamma_method = "provisional:nearest_zero"  # or "user"
cut.S.apply_offsets({"phi": phi0})

kdata = convert_to_kspace(cut)  # or resolution= / kp=np.linspace(...)
# Report: geometry, gamma_method, output coords, grid

stem = "example_cut"
out = Path("analysis/kspace") / f"{stem}_k.npz"
out.parent.mkdir(parents=True, exist_ok=True)
np.savez_compressed(
    out,
    intensity=np.asarray(kdata.values),
    **{d: np.asarray(kdata.coords[d]) for d in kdata.dims},
    dims=np.array(kdata.dims),
    source_path=np.array("example_data.cut"),
    gamma_method=np.array(gamma_method),
    energy_convention=np.array("EF-aligned binding-like"),
    grid_spec=np.array("pyarpes_default"),
    assumptions=np.array(f"gamma={gamma_method}; no absolute kz"),
    created_utc=np.array(datetime.now(timezone.utc).isoformat()),
    skill_ref=np.array("arpes"),
)
kdata.S.plot()
```

**Agent narrative:** State energy convention, Γ method (provisional vs user),
geometry, and that axes are Å⁻¹ only after conversion. Prefer reload from npz
next time if meta still matches.

## 2. hv scan → kz (state V₀)

```python
from arpes.io import example_data
from arpes.utilities.conversion import convert_to_kspace
import numpy as np
from pathlib import Path
from datetime import datetime, timezone

spectrum = example_data.photon_energy

# Γ offset (provisional or user) — apply before convert
gamma_method = "provisional:nearest_zero"
# spectrum.S.apply_offsets({...})

V0 = 10.0  # eV — ASK user if unknown; state source (guess/literature/user)
spectrum.attrs["inner_potential"] = V0

# Default grid OK; user may pass N / linspace
kz_data = convert_to_kspace(
    spectrum.S.fermi_surface,
    kp=np.linspace(-2, 2, 500),
    kz=np.linspace(3.5, 5.2, 400),
)

stem = "example_hv"
out = Path("analysis/kspace") / f"{stem}_kz.npz"
out.parent.mkdir(parents=True, exist_ok=True)
np.savez_compressed(
    out,
    intensity=np.asarray(kz_data.values),
    **{d: np.asarray(kz_data.coords[d]) for d in kz_data.dims},
    dims=np.array(kz_data.dims),
    source_path=np.array("example_data.photon_energy"),
    gamma_method=np.array(gamma_method),
    inner_potential=np.array(V0),
    energy_convention=np.array("EF-aligned binding-like"),
    grid_spec=np.array("kp=linspace(-2,2,500); kz=linspace(3.5,5.2,400)"),
    assumptions=np.array(f"V0={V0} eV (stated); absolute kz depends on V0"),
    created_utc=np.array(datetime.now(timezone.utc).isoformat()),
    skill_ref=np.array("arpes"),
)
```

**Agent narrative:**

- Print **V₀** and source; absolute kz scales with V₀.
- No separate KE matrix if `hv` + EF-aligned `eV` are present.
- Prefer periodicity check vs hv when data allow.
- If user changes Γ / V₀ / grid → recompute and overwrite npz.

## Rules

| Rule | Detail |
|------|--------|
| Quick report | No conversion |
| State V₀ | Before absolute kz |
| User Γ wins | Overrides provisional |
| Cache | `analysis/kspace/*.npz` with meta |

See `reference/failure-modes.md`.
