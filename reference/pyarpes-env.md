# PyARPES environment (Python 3.8 venv)

PyARPES (`arpes` on PyPI, v3.0.x) declares:

```text
python_requires = ">=3.8.0,<3.9"
```

So it needs a **dedicated Python 3.8** environment. Do **not** install into the
user’s default 3.10/3.11/3.12 system or project venv — that confuses new users
and the install fails or breaks other work.

## Agent checklist (mandatory)

1. Check whether `arpes` already imports **and** `sys.version_info` is 3.8.x.
2. If missing or wrong Python — **STOP and ask** (see `SKILL.md` Stack policy).
3. On yes: create a **new** venv named for this purpose (default
   `.venv-arpes` in the project root, or `~/arpes-py38-venv` if user prefers).
4. Install with that venv’s `pip` only.
5. Re-verify import + version before any analysis.
6. Run all later analysis commands with that interpreter
   (`.venv-arpes/bin/python`), not bare `python3`.

## Find Python 3.8

```bash
command -v python3.8
python3.8 -V   # expect Python 3.8.x
```

If `python3.8` is missing, tell the user and offer one of:

**macOS (Homebrew)** — if available:

```bash
brew install python@3.8
# then use: $(brew --prefix python@3.8)/bin/python3.8
```

**conda / mamba** (often easiest on modern Macs):

```bash
conda create -n arpes-py38 python=3.8 -y
conda activate arpes-py38
pip install arpes
```

Do not invent a Python 3.8 install path. If none exists, ask the user which
route they want (Homebrew vs conda vs already-installed 3.8 path).

## Create venv + install (preferred when `python3.8` exists)

From the **analysis project root** (not inside the raw data folder):

```bash
# Pick 3.8 explicitly
PY38="$(command -v python3.8)"
"$PY38" -V   # must print 3.8.x

"$PY38" -m venv .venv-arpes
source .venv-arpes/bin/activate   # Windows: .venv-arpes\Scripts\activate

python -V    # must still be 3.8.x
python -m pip install --upgrade pip
python -m pip install arpes

python -c "import arpes, sys; print('arpes OK', sys.version)"
```

Optional scientific helpers if needed later:

```bash
python -m pip install matplotlib h5py xarray
```

(Many come with `arpes`; install extras only if import fails.)

## After install — Cursor / agent

- Prefer running analysis with:  
  `/absolute/path/to/project/.venv-arpes/bin/python`
- Tell the user once: *“Using `.venv-arpes` (Python 3.8) for PyARPES.”*
- If they open a new terminal, remind them to `source .venv-arpes/bin/activate`
  or use the venv python path.

## Wrong Python — what to say

If active Python is 3.9+:

> PyARPES requires Python **3.8.x** only (`>=3.8,<3.9`). Your current
> interpreter is X.Y. I should create a separate `.venv-arpes` with Python 3.8
> and install there — OK?

Do **not** run `pip install arpes` on 3.9+.

## Inspect-only fallback

Only if the user declines the venv/install: xarray + h5py load/inspect
(see `formats-and-axes.md`). No fit / k / kz.
