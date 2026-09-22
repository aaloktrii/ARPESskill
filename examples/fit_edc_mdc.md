# Example: Fit EDC and MDC with PyARPES

**Goal:** Extract one EDC from tutorial or user data, fit with a named lineshape,
then sketch a broadcast fit on a 2D cut to derive center vs momentum and width vs
energy. Follow `reference/safe-reduction.md` steps 1–5 and
`reference/edc-mdc-fitting.md`.

## 1. Select data and extract one EDC

```python
from arpes.io import example_data

cut = example_data.cut.spectrum  # or user-loaded DataArray

# State axes + units before extraction
print(cut.dims, list(cut.coords))

# EDC: fixed angle/momentum — use actual coord name from .coords
# Example: select mid-index on the non-energy dim
non_eV = [d for d in cut.dims if d != "eV"][0]
edc = cut.isel({non_eV: cut.sizes[non_eV] // 2})
print(f"EDC at {non_eV}={float(edc.coords[non_eV]):.4g}")
```

## 2. Single-curve fit (name the lineshape)

Pick **Gaussian**, **Lorentzian**, or **Voigt** for the physics question. Always
state the lineshape name in the report — not just "peak fit".

```python
from arpes.fits.fit_models import LorentzianModel  # or VoigtModel, GaussianModel

result = LorentzianModel().guess_fit(edc)
print(result.fit_report())
# Report: lineshape name, center (eV), width (σ/γ/FWHM — state which), amplitude
```

**Voigt alternative:**

```python
from arpes.fits.fit_models import VoigtModel

result = VoigtModel().guess_fit(edc)
print(result.fit_report())
# Agent reply must name "Voigt" and report center, width, amplitude
```

## 3. Broadcast fit on a 2D cut

Apply the same lineshape along one axis to track peak position and width across
the cut. State which mode (MDC vs EDC) you used.

```python
from arpes.fits.utilities import broadcast_model
from arpes.fits.fit_models import LorentzianModel

# Fit MDCs vs energy — use actual coord name from .coords
fit_results = broadcast_model(LorentzianModel, cut, "eV")

# Extract fitted parameters (structure varies by PyARPES version; inspect .values)
# fit_results → peak center vs k or E; width vs k or E
```

**Typical derived plots (describe or show at least one):**

| Plot | X axis | Y axis | Mode |
|------|--------|--------|------|
| Center vs momentum | k (Å⁻¹) | E (eV) | MDC broadcast along `"eV"` → dispersion E(k) |
| Width vs energy | E (eV) | width (eV) | EDC broadcast along k at each energy slice |

```python
import matplotlib.pyplot as plt

# Sketch: extract centers and widths from fit_results, then plot
# centers = ...  # from fit_results
# widths = ...
# plt.plot(k_axis, centers); plt.xlabel("k (Å⁻¹)"); plt.ylabel("E (eV)")
# plt.figure(); plt.plot(e_axis, widths); plt.xlabel("E (eV)"); plt.ylabel("width (eV)")
```

Label axes with units. State lineshape (Lorentzian / Voigt / Gaussian) and whether
width is σ, γ, or FWHM.

## Agent checklist

1. EDC or MDC extracted with stated window (`safe-reduction.md` step 5).
2. **Lineshape named** (Gaussian, Lorentzian, or Voigt) in every reply.
3. Background stated if present.
4. Center, width (with σ/γ/FWHM clarity), amplitude reported.
5. For broadcast: mode stated (MDC vs EDC); derived plot(s) described or shown.
6. No lifetime / Γ claims without explicit assumptions.
