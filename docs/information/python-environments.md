# Python Environments

For the current local macOS setup, command locations, install rules, and environment inventory, see [Workstation Setup](../getting-started/workstation-setup.md).

Use this page for general rules about Python environments. Use
[Environment](environment.md) for the broader mental model that includes Docker,
Singularity/Apptainer, and workflow systems.

## Default Rule

Use the current Miniconda `base` environment for small daily scripts:

```text
CSV cleanup
DICOM header inspection
small nibabel/pydicom utilities
simple plotting
one-off file manifests
```

Create a separate environment when:

- a project has an `environment.yml` or `requirements.txt`
- a package requires an older Python
- installing the package would downgrade core libraries such as `numpy`,
  `scipy`, `torch`, or `tensorflow`
- the workflow must be reproducible later
- the tool has fragile compiled dependencies

## Conda vs venv

Prefer conda for neuroimaging/scientific work because many packages rely on
compiled libraries.

```bash
conda create -n myenv python=3.11
conda activate myenv
conda install numpy scipy pandas matplotlib nibabel
```

Use `venv` only for lightweight pure-Python tools or web/app projects that
already expect Python virtual environments:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

## pip Rule

Always install pip packages through the active Python:

```bash
python -m pip install package-name
```

Avoid:

```bash
pip install package-name
sudo pip install package-name
```

This prevents installing into a different Python than the one used to run the
script.

## Common Neuroimaging Packages

| Package | Use |
|---|---|
| `nibabel` | Read/write NIfTI, MGZ, GIFTI, CIFTI. |
| `pydicom` | Read DICOM headers and metadata. |
| `nilearn` | GLM utilities, masking, plotting, connectomes. |
| `pybids` | Query BIDS datasets and metadata. |
| `nipype` | Legacy workflow interfaces and wrappers. |
| `rsatoolbox`, `brainiak` | RSA and multivariate analysis workflows. |

Install project-specific packages in a project env, not in `base`, if they bring
large dependency trees or strict version pins.

## Jupyter

For notebooks, install `ipykernel` inside the environment you want to use:

```bash
conda activate myenv
python -m pip install ipykernel
python -m ipykernel install --user --name myenv --display-name "Python (myenv)"
```

Then choose `Python (myenv)` from Jupyter.

## Quick Sanity Check

Run this before debugging a Python/package issue:

```bash
echo "$CONDA_DEFAULT_ENV"
which python
python --version
python -m pip --version
python - <<'PY'
import sys
print(sys.executable)
PY
```
