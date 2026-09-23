# Formats and axes

Reference for identifying ARPES file types, locking coordinate names and
units, and inspecting loaded data before any reduction or conversion.

## Common formats

| Format | Typical extension | Notes |
|--------|-------------------|-------|
| **MAESTRO HDF5 (MH1)** | `.h5`, `.hdf5` | ALS MAESTRO; needs **h5py**. Stock plugins often fail — if sibling `.fits` exists, prefer FITS first |
| **FITS** | `.fits`, `.fit` | Preferred MAESTRO path for PyARPES (`location='MAESTRO'`); needs **astropy**. **Not** peak-fitting |
| **NeXus** | `.nxs`, `.h5` | Community standard; may embed MAESTRO or other beamlines |
| **Igor** | `.pxp`, `.ibw` | WaveMetrics Igor Pro exports; axis metadata varies by export script |
| **Generic HDF5** | `.h5`, `.hdf5` | Raw or custom layouts; inspect structure before assuming axis names |
| **PyARPES `.nc`** | `.nc` | NetCDF exports from PyARPES pipelines; coords usually preserved |

**Do not confuse:** “fits” in loader messages = **FITS file format**. Peak fitting
(Gaussian / Lorentzian / Voigt) is separate — see `edc-mdc-fitting.md`.

Prefer **PyARPES** loaders when available. If `import arpes` fails, **stop and
ask** the user whether to create a Python 3.8 env and install (see
`pyarpes-env.md`) before continuing. Only after they decline install (or choose
inspect-only) may you use xarray + h5py for load/inspect (see
[xarray fallback](#xarray-fallback) below).

If PyARPES imports but loading `.h5` / `.fits` fails with missing-module errors,
install **`h5py`** (HDF5) and/or **`astropy`** (FITS) into the same env, then
retry. Pass `location=` for MAESTRO when known (e.g. micro/nano endstation).

## Typical dimensions

After load, inspect `.coords` and `.attrs` — never assume names from memory.

| Dimension | Common names | Notes |
|-----------|--------------|-------|
| **Energy** | `eV`, `binding`, `kinetic` | PyARPES often uses **binding energy** (eV), negative below EF |
| **Analyzer angles** | `phi`, `psi`, `beta`, … | Detector/emission angles in **degrees** |
| **Manipulator angles** | `theta`, `beta`, `chi`, … | Sample/manipulator angles in **degrees** |
| **Photon energy** | `hv`, `photon_energy` | Scan axis for hv-dependent or kz work (eV) |
| **Spatial** | `x`, `y` | Beam spot or raster position — mention only; no linked-viewer recipes in v1 |

A single file may stack several of these (e.g. `phi` × `eV`, or `hv` × `phi` × `eV`).

## Units table

| Quantity | Unit | Symbol / convention |
|----------|------|---------------------|
| Angle | degrees | ° |
| Momentum | inverse ångström | Å⁻¹ |
| Energy | electron volt | eV |
| Photon energy | electron volt | eV (same symbol as binding/kinetic energy — check context) |

State units explicitly in plots and when reporting numeric windows.

## Binding vs kinetic — never swap silently

Always **tell the user** which energy axis the load has:

| Label | Meaning |
|-------|---------|
| **Ek** | Absolute kinetic |
| **Eb** | Binding (sign may vary) |
| **E−EF** | EF-aligned (PyARPES: often ≤0 below EF) |
| **ambiguous** | Ask |

Clues: coord name, attrs/units, whether `0` is in range. **Never** silently
relabel or flip sign.

Before any **k conversion** on a cut: run PyARPES **EF finder**, report
`EF_fit` and deviation from 0 eV, shift EF→0; if claimed E−EF/Eb and
`|EF_fit| > 50 meV`, warn **possible charging**. Details:
`reference/k-and-kz-conversion.md`.

If converting BE ↔ KE for other reasons, show the formula and `hv` used.

## MAESTRO via PyARPES

ALS MAESTRO data is handled by PyARPES endstation plugins:

- `MAESTROMicroARPESEndstation` — micro-ARPES branch
- `MAESTRONanoARPESEndstation` — nano-ARPES branch

**File choice (important):** beamline folders often contain both **`.fits`** and
**MH1 `.h5`**. Prefer **`.fits` + PyARPES** when a FITS file exists for the same
scan — that is the path the stock MAESTRO plugins support best. Do not jump to a
custom MH1 HDF5 parser while a sibling `.fits` is unused.

**Load path (preferred):**

```python
from arpes.io import load_data

# Prefer .fits when present; pass location when known
data = load_data("/path/to/maestro_scan.fits", location="MAESTRO")
# Micro/nano variants: location="MAESTRO_MICRO" / "MAESTRO_NANO" if required
```

Prefer **`arpes.io.load_data`** (and any loader already in the user’s project)
over writing ad-hoc HDF5 parsers. If only MH1 `.h5` is available and the
official plugin fails, **ask** before a custom loader — see
`reference/package-first.md`.

### Spectrum selection after load

Loaded objects may expose several intensity arrays. Prefer the main spectrum
variable (often named like `spectrum` / `spectrum-*`). **Skip** companion
arrays whose names suggest counters or monitors (e.g. `*_num_*`, `num_*`).
If several candidates remain, pick the largest intensity array and **state**
which name was used. Do not assume `S.spectra[0]` is the science spectrum.

### Axes after load

FITS-loaded MAESTRO data and MH1 HDF5 layouts can disagree on which axis is
longer or how motors are named. **Trust the loaded `.dims` / `.coords`** —
do not apply silent `rot90` / axis swaps “to look like MH1.” Label pixels vs
angles from attrs; if a detector axis is still in **pixels**, say so — do not
relabel as degrees without conversion metadata.

**Before analysis**, inspect the returned `xr.DataArray` or dataset:

```python
print(data.shape)
print(list(data.coords))
print(data.attrs)
for name, coord in data.coords.items():
    print(name, float(coord.min()), float(coord.max()), coord.attrs.get("units", "?"))
```

Confirm energy convention (binding vs kinetic), angle names, and any `hv` present.
Do not invent motor or pixel names — read them from `.coords` and `.attrs`.

## Sanity print template

Run this (or equivalent) immediately after load and before reduction:

```python
def sanity_print(da, label="data"):
    print(f"=== {label} ===")
    print(f"shape: {da.shape}")
    print(f"dims:  {da.dims}")
    print(f"coords: {list(da.coords)}")
    for name in da.dims:
        c = da.coords[name]
        lo, hi = float(c.min()), float(c.max())
        units = c.attrs.get("units", da.attrs.get(f"{name}_units", "?"))
        print(f"  {name}: [{lo:.4g}, {hi:.4g}] {units}")
    # One mid-index slice summary (adjust dim names to match file)
    if "eV" in da.dims and da.sizes.get("eV", 0) > 0:
        mid = da.sizes["eV"] // 2
        sl = da.isel(eV=mid)
        print(f"  mid-eV slice stats: min={float(sl.min()):.4g}, max={float(sl.max()):.4g}")
    print(f"attrs sample: { {k: da.attrs[k] for k in list(da.attrs)[:8]} }")
```

Adapt dim names to the file. If a expected dim is missing, report that — do not
rename arrays to match expectations.

## xarray fallback

Use **only after** the user declined installing PyARPES (or explicitly asked
for inspect-only), or when a file has no PyARPES loader after PyARPES is
already installed. **Do not** silently choose this path when `import arpes`
fails.
State clearly: *"PyARPES not used; xarray/h5py inspect only (user declined install or chose inspect-only)."*

```python
import h5py

with h5py.File("/path/to/file.h5", "r") as f:
    def walk(name, obj):
        if isinstance(obj, h5py.Dataset):
            print(name, obj.shape, obj.dtype)
    f.visititems(walk)
```

List groups and datasets; note shapes and dtypes. **Do not invent axis labels**
from dataset order alone. If metadata strings exist (`attrs`, JSON sidecars, Igor
wave notes), read those before naming dimensions.

For partial NetCDF or xarray-native files:

```python
import xarray as xr
ds = xr.open_dataset("/path/to/file.nc")
print(ds)
```

Report what is readable; defer cuts, fits, and k/kz conversion until PyARPES
or unambiguous axis metadata is available.
