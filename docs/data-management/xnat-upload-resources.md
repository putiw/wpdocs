# Uploading Resources to XNAT

This page covers how to manually upload derivative data (e.g. FreeSurfer output, raw BIDS data) directly to the XNAT archive filesystem and register it so the browser UI and REST API can see it.

Use this approach when data is too large or too structured to upload via the XNAT web UI, or when you want to stage derivatives alongside an existing session without re-running a pipeline inside XNAT.

---

## How XNAT Stores Resources

XNAT sessions live under:

```text
/Volumes/CTP-XNAT/xnat-main/xnat-data/archive/<project>/arc001/<session>/
```

Each session has a `RESOURCES/` directory. Every subdirectory under `RESOURCES/` is a named resource (e.g. `rawdata`, `freesurfer`). Each resource directory contains its data files and a `{label}_catalog.xml` file that XNAT reads to list the contents.

```text
RESOURCES/
├── rawdata/
│   ├── rawdata_catalog.xml      ← XNAT reads this
│   └── sub-0248/ses-01/...
└── freesurfer/
    ├── freesurfer_catalog.xml   ← XNAT reads this
    ├── sub-0248/
    ├── fsaverage/
    └── fsaverage6/
```

XNAT's browser and REST API pull file listings from the catalog XML, not from a raw directory scan. The catalog also stores MD5 checksums and byte sizes for each file. The XNAT database separately caches the total `file_count` and `file_size` per resource — these require an explicit refresh call to populate.

Project-level resources use a slightly different path shape. For example, the
project-level dcm2bids config resource lives at:

```text
/Volumes/CTP-XNAT/xnat-main/xnat-data/archive/<project>/resources/configs/
```

The same catalog rules apply there: XNAT reads `configs_catalog.xml`, and
refresh calls need the XNAT URI for the project resource rather than a filesystem
path.

---

## Authentication

Use an alias/secret token pair instead of your username and password for REST calls. Generate one in the XNAT UI under **My Account → Manage Alias Tokens**.

```bash
XNAT_ALIAS="your-alias-uuid"
XNAT_SECRET="your-secret-string"
XNAT_HOST="https://xnat.abudhabi.nyu.edu"
```

Pass credentials with `-u "$XNAT_ALIAS:$XNAT_SECRET"` in every curl command below.

---

## Step 1 — Copy Files to the Archive

Copy your derivative data directly into the session's `RESOURCES/` directory on the mounted fileshare.

```bash
ARCHIVE=/Volumes/CTP-XNAT/xnat-main/xnat-data/archive
SESSION=$ARCHIVE/bi_MRI_QualityControl_2026/arc001/Subject_0248_ses_01/RESOURCES

# FreeSurfer output (subject + template surfaces)
cp -r /path/to/freesurfer/sub-0248   $SESSION/freesurfer/sub-0248
cp -r /path/to/freesurfer/fsaverage  $SESSION/freesurfer/fsaverage
cp -r /path/to/freesurfer/fsaverage6 $SESSION/freesurfer/fsaverage6

# Raw BIDS anatomicals
cp -r /path/to/rawdata/sub-0248/ses-01/anat \
       $SESSION/rawdata/sub-0248/ses-01/anat
```

On Monai the archive is at `/mnt/CTP/xnat-main/xnat-data/archive/`.

---

## Step 2 — Generate the Catalog XML

XNAT will not see the files without a valid `{label}_catalog.xml`. Use the script below to walk the resource directory, compute MD5 checksums, and write the catalog.

```python
#!/usr/bin/env python3
"""Generate an XNAT resource catalog XML from files already on disk."""
import hashlib
from datetime import datetime, timezone
from pathlib import Path

# ── configure ────────────────────────────────────────────────────────────────
RESOURCE_DIR = Path("/Volumes/CTP-XNAT/xnat-main/xnat-data/archive"
                    "/bi_MRI_QualityControl_2026/arc001/Subject_0248_ses_01"
                    "/RESOURCES/freesurfer")
LABEL   = "freesurfer"            # must match the directory name
PROJECT = "bi_MRI_QualityControl_2026"
EVENT_ID = "2037109"              # arbitrary; use a number higher than existing ones
# ─────────────────────────────────────────────────────────────────────────────

NOW      = datetime.now(timezone.utc).strftime("%Y-%m-%dT%H:%M:%S.000")
NOW_AUDIT = datetime.now(timezone.utc).strftime("%a %b %d %H:%M:%S UTC %Y")
NS = ('xmlns:arc="http://nrg.wustl.edu/arc" '
      'xmlns:cat="http://nrg.wustl.edu/catalog" '
      'xmlns:icr="http://icr.ac.uk/icr" '
      'xmlns:pipe="http://nrg.wustl.edu/pipe" '
      'xmlns:prov="http://www.nbirn.net/prov" '
      'xmlns:scr="http://nrg.wustl.edu/scr" '
      'xmlns:val="http://nrg.wustl.edu/val" '
      'xmlns:wrk="http://nrg.wustl.edu/workflow" '
      'xmlns:xdat="http://nrg.wustl.edu/security" '
      'xmlns:xnat="http://nrg.wustl.edu/xnat" '
      'xmlns:xnat_a="http://nrg.wustl.edu/xnat_assessments" '
      'xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"')

CATALOG = RESOURCE_DIR / f"{LABEL}_catalog.xml"

def md5(path):
    h = hashlib.md5()
    with open(path, "rb") as f:
        for chunk in iter(lambda: f.read(1024 * 1024), b""):
            h.update(chunk)
    return h.hexdigest()

entries = []
for f in sorted(RESOURCE_DIR.rglob("*")):
    if not f.is_file() or f == CATALOG:
        continue
    rel    = str(f.relative_to(RESOURCE_DIR))
    digest = md5(f)
    size   = f.stat().st_size
    name   = f.name
    entries.append(
        f'\t<cat:entry ID="{rel}" URI="{rel}" cachePath="{rel}" '
        f'createdEventId="{EVENT_ID}" createdTime="{NOW}" '
        f'digest="{digest}" name="{name}">\n'
        f'\t\t<cat:metaFields>\n'
        f'\t\t\t<cat:metaField name="SIZE">{size}</cat:metaField>\n'
        f'\t\t</cat:metaFields>\n'
        f'\t</cat:entry>'
    )

xml = f"""<?xml version="1.0" encoding="UTF-8"?>
<cat:Catalog ID="{LABEL}" {NS}>
<cat:metaFields>
\t<cat:metaField name="PROJECT">{PROJECT}</cat:metaField>
\t<cat:metaField name="AUDIT">{EVENT_ID}:{NOW_AUDIT}=Added:{len(entries)}</cat:metaField>
</cat:metaFields>
<cat:entries>
{chr(10).join(entries)}
</cat:entries>
</cat:Catalog>"""

CATALOG.write_text(xml)
print(f"Wrote {len(entries)} entries → {CATALOG}")
```

Run this for each new resource directory. For large FreeSurfer trees (~1 700 files) this takes a few minutes due to MD5 computation.

---

## Step 3 — Register the Resource via REST API

!!! warning "Order matters"
    **Register the resource via the API only after the catalog XML already exists on disk.** XNAT's `PUT` call to create a resource immediately writes an empty `{label}_catalog.xml`, overwriting any file already there. If you register first, re-run the catalog generation script afterwards.

Find the experiment ID from the session URL (e.g. `XNAT_NYUAD_E05410`):

```bash
curl -u "$XNAT_ALIAS:$XNAT_SECRET" -X PUT \
  "$XNAT_HOST/data/experiments/XNAT_NYUAD_E05410/resources/freesurfer?format=FREESURFER"
```

Common `format` values: `FREESURFER`, `BIDS`, `NIFTI`, `DICOM`. Leave blank if none applies.

After this call, the resource appears in the XNAT database but XNAT has overwritten the catalog with an empty file. Re-run the catalog generation script immediately:

```bash
python3 generate_catalog.py
```

---

## Step 4 — Populate Database Stats

XNAT caches `file_count` and `file_size` per resource separately from the catalog XML. Use the catalog refresh service to populate them:

```bash
curl -u "$XNAT_ALIAS:$XNAT_SECRET" -X POST \
  "$XNAT_HOST/data/services/refresh/catalog?options=populateStats\
&resource=/archive/projects/bi_MRI_QualityControl_2026/experiments/XNAT_NYUAD_E05410/resources/freesurfer"
```

Wait a few seconds then verify:

```bash
curl -s -u "$XNAT_ALIAS:$XNAT_SECRET" \
  "$XNAT_HOST/data/experiments/XNAT_NYUAD_E05410/resources?format=json" \
  | python3 -c "
import sys, json
for r in json.load(sys.stdin)['ResultSet']['Result']:
    print(r['label'], r['file_count'], 'files', r['file_size'], 'bytes')"
```

The resource will now show the correct file count and size in the browser.

### Helper script

For routine refreshes, use the local helper script:

```bash
export XNAT_ALIAS="<alias-token>"
export XNAT_SECRET="<alias-secret>"

/Users/pw1246/Documents/GitHub/pipelines/xnat/refresh_xnat_resource_from_catalog.sh \
  /Volumes/CTP-XNAT/xnat-main/xnat-data/archive/bi_MRI_QualityControl_2026/arc001/Subject_0037_ses_01/RESOURCES/rawdata/rawdata_catalog.xml
```

It derives the project, session/resource label, experiment ID, and XNAT resource
URI from the catalog path. It also supports project-level resources:

```bash
/Users/pw1246/Documents/GitHub/pipelines/xnat/refresh_xnat_resource_from_catalog.sh \
  /Volumes/CTP-XNAT/xnat-main/xnat-data/archive/bi_MRI_QualityControl_2026/resources/configs/configs_catalog.xml
```

Do not hard-code alias tokens or secrets in this script. Generate an alias token
in XNAT and export it in the current shell when needed.

---

## Adding Files to an Existing Resource

If a resource already exists and you need to add more files (e.g. adding `anat/` to an existing `rawdata` resource):

1. Copy the new files into the resource directory.
2. Edit the existing `{label}_catalog.xml` to append new `<cat:entry>` elements, or re-run the catalog script to regenerate it fully.
3. Use `append` together with `populateStats` in the refresh call so XNAT picks up the new entries and updates the count:

```bash
curl -u "$XNAT_ALIAS:$XNAT_SECRET" -X POST \
  "$XNAT_HOST/data/services/refresh/catalog?options=populateStats,append\
&resource=/archive/projects/<project>/experiments/<exp_id>/resources/<label>"
```

No `PUT` to register is needed — the resource is already in the database.

---

## Catalog Refresh Options

The `options` parameter accepts a comma-separated list:

| Option | Effect |
|---|---|
| `populateStats` | Update `file_count` and `file_size` in the XNAT database |
| `append` | Add catalog entries for files present on disk but missing from the XML |
| `delete` | Remove catalog entries that point to files no longer on disk |
| `checksum` | Generate MD5 digests for entries that are missing them |

Reference: [XNAT Catalog Refresh API](https://wiki.xnat.org/xnat-api/catalog-refresh-api)

### Stale catalog rows

`append` only adds rows for files present on disk but missing from the catalog.
It does not remove catalog rows for files that were deleted from disk.

If refresh returns HTTP 500 and the catalog count is larger than the resource
file count, check for stale `<cat:entry>` rows. Either remove those stale entries
from the XML or run a documented refresh that includes `delete`:

```bash
curl -u "$XNAT_ALIAS:$XNAT_SECRET" -X POST \
  "$XNAT_HOST/data/services/refresh/catalog?options=delete,checksum,populateStats\
&resource=/archive/projects/<project>/resources/<label>"
```

For session-level resources, use:

```text
/archive/projects/<project>/experiments/<exp_id>/resources/<label>
```

For project-level resources, use:

```text
/archive/projects/<project>/resources/<label>
```

After repair, verify the REST file endpoint and the catalog entries agree. The
aggregate `file_count` summary can lag or stay stale on some project-level
resources even when the REST file listing and catalog are correct.

---

## FreeSurfer Reuse with fMRIPrep

To make fMRIPrep skip FreeSurfer recon and reuse existing output, the FreeSurfer subject directory must be accessible at the path you pass to `--fs-subjects-dir`. When running via XNAT pipelines or with the archive mounted, point fMRIPrep at the `freesurfer/` resource directory:

```bash
--fs-subjects-dir /Volumes/CTP-XNAT/xnat-main/xnat-data/archive/<project>/arc001/<session>/RESOURCES/freesurfer
```

fMRIPrep will detect `sub-XXXX/` inside that directory and skip recon if the `scripts/recon-all.done` touch file is present.

---

## Common Mistakes

| Problem | Fix |
|---|---|
| Resource visible on fileshare but not in XNAT browser | Resource not registered in database — run the `PUT` to create it |
| Browser shows "files, 0 bytes" | Stats not populated — run the `populateStats` refresh call |
| Catalog XML is empty after `PUT` | XNAT overwrote it — re-run the catalog generation script |
| Files accessible via REST but file count is wrong | Run `populateStats,append` refresh |
| Refresh returns HTTP 500 after files were deleted | Remove stale catalog rows or use refresh options including `delete` |
| Project-level `resources/configs` path is rejected | Use the helper script version that supports `/archive/<project>/resources/<resource>/...` |
| `PUT` resource returns "Request Rejected" (WAF block) | Avoid passing filesystem paths as query parameters; use the XNAT URI format `/archive/projects/...` |
| `populateStats` has no effect | Check the `resource` parameter — it must be the XNAT URI path, not the filesystem path |
