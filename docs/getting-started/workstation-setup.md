# Workstation Setup

This page documents the current local macOS setup for daily neuroimaging work.

Use this page for "what is installed here and what should I run?" Use [Environment](../information/environment.md) for the general explanation of Python, conda, pip, Docker, Singularity, and Nextflow.

## Current Default

Opening a new terminal starts in Miniconda `base`.

```bash
which python
python --version
which pip
python -m pip --version
```

Expected:

```text
/Users/pw1246/miniconda3/bin/python
Python 3.12.9
/Users/pw1246/miniconda3/bin/pip
pip ... from /Users/pw1246/miniconda3/lib/python3.12/site-packages/pip
```

`python`, `python3`, `pip`, and `pip3` should all resolve to Miniconda while `base` is active.

## Shell Startup

| File | Purpose |
|---|---|
| `/Users/pw1246/.zprofile` | Minimal login-shell setup. Runs Homebrew shell setup only. |
| `/Users/pw1246/.zshrc` | Interactive terminal setup: paths, FSL, FreeSurfer, conda, aliases. |
| `/Users/pw1246/.zshrc.backup-20260613-codex-python-cleanup` | Backup from before the Python cleanup. |
| `/Users/pw1246/.zprofile.backup-20260613-codex-python-cleanup` | Backup from before the Python cleanup. |

The current startup goal:

```text
Miniconda Python wins.
FSL and FreeSurfer commands are available immediately.
Homebrew commands are available.
FSL Python is not used as the daily Python.
Homebrew Python is not used as the daily Python.
```

## Python Locations

| Python source | Path | Use |
|---|---|---|
| Miniconda base | `/Users/pw1246/miniconda3/bin/python` | Daily Python. This is the default. |
| FSL Python | `/Users/pw1246/fsl/bin/python` | Tool-internal. Do not use as daily Python. |
| Homebrew Python | `/opt/homebrew/bin/python3` and `/opt/homebrew/opt/python/libexec/bin/python` | Leave installed for Homebrew tools. Do not use for analysis unless a Homebrew tool specifically needs it. |
| python.org Python | `/Library/Frameworks/Python.framework/Versions/3.12/bin/python3` | Old/manual install. Avoid for daily work. |
| Apple system Python | `/usr/bin/python3` | System Python. Do not modify. |

Use:

```bash
python script.py
python -m pip install package-name
```

Avoid:

```bash
pip install package-name
sudo pip install package-name
```

`python -m pip` is preferred because it installs into the same Python that will run the script.

## Installing Things

| Need | First choice | Notes |
|---|---|---|
| macOS command-line tool | `brew install ...` | Use for tools such as `dcmtk`, `git`, `wget`, `tree`, Java, and app-style utilities. |
| Python/scientific package | `conda install ...` | Use inside the active conda env. Prefer this for compiled scientific packages. |
| Python package not available in conda | `python -m pip install ...` | Use only after confirming the correct env is active. |
| Whole reproducible pipeline | Docker / Singularity / Nextflow | Use when the workflow provides a container or Nextflow pipeline. |

Default rules:

```text
Use brew for terminal programs.
Use conda for Python environments and scientific packages.
Use python -m pip only for Python packages missing from conda.
Do not use FSL's Python for general scripts.
Do not install analysis packages into Homebrew Python.
```

## Neuroimaging Commands

These commands should work immediately in a new terminal.

| Command | Current location | Purpose |
|---|---|---|
| `fslmaths` | `/Users/pw1246/fsl/share/fsl/bin/fslmaths` | FSL image math. |
| `freeview` | `/Applications/freesurfer/7.4.1/bin/freeview` | FreeSurfer viewer. |
| `mri_convert` | `/Applications/freesurfer/7.4.1/bin/mri_convert` | FreeSurfer image conversion. |
| `dcm2niix` | `/opt/homebrew/bin/dcm2niix` | DICOM to NIfTI conversion. |
| `dcmdump` | `/opt/homebrew/bin/dcmdump` | Inspect DICOM headers. |
| `docker` | `/usr/local/bin/docker` | Local container runtime. |
| `nextflow` | `/Users/pw1246/.local/bin/nextflow` | Workflow runner. |
| `java` | `/usr/bin/java` | Required by some workflow tools, including Nextflow. |

Quick check:

```bash
which python pip fslmaths freeview mri_convert dcm2niix dcmdump docker nextflow java
```

## Conda Setup

Current active conda installation:

```text
/Users/pw1246/miniconda3
```

Current conda settings:

| Setting | Value |
|---|---|
| Active default env | `base` |
| `auto_activate_base` | `True` |
| Main channel | `defaults` |
| Channel priority | `flexible` |
| Env dirs | `/Users/pw1246/.conda/envs`, `/Users/pw1246/miniconda3/envs` |

`base` is the daily lightweight Python environment. Keep it usable, but do not turn it into a dumping ground for old or fragile project dependencies.

## Conda Environments

| Environment | Path | Use |
|---|---|---|
| `base` | `/Users/pw1246/miniconda3` | Daily Python. Contains Python 3.12, `numpy`, `pandas`, `scipy`, `matplotlib`, `nibabel`, `pydicom`, `torch`, and `SimpleITK`. |
| `afni_env` | `/Users/pw1246/miniconda3/envs/afni_env` | AFNI-related Python environment. Use only when a workflow requires it. |
| `laminatereport` | `/Users/pw1246/miniconda3/envs/laminatereport` | Laminate/reporting work. Contains common scientific packages including `numpy`, `scipy`, `matplotlib`, and `nibabel`. |
| `synthsr_env` | `/Users/pw1246/miniconda3/envs/synthsr_env` | SynthSR/SynthSeg-style work. Contains older scientific stack and TensorFlow. Keep separate from `base`. |
| `vidtools` | `/Users/pw1246/miniconda3/envs/vidtools` | Video/tooling environment. Use only when needed. |
| `xnat_env` | `/Users/pw1246/miniconda3/envs/xnat_env` | XNAT scripting. Contains `xnat` and `pydicom`. |
| FSL envs | `/Users/pw1246/fsl` and `/Users/pw1246/fsl/envs/*` | FSL/tool-internal conda environments. Do not use for general analysis scripts. |
| Miniforge envs | `/Users/pw1246/miniforge3` and `/Users/pw1246/miniforge3/envs/sam` | Present but not the active conda installation. Ignore unless a specific workflow needs it. |

Activate a special env only when needed:

```bash
conda activate xnat_env
python script.py
conda deactivate
```

## When To Make A New Env

Use `base` for small, normal scripts:

```text
CSV cleanup
DICOM header inspection
small nibabel/pydicom scripts
plotting
simple data transfers
```

Make or use a separate env when:

```text
a tool requires an old Python version
a package wants to downgrade numpy/scipy/tensorflow
a project has a requirements.txt or environment.yml
a workflow is fragile or hard to reinstall
```

## Sanity Checks

Run this after changing shell setup:

```bash
echo $CONDA_DEFAULT_ENV
which python
python --version
which pip
python -m pip --version
which fslmaths
which freeview
which mri_convert
which dcm2niix
which dcmdump
```

Expected:

```text
base
/Users/pw1246/miniconda3/bin/python
Python 3.12.9
/Users/pw1246/miniconda3/bin/pip
/Users/pw1246/fsl/share/fsl/bin/fslmaths
/Applications/freesurfer/7.4.1/bin/freeview
/Applications/freesurfer/7.4.1/bin/mri_convert
/opt/homebrew/bin/dcm2niix
/opt/homebrew/bin/dcmdump
```

## Known Notes

Homebrew Python is installed and may be required by Homebrew-managed tools. That is fine. It should not be the Python used for neuroimaging analysis.

FreeSurfer and FSL are loaded on terminal startup so commands such as `freeview`, `mri_convert`, and `fslmaths` are available immediately.

If `conda list` prints warnings about old `conda-meta/*.json` files, reboot first. If the warning persists, inspect the listed files before removing anything manually.
