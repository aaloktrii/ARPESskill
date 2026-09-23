# Example: near-EF cut — metal EF, FD divide, symmetrize

Follow `reference/near-ef-gap.md` and `reference/package-first.md`.
**Only when the user asks** for near-EF / metal EF / FD / gap / pseudogap.

## Outline

```python
from arpes.io import load_data
from arpes.fits.fit_models import AffineBroadenedFD
from arpes.analysis.gap import symmetrize
# Optional: package broadened-FD helper — check installed name, e.g.
# from arpes.analysis.gap import determine_broadened_fermi_distribution

metal = load_data("path/to/metal_ref", location="MAESTRO")  # or project loader
sample = load_data("path/to/sample_cut", location="MAESTRO")

# 1) Fit metal Fermi edge (package model only)
metal_edge = metal.S.spectrum  # or angle-integrated EDC near EF — state choice
fit = AffineBroadenedFD().guess_fit(metal_edge)
print(fit.fit_report())
# Read EF (and T / resolution if in params); shift sample so EF → 0
# sample_shifted = sample.G.shift_by(...)  # report ΔE in meV

# 2) Optional: divide by resolution-broadened FD (state T + resolution)
# fd = determine_broadened_fermi_distribution(...)  # if available
# corrected = sample_shifted / fd   # guard against FD ~ 0

# 3) EDC of interest (state k / angle; often near kF for gap work)
# edc = sample_shifted.sel(kx=kF, method="nearest")  # real dim names

# 4) If gap / pseudogap → symmetrize about E = 0
# sym = symmetrize(edc)
# Plot raw / FD-divided / symmetrized; echo assumptions (p–h symmetry, T, res)
```

## Report must include

- Metal fit ΔE (meV); T; resolution (and that FD was **resolution-broadened** if divided).
- EDC k/angle; whether symmetrized; p–h symmetry note for gap/pseudogap.
- No numerical Δ unless a **named** method was used (or ask first).

Save figures under `analysis/`. Do not dump full arrays into chat.
