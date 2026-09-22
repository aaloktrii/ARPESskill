# Failure modes

Common agent mistakes in ARPES analysis and the correct behavior. Cross-check against
`SKILL.md` hard rules before reporting results.

| Failure | Correct |
|---------|---------|
| Plot angle axis labeled as k | Convert first with `convert_to_kspace`, or label axes in **degrees** (°) |
| hv scan plotted as kz without V₀ | Set or ask for `inner_potential`; state uncertainty if V₀ unknown |
| Swap binding ↔ kinetic | Check PyARPES convention (binding often ≤0 below EF); state which is used |
| Invent MAESTRO motor names | Read coords/attrs from file — never guess `phi`, `theta`, etc. |
| "Γ is at image center" | No — state method to find Γ (manual pick, fit, symmetry, model) |
| Fit without lineshape | Name Gaussian, Lorentzian, or Voigt (+ background if used) |
| Use TensorSpec APIs in v1 | Defer; use PyARPES for load, reduce, fit, and k/kz |
| Launch QtTool as only path | Prefer scripted PyARPES + matplotlib; GUIs are optional |
| PyARPES missing → silent xarray fallback | **STOP**; ask to create Python **3.8** `.venv-arpes` + install; wait |
| Install PyARPES without asking | Ask first; install only if the user says yes |
| `pip install arpes` on Python 3.9+ / default env | Refuse; create dedicated 3.8 venv (`reference/pyarpes-env.md`) |
| Bare `pip install arpes` hangs on PyQt / qmake | Use conda `pyqt=5` first, then `pip install arpes --no-deps` |
| Loader complains about **h5** / **fits** | Install **h5py** (HDF5 `.h5`) and **astropy** (FITS `.fits`) — not peak-fitting |
| Dump full arrays / whole beamtime into chat | Warn (token note); write scripts + files under `analysis/` instead |
| Cut overview = 1D line / Fermi or hv = single edge frame | Use `default-overview-plots.md`: cut=1; Fermi trio; hv/kz trio (isoenergy + photon-axis dispersion) |
| Silent custom loader when PyARPES fails | Stop; report error; ask before new code (`package-first.md`) |
| Only try MH1 `.h5` when sibling `.fits` exists | Prefer `.fits` + `load_data(..., location='MAESTRO')` first (`formats-and-axes.md`) |
| Blind `S.spectra[0]` / first spectrum var | Skip `*num*`; pick largest `spectrum-*` intensity image |
| Invent `rot90` / rename axes to match another format | Trust loaded coords; ask if display orientation is wrong |
| Trust log “EPH/Cut” over dims for overview kind | **Dims win**; log is comment only (`default-overview-plots.md`) |
| Report omits overview assumptions | Echo defaults + anti-claims (Γ / k / EF / kz) in report header |
| Mid pixel / mid ψ claimed as Γ or E=0 as calibrated EF | Anti-claims in `default-overview-plots.md` assumptions section |
| `convert_to_kspace` during quick-report trios | Quick = angle-space only; k/kz = analysis / user ask (`k-and-kz-conversion.md`) |
| Invent absolute KE matrix when EF-aligned `eV` + `hv` exist | Use PyARPES convention; WF only for EF calibration if needed |
| Γ claimed with no method / ignore user offset | Label provisional heuristic; **user offset wins**; persist in npz |
| Reuse stale k npz after Γ / V₀ / grid change | Recompute and overwrite (or version); meta must match |
| Reimplement fit / k-conversion by hand | Use PyARPES APIs; ask if truly unavailable |

## Additional guidance

- **Angle vs momentum:** detector or manipulator angles are in degrees until
  `convert_to_kspace` produces k coordinates. See `reference/k-and-kz-conversion.md`.
- **Inner potential:** absolute kz from hv scans requires V₀ in
  `spectrum.attrs["inner_potential"]`. If unknown, report relative kz or ask one
  sharp question.
- **Γ (gamma point):** for **overview** plots, mid-frame ≠ Γ. For **k conversion**,
  use provisional heuristic labeled as such, or user offset (wins). Never claim
  Γ without method. See `reference/k-and-kz-conversion.md`.
- **k/kz:** analysis mode only; cache under `analysis/kspace/*.npz`. Quick
  report must not convert.
- **Fits:** every reported fit must name the lineshape and any background model. See
  `reference/edc-mdc-fitting.md`.
- **Stack policy:** v1 uses PyARPES for analysis. If PyARPES is missing, ask to
  install before any fallback. TensorSpec and Qt-based tools are out of scope
  unless the user explicitly chooses inspect-only or future work.

When in doubt, read the matching `reference/` file and ask the user one sharp question
rather than inventing axes, units, or physics assumptions.
