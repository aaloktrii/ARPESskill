# Example: Load MAESTRO data with PyARPES

**Goal:** Load MAESTRO-like HDF5 data, print axes and units, then plot one cut or
near-EF map. Follow `reference/formats-and-axes.md` and `reference/safe-reduction.md`
steps 1–4 before plotting.

## Preferred: PyARPES loader

Use `arpes.io.load_data` or a project-specific MAESTRO loader. Endstation plugins:
`MAESTROMicroARPESEndstation`, `MAESTRONanoARPESEndstation`.

```python
from arpes.io import load_data

# User path — no real data bundled in this skill repo
# data = load_data("PATH/TO/maestro.h5")

# Step 1–2: state axes + units before any plot
# print(data.dims, list(data.coords), data.attrs)
# for name, coord in data.coords.items():
#     print(name, float(coord.min()), float(coord.max()),
#           coord.attrs.get("units", "?"))
```

After load, confirm **binding vs kinetic** energy and angle coord names from
`.coords` — never invent motor or pixel names.

## Tutorial fallback (no user file)

When no experimental file is available, use PyARPES bundled tutorial data to
demonstrate the inspect → plot workflow:

```python
from arpes.io import example_data

cut = example_data.cut.spectrum

# State axes + units before plot
print(cut.dims, dict(cut.coords))
for name in cut.dims:
    c = cut.coords[name]
    print(f"  {name}: [{float(c.min()):.4g}, {float(c.max()):.4g}] "
          f"{c.attrs.get('units', '?')}")

cut.S.plot()
```

**Agent narrative:** Report dim names, coord ranges, units, and binding vs kinetic
convention before showing or saving any figure.

## xarray / h5py fallback (inspect only)

Use only when PyARPES is unavailable or the file has no PyARPES loader. State
clearly: *"PyARPES not used; xarray/h5py inspect only."*

```python
import h5py

with h5py.File("PATH/TO/maestro.h5", "r") as f:
    def walk(name, obj):
        if isinstance(obj, h5py.Dataset):
            print(name, obj.shape, obj.dtype)
    f.visititems(walk)
```

List groups and datasets; note shapes and dtypes. **Do not invent axis labels**
from dataset order alone. Defer cuts, fits, and k/kz conversion until PyARPES or
unambiguous axis metadata is available.

## Next steps

- Near-EF map: integrate over a stated window (e.g. ±25 meV binding) — see
  `reference/safe-reduction.md` step 4.
- EDC/MDC extraction: `examples/fit_edc_mdc.md`
- k / kz conversion: `examples/convert_k_kz.md`
