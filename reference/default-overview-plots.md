# Default overview plots by scan kind

When the agent (or a catalog script) needs a **default preview image** of an
ARPES file, use these conventions. Goal: show a useful **dispersion** (image),
not a 1D line or a random off-center slice.

Always: **energy on the vertical axis** when plotting 2D intensity.

## Defaults

| Scan kind | How to recognize (typical) | Default overview plot |
|-----------|----------------------------|------------------------|
| **Cut / 1D dispersion** | Single frame along analyzer (`scan` size ≈ 1), or log type “Cut” | Full **detector × energy** image (e.g. `pixel` × `eV`). **Not** a line cut through it. |
| **Fermi map / angle sweep** | Scan motor is deflection / polar / similar (e.g. `Slit_Defl`), many frames | Same **detector × energy** dispersion at the **mid-scan** frame — prefer the index nearest **0°** if that angle is in range; else mid index. State the scan value used. |
| **Photon-energy (EPH / hv) stack** | Scan motor is `hv` / `mono_eV` / Beamline energy with n>1 | Same: **detector × energy** at **mid hv** frame (mid index). State hv used. |
| **XY / spatial map** | Scan motors are `x`,`y` (or similar) | Prefer a **spatial intensity map** (integrated over detector+energy or near-EF window stated in meV) **or** dispersion at mid (x,y) — say which. Linked XY‖dispersion viewer = TensorSpec later; for scripts use one clear default and label it. |

## Rules

1. **Never** default a cut overview to a 1D line (mean over energy or mean over pixel) — that hides band dispersion.
2. **Never** default a Fermi-map overview to an arbitrary edge frame (start of deflection sweep) — use **center / 0°** when possible.
3. Label the figure title with: file stem, hv if known, and which frame (e.g. `Slit_Defl[60]=0°`).
4. Downsample huge axes for PNG previews if needed; keep aspect readable.
5. If the spectrum array is all zeros, say so in the report — empty DAQ, not a plot bug.

## Pseudocode (MH1-style `pixel, eV, scan`)

```python
# cut (n_scan == 1): plot ds[:, :, 0] as image (transpose so eV vertical)
# fermi / eph (n_scan > 1):
#   if scan is deflection and 0 in range: idx = argmin(|scan - 0|)
#   else: idx = n_scan // 2
#   plot ds[:, :, idx] as image (eV vertical)
```

## Token note

Generating overview PNGs for **many** files is fine on disk; avoid pasting every
image into chat — see `token-usage.md`.
