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
`theta`, `psi`, …). **User-defined offset always wins.** Never claim “Γ found”
without naming method. Overview mid-detector / mid-scan ≠ Γ for k conversion.

### Cuts (dispersion)

1. Prefer existing `S.offsets` if already set by the loader — label *loader offsets*.
2. Else **provisional** nearest-0° / mid if 0 in range — label
   *provisional:nearest_zero* (not a PyARPES auto-Γ API).
3. User offset overrides; persist and recompute if changed.

### Fermi maps (stricter — package only + ask)

There is **no** general PyARPES `estimate_gamma_from_isoenergy`. Do **not** invent
centroid / argmax / symmetry scripts.

Order:

1. **User offset** if provided → `apply_offsets`; method `user`.
2. Else print existing **`S.offsets`** if present → method `loader_offsets`;
   ask whether to keep or replace.
3. Else, if the isoenergy is clearly a **single pocket**, may call
   **`arpes.analysis.pocket.pocket_parameters`** (package only) and propose its
   center as offsets — label *package:pocket_parameters*; **ask before applying**.
4. Else offer interactive **`arpes.plotting.qt_ktool.ktool`** / `widgets.kspace_tool`
   if the user wants GUI — **ask first** (skill prefers scripted path).
5. Else **STOP and ask** for Γ / normal-emission angles (or a clickable point).
   Do not freestyle a center finder.

Any **new** center-finding code → ask A/B/C (`package-first.md`); do not write it
silently.

## Prerequisites

Before `convert_to_kspace` on a **cut or Fermi map**:

1. **Energy axis notice** (Ek / Eb / E−EF / ambiguous) — same for both.
2. **EF finder** (PyARPES only) → report EF_fit + deviation from 0 → shift EF→0;
   charging warning if claimed E−EF/Eb and `|EF_fit| > 50 meV`.
3. Identify which **angles** map to in-plane momentum — read `.coords`.
4. Set Γ offsets per [Γ policy](#γ--zero-momentum-policy) (Fermi = stricter).
5. State sample geometry in the report.
6. For kz: set or ask for **V₀** (`attrs["inner_potential"]`).

## In-plane k — cut

**Skip** this path if the file is **suspected core-as-2D**
(`reference/default-overview-plots.md`) unless the user overrides to valence.

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

**Energy:** same as cut — axis notice + PyARPES EF finder + EF_fit/deviation +
charging warn at 50 meV + shift EF→0. Do **not** skip EF because “it’s a map.”

**Center / Γ:** follow [Fermi maps (stricter)](#fermi-maps-stricter--package-only--ask)
— package offsets / optional `pocket_parameters` / optional `ktool` / **ask**.
No invent centroid code.

Convert the near-EF isoenergy (or stated window) and/or the volume as the task
requires. Finite energy window ≠ true FS if bands disperse strongly — echo that
caveat.

```python
from arpes.fits.utilities import broadcast_model
from arpes.fits.fit_models import AffineBroadenedFD
from arpes.utilities.conversion import convert_to_kspace
# optional, only if clearly a pocket and user agrees:
# from arpes.analysis.pocket import pocket_parameters

# 1) Energy axis notice (Ek / Eb / E−EF / ambiguous)
# 2) EF finder on a near-EF cut through the map (package only) — same as cut
#    e.g. mid-psi or angle-summed strip → AffineBroadenedFD → shift_by
# 3) Γ: user | S.offsets | ask about pocket_parameters | ask about ktool | STOP ask
#    fmap.S.apply_offsets({...})  # only after user/package path chosen
# 4) Isoenergy or .S.fermi_surface as appropriate
k_fs = convert_to_kspace(fs_slice)  # or full fmap after EF+offsets
# Report: energy axis, EF_fit + meV from 0, gamma_method, grid
```

## hv → kz (Eph stacks)

### Energy on hv stacks

Absolute **Ek differs per hv slice**. After a correct PyARPES load you normally
have:

- `eV` — shared energy coord (intended **Eb / E−EF**), and  
- `hv` — photon energy dim and/or attrs  

Do **not** invent a separate kinetic-energy cube when those exist. KE for
conversion is implied by **hv + EF-aligned `eV`**. If the load still looks like
raw analyzer KE with one grid for all hv → state that; align EF **per hv**
before kz.

Always give the energy-axis notice (Ek / Eb / E−EF / ambiguous).

### EF align across hv (required before analysis / kz)

Mono / undulator drift can move the edge differently at each hv. Use **PyARPES
only** ([Fermi edge corrections](https://arpes.readthedocs.io/en/latest/notebooks/fermi-edge-correction.html)):

```python
from arpes.fits.utilities import broadcast_model
from arpes.fits.fit_models import AffineBroadenedFD

# Near-EF strip (adapt phi / eV windows to data)
edge = hv_scan.sel(eV=slice(-0.1, 0.1), phi=slice(...)).sum("phi")  # example
results = broadcast_model(AffineBroadenedFD, edge, "hv")
# Report EF_fit(hv) and deviation from 0 for each slice (or summary stats)
hv_ef = hv_scan.G.shift_by(
    results.F.p("fd_center"), shift_axis="eV", shift_coords=True
)
```

Do **not** invent a custom per-hv aligner. If the fit fails → ask.

### Slit / Γ offset (after EF align)

Same family as **cut** Γ: `S.apply_offsets` — no invent center finder.

**Default skill rule:** determine the analyzer/slit offset on the **lowest-hv**
slice available (after EF align). Reason: photon-momentum push grows with hv
(soft X-ray). Apply that offset to the **whole** cube, then convert.

If lowest-hv slice is too noisy / no clear normal emission → ask user which
slice or which offset to use.

### V₀ and convert

1. Confirm `hv` present — **Stop** if missing.  
2. Set `attrs["inner_potential"]` = V₀ — **ask or state source**; never silent.  
3. Soft X-ray / photon momentum — see below.  
4. `convert_to_kspace(...);` prefer **periodicity** check; absolute kz depends on V₀.

```python
import numpy as np
from arpes.utilities.conversion import convert_to_kspace

hv_ef.attrs["inner_potential"] = V0  # eV — MUST state
kz_data = convert_to_kspace(
    hv_ef,  # or .S.fermi_surface / appropriate reduction
    # kp=np.linspace(...), kz=np.linspace(...),  # or resolution=
)
```

Typical V₀ ~5–15 eV (material/surface-dependent). Do not silently assume 10 eV.

### Photon momentum (soft X-ray) — gap + beamline defaults

UV ARPES often **neglects** photon momentum. Soft X-ray (rough skill flag:
**hv ≳ 100 eV**, or user says soft X-ray) may need a correction and the
**photon incidence geometry**.

PyARPES tutorials document `convert_to_kspace` + **`inner_potential`**; they do
**not** document a simple public “photon momentum on + incidence angle” switch.
Treat full photon-momentum correction as a **known gap**:

1. Warn the user when hv is in soft X-ray / high-hv range.  
2. Load curated defaults from `reference/beamline-geometry.md` (**ALS MAESTRO**
   and **ALBA LOREA**: default incidence **55°** — ask before use; **SLS** soft
   X-ray ARPES postponed / dark time).  
3. **Ask:** accept default geometry notes / edit numbers / ignore photon
   momentum for this run.
4. If a correction requires **new** code beyond package APIs → ask A/B/C
   (`package-first.md`); do not invent formulas silently.

### Pipeline summary

```text
load hv stack → state energy axis (expect Eb / E−EF + hv)
  → EF align vs hv (broadcast AffineBroadenedFD on hv + shift_by)
  → slit/Γ offset from lowest-hv slice (cut-like; ask if unclear)
  → state V₀
  → soft X-ray? → beamline geometry default + ask (photon momentum)
      (ALS MAESTRO 55°; ALBA LOREA 55° — ask; SLS soft X-ray postponed)
  → convert_to_kspace → analysis/kspace/<stem>_kz.npz
```

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
| **EF finder before cut/Fermi → k** | PyARPES edge fit; always report EF_fit + deviation from 0 |
| **Charging warn** | Claimed E−EF/Eb and \|EF_fit\| > 50 meV |
| **Fermi Γ** | Package offsets / pocket_parameters / ktool / **ask** — no invent center |
| **EF align hv stacks** | `broadcast_model(..., "hv")` + `shift_by` before kz |
| **Slit offset for kz** | Prefer **lowest-hv** slice after EF align (cut-like offsets) |
| **Photon momentum** | Soft X-ray: warn + `beamline-geometry.md` defaults + **ask**; no invent |
| **State V₀** | Before absolute kz; ask if unknown |
| **Prefer periodicity** | Cross-check bands vs hv when possible |
| **No fake Å⁻¹** | Until `convert_to_kspace` runs |
| **State geometry + Γ method** | Named method; user offset wins |
| **User offset wins** | Overrides; update cache |
| **No invented KE matrix / k formulas / center finders** | Package APIs only; ask before new code |
| **Cache after convert** | `analysis/kspace/*.npz` with required meta |

## Common mistakes

See `reference/failure-modes.md`.
