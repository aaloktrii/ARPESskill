# ARPESskill

General LLM agent skill for ARPES analysis via **PyARPES** (v1).

## What this is

Teaches agents to load ARPES data, lock axes/units, extract EDC/MDC,
fit peaks (Gaussian / Lorentzian / Voigt), convert cuts/Fermi maps to
k-space, convert photon-energy scans to kz, and (when asked) run near-EF
cut analysis for gap/pseudogap (metal EF, resolution-broadened FD divide,
symmetrize) — without inventing coordinates or physics assumptions.
Default backend **PyARPES**; optional **user-map** of project functions via
a living capability inventory when PyARPES is declined.

## What this is not (v1)

- Not tied to TensorSpec / TensorSpec_GUI (future separate bridge)
- Not a Python package; not interactive Qt GUI control
- Does not ship MAESTRO data files

## Install

### Cursor
```bash
git clone https://github.com/fawkesdx/ARPESskill.git
mkdir -p ~/.cursor/skills
ln -s "$(pwd)/ARPESskill" ~/.cursor/skills/arpes
```
(Or copy the repo contents into `~/.cursor/skills/arpes/` so that
`~/.cursor/skills/arpes/SKILL.md` exists.)

### Claude Code
Clone the repo and install into Claude Code's skills directory so that
`SKILL.md` is discoverable (same files). If your Claude Code version
uses `~/.claude/skills/`, symlink similarly:
```bash
mkdir -p ~/.claude/skills
ln -s /absolute/path/to/ARPESskill ~/.claude/skills/arpes
```
Confirm the path for your Claude Code version if it differs.

### Other LLM agents
Point the agent at this repo, or inject `SKILL.md` plus needed files
under `reference/` into context.

## Requires (for full analysis)

- **Python 3.8.x** only (PyARPES: `>=3.8,<3.9`)
- Dedicated venv (e.g. `.venv-arpes`) + `pip install arpes` inside it  
  See skill `reference/pyarpes-env.md` — agent should ask before creating it
- For load/inspect fallback only: `xarray`, `h5py`

## Skill layout

What users need:

- `SKILL.md` — entry + hard rules  
- `reference/` — workflows (load, fit, k/kz, near-EF, backend map, …)  
- `examples/` — short recipes  
- `LICENSE` · `README.md`

No planning / design-history folders in this repo.

## Citation / contact

Sandy Adhitia Ekahana
