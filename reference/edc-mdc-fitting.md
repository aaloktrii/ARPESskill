# EDC and MDC fitting

Reference for PyARPES peak fits on **EDC** (energy distribution curve) and **MDC**
(momentum distribution curve) lines extracted after the safe-reduction checklist
(`reference/safe-reduction.md` step 5). Requires PyARPES; do not fit on raw
angle–energy data without prior inspection and extraction.

**PyARPES docs:** [Curve fitting](https://arpes.readthedocs.io/en/latest/curve-fitting.html)

## Lineshape models

Use one of these PyARPES fit models — state the choice in every report:

| Model | Import path |
|-------|-------------|
| **Gaussian** | `arpes.fits.fit_models.GaussianModel` |
| **Lorentzian** | `arpes.fits.fit_models.LorentzianModel` |
| **Voigt** | `arpes.fits.fit_models.VoigtModel` |

- **Gaussian:** instrument-dominated broadening; width parameter is σ (report FWHM = 2√(2 ln 2) σ if converting).
- **Lorentzian:** lifetime / natural-width dominated; width parameter is HWHM γ (report FWHM = 2γ if needed).
- **Voigt:** convolution of Gaussian and Lorentzian; use when both instrument and lifetime matter.

Pick the lineshape for the physics question; do not default to Voigt without reason.

## Single-curve fit

Fit one extracted EDC or MDC (1D `DataArray` with the correct coordinate):

```python
from arpes.fits.fit_models import VoigtModel  # or GaussianModel, LorentzianModel

# edc: 1D DataArray with energy coordinate
result = VoigtModel().guess_fit(edc)
print(result.fit_report())
# Always report: lineshape name, center, width (FWHM or model σ/γ — state which), amplitude
```

**Reporting rules (always):**

1. **Lineshape name** — e.g. "Voigt", not "peak fit".
2. **Center** — energy (eV) for EDC, momentum (Å⁻¹) for MDC; state binding vs kinetic.
3. **Width** — give the model parameter (σ, γ, or Voigt components) **and** state whether you quote FWHM or σ/γ.
4. **Amplitude** — intensity at peak or area, as returned by the model.
5. **Background** — if the fit includes a linear/polynomial background or composite model, name it explicitly.

Do not claim quasiparticle lifetime (Γ, τ) from a Lorentzian width without stating assumptions (temperature, matrix elements, impurity scattering, whether the width is truly lifetime-limited).

## EDC vs MDC — when to use which

| Curve | Fixed axis | Best for |
|-------|------------|----------|
| **EDC** | fixed k (or angle) | Energy distribution, DOS-like features, gap edges, binding-energy shifts |
| **MDC** | fixed E | Dispersion tracking E(k), band velocity, Fermi-surface crossings |

- **Prefer MDC fits** when building **dispersion E(k)** or tracking a band across momentum.
- **Prefer EDC fits** for **energy-distribution** studies at a single k point (gaps, pseudogaps, spectral weight vs binding energy).

Complete `reference/safe-reduction.md` steps 1–5 before fitting.

## Broadcast fits (dispersion / parameter maps)

Apply the same lineshape along one axis of a 2D cut to track peak position and width:

```python
from arpes.fits.utilities import broadcast_model
from arpes.fits.fit_models import LorentzianModel

# Fit MDCs vs energy (or EDCs vs momentum) — state which mode
fit_results = broadcast_model(LorentzianModel, data_2d, "eV")  # example axis name; use actual coord
# Derive plots: peak center vs k or E; width vs k or E
```

**Axis argument:** pass the coordinate name along which you slice (e.g. `"eV"` to fit an MDC at each energy, or `"pixel"` / momentum dim to fit EDCs vs k). Use the **actual** coord name from `.coords` — never invent it.

**Typical modes:**

- **MDCs vs energy** — `broadcast_model(..., data_k_eV, "eV")` on a k×E cut → center gives **E vs k** (dispersion); width gives **width vs k** at fixed energy slices or vs E depending on slice direction.
- **EDCs vs momentum** — broadcast along k at each energy → **width vs E** at fixed k, or center vs k for weakly dispersing features.

State which mode you used in the report.

## Derived plots (after broadcast)

Always plot at least one derived quantity when broadcast was used:

| Plot | X axis | Y axis | Typical use |
|------|--------|--------|-------------|
| **Center vs momentum** | k (Å⁻¹) | E (eV) | Band dispersion E(k) |
| **Width vs E** | E (eV) | width (eV or Å⁻¹) | Scattering / broadening vs binding energy |
| **Width vs k** | k (Å⁻¹) | width | Momentum-dependent broadening |

Extract fitted parameters from `fit_results` (structure depends on PyARPES version; inspect `.values` / fit components). Label axes with units. State lineshape and whether width is σ, γ, or FWHM.

## Checklist before reporting fits

1. EDC or MDC extracted with stated window (`safe-reduction.md` step 5).
2. Lineshape named (Gaussian, Lorentzian, or Voigt).
3. Background stated if present.
4. Center, width (with σ/γ/FWHM clarity), amplitude reported.
5. For broadcast: mode stated (MDC vs EDC); derived plot(s) described or shown.
6. No lifetime / Γ claims without explicit assumptions.
