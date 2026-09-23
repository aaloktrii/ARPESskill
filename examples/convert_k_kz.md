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
from arpes.fits.utilities import broadcast_model
from arpes.fits.fit_models import AffineBroadenedFD
from arpes.utilities.conversion import convert_to_kspace
import numpy as np
from pathlib import Path
from datetime import datetime, timezone

spectrum = example_data.photon_energy.spectrum

# Energy axis notice: expect Eb / E−EF + hv (not a single Ek for all slices)

# EF align across hv (required) — PyARPES only
edge = spectrum.sel(eV=slice(-0.1, 0.1))  # adapt; often sum a phi window first
# edge = spectrum.sel(phi=slice(...), eV=slice(-0.1, 0.1)).sum("phi")
results = broadcast_model(AffineBroadenedFD, edge, "hv")
# Report EF_fit(hv) + deviations from 0
spectrum_ef = spectrum.G.shift_by(
    results.F.p("fd_center"), shift_axis="eV", shift_coords=True
)

# Slit / Γ offset: prefer lowest-hv slice (cut-like); user wins; ask if unclear
hv_min = float(spectrum_ef.coords["hv"].min())
low = spectrum_ef.sel(hv=hv_min, method="nearest")
gamma_method = "provisional:lowest_hv"  # or "user" after ask
# low.S.apply_offsets({...}); then apply same offsets to spectrum_ef

V0 = 10.0  # eV — ASK if unknown; state source
spectrum_ef.attrs["inner_potential"] = V0

# Soft X-ray (hv ≳ 100 eV): warn; see reference/beamline-geometry.md (MAESTRO);
# ask accept / edit incidence / ignore photon momentum — no invent angles

kz_data = convert_to_kspace(
    spectrum_ef.S.fermi_surface,
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
    energy_convention=np.array("E-EF_after_per_hv_shift"),
    grid_spec=np.array("kp=linspace(-2,2,500); kz=linspace(3.5,5.2,400)"),
    assumptions=np.array(
        f"V0={V0} eV; offset from lowest hv; photon momentum TBD/ask"
    ),
    created_utc=np.array(datetime.now(timezone.utc).isoformat()),
    skill_ref=np.array("arpes"),
)
```

**Agent narrative:**

- Expect **Eb / E−EF + hv**; no invented KE cube.
- Print **EF_fit(hv)** (or summary) after broadcast align.
- Slit offset from **lowest hv**; user override wins.
- Print **V₀** and source; absolute kz scales with V₀.
- Soft X-ray: `beamline-geometry.md` — MAESTRO / ALBA LOREA propose **55°**
  (ask); SLS soft X-ray postponed.
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
