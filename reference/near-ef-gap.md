# Near-EF cut analysis (metal EF, FD divide, symmetrize)

**Gate:** run only when the user asks for near-EF / metal-referenced EF /
Fermi–Dirac correction / **gap** / **pseudogap** on a **cut** (or a stated EDC
from a cut). Not part of the default quick report.

**Capabilities:** `fit_fermi_edge`, `shift_energy`, `extract_edc_mdc`,
`fd_broadened`, `symmetrize_edc`, `gap_delta_fit` —
`reference/backend-capability-map.md` (update that table if this workflow gains
steps).

**Docs / package:** [Fermi edge corrections](https://arpes.readthedocs.io/en/latest/notebooks/fermi-edge-correction.html) ·
`arpes.analysis.gap` (`symmetrize`, helpers) · fit models
`AffineBroadenedFD` / `FermiDiracModel` (check installed names).

Complete load + cut overview first (`safe-reduction.md`,
`default-overview-plots.md`). Prefer k-space EDCs when already converted
(`k-and-kz-conversion.md`); angle-space OK if labeled provisional.

---

## Pipeline (package only)

```text
1. Metal reference → fit Fermi edge → energy shift
2. Apply same EF offset to sample cut (report ΔE)
3. Optional: divide by resolution-broadened FD (state T + resolution)
4. Select EDC(s) of interest (state k / angle; often near kF)
5. If pseudogap / gap question → symmetrize about E = 0
6. Plot + assumptions; quantify Δ only with a stated method (or ask)
```

### 1–2. Metal reference → sample EF

1. Load **metallic** reference (same beamline / analyzer conditions when possible).
2. Fit the metal edge with package models only, e.g. `AffineBroadenedFD`
   (or `FermiDiracModel` + affine background if that is what is installed).
3. Read fitted **EF** (and, if returned, resolution / T parameters — state them).
4. Shift the **sample** spectrum so EF → 0 (same family as other EF aligns:
   `G.shift_by` / equivalent). Report **ΔE** applied (meV).
5. Assumptions to echo: metal and sample share analyzer EF calibration;
   metal edge ≈ sample edge position for calibration purposes.

Do **not** skip stating T / resolution if they entered the metal fit.

### 3. Divide by resolution-broadened Fermi–Dirac (optional)

Use when the user wants FD-corrected intensity near/above EF (thermal cutoff
removed).

**Required:**

- **Always convolve FD with energy resolution** — bare FD at finite T alone is
  not enough. Prefer package helpers if present (e.g.
  `arpes.analysis.gap.determine_broadened_fermi_distribution` — **check** the
  installed version before claiming the name).
- State **temperature T** and **resolution width** (σ or FWHM — say which).
  Prefer values from the metal-edge fit when available; else ask.
- Build the broadened FD on the **same energy grid** as the data; divide
  intensity by that curve (thin glue OK). Avoid dividing where the FD is
  numerically ~0 (state a cutoff or energy window).

If no clear package helper exists → **ask** (package-first A/B/C) before a
custom broadened-FD script.

FD-divide and symmetrize are **independent options**. Do not force both unless
the user wants both plots.

### 4. EDC of interest

- Extract EDC at a stated **k** (prefer) or angle window.
- For gap / pseudogap work, default interest is near **kF** when known; if kF
  unknown → **ask** or show a few EDCs along the cut and say so.
- Prefer energy axis already EF-aligned (steps 1–2).

### 5. Symmetrize about E = 0 — **when gap / pseudogap**

**Do this if** the user asks about a **gap** or **pseudogap** (or explicitly
asks to symmetrize).

```python
from arpes.analysis.gap import symmetrize

sym = symmetrize(edc)  # energy already EF-aligned; about μ = 0
```

- Result is \(I(E)+I(-E)\) style (package definition) for visualization.
- Reading: double peak ↔ gapped; single peak at EF ↔ filled through EF
  (**under particle–hole symmetry**).
- Always state the **p–h symmetry** assumption when reporting symmetrized EDCs.

Do **not** invent a DIY mirror-add if `symmetrize` exists.

### 6. Quantifying the gap

Symmetrized / FD-divided EDCs are primarily **visualization**.

To **quote Δ** (leading edge, peak–peak, Norman-type, Dynes, …): name the
method and use package models if available. If the method is unclear or not in
the package → **ask** before writing a custom gap fitter.

---

## Assumptions checklist (always echo)

| Item | Require |
|------|---------|
| Metal + sample EF share | Same-run / same analyzer when possible; state if not |
| ΔE from metal fit | Report in meV |
| T | Stated (FD divide and/or metal fit) |
| Resolution | Stated; **convolved into FD** when dividing |
| EDC k / angle | Stated; kF noted if used |
| Symmetrize | Only for gap/pseudogap (or explicit ask); state p–h symmetry |
| Energy axis | EF-aligned for these plots |

---

## Package map

| Step | Prefer |
|------|--------|
| Metal / edge fit | `AffineBroadenedFD` / `FermiDiracModel` (+ affine) via `guess_fit` |
| Shift sample | `G.shift_by` (or same EF-align path as k-conversion) |
| Broadened FD | Package gap helper if present; else ask before custom |
| Symmetrize | `arpes.analysis.gap.symmetrize` |
| Gap Δ fit | Package first; **ask** before new model |

---

## Do not

- Run this pipeline in **quick report** by default.  
- Divide by **bare** FD without resolution broadening.  
- Symmetrize every EDC “for completeness” when the question is not gap/pseudogap.  
- Claim a numerical gap without a named method.  
- Invent symmetrize / broadened-FD / gap-fitter when package APIs exist — ask if missing.  
- Confuse this metal-EF workflow with the **per-file EF finder** used before
  k-conversion (`k-and-kz-conversion.md`) — related idea, different gate.

---

## Checklist before reporting

1. User asked for near-EF / metal EF / FD / gap / pseudogap on a cut.  
2. Metal fit done (or user declined metal and stated alternate EF).  
3. Sample shifted; ΔE reported.  
4. If FD-divide: T + resolution stated; FD **resolution-broadened**.  
5. EDC k/angle stated.  
6. If gap/pseudogap: symmetrized about E=0; p–h symmetry stated.  
7. Plots saved under `analysis/`; assumptions echoed; no invented Δ.
