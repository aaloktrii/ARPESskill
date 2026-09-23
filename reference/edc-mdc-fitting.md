# Peak fitting (core, EDC, MDC)

PyARPES-only peak fits. Do **not** invent lineshapes or fit engines. Thin glue
that **calls** package APIs is OK; any new model → ask first (`package-first.md`).

**Docs:** [Curve fitting](https://arpes.readthedocs.io/en/latest/curve-fitting.html) ·
[XPS example](https://arpes.readthedocs.io/en/latest/notebooks/full-analysis-xps.html)

Complete load/inspect first (`safe-reduction.md`). For folder work, prefer
manifest recall (`folder-manifest.md`).

## Shared rules (all fit kinds)

1. Name the **lineshape(s)** and **background** every time.  
2. Report **center**, **width** (state σ / γ / FWHM), **amplitude** (or area).  
3. State energy convention (Ek / Eb / E−EF).  
4. Use **only** package models unless user approves new code.  
5. Save fit params / figures under `analysis/`; do not dump huge residuals into chat
   (`token-usage.md`).

### Package models (common)

| Model | Import | Typical use |
|-------|--------|-------------|
| Gaussian | `arpes.fits.fit_models.GaussianModel` | Core / instrument-dominated |
| Lorentzian | `arpes.fits.fit_models.LorentzianModel` | Lifetime-dominated valence |
| Voigt | `arpes.fits.fit_models.VoigtModel` | Mixed instrument + lifetime |
| Affine background | `arpes.fits.fit_models.AffineBackgroundModel` | Linear/const background |
| Multi-peak | Sum with `prefix=` e.g. `GaussianModel(prefix="a_") + …` | Several lines at once |
| Broadcast | `arpes.fits.utilities.broadcast_model` | Same model along 1+ dims |

**Fermi edge** (not a core/valence peak): `AffineBroadenedFD` / step models — see
k-conversion EF finder; do not use as a core peak model.

**Doniach–Šunjić / exotic XPS shapes:** check whether they exist in the installed
`arpes.fits.fit_models` before claiming. If missing → **ask** before writing a
custom lineshape.

---

## 1. Core-level fitting (first)

### When

- Suspected **core-as-2D** (`default-overview-plots.md`), or user asks for core /
  XPS / element edge fits.
- Input is usually an **angle-integrated** (or spatially selected) **EDC-like**
  curve vs energy — not a dispersing valence band fit.

### Pipeline (package only)

```text
identify core / core-as-2D
  → angle-integrate (mean/sum over detector) if still 2D
  → state energy axis; select ROI in eV around the core(s)
  → optional Shirley: arpes.analysis.shirley.remove_shirley_background
  → compose peaks + background (Gaussian/Voigt + AffineBackgroundModel, …)
  → guess_fit on one curve; report lineshape + params
  → optional broadcast_model across x/y (or other dims) with params hints
  → save results under analysis/
```

```python
from arpes.fits.fit_models import GaussianModel, AffineBackgroundModel
from arpes.analysis.shirley import remove_shirley_background
from arpes.fits.utilities import broadcast_model, result_to_hints

# core_edc: 1D intensity vs eV (after angle integrate if needed)
roi = core_edc.sel(eV=slice(e_lo, e_hi))  # state window
clean = remove_shirley_background(roi)  # optional; state if used

model = (
    AffineBackgroundModel()
    + GaussianModel(prefix="a_")
    + GaussianModel(prefix="b_")
)
result = model.guess_fit(
    clean - clean.min(),  # follow XPS example patterns as appropriate
    params={
        "a_center": {"value": c1},
        "b_center": {"value": c2},
    },
)
print(result.fit_report())

# Optional map broadcast (nano-XPS style) — token note if many pixels
# results = broadcast_model(
#     [GaussianModel, GaussianModel],
#     bkg_removed_map,
#     ["x", "y"],
#     params=result_to_hints(result),
# )
```

### Core reporting extras

- Element / edge name if known (user or log) — do not invent assignments.  
- Shirley (or other) background: **on/off** and ROI.  
- Spin–orbit pair constraints only if user/package params set them — state them.  
- Link to core-as-2D overview PNGs / manifest row when present.

### Do not

- Run valence **k-conversion** as part of core fitting by default.  
- Invent Doniach–Šunjić or SO splitting if not in package / user input.  
- Fit the full deep survey without an explicit energy ROI when many edges exist —
  ask which edge(s).

---

## 2. EDC and MDC fitting

After safe-reduction step 5 (extract EDC/MDC with stated window).

### EDC vs MDC

| Curve | Fixed axis | Best for |
|-------|------------|----------|
| **EDC** | fixed k (or angle) | Energy distribution, gaps, binding shifts |
| **MDC** | fixed E | Dispersion E(k), velocity, FS crossings |

- Prefer **MDC** for band tracking / E(k).  
- Prefer **EDC** for energy-distribution at one k.  
- **Several lines at once:** compose prefixed models on one curve (same as core).

### Single-curve fit

```python
from arpes.fits.fit_models import LorentzianModel  # or VoigtModel, GaussianModel

result = LorentzianModel().guess_fit(edc_or_mdc)
print(result.fit_report())
```

Do not claim quasiparticle lifetime from Lorentzian width without assumptions
(temperature, matrix elements, impurity scattering, etc.).

### Broadcast fits (dispersion / parameter maps)

```python
from arpes.fits.utilities import broadcast_model
from arpes.fits.fit_models import LorentzianModel

fit_results = broadcast_model(LorentzianModel, data_2d, "eV")  # use real dim name
```

- `"eV"` → fit MDCs vs energy (typical dispersion track).  
- Momentum / `phi` / `pixel` → fit EDCs vs that axis.  
- Multi-peak: pass a list of model classes or a composite, with `params=`.

### Derived plots (after broadcast)

| Plot | X | Y | Use |
|------|---|---|-----|
| Center vs momentum | k (Å⁻¹) | E (eV) | Dispersion E(k) |
| Width vs E | E (eV) | width | Broadening vs binding |
| Width vs k | k (Å⁻¹) | width | Momentum-dependent width |

Warn before large broadcasts (`token-usage.md`).

---

## Checklist before reporting

### Core

1. Curve is angle-integrated (or stated spatial ROI), not treated as valence cut.  
2. Energy ROI stated; Shirley on/off stated.  
3. Lineshape(s) + background named; centers/widths/amplitudes reported.  
4. No invent element assignment or exotic lineshape.

### EDC / MDC

1. EDC/MDC extracted with stated window.  
2. Lineshape + background named.  
3. Center/width/amplitude reported (σ/γ/FWHM clear).  
4. Broadcast: mode stated; derived plot described or saved.  
5. No lifetime claims without assumptions.
