# RSA (Representational Similarity Analysis)

Representational Similarity Analysis for fMRI — comparing neural representational geometry to model predictions.

!!! info "Status"
    This page is a detailed outline. The full walkthrough (with code from Haneen's implementation) will be filled in incrementally.

## Overview

RSA compares the representational geometry of brain regions to theoretical models by computing pairwise dissimilarities between neural activity patterns and correlating those with model-predicted dissimilarity matrices (RDMs).

## Pipeline Outline

### 1. Prerequisites

- fMRIPrep-preprocessed BOLD data (in a chosen output space)
- Event/condition labels per run
- ROI masks or searchlight setup
- Python environment with: `rsatoolbox`, `nilearn`, `nibabel`, `scipy`, `numpy`, `pandas`

### 2. Extract Condition-Level Patterns

**Goal:** Go from preprocessed BOLD time series to one beta map per condition.

- Run a first-level GLM (one regressor per condition, per run)
- Or use LSA (Least Squares All) / LSS (Least Squares Separate) for trial-level estimates
- Consider: which confounds to include, whether to z-score within runs

```python
# TODO: Add code from Haneen's implementation
# - GLM setup with nilearn or nipype
# - Beta extraction per condition
```

### 3. Define ROIs or Searchlight

- Atlas-based ROIs (e.g., Schaefer, Harvard-Oxford)
- Functionally-defined ROIs (from a localizer)
- Searchlight spheres (specify radius, typically 3-4 voxels)

```python
# TODO: Add ROI masking code
# - NiftiMasker setup
# - Searchlight sphere definition
```

### 4. Compute Neural RDMs

- Extract pattern vectors for each condition from each ROI
- Compute pairwise dissimilarity (1 - Pearson correlation, or crossnobis/Mahalanobis)
- One RDM per ROI per subject

```python
# TODO: Add RDM computation code
# - rsatoolbox.data.Dataset creation
# - rsatoolbox.rdm.calc_rdm()
# - Crossnobis estimator for noise-normalized distances
```

### 5. Construct Model RDMs

- Theoretical models (e.g., categorical, semantic, perceptual)
- Computational model features (e.g., DNN layer activations)
- Each model produces a predicted RDM

```python
# TODO: Add model RDM construction
# - From category labels
# - From feature vectors
# - rsatoolbox.model setup
```

### 6. Compare Neural and Model RDMs

- Correlation methods: Spearman rank, Kendall tau, Pearson
- Partial correlation to control for competing models
- Noise ceiling estimation

```python
# TODO: Add RSA comparison code
# - rsatoolbox.rdm.compare()
# - Noise ceiling (upper and lower bounds)
```

### 7. Statistical Inference

- Within-subject: bootstrap or permutation of condition labels
- Group-level: one-sample t-test or Wilcoxon on subject-level correlations
- Multiple comparison correction across ROIs
- Searchlight: cluster-based correction

```python
# TODO: Add statistical testing code
# - Permutation framework
# - Group-level stats
```

### 8. Visualization

- RDM heatmaps (with condition labels)
- MDS plots of representational geometry
- Bar plots of model comparisons with noise ceiling
- Searchlight maps projected on brain

```python
# TODO: Add visualization code
# - rsatoolbox.vis
# - nilearn plotting
```

## Key Decisions to Document

- [ ] LSA vs LSS vs multi-condition GLM for beta estimation
- [ ] Which dissimilarity metric (correlation distance vs crossnobis)
- [ ] How to handle multiple runs (average RDMs vs concatenate)
- [ ] Cross-validated vs non-cross-validated distances
- [ ] Noise ceiling method

## Reference Code

This walkthrough will be based on **Haneen's RSA implementation**. When filling in, include:

- Full runnable scripts
- Expected input/output file structure
- Example outputs (RDM plots, stats tables)

## Further Reading

- Kriegeskorte et al. (2008) — Original RSA paper
- Nili et al. (2014) — RSA toolbox paper
- Walther et al. (2016) — Crossnobis estimator
- [rsatoolbox documentation](https://rsatoolbox.readthedocs.io/)
