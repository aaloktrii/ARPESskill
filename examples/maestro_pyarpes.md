# Example: Load MAESTRO data with PyARPES

**Goal:** Load ALS MAESTRO data via PyARPES, pick the main spectrum, print axes
and units, then plot one overview. Follow `reference/formats-and-axes.md`,
`reference/package-first.md`, and `reference/default-overview-plots.md`.

## Preferred: FITS + `location='MAESTRO'`

When a scan has both `.fits` and MH1 `.h5`, prefer **FITS**. Pass a MAESTRO
`location=` so the endstation plugin runs.

```python
from arpes.io import load_data

path = "PATH/TO/maestro_scan.fits"  # user path — no data bundled here
ds = load_data(path, location="MAESTRO")
# If micro/nano is known, use the matching location string from PyARPES

# Pick main photoemission image (skip scaler/num channels)
cands = [
    k for k in ds.data_vars
    if "spectrum" in k.lower() and "num" not in k.lower()
]
if not cands:
    # Single DataArray load, or different naming — inspect ds / use spectrum attr
    spectrum = ds if hasattr(ds, "dims") else ds[list(ds.data_vars)[0]]
else:
    key = max(cands, key=lambda k: ds[k].nbytes)
    spectrum = ds[key]

# State axes + units before any plot
print(spectrum.dims, list(spectrum.coords))
for name in spectrum.dims:
    c = spectrum.coords[name]
    print(
        f"  {name}: [{float(c.min()):.4g}, {float(c.max()):.4g}] "
        f"{c.attrs.get('units', '?')}"
    )
```

Confirm **binding vs kinetic** and angle/scan names from `.coords` — never invent
motors. Detector may still be labeled `pixel`; report as loaded.

## Overview plots

**Cuts:** one mid-frame detector × energy image (energy vertical) is enough —
see snippet below.

**Fermi maps / hv–kz stacks:** do **not** stop at one dispersion. Save the
**trio** from `reference/default-overview-plots.md` and link all three in the
report (analyzer or mid-hv dispersion; isoenergy near EF or 1/4-from-top;
perpendicular eV×scan or eV×hv).

### Cut / single-frame sketch

```python
import matplotlib.pyplot as plt

# Discover scan dim (names vary: psi, Slit_Defl, hv, …)
scan_candidates = [d for d in spectrum.dims if d not in ("eV", "pixel", "phi", "ky", "kx")]
# Prefer a known deflection/hv name if present
for preferred in ("psi", "Slit_Defl", "hv", "theta"):
    if preferred in spectrum.dims:
        scan_dim = preferred
        break
else:
    scan_dim = scan_candidates[0] if scan_candidates else None

if scan_dim is None or spectrum.sizes.get(scan_dim, 1) <= 1:
    frame = spectrum
    title_extra = "cut"
else:
    scan = spectrum.coords[scan_dim]
    if float(scan.min()) <= 0 <= float(scan.max()):
        idx = int(abs(scan - 0).argmin())
    else:
        idx = spectrum.sizes[scan_dim] // 2
    frame = spectrum.isel({scan_dim: idx})
    title_extra = f"{scan_dim}={float(scan[idx]):.3g}"

# Energy vertical: put eV first among remaining dims (no invent rot90)
if "eV" in frame.dims:
    other = [d for d in frame.dims if d != "eV"]
    plot_da = frame.transpose("eV", *other)
else:
    plot_da = frame
plot_da.plot()
plt.title(f"{path} | {title_extra}")
```

Do **not** invent `np.rot90` to match another file format. If orientation looks
wrong, check dims/coords and ask the user.

### Fermi / hv trio (outline)

```python
# After identifying scan_dim, det_dim, eV:
# E pick: 0 if in range else E_max - 0.25*(E_max-E_min)
# 1) spectrum.isel({scan_dim: idx0})           -> detector × eV
# 2) spectrum.sel(eV=E, method="nearest")      -> scan × detector  (isoenergy)
# 3) spectrum.isel({det_dim: mid})             -> eV × scan_dim   (perp / hv cut)
# Save three PNGs; titles must state fixed coords + E.
```

## Tutorial fallback (no user file)

```python
from arpes.io import example_data

cut = example_data.cut.spectrum
print(cut.dims, dict(cut.coords))
cut.S.plot()
```

## If only MH1 `.h5` / plugin fails

1. Look for a sibling `.fits` of the same scan.
2. Quote the error from `load_data` on `.h5`.
3. Ask before any custom loader — see `reference/package-first.md`.

## xarray / h5py fallback (inspect only)

Use only when PyARPES is unavailable or the user chose inspect-only. State:
*"PyARPES not used; xarray/h5py inspect only."*

```python
import h5py

with h5py.File("PATH/TO/maestro.h5", "r") as f:
    def walk(name, obj):
        if isinstance(obj, h5py.Dataset):
            print(name, obj.shape, obj.dtype)
    f.visititems(walk)
```

## Next steps

- Near-EF map: integrate over a stated window — `reference/safe-reduction.md`
- EDC/MDC: `examples/fit_edc_mdc.md`
- k / kz: `examples/convert_k_kz.md`
