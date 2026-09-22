# Formats and axes

Reference for identifying ARPES file types, locking coordinate names and
units, and inspecting loaded data before any reduction or conversion.

## Common formats

| Format | Typical extension | Notes |
|--------|-------------------|-------|
| **MAESTRO HDF5** | `.h5`, `.hdf5` | ALS MAESTRO endstation scans; load via PyARPES endstation plugins |
| **NeXus** | `.nxs`, `.h5` | Community standard; may embed MAESTRO or other beamlines |
| **Igor** | `.pxp`, `.ibw` | WaveMetrics Igor Pro exports; axis metadata varies by export script |
| **Generic HDF5** | `.h5`, `.hdf5` | Raw or custom layouts; inspect structure before assuming axis names |
| **PyARPES `.nc`** | `.nc` | NetCDF exports from PyARPES pipelines; coords usually preserved |

Prefer **PyARPES** loaders when available. Fall back to xarray + h5py only
for load/inspect (see [xarray fallback](#xarray-fallback) below).

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

- **Binding energy** (BE): often negative below the Fermi level in PyARPES
  conventions; EF ≈ 0 eV binding.
- **Kinetic energy** (KE): measured electron energy; related to BE and `hv` via
  the photoemission equation.

Before any cut, fit, or k conversion:

1. Read which convention the loaded array uses (coord name, attrs, or loader docs).
2. State it in the analysis log.
3. If converting BE ↔ KE, show the formula and `hv` used.
4. **Never** relabel an axis or flip a sign without stating the change.

If the convention is ambiguous, stop and ask one sharp question.

## MAESTRO via PyARPES

ALS MAESTRO data is handled by PyARPES endstation plugins:

- `MAESTROMicroARPESEndstation` — micro-ARPES branch
- `MAESTRONanoARPESEndstation` — nano-ARPES branch

**Load path (preferred):**

```python
from arpes.io import load_data

# Project-specific loader if one exists — use that first
data = load_data("/path/to/maestro_scan.h5")
```

Or register/use the appropriate MAESTRO endstation context per your PyARPES
project setup. Prefer **`arpes.io.load_data`** and existing project loaders over
writing ad-hoc HDF5 parsers.

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

Use only when PyARPES is unavailable or the file has no PyARPES loader.
State clearly: *"PyARPES not used; xarray/h5py inspect only."*

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
