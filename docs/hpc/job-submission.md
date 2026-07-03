# Job Submission (SLURM)

Jubail uses Slurm for batch jobs. Use this page for basic submission,
monitoring, and debugging commands.

## Basic Script

```bash
#!/usr/bin/env bash
#SBATCH --job-name=my_job
#SBATCH --partition=compute
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=08:00:00
#SBATCH --output=slurm-%j.out
#SBATCH --error=slurm-%j.err

set -euo pipefail

hostname
date

# Run your command here
```

Submit:

```bash
sbatch my_job.slurm
```

Test script validity without submitting:

```bash
sbatch --test-only my_job.slurm
```

## Monitoring

```bash
squeue -u "$USER"
scontrol show job <job_id>
sacct -j <job_id> --format=JobID,JobName,State,Start,End,Elapsed,ExitCode
```

For queue/partition state:

```bash
sinfo
sinfo -p compute -N -h -o "%N %t %c %m %f" | head
```

## Common Partitions

| Partition | Typical use |
|---|---|
| `compute` | CPU jobs. |
| `nvidia` | GPU jobs. |
| `bigmem` | Large-memory jobs. |

Always check current availability and limits with `sinfo`, `scontrol show
partition`, and account/QOS commands when a job is stuck pending.

## Array Jobs

Use array jobs for subject-level parallelism:

```bash
#SBATCH --array=1-20

SUBJECT=$(sed -n "${SLURM_ARRAY_TASK_ID}p" subjects.txt)
echo "Running ${SUBJECT}"
```

Keep each array task independent and write outputs to subject-specific folders.

## XNAT Wrapper Jobs

XNAT wrappers can create Slurm scripts before Slurm assigns a job ID. For
example, the fMRIPrep wrapper names files from `XNAT_WORKFLOW_ID`:

```text
/home/mri/<workflow_id>.slurm
/home/mri/slurm-<workflow_id>.out
/home/mri/slurm-<workflow_id>.err
/scratch/mri/<workflow_id>/
```

Then Slurm assigns a separate job ID such as `16403349_1`.

If you see a Slurm command or job name like:

```text
.slurm
```

that usually means the wrapper created a script with an empty workflow ID. Stop
and inspect the wrapper before trusting cleanup logic.

## Pending Jobs

Useful checks:

```bash
scontrol show job <job_id>
sbatch --test-only /home/mri/<workflow_id>.slurm
scontrol show reservations
sacctmgr show assoc user="$USER" format=User,Account,Partition,QOS,DefaultQOS,MaxJobs,MaxSubmit,MaxTRES,GrpTRES
```

If idle nodes exist but your job does not start, check for:

- feature constraints
- reservations
- walltime too long for backfill
- account/QOS limits
- invalid script syntax
- requested GPU/CPU architecture that is not currently available

Do not assume idle nodes are usable until the job's pending reason and feature
constraints are clear.
