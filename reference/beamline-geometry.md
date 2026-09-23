# Beamline geometry defaults (photon incidence)

Curated **defaults** for soft X-ray / photon-momentum discussions during
**hv → kz** conversion. Always **ask the user** before treating numbers as
final. Prefer values from beamline staff / experiment notes over this table.

See `reference/k-and-kz-conversion.md` § Photon momentum.

## How agents use this

1. Identify beamline / endstation from path, attrs, log, or user.  
2. Look up the row below (or say *unknown beamline*).  
3. If hv ≳ **100 eV** (or user says soft X-ray): warn that photon momentum may
   matter; show the default notes; **ask** accept / edit / ignore.  
4. Do **not** invent incidence angles. If the cell says *unverified*, ask.  
5. Do **not** write a custom photon-momentum corrector unless the user chooses
   package-first option (C) after A/B fail.

## Active curated entries

### ALS MAESTRO (BL 7.0.2)

| Field | microARPES (7.0.2.1) | nanoARPES (7.0.2.2) |
|-------|----------------------|---------------------|
| Facility | ALS | ALS |
| Photon energy range (facility) | ~20–1000 eV (EPU7) | ~20–1000 eV (EPU7) |
| Analyzer (typical) | Scienta-family / deflector setup | R4000 with deflectors (beamline notes) |
| Photon incidence (normal emission) | **55°** (skill default — ask before use) | **55°** (skill default — ask before use) |
| Photon-momentum in stock PyARPES | Not a documented one-click switch | Same |
| Skill default action | At soft X-ray hv: propose **55°**; **ask** accept / edit / ignore | Same |

Facility pages confirm endstations and hv range. Incidence **55°** is the
skill default for both micro and nano (user-confirmed); still **ask** before
treating as final, and prefer staff / experiment notes if they disagree.
Record the chosen value in the project manifest / npz `assumptions`.

### ALBA LOREA (BL20) — soft X-ray ARPES active

| Field | Value |
|-------|--------|
| Facility | ALBA Synchrotron |
| Endstation | BL20 LOREA |
| Photon energy range | ~10–1000 eV (VUV + soft X-ray ARPES feasible ~200–600 eV per beamline notes) |
| Analyzer (typical) | Hemispherical (MBS A-1 / LOREA endstation docs) |
| Photon incidence (normal emission) | **55°** (beamline sample-environment table: “Incidence angle for normal emission = 55 deg”) |
| Source | [ALBA LOREA sample environments](https://www.cells.es/en/instruments/beamlines/bl20-lorea/sample-environments-preparation); overview [BL20 LOREA](https://www.cells.es/en/instruments/beamlines/bl20-lorea) |
| Photon-momentum in stock PyARPES | Not a documented one-click switch — still **ask** before any custom correction |
| Skill default action | At soft X-ray hv: propose **55°** incidence default; **ask** accept / edit / ignore |

## Postponed

### SLS soft X-ray ARPES

**Status:** postponed — facility / soft X-ray ARPES in **dark time** (do not
curate incidence defaults until operations resume and numbers are confirmed).

If user data are from SLS soft X-ray ARPES now: say postponed; **ask** for
incidence / geometry from their experiment notes; do not invent SLS angles.

## Adding another beamline

Append a section with: facility, endstation, hv range, analyzer, incidence
(with **source citation**), and whether PyARPES plugin exists. Never copy
angles from an unrelated beamline.
