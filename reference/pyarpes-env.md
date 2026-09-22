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

**conda / mamba** (recommended on modern Macs — see full recipe below):

```bash
conda create -n arpes_agent_test python=3.8 -y
conda activate arpes_agent_test
# then follow "Create env + install (recommended on Mac: conda)" below
# (do NOT bare `pip install arpes` first — PyQt5/qmake trap)
```

Do not invent a Python 3.8 install path. If none exists, ask the user which
route they want (Homebrew vs conda vs already-installed 3.8 path).

## Create env + install (recommended on Mac: conda)

Bare `pip install arpes` often **fails or hangs on PyQt5** (tries to compile
from source; needs `qmake`). Prefer **conda for Qt/HDF stacks**, then install
`arpes` without letting pip rebuild PyQt.

```bash
source "$(conda info --base)/etc/profile.d/conda.sh"

conda create -n arpes_agent_test python=3.8 -y
conda activate arpes_agent_test

# Qt + I/O libs as binaries (avoids qmake / source builds)
conda install -c conda-forge "pyqt=5" pyqtgraph h5py netcdf4 xarray \
  matplotlib numpy scipy astropy -y

# Wheel PyQt5 if needed (optional; conda pyqt often enough)
pip install "PyQt5==5.15.10"

# Install arpes WITHOUT re-resolving pinned PyQt5==5.15 (source build trap)
pip install "arpes==3.0.1" --no-deps
pip install "pyqtgraph>=0.12.0,<0.13.0" colorcet pint pandas \
  "numpy>=1.20.0,<2.0.0" "scipy>=1.6.0,<2.0.0" "lmfit>=1.0.0,<2.0.0" \
  scikit-learn "matplotlib>=3.0.3" "bokeh>=2.0.0,<3.0.0" \
  "ipywidgets>=7.0.1,<8.0.0" packaging colorama imageio titlecase tqdm rx dill \
  "ase>=3.17.0,<3.22.0" "numba>=0.53.0,<1.0.0" netCDF4

python -c "import arpes, h5py, astropy, sys; print('OK', sys.version)"
```

**Loader I/O packages (required for real files):**

| Package | Why |
|---------|-----|
| **h5py** | Read **`.h5` / HDF5** (modern MAESTRO) |
| **astropy** | Read **`.fits`** (older MAESTRO / some beamlines) via `astropy.io.fits` |
| **netCDF4** | NetCDF / some exported datasets |

If the loader “complains about h5 and fits”, it usually means those file-type
plugins need **h5py** and **astropy** — not peak-fitting. Install them in the
same env, then retry `load_data`.

## Create venv + install (when `python3.8` exists, no conda)

From the **analysis project root** (not inside the raw data folder):

```bash
PY38="$(command -v python3.8)"
"$PY38" -V   # must print 3.8.x

"$PY38" -m venv .venv-arpes
source .venv-arpes/bin/activate   # Windows: .venv-arpes\Scripts\activate

python -V    # must still be 3.8.x
python -m pip install --upgrade pip
# Prefer binary wheels; if PyQt5 build fails (qmake), switch to conda recipe above
python -m pip install "PyQt5==5.15.10" h5py astropy netCDF4
python -m pip install "arpes==3.0.1" --no-deps
# then install remaining arpes deps (same pip list as conda recipe, minus PyQt5)

python -c "import arpes, h5py, astropy, sys; print('arpes OK', sys.version)"
```

## After install — Cursor / agent

- Prefer running analysis with the env python, e.g.  
  `/opt/homebrew/Caskroom/miniforge/base/envs/arpes_agent_test/bin/python`  
  or `…/project/.venv-arpes/bin/python`
- Tell the user once which interpreter is in use.
- For MAESTRO `.h5`, pass an explicit `location=` (micro/nano) when known.

## Wrong Python — what to say

If active Python is 3.9+:

> PyARPES requires Python **3.8.x** only (`>=3.8,<3.9`). Your current
> interpreter is X.Y. I should create a separate conda env / `.venv-arpes`
> with Python 3.8 and install there — OK?

Do **not** run `pip install arpes` on 3.9+.

## Inspect-only fallback

Only if the user declines the env/install: xarray + h5py load/inspect
(see `formats-and-axes.md`). No fit / k / kz.
