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
| Core fit without ROI / invent Doniach–Šunjić | Shirley + package peaks; ask if exotic model missing (`edc-mdc-fitting.md`) |
| Broadcast fit with no E vs k / width follow-up plots | Save default derived curves (`edc-mdc-fitting.md`) |
| Quote vF / m* without band model or k window | Linear/Quadratic on centers; state units + window; prefer k-space |
| Treat core-as-2D as valence EDC/MDC broadcast | Angle-integrate; core section first; no default k-convert |
| Use TensorSpec APIs in v1 | Defer; use PyARPES for load, reduce, fit, and k/kz |
| Launch QtTool as only path | Prefer scripted PyARPES + matplotlib; GUIs are optional |
| PyARPES missing → silent xarray fallback | **STOP**; ask `.venv-arpes`; if declined → user-map then inspect-only |
| Call user functions without confirmed map | Propose map; wait (`backend-capability-map.md`) |
| New skill workflow without capability row | Same-change update to `backend-capability-map.md` |
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
| Skip energy-axis notice on load | Always state Ek / Eb / E−EF / ambiguous |
| Convert cut to k without PyARPES EF finder | Fit edge first; report EF_fit + meV from 0; shift EF→0 |
| Claimed E−EF/Eb but \|EF_fit\| > 50 meV, no note | Warn **possible charging**; still print deviation |
| Invent k formula or auto-Γ / FS-center finder | `convert_to_kspace` + `apply_offsets` only; Fermi: ask if no package path |
| Fermi map → k without EF finder | Same energy rules as cut (`k-and-kz-conversion.md`) |
| hv stack → kz without per-hv EF align | Angle-summed near-EF + package fit vs hv + `shift_by` first |
| Mid-φ / single-pixel EDC as default hv EF fit | **Banned**; sum/mean over φ (or wide window) |
| Mean-only EF report for hv stack | Report **per-hv** EF_fit + meV from 0; plot EF_fit vs hv |
| Skip EF QC / post-shift then claim kz FS | Hard-stop on junk/pinned/stderr; verify ≈0 at low/mid/high hv |
| hv npz with only scalar `ef_fit_eV` | Require `ef_fit_per_hv` (+ optional plot path) |
| broadcast_model broken → mid-φ fallback | Loop **summed-φ** EDCs + `AffineBroadenedFD`; ask if still fails |
| Slit offset from high-hv soft X-ray slice only | Prefer **lowest-hv** slice after EF align |
| Invent photon-momentum / incidence angles | Use `beamline-geometry.md`; MAESTRO / ALBA LOREA **55°** default then ask; SLS soft X-ray postponed |
| Invent KE cube when `eV`+`hv` present | Use EF-aligned `eV` + `hv`; no invented matrix |
| Long swept “Cut” treated as valence only | Check core-as-2D heuristics; report image + angle-integrated EDC (`default-overview-plots.md`) |
| Valence k-conversion on suspected core-as-2D | Stop / ask; user must override science kind |
| Re-walk folder / paste full catalog every turn | Build `analysis/manifest.json`; recall later (`folder-manifest.md`) |
| Ignore stale manifest after files change | Refresh rows when mtime/hash differs |
| Treat log “Cut” as kind without dims/heuristics | Dims + core-as-2D rules; log → `log_comment` only |
| Reimplement fit / k-conversion by hand | Use PyARPES APIs; ask if truly unavailable |
| Near-EF FD divide without resolution | Always convolve FD with resolution (`near-ef-gap.md`) |
| Symmetrize every EDC by default | Only for gap/pseudogap (or explicit ask); state p–h symmetry |
| DIY symmetrize / invent gap Δ fitter | `arpes.analysis.gap.symmetrize`; ask before custom Δ |
| Metal EF / gap workflow in quick report | User-asked cut analysis only (`near-ef-gap.md`) |

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
- **Folder inventory:** multi-file work starts with `analysis/manifest.json`
  (`reference/folder-manifest.md`); recall instead of re-cataloging in chat.
- **Fits:** every reported fit must name the lineshape and any background model. See
  `reference/edc-mdc-fitting.md`.
- **Stack policy:** prefer PyARPES; if missing, offer venv then **user-map**
  (`backend-capability-map.md`); inspect-only last. TensorSpec deferred.
  New workflows must update the capability inventory (living list).

When in doubt, read the matching `reference/` file and ask the user one sharp question
rather than inventing axes, units, or physics assumptions.
