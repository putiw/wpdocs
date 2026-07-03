# dcm2bids Config Files

A dcm2bids config file is a JSON file that tells dcm2bids which DICOM-derived
sidecars belong to each BIDS output file and what metadata should be added or
changed during conversion. The basic workflow is:

1. Run `dcm2bids_helper` on a representative scan session.
2. Inspect the temporary dcm2niix JSON sidecars.
3. Write one `descriptions` entry per acquisition type.
4. Test on one subject/session before using the config broadly.

The official dcm2bids guide describes the same pattern: each acquisition is
matched by `criteria`, renamed with `datatype`, `suffix`, and
`custom_entities`, and optionally edited with `sidecar_changes`.

## Minimal Structure

Every config starts with a top-level `descriptions` list:

```json
{
  "descriptions": [
    {
      "id": "task_hrf",
      "datatype": "func",
      "suffix": "bold",
      "custom_entities": ["task-hrf", "run"],
      "criteria": {
        "SeriesDescription": "func-bold_task-hrf_run-*"
      },
      "sidecar_changes": {
        "TaskName": "hrf"
      }
    }
  ]
}
```

Common fields:

- `id`: local dcm2bids label used by `IntendedFor`; use stable names such as
  `task_hrf` or `task_rest`.
- `datatype`: BIDS folder, such as `anat`, `func`, `fmap`, or `dwi`.
- `suffix`: BIDS suffix, such as `T1w`, `bold`, `sbref`, or `epi`.
- `custom_entities`: extra BIDS filename entities, such as `task-hrf`, `dir-AP`,
  or `run`. Keep entity order BIDS-compatible.
- `criteria`: matching rules against dcm2niix sidecar fields. All criteria must
  match.
- `sidecar_changes`: metadata to add to or replace in the output JSON sidecar.

## Choosing Criteria

Prefer criteria that identify the sequence across subjects and sessions:

- `SeriesDescription`
- `ProtocolName`
- `SequenceName` or `PulseSequenceName`
- `ImageType`
- acquisition geometry such as `AcquisitionMatrixPE` when fieldmaps differ by
  task or resolution

Use `SeriesNumber` cautiously because repeated scans or extra localizers can
shift numbering. Avoid relying on temporary dcm2niix filenames unless there is no
better identifier.

Keep DICOM matching separate from BIDS output naming. For example, a DICOM
`SeriesDescription` can contain `flex1.6`, but a BIDS `task-` entity should not
contain a dot. Match the DICOM value in `criteria`, then write a BIDS-safe output
label such as `task-flex16` and keep the human-readable task name in `TaskName`.

Example:

```json
{
  "datatype": "fmap",
  "suffix": "epi",
  "custom_entities": ["dir-AP", "run"],
  "criteria": {
    "SeriesDescription": "fmap_acq-se_run-01_dir-AP_topup",
    "AcquisitionMatrixPE": 80
  }
}
```

## Fieldmaps

For fMRIPrep and other downstream tools, fieldmap metadata is usually the most
important part of the config. For PEpolar fieldmaps, the important sidecar fields
are:

- `PhaseEncodingDirection`: for example `j` and `j-`.
- `TotalReadoutTime`: must be correct and should match the acquisition.
- `B0FieldIdentifier`: placed on the fieldmap input sidecars.
- `B0FieldSource`: placed on the BOLD/SBRef/DWI sidecars that should use that
  fieldmap estimate.
- `IntendedFor`: placed on the fieldmap sidecar to point to the scans it should
  correct.

BIDS recommends `B0FieldIdentifier` and `B0FieldSource` for expressing fieldmap
intent. BIDS also supports `IntendedFor`; using both is useful for compatibility
with tools that still rely on `IntendedFor`.

Before assigning a fieldmap to a BOLD run, compare geometry from the DICOM or
dcm2niix sidecars. The distortion pattern is geometry-dependent, so check:

- phase-encoding axis and polarity pair
- matrix size
- pixel spacing / field of view
- slice thickness and spacing
- orientation
- number of slices
- first slice position

Different `TotalReadoutTime` values between the fieldmap images and the BOLD run
do not automatically invalidate a PEpolar pair; preserve the scanner-derived
metadata and let the downstream tool use the correct values for each input.

## B0FieldIdentifier and B0FieldSource

`B0FieldIdentifier` names a fieldmap estimation group. All images used together
to estimate the same B0 field should share the same identifier. The target scans
then refer to that identifier with `B0FieldSource`.

Use simple identifier values with letters, numbers, and underscores only.
Do not use hyphens in these values.

Good:

```json
"B0FieldIdentifier": "pepolarhrf"
```

Also good:

```json
"B0FieldIdentifier": "pepolar_hrf"
```

Avoid:

```json
"B0FieldIdentifier": "pepolar-hrf"
```

The hyphenated value is legal-looking JSON, but it can break downstream software
that converts the identifier into an internal Python/Nipype attribute name. For
example, fMRIPrep may try to access an input named like `in_pepolar-hrf`, which
is not a valid trait attribute. Keep the value consistent between
`B0FieldIdentifier` and `B0FieldSource`.

Example PEpolar config:

```json
{
  "descriptions": [
    {
      "id": "task_hrf_bold",
      "datatype": "func",
      "suffix": "bold",
      "custom_entities": ["task-hrf", "run"],
      "criteria": {
        "SeriesDescription": "func-bold_task-hrf_run-*"
      },
      "sidecar_changes": {
        "TaskName": "hrf",
        "B0FieldSource": "pepolarhrf"
      }
    },
    {
      "id": "task_hrf_sbref",
      "datatype": "func",
      "suffix": "sbref",
      "custom_entities": ["task-hrf", "run"],
      "criteria": {
        "SeriesDescription": "func-bold_task-hrf_run-*_sbref"
      },
      "sidecar_changes": {
        "TaskName": "hrf",
        "B0FieldSource": "pepolarhrf"
      }
    },
    {
      "datatype": "fmap",
      "suffix": "epi",
      "custom_entities": ["dir-AP", "run"],
      "criteria": {
        "SeriesDescription": "fmap_acq-se_run-01_dir-AP_topup"
      },
      "sidecar_changes": {
        "B0FieldIdentifier": "pepolarhrf",
        "IntendedFor": ["task_hrf_bold", "task_hrf_sbref"]
      }
    },
    {
      "datatype": "fmap",
      "suffix": "epi",
      "custom_entities": ["dir-PA", "run"],
      "criteria": {
        "SeriesDescription": "fmap_acq-se_run-02_dir-PA_topup"
      },
      "sidecar_changes": {
        "B0FieldIdentifier": "pepolarhrf",
        "IntendedFor": ["task_hrf_bold", "task_hrf_sbref"]
      }
    }
  ]
}
```

## IntendedFor

In dcm2bids, `IntendedFor` is added through `sidecar_changes` on the fieldmap
description. The values can refer to `id` values from other descriptions, and
dcm2bids resolves them to the converted BIDS files.

```json
{
  "id": "task_rest",
  "datatype": "func",
  "suffix": "bold",
  "custom_entities": "task-rest",
  "criteria": {
    "SeriesDescription": "func-bold_task-rest_run-*"
  },
  "sidecar_changes": {
    "TaskName": "rest",
    "B0FieldSource": "pepolarrest"
  }
}
```

```json
{
  "datatype": "fmap",
  "suffix": "epi",
  "custom_entities": "dir-AP",
  "criteria": {
    "SeriesDescription": "fmap_acq-se_run-01_dir-AP_topup"
  },
  "sidecar_changes": {
    "B0FieldIdentifier": "pepolarrest",
    "IntendedFor": ["task_rest"]
  }
}
```

Keep the distinction clear:

- BIDS filenames and paths still contain hyphens, such as `sub-0248`,
  `ses-03`, and `task-hrf`. Do not remove these.
- B0 identifier values should avoid hyphens, such as `pepolarhrf` or
  `pepolar_hrf`.
- `IntendedFor` paths, when written directly, should point to `.nii` or
  `.nii.gz` BIDS files, not JSON sidecars. Current BIDS also supports BIDS URI
  values such as `bids::sub-0248/ses-01/func/..._bold.nii.gz`; use the form
  expected by the downstream tool and BIDS validator version you are targeting.

## XNAT dcm2bids Pipeline Configs

The XNAT dcm2bids pipeline looks for config files in the project-level
`resources/configs` resource. For a project such as
`bi_MRI_QualityControl_2026`, the archive path looks like:

```text
/Volumes/CTP-XNAT/xnat-main/xnat-data/archive/<PROJECT>/resources/configs/
```

The resource can contain more than one config file, for example:

```text
resources/configs/
  config.json
  config3.json
  configs_catalog.xml
```

Use multiple config files when different sessions in the same XNAT project have
different acquisition protocols. This is common when scanner protocols change
mid-project, when one session has a special task, or when fieldmap geometry
differs between sessions.

Recommended pattern:

1. Keep the broad/default protocol in `config.json`.
2. Add separate configs for protocol variants, such as `config_hrf.json`,
   `config_ses03.json`, or `config_v2.json`.
3. Make each config internally complete for the session it is meant to convert.
4. Use clear names and keep a short note in the project README or XNAT resource
   description explaining which sessions use which config.
5. Before running fMRIPrep, verify that each converted session has matching
   `B0FieldIdentifier`, `B0FieldSource`, and `IntendedFor` metadata.

Do not try to force one very broad config to cover incompatible protocols if the
matching criteria become ambiguous. Separate config files are easier to test and
safer for XNAT sessions that genuinely differ.

## Validation Checklist

Before running the XNAT pipeline or a batch conversion:

- The config is valid JSON: `jq empty config.json`.
- Each important acquisition matches exactly one description.
- Every `func` BOLD has `TaskName`.
- Fieldmaps have correct `PhaseEncodingDirection` and `TotalReadoutTime`.
- Fieldmap `B0FieldIdentifier` values have no hyphens.
- Target scans use matching `B0FieldSource` values.
- `IntendedFor` resolves to the intended BOLD/SBRef/DWI files.
- BIDS entity labels are valid: no dots in `task-` labels, no spaces, and no
  scanner punctuation copied directly into filenames.
- `RepetitionTime` agrees with the scanner-derived dcm2niix sidecar. Do not
  force it unless verified.
- If `SliceTiming` exists, all values are less than the declared
  `RepetitionTime`.
- A test conversion passes the BIDS validator before large-scale conversion.

## fMRIPrep SliceTiming Failure

If fMRIPrep fails in AFNI `3dTshift` with an error like:

```text
Illegal value 1.385 in tpattern file
```

check whether the config forced the wrong `RepetitionTime`. One observed case
had a dcm2bids config forcing:

```json
"RepetitionTime": 1
```

while dcm2niix reported the actual TR as `2`, and `SliceTiming` values reached
about `1.78`. The fix was to correct the config to the scanner-derived TR, rerun
dcm2bids, refresh the XNAT `configs` catalog, and rerun fMRIPrep.

## References

- [dcm2bids: Create a config file](https://unfmontreal.github.io/Dcm2Bids/3.2.0/how-to/create-config-file/)
- [dcm2bids tutorial: Building the configuration file](https://unfmontreal.github.io/Dcm2Bids/3.2.0/tutorial/first-steps/#building-the-configuration-file)
- [BIDS MRI specification: B0 fieldmaps](https://bids-specification.readthedocs.io/en/stable/modality-specific-files/magnetic-resonance-imaging-data.html#b0-fieldmaps)
- [BIDS MRI specification: IntendedFor metadata](https://bids-specification.readthedocs.io/en/stable/modality-specific-files/magnetic-resonance-imaging-data.html#using-intendedfor-metadata)
