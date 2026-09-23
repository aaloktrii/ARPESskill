# Example: Convert angle cut to k and hv scan to kz

**Goal:** Analysis-mode conversion only (not quick report). Follow
`reference/k-and-kz-conversion.md`: energy-axis notice, EF finder, provisional
or user Γ, stated V₀ for kz, save `analysis/kspace/*.npz`.

## 0. Mode check

If the task is a **quick catalog / overview trio** — skip this example; stay in
angle space (`default-overview-plots.md`).

## 1. Angle cut → in-plane k

```python
from arpes.io import example_data
from arpes.fits.utilities import broadcast_model
from arpes.fits.fit_models import AffineBroadenedFD
from arpes.utilities.conversion import convert_to_kspace
from pathlib import Path
import numpy as np
from datetime import datetime, timezone

cut = example_data.cut.spectrum  # or user-loaded cut

# --- Energy axis notice (always) ---
# Tell user: Ek / Eb / E−EF / ambiguous from coords+attrs+range
print(cut.dims, list(cut.coords))

# --- EF finder (required before convert; PyARPES only) ---
near_ef = cut.sel(eV=slice(-0.15, 0.1))  # adapt window
results = broadcast_model(AffineBroadenedFD, near_ef, "phi")
ef_fit = float(results.F.p("fd_center").mean())
ef_dev_meV = abs(ef_fit) * 1000.0
# ALWAYS report EF_fit and deviation from 0 eV
# If claimed E−EF/Eb and ef_dev_meV > 50: warn possible charging
cut_ef = cut.G.shift_by(-ef_fit, "eV")

# --- Γ offsets (provisional or user; no auto-Γ API) ---
phi0 = 0.0
gamma_method = "provisional:nearest_zero"  # or "user"
cut_ef.S.apply_offsets({"phi": phi0})

kdata = convert_to_kspace(cut_ef)

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
    energy_convention=np.array("E-EF_after_shift"),
    ef_fit_eV=np.array(ef_fit),
    ef_deviation_meV=np.array(ef_dev_meV),
    charging_warning=np.array(ef_dev_meV > 50.0),
    grid_spec=np.array("pyarpes_default"),
    assumptions=np.array(
        f"EF_fit={ef_fit:.4f} eV ({ef_dev_meV:.1f} meV from 0); "
        f"gamma={gamma_method}"
    ),
    created_utc=np.array(datetime.now(timezone.utc).isoformat()),
    skill_ref=np.array("arpes"),
)
kdata.S.plot()
```

**Agent narrative:** State energy axis kind, EF_fit + deviation from 0 (charging
warn if >50 meV on claimed E−EF/Eb), Γ method, geometry. Å⁻¹ only after convert.

## 1b. Fermi map → in-plane k (sketch)

Same **energy** path as the cut (axis notice + EF finder + shift).

**Γ / center:** do **not** invent a centroid. Order: user offset → existing
`S.offsets` (ask keep?) → optional `pocket_parameters` if clearly a pocket and
user agrees → optional `ktool` if user wants GUI → else **ask** for offsets.
Then `convert_to_kspace` on the near-EF isoenergy / map. Save
`analysis/kspace/<stem>_k.npz` with `gamma_method` and EF meta.

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
| Energy axis | State Ek / Eb / E−EF on load |
| EF before cut→k | Fit + report deviation; charging warn if >50 meV on E−EF/Eb |
| State V₀ | Before absolute kz |
| User Γ wins | Overrides provisional |
| Cache | `analysis/kspace/*.npz` with meta |

See `reference/failure-modes.md`.
