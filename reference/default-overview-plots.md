# Default overview plots by scan kind

When the agent (or a catalog / report script) needs **default preview images**,
use these conventions. Goal: useful **images**, not a 1D line or a random
off-center slice.

Always: **energy on the vertical axis** when the plot includes energy
(dispersion / hv-dispersion). Isoenergy maps: state the energy (and window if
integrated).

## Defaults

| Scan kind | How to recognize (**dims first**) | Default overview plot(s) |
|-----------|-----------------------------------|---------------------------|
| **Cut / valence dispersion** | Scan size ≈ 1; **not** flagged as [core-as-2D](#core-level-saved-as-2d-image) | One image: full **detector × energy**. **Not** a 1D line alone. |
| **Core-level as 2D image** | Cut-shaped dims, but [core-as-2D heuristics](#core-level-saved-as-2d-image) fire | **Two** plots: detector×energy **and** angle-integrated EDC |
| **Fermi map / angle sweep** | Deflection / polar scan dim (`psi`, `Slit_Defl`, …) with n>1 | **≥3 images** — [Fermi map trio](#fermi-map-trio-required) |
| **Photon-energy / kz (EPH, hv stack)** | Scan dim is `hv` / `mono_eV` / beamline energy with n>1 | **≥3 images** — [hv / kz trio](#hv--kz-eph-trio-required) |
| **XY / spatial map** | Scan motors `x`,`y` | Spatial intensity map (state energy window) **or** dispersion at mid (x,y) — say which. |

**Kind rule:** classify from **loaded dims / sizes**. Measurement-log text
(“Cut”, “EPH”, “Fermi Map”) is a **comment only** — if log disagrees with dims
(e.g. log says EPH but `hv` size = 1), **dims win** for which overview set to
make. Exception: a cut-shaped file may still be **science-kind**
`core_level_2d` when [core-as-2D](#core-level-saved-as-2d-image) heuristics fire
(log “Cut” does not override that suspicion).

**Hard rule for reports:** if the file is a Fermi map or an hv/kz stack, the
report (or catalog entry) must include **all three** PNGs below — not only the
analyzer dispersion. Also echo the [default overview assumptions](#default-overview-assumptions).

**Quick report stays in angle space.** Do **not** run `convert_to_kspace` for
overview trios — k/kz only in analysis mode (`reference/k-and-kz-conversion.md`).
Do **not** run valence band → k on suspected core-as-2D unless the user says
it is valence.

## Core-level saved as 2D image

DAQ sometimes records a **core-level** (or wide survey) as a **2D detector×energy
image** — same shape as a dispersion cut — especially in **swept** mode over a
**deep / wide** energy window, instead of collapsing to a 1D XPS line. Log may
still say “Cut.”

### Detection (primary = C, soft numeric = B)

**Primary (C):** flag **suspected core-as-2D** when the array is cut-shaped
(scan size ≈ 1) **and**:

- Attrs / log / DAQ mention **swept** (vs fixed), **and**
- Deep/wide energy window **or** log/attrs/comment suggest **core** / element
  edge / XPS-like / high-hv core run.

**Soft numeric flag (B)** — also raise suspicion (even if mode unknown) when
energy is E−EF/Eb-like and either:

- Span \(E_\mathrm{max} - E_\mathrm{min}\) **≳ 10 eV**, or
- Deepest end **≳ 5 eV** below EF (e.g. \(E_\mathrm{min} \lesssim -5\) eV if EF≈0).

Extra clues (strengthen the flag, not required alone): intensity nearly **flat
in angle** with peaks only vs energy; comment names a core level.

Always **tell the user** the suspicion and which clues fired. If the user says
it is valence → treat as valence cut and state that override.

### Quick report (required if suspected)

Save **both**:

1. **Detector × energy** image (energy vertical) — same as a cut overview.
2. **Angle-integrated EDC** — mean (or sum) over detector/angle → intensity vs E
   (the natural 1D core spectrum).

Label kind `core_level_2d` (suspected) in the catalog/report.

### Analysis defaults

- Prefer angle-integrated line for core peak fitting; **ask** before inventing
  an XPS lineshape. Package path: Shirley + Gaussian/Voigt composites —
  `reference/edc-mdc-fitting.md` § Core-level fitting.
- **Do not** default to valence EF→k / band-dispersion workflow.

## Fermi map trio (required)

Save **at least three** PNGs and link/embed them in the report:

| # | Name | What to plot | Fixed coords |
|---|------|--------------|--------------|
| 1 | **Dispersion (analyzer)** | detector × energy | Mid deflection: nearest **0°** if in range, else mid index. Energy vertical. |
| 2 | **Isoenergy (FS-like)** | scan × detector | Energy near EF — see [isoenergy energy pick](#isoenergy-energy-pick-shared). State E (±window if used). |
| 3 | **Perpendicular dispersion** | energy × **scan motor** | Mid detector (`pixel` / `phi`). Energy vertical. Orthogonal to plot #1. |

Example dim names after PyARPES: `(eV, pixel)` @ fixed `psi`; `(psi, pixel)` @
fixed `eV`; `(eV, psi)` @ fixed mid `pixel`. Discover real names from `.dims`.

## hv / kz (EPH) trio (required)

For photon-energy stacks (relative **kz** along hv — do **not** claim absolute
kz without stated V₀):

| # | Name | What to plot | Fixed coords |
|---|------|--------------|--------------|
| 1 | **Dispersion at mid hv** | detector × energy | Mid `hv` index. Energy vertical. |
| 2 | **Isoenergy vs hv** | **hv × detector** (or hv × angle) | Energy near EF — same pick rule as Fermi. State E. |
| 3 | **Dispersion along photon axis** | energy × **hv** | Mid detector. Energy vertical. This is the hv-dependent (kz-like) cut. |

## Isoenergy energy pick (shared)

Prefer near the **Fermi level**. Fallback = **~1/4 of the way down from the top**
of the energy axis (high-energy end of the spectrogram — usually near EF when
binding ≤0 is plotted with EF at the top).

```text
E_min, E_max = energy coord min/max
if 0 is inside [E_min, E_max]:
    use E ≈ 0
    optional: mean over ±25 meV if that window fits; else nearest plane
else:
    # ~1/4 from the top (toward deeper binding / lower KE from E_max)
    E = E_max - 0.25 * (E_max - E_min)
```

Always state the energy (and integration half-width if any) in the figure title.

## Default overview assumptions

These are **process defaults** for first-look / catalog plots — not calibrated
physics. Agents **must list them** (or the subset used) in the report header or
figure captions.

### Defaults in force

| Topic | Default |
|-------|---------|
| Kind | From **dims/sizes**; log text is comment only; cut-shaped + core-as-2D heuristics → `core_level_2d` |
| Core-as-2D detect | Primary: **swept** + deep/core clues; soft: span ≳10 eV or deepest ≳5 eV below EF |
| Core-as-2D plots | Detector×energy **and** angle-integrated EDC |
| Slice pick | **Center**, not peak-find: nearest **0°** deflection if in range else mid index; **mid hv**; **mid detector** (`n//2`) |
| Extra dims | Squeeze at mid index if needed |
| Energy axis | Use loaded `eV` as-is — **no** silent EF / work-function / analyzer recalibration |
| Isoenergy E | ≈0 if in range (optional ±25 meV mean); else ¼-from-top |
| hv / kz overview | Photon-axis view only — **not** absolute kz unless V₀ stated |

### Do **not** claim from these defaults

- Mid deflection / mid detector ≠ **Γ** (no Γ without a stated method).
- `pixel` (or unconverted angle) ≠ **Å⁻¹** until `convert_to_kspace`.
- E≈0 ≠ **calibrated EF** without an EF check.
- Finite isoenergy window ≠ a true Fermi surface if bands disperse strongly in that window.
- hv trio ≠ absolute **kz** without stated V₀.

## Pseudocode sketch

```python
# Fermi: scan_dim = psi / Slit_Defl / … ; det = pixel / phi / …
# 1) spectrum.isel({scan_dim: idx0}).transpose("eV", det)     # analyzer dispersion
# 2) spectrum.sel(eV=E, method="nearest")  or  .sel(eV=slice(...)).mean("eV")
# 3) spectrum.isel({det: mid_det}).transpose("eV", scan_dim) # perpendicular

# hv/kz: same pattern with scan_dim = hv
# 1) mid hv detector×eV
# 2) isoenergy: hv × detector @ E≈EF or 1/4-from-top
# 3) mid detector: eV × hv
```

## Rules

1. **Never** default a **valence** cut overview to a 1D line alone — use the 2D image.
   Suspected **core-as-2D** must include **both** the 2D image and an angle-integrated EDC.
2. **Never** default Fermi/hv overviews to an arbitrary edge frame — use center / 0° / mid hv.
3. **Never** ship a Fermi-map or hv/kz **report with only one** dispersion PNG — complete the trio.
4. Title: stem, hv if known, fixed coords (e.g. `psi=0°`, `E=−0.02±0.025 eV`).
5. Downsample huge axes for PNG previews if needed.
6. All-zero arrays → report empty DAQ, not a plot bug.
7. **Trust loaded coords.** `transpose` so eV is vertical. **No** invented `rot90`.
8. Discover scan / detector dims from `.dims` / `.coords` — do not hard-code one motor name.
9. **Echo assumptions** in the report (see [Default overview assumptions](#default-overview-assumptions)).
10. Kind from dims; log is comment only — except core-as-2D science kind from heuristics above.
11. Do not run valence **k conversion** on suspected core-as-2D unless the user overrides.

## Token note

Many PNGs on disk are fine; avoid dumping every image into chat —
see `token-usage.md`. For catalogs: write all trio files under `analysis/`;
summarize in chat (paths + which E / frame).
