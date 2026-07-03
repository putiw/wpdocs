# Software Installation

This page covers installing neuroimaging software on lab machines, with a focus on **monai** (the main Linux VM). Most of the hard-won lessons here came from setting up MELD Graph, but the patterns apply to any tool you install on monai.

For local macOS setup, see [Workstation Setup](workstation-setup.md). For machine addresses and SSH commands, see [Machines and Fileshares](machines-and-fileshares.md).

## Monai Quick Reference

RHEL 8.10, 502 GB RAM, 128 CPUs. SSH on a non-standard port:

```bash
ssh -p 4410 mri@monai.abudhabi.nyu.edu
```

Conda is at `/opt/miniconda3`. FSL is at `/home/mri/fsl`. FreeSurfer 7.2.0 is at `/home/mri/software/freesurfer`.

## The FSL PATH Problem

FSL's installer puts `/home/mri/fsl/bin` on `PATH`. That directory contains its own `python`, `pip`, and `conda` binaries. Because of PATH ordering, these can silently win over the conda environment you think you're using:

```bash
conda activate my_env
which pip
# Expected: /opt/miniconda3/envs/my_env/bin/pip
# Actual:   /home/mri/fsl/bin/pip          <-- WRONG
```

This causes cascading problems: packages install into FSL's site-packages, builds run under FSL's Python (wrong version, missing dependencies), and you get confusing `ModuleNotFoundError` for packages you just installed.

### How to work around it

**Use full absolute paths** to the conda environment's Python and pip. Do not rely on `which python` or bare `pip` after `conda activate`:

```bash
# ALWAYS do this on monai when FSL is installed:
/opt/miniconda3/envs/my_env/bin/python -m pip install <package>
/opt/miniconda3/envs/my_env/bin/python my_script.py
```

This bypasses PATH entirely and guarantees you're using the right interpreter.

**Quick check** — after activating any conda env on monai, verify before installing anything:

```bash
which python && which pip
# If either points to /home/mri/fsl/bin/ — use absolute paths instead.
```

## FreeSurfer bashrc and scp/rsync

FreeSurfer's `SetUpFreeSurfer.sh` prints banner text to stdout when sourced. If you add `source $FREESURFER_HOME/SetUpFreeSurfer.sh` to `~/.bashrc` without a guard, it runs during non-interactive logins too. This breaks `scp` and `rsync` because they interpret the banner as protocol data:

```text
scp: Received message too long 757935405
rsync: error: unexpected tag 94
```

### Fix: wrap in an interactive shell guard

```bash
# In ~/.bashrc on monai:
if [[ $- == *i* ]]; then
    export FREESURFER_HOME=/home/mri/software/freesurfer
    source $FREESURFER_HOME/SetUpFreeSurfer.sh 2>/dev/null
fi
```

The `$- == *i*` check ensures the block only runs in interactive terminals. The `2>/dev/null` suppresses any stray stderr. After this change, `scp` and `rsync` work normally again.

!!! warning "Any tool that prints to stdout in bashrc"
    This is not FreeSurfer-specific. Any software that prints during shell init (ANTS, custom banners, `echo` statements) will break non-interactive SSH commands. Always guard with `[[ $- == *i* ]]`.

## Installing Old PyTorch Versions

Some tools (e.g., MELD Graph) require old PyTorch versions like 1.10.0. These old wheels have been removed from PyPI and the PyTorch pip archive, so `pip install torch==1.10.0` will fail with "No matching distribution found."

**Use conda instead** — old versions are still in the `pytorch` conda channel:

```bash
conda install pytorch==1.10.0 cpuonly -c pytorch
```

General rule: if a `pip install` of an old version fails, try `conda install` from the tool's official channel before giving up.

## Installing Packages That Build C Extensions

Packages like `torch-scatter` compile C/C++ extensions during install. On monai with FSL present, two things go wrong:

1. The build process finds `setuptools` in FSL's site-packages instead of your conda env's, causing import errors.
2. The build runs under FSL's Python (e.g., 3.10) while your env has a different version (e.g., 3.9) with the required dependencies.

**Fix:** use the full path to the conda env's Python:

```bash
/opt/miniconda3/envs/my_env/bin/python -m pip install torch-scatter \
    -f https://data.pyg.org/whl/torch-1.10.0.html
```

This ensures the correct Python version runs the build and finds `torch` in the right site-packages.

## FreeSurfer Installation (Terminal Only)

FreeSurfer can be installed without a GUI. Download the tarball and extract:

```bash
mkdir -p /home/mri/software
cd /home/mri/software
wget https://surfer.nmr.mgh.harvard.edu/pub/dist/freesurfer/7.2.0/freesurfer-linux-centos7_x86_64-7.2.0.tar.gz
tar -xzf freesurfer-linux-centos7_x86_64-7.2.0.tar.gz -C /home/mri/software/
```

Then add the bashrc entry with the interactive guard shown above. Copy your `license.txt` from your local Mac:

```bash
# From local Mac:
scp -P 4410 /Applications/freesurfer/7.4.1/license.txt \
    mri@monai.abudhabi.nyu.edu:/home/mri/software/freesurfer/license.txt
```

!!! note "Version compatibility"
    Different tools require different FreeSurfer versions. MELD Graph needs exactly 7.2.0 (not 7.3+). Multiple versions can coexist — just point `FREESURFER_HOME` to the one you need.

## General Patterns for Monai

### Always use absolute paths

Scripts and documentation should use full absolute paths (`/home/mri/software/...`, `/opt/miniconda3/envs/...`). Relative paths break when you're in the wrong directory, which happens more often than you'd think on a shared VM.

### SSH and rsync always need `-p 4410`

Monai uses port 4410, not the default 22. Every SSH-based command needs the port flag:

```bash
ssh -p 4410 mri@monai.abudhabi.nyu.edu
scp -P 4410 file mri@monai.abudhabi.nyu.edu:/path/    # note: capital -P for scp
rsync -avz -e "ssh -p 4410" src/ mri@monai.abudhabi.nyu.edu:/dest/
```

### Run remote commands non-interactively

When running a pipeline via SSH from your local Mac, remember that `~/.bashrc` guards may skip loading FreeSurfer/FSL. Export paths explicitly in your SSH command:

```bash
ssh -p 4410 mri@monai.abudhabi.nyu.edu 'bash -l -c "
    export FREESURFER_HOME=/home/mri/software/freesurfer
    source \$FREESURFER_HOME/SetUpFreeSurfer.sh
    /opt/miniconda3/envs/meld_graph/bin/python your_script.py
"'
```

Using `bash -l` gets a login shell (loads `~/.bash_profile`), and the explicit exports handle the rest.

### Check `which` before every install

Before running `pip install` or `python setup.py`, always check that `which python` and `which pip` point where you expect. If they point to `/home/mri/fsl/bin/`, use absolute paths.

## Software Currently on Monai

| Software | Version | Path | Notes |
|---|---|---|---|
| FreeSurfer | 7.2.0 | `/home/mri/software/freesurfer` | For MELD Graph. Bashrc-guarded. |
| FSL | (system) | `/home/mri/fsl` | Pre-installed. Hijacks PATH — see above. |
| Miniconda | (system) | `/opt/miniconda3` | System conda. |
| MELD Graph | v2.2.5 | `/home/mri/software/meld_graph` | Conda env: `meld_graph` (Python 3.9, PyTorch 1.10.0). |

## Software on Local Mac

See [Workstation Setup](workstation-setup.md) for the full list. Key installs:

| Software | Path |
|---|---|
| FreeSurfer 7.4.1 | `/Applications/freesurfer/7.4.1` |
| FSL | `/Users/pw1246/fsl` |
| Miniconda | `/Users/pw1246/miniconda3` |
| Docker Desktop | `/usr/local/bin/docker` |
| dcm2niix | `/Users/pw1246/miniconda3/bin/dcm2niix` |

For installing additional tools (ANTs, AFNI, ITK-SNAP, Singularity), follow each tool's official docs and prefer conda or brew where possible. Use a dedicated conda environment for tools with complex or conflicting dependencies.
