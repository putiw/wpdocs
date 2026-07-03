# CCAD Epilepsy BIDS Conversion

Use this after DICOMs have been exported to Asif's iMac.

## Paths

Pipeline script:

```text
/Users/pw1246/Documents/GitHub/pipelines/ccad/scripts/convert_missing_dicoms_to_bids.py
```

Remote machine:

```text
imac1@10.157.29.87
```

Remote BIDS root:

```text
/Volumes/fMRI/Epilepsy_Cases/BIDS
```

Local BIDS root:

```text
/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi
```

Config:

```text
/Users/pw1246/Documents/GitHub/pipelines/ccad/configs/ccad_dcm2bids_config.json
```

## Dry Run

From the local Mac:

```bash
python3 /Users/pw1246/Documents/GitHub/pipelines/ccad/scripts/convert_missing_dicoms_to_bids.py
```

This reports subjects present in remote `sourcedata` but missing from remote `rawdata`.

## Convert Missing DICOMs

```bash
python3 /Users/pw1246/Documents/GitHub/pipelines/ccad/scripts/convert_missing_dicoms_to_bids.py --convert
```

The script:

1. Connects to `imac1@10.157.29.87`.
2. Checks `/Volumes/fMRI/Epilepsy_Cases/BIDS/sourcedata`.
3. Finds missing subjects in `/Volumes/fMRI/Epilepsy_Cases/BIDS/rawdata`.
4. Uploads the dcm2bids config to the remote Mac.
5. Runs `dcm2bids` on the remote Mac.
6. Syncs remote `rawdata` back to:

```text
/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/rawdata
```

The script treats `--local-bids-root` as the BIDS root and syncs into `<root>/rawdata`.

## Sync Only

Use this when conversion is already done remotely.

```bash
python3 /Users/pw1246/Documents/GitHub/pipelines/ccad/scripts/convert_missing_dicoms_to_bids.py --sync-only
```

## Useful Options

```bash
python3 /Users/pw1246/Documents/GitHub/pipelines/ccad/scripts/convert_missing_dicoms_to_bids.py --convert --no-sync
```

```bash
python3 /Users/pw1246/Documents/GitHub/pipelines/ccad/scripts/convert_missing_dicoms_to_bids.py --convert --clobber
```

## Remote Tool Check

The remote Mac needs:

- `dcm2niix`
- `dcm2bids`

The script checks both before conversion.
