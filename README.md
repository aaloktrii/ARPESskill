# ARPESskill

General LLM agent skill for ARPES analysis via **PyARPES** (v1).

## What this is

Teaches agents to load ARPES data, lock axes/units, extract EDC/MDC,
fit peaks (Gaussian / Lorentzian / Voigt), convert cuts/Fermi maps to
k-space, and convert photon-energy scans to kz — without inventing
coordinates or physics assumptions.

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

- Python 3.8+
- `pip install arpes` (PyARPES) plus its usual scientific stack
- For load/inspect fallback only: `xarray`, `h5py`

## Skill layout

See `SKILL.md` and `reference/`.

## Citation / contact

Sandy Adhitia Ekahana
