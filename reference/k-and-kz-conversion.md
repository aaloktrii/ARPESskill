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

## Energy convention (PyARPES-first)

Do **not** invent a separate absolute-kinetic-energy matrix when PyARPES data
already has:

- EF-aligned energy coord (`eV`, typically ≤0 below EF), and
- photon energy `hv` in attrs and/or coords.

PyARPES uses **binding-like energy with EF at 0** (docs: kinetic offset so zero
is at Fermi). Together with geometry and `hv`, that is enough for
`convert_to_kspace`.

| Situation | Action |
|-----------|--------|
| Energy EF-aligned (0 in range or stated EF) | Proceed; state convention |
| Binding-like but EF not calibrated | Prefer EF check / ask; may convert with stated caveat |
| Raw analyzer kinetic only | Calibrate EF (WF/hv as needed) **before** conversion; ask if missing |
| hv missing on hv-stack | **Stop** — cannot do absolute kz |

**Work function:** for EF calibration from analyzer KE — not a usual extra
argument to `convert_to_kspace` once EF is set.

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

Before `convert_to_kspace`:

1. Confirm energy convention (table above).
2. Identify which **angles** map to in-plane momentum — read `.coords`, do not invent.
3. Set Γ offsets (provisional → user override).
4. State sample geometry (normal emission, manipulator settings) in the report.
5. For kz: set or ask for **V₀** (`attrs["inner_potential"]`).

## In-plane k — cut

```python
from arpes.utilities.conversion import convert_to_kspace

# Provisional or user offsets (example keys — use dims present on the data)
cut.S.apply_offsets({"phi": phi0, "psi": psi0})  # radians or degrees per PyARPES convention
# Report: gamma_method = "provisional:nearest_zero" | "user"

kdata = convert_to_kspace(cut)  # or pass resolution= / kp=linspace(...)
# State: geometry, offsets + method, output coords (Å⁻¹), grid
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
| `energy_convention` | short string |
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
| **State V₀** | Before absolute kz; ask if unknown |
| **Prefer periodicity** | Cross-check bands vs hv when possible |
| **No fake Å⁻¹** | Until `convert_to_kspace` runs |
| **State geometry + Γ method** | Provisional or user |
| **User offset wins** | Overrides heuristic; update cache |
| **No invented KE matrix** | If EF-aligned `eV` + `hv` present |
| **Cache after convert** | `analysis/kspace/*.npz` with required meta |

## Common mistakes

See `reference/failure-modes.md`.
