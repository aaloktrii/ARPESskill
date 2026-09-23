# k and kz conversion

Reference for converting angle-space ARPES data to **in-plane momentum** (k,
kx, ky) and **out-of-plane momentum** (kz) via PyARPES. Also covers **when**
to convert and how to **cache** products under `analysis/`.

**PyARPES docs:** [Converting to k-space](https://arpes.readthedocs.io/en/latest/notebooks/converting-to-kspace.html)

Requires completed load/inspect (`reference/safe-reduction.md`). Never label
axes as Å⁻¹ until `convert_to_kspace` has actually run.

## Modes (important)

| Mode | When | k / kz |
|------|------|--------|
| **Quick report** | Catalogs, first-look overview trios | **Off** — angle-space only (`default-overview-plots.md`) |
| **Analysis / requested report** | User asks for k, kz, momentum axes, or a report that includes conversion | **On** — this document |

If unclear: ask one sharp question. **Do not convert silently** during quick report.

## Energy axis notice (always — load & analysis)

On **every load**, state the energy axis kind to the user. Do **not** silently
relabel.

| Label | Meaning | Typical clues (not absolute) |
|-------|---------|------------------------------|
| **Ek** | Absolute kinetic (analyzer scale) | Range ≫ 0; attrs/units say kinetic |
| **Eb** | Binding (sign conventions vary) | Attrs say binding; may be positive-down |
| **E−EF** | EF-aligned (PyARPES default: ≤0 below EF) | `0` in range; negative below EF |
| **ambiguous** | Cannot tell | Say so; ask one sharp question |

Example line:  
`Energy axis: E−EF (claimed); range [−1.2, 0.05] eV`

## EF finder before cut → k (required)

Before `convert_to_kspace` on a **dispersion cut** (analysis mode):

1. **Always** run a PyARPES Fermi-edge fit — even if the axis is already labeled
   E−EF or Eb (charging / mono drift can move the edge off 0).
2. Use **only** package tools (e.g. `AffineBroadenedFD` via `broadcast_model`
   and/or a mid-cut EDC fit; then `G.shift_by` / documented energy correction).
   **Do not invent** a custom edge fitter or hand-rolled \(k=\ldots\) formulas.
3. Report **EF_fit** and **deviation from 0**:  
   `EF_fit = X eV → |X| = … meV from 0 eV`.
4. Shift so EF → 0, then convert.

| Loaded claim | After EF fit | Tell the user |
|--------------|--------------|---------------|
| **Ek** | Fit + shift to EF=0 | Was kinetic; EF calibrated with PyARPES edge fit (state EF_fit + deviation). **Do not** convert as raw Ek. |
| **E−EF** or **Eb**, \|EF_fit\| ≤ **50 meV** | Shift if needed | Print EF_fit and deviation from 0; no charging flag. |
| **E−EF** or **Eb**, \|EF_fit\| > **50 meV** | Shift to 0 for conversion | Print EF_fit + deviation; **possible sample charging** (or bad energy cal). Warning only — not a proven diagnosis. |

**Charging flag threshold:** `|EF_fit| > 50 meV` when the axis was claimed E−EF or Eb.
Always print the deviation regardless of threshold.

Do **not** invent a separate absolute-KE matrix when data already has EF-aligned
`eV` (after this step) and `hv` in attrs/coords. PyARPES `convert_to_kspace`
uses that spectral model.

| Situation | Action |
|-----------|--------|
| After EF→0 + `hv` present | Proceed to offsets + `convert_to_kspace` |
| EF fit fails / no clear edge | Stop; ask user (metal EF region? insulator?) |
| hv missing when conversion needs it | Stop; ask |
| hv missing on hv-stack (kz) | **Stop** — cannot do absolute kz |

**Work function:** for EF calibration from analyzer KE when needed — not a usual
extra argument to `convert_to_kspace` once EF is at 0.

## Γ / zero momentum (policy)

Mechanism: `data.S.apply_offsets({...})` on present angle motors (`phi`,
`theta`, `psi`, …).

1. **Provisional heuristic** (default for analysis reports): nearest-0° / mid
   if 0 is in range and looks centered; or a simple FS intensity centroid /
   symmetry hint when a 2D map allows. Apply offsets; label
   *provisional Γ — method \<name\>*.
2. **User-defined offset always wins.** Persist user value; report
   *user-defined*. Recompute k if offsets change.
3. Never claim “Γ found” without naming method (`provisional:…` | `user` |
   literature).

Overview mid-detector / mid-scan slices are **not** the same as Γ for k
conversion — do not conflate them.

## Prerequisites

Before `convert_to_kspace` on a cut:

1. **Energy axis notice** (Ek / Eb / E−EF / ambiguous).
2. **EF finder** → report EF_fit + deviation from 0 → shift EF→0; charging
   warning if claimed E−EF/Eb and `|EF_fit| > 50 meV`.
3. Identify which **angles** map to in-plane momentum — read `.coords`.
4. Set Γ offsets (provisional → user override). No auto-Γ API in PyARPES.
5. State sample geometry in the report.
6. For kz: set or ask for **V₀** (`attrs["inner_potential"]`).

## In-plane k — cut

```python
from arpes.fits.utilities import broadcast_model
from arpes.fits.fit_models import AffineBroadenedFD
from arpes.utilities.conversion import convert_to_kspace

# 1) State energy axis kind to user (Ek / Eb / E−EF / ambiguous)

# 2) EF finder (PyARPES only) — even if already labeled E−EF
#    Adapt ROI / broadcast dim to the cut; example pattern from docs:
near_ef = cut.sel(eV=slice(-0.15, 0.1))  # adjust window to data
# Single EDC or broadcast along detector — use package fit models only
results = broadcast_model(AffineBroadenedFD, near_ef, "phi")  # or mid EDC fit
ef_fit = float(results.F.p("fd_center").mean())  # or appropriate reduction
# ALWAYS report: EF_fit and |EF_fit| in meV from 0
# If claimed E−EF/Eb and abs(ef_fit) > 0.05: warn possible charging
cut_ef = cut.G.shift_by(-ef_fit, "eV")  # EF → 0; follow PyARPES shift API

# 3) Provisional or user Γ offsets (no invent auto-Γ)
cut_ef.S.apply_offsets({"phi": phi0})  # keys = dims present
# gamma_method = "provisional:…" | "user"

# 4) Convert — package only
kdata = convert_to_kspace(cut_ef)  # or resolution= / kp=linspace(...)
```

## In-plane k — Fermi map

Same energy + Γ requirements. Convert the near-EF isoenergy (or stated window)
and/or the map as the task requires. Finite energy window ≠ true FS if bands
disperse strongly — echo that caveat.

```python
k_fs = convert_to_kspace(fs_slice)  # near-EF slice or integrated map
```

## hv → kz

1. Γ via analyzer/slit-related offset (same `apply_offsets` API).
2. Confirm `hv` per frame from coords/attrs — **no invented KE matrix** if
   present and energy is EF-aligned.
3. Set inner potential; **state source** (user | guess | literature). Never silent.
4. Convert; prefer **periodicity** cross-check when data allow.
5. Absolute kz **depends on V₀**.

```python
import numpy as np
from arpes.utilities.conversion import convert_to_kspace

hv_scan.attrs["inner_potential"] = V0  # eV — MUST state; ask if unknown
kz_data = convert_to_kspace(
    hv_scan,  # or .S.fermi_surface / appropriate reduction
    # default resolution OK; or user grids:
    # kp=np.linspace(-2, 2, N),
    # kz=np.linspace(kz_lo, kz_hi, M),
)
```

Typical V₀ ~5–15 eV (material/surface-dependent). Do not silently assume 10 eV.

## Output grid / resolution

- **Default:** PyARPES `resolution=` / auto bounds from angle range — reasonable sampling.
- **User override:** explicit `N` or `np.linspace` for `kp` / `kx` / `ky` / `kz`.
- Always state the grid in the report and in npz meta.

## Analysis cache (`analysis/kspace/*.npz`)

After a successful conversion (offsets / V₀ / energy locked), save under the
user’s project so later agents can reload products instead of raw files.

```text
analysis/kspace/<stem>_k.npz    # cut or FS → in-plane k
analysis/kspace/<stem>_kz.npz   # hv stack → kz (+ in-plane as present)
```

### Required npz keys

| Key | Content |
|-----|---------|
| `intensity` | Converted array |
| named axes | e.g. `eV`, `kp`, `kx`, `ky`, `kz` (1D) |
| `dims` | Ordered dim names (object/string array OK) |
| `source_path` | Original raw path |
| `offsets` | Angle offsets used (serialized dict) |
| `gamma_method` | `provisional:<name>` \| `user` \| … |
| `inner_potential` | float; omit or NaN if N/A |
| `hv` | scalar or array |
| `energy_convention` | short string (Ek / Eb / E−EF as claimed + after shift) |
| `ef_fit_eV` | Fitted Fermi edge before shift |
| `ef_deviation_meV` | `|EF_fit| × 1000` from 0 |
| `charging_warning` | bool / flag if claimed E−EF/Eb and \|EF_fit\| > 50 meV |
| `grid_spec` | resolution / linspace description |
| `assumptions` | Free-text echo |
| `created_utc` | ISO timestamp |
| `skill_ref` | e.g. `arpes` + date |

### Save / load sketch

```python
from pathlib import Path
import numpy as np

out = Path("analysis/kspace") / f"{stem}_k.npz"
out.parent.mkdir(parents=True, exist_ok=True)
np.savez_compressed(
    out,
    intensity=np.asarray(kdata.values),
    **{name: np.asarray(kdata.coords[name]) for name in kdata.dims},
    dims=np.array(kdata.dims),
    source_path=np.array(str(raw_path)),
    # serialize offsets / meta as JSON strings if needed
    gamma_method=np.array(gamma_method),
    energy_convention=np.array("EF-aligned binding-like"),
    grid_spec=np.array(grid_spec),
    assumptions=np.array(assumptions_text),
    created_utc=np.array(created_utc),
    skill_ref=np.array("arpes"),
)

# Reload: prefer npz when meta matches current offsets/V₀/grid (or user accepts cache)
# If user changes Γ / V₀ / grid → recompute and overwrite (or versioned name)
```

Quick report never requires npz. Prefer **read analysis npz** over raw when cache
is valid.

## Rules summary

| Rule | Detail |
|------|--------|
| **No k in quick report** | Overviews stay angle-space |
| **State energy axis** | Ek / Eb / E−EF / ambiguous on every load |
| **EF finder before cut→k** | PyARPES edge fit; always report EF_fit + deviation from 0 |
| **Charging warn** | Claimed E−EF/Eb and \|EF_fit\| > 50 meV |
| **State V₀** | Before absolute kz; ask if unknown |
| **Prefer periodicity** | Cross-check bands vs hv when possible |
| **No fake Å⁻¹** | Until `convert_to_kspace` runs |
| **State geometry + Γ method** | Provisional or user; no invent auto-Γ |
| **User offset wins** | Overrides heuristic; update cache |
| **No invented KE matrix / k formulas** | Package `convert_to_kspace` only |
| **Cache after convert** | `analysis/kspace/*.npz` with required meta |

## Common mistakes

See `reference/failure-modes.md`.
