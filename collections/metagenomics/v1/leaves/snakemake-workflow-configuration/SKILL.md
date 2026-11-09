---
name: snakemake-workflow-configuration
description: Use when paired-end 16S amplicon FASTQ files must be processed end-to-end
  into an annotated OTU table and read merging is to be avoided in favor of separate
  R1/R2 clustering with CD-HIT-OTU-Miseq.
when_to_use_negative:
- Input is already merged/concatenated reads or an existing OTU/feature table — the
  skill's CD-HIT-OTU-Miseq scheme requires unmerged paired-end FASTQ files
- Reads have no clearly demarcated good-quality region (e.g. no plateau of quality
  ~25-30 before a sudden drop), making p/q cropping to 200/180 bp inappropriate
- The 16S database is not reformatted so that sequences with more specific annotations
  appear at the beginning of the file, as CD-HIT-OTU-Miseq installation step 4 requires
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Snakemake
  role: Workflow engine that wraps CD-HIT-OTU-Miseq utilities for one-command, reproducible
    launch of the whole pipeline
  repo: github.com/ydkondratenko/cdsnake
- name: CD-HIT-OTU-Miseq
  role: Underlying utility suite performing separate 99% R1/R2 clustering and 97%
    annotated OTU clustering without read merging
  repo: github.com/ydkondratenko/cdsnake
- name: Trimmomatic
  role: Crops R1/R2 to good-quality parts (p=200, q=180 default) and filters reads
    shorter than MINLEN 100 nucleotides
- name: CD-HIT
  role: Provides cd-hit-est used to cluster filtered reads at 99% identity for chimera
    removal
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:957de4ed5f6b29ccb507171f97184c76b8985d395328ac330dde7f32e765009e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT
  - CD-HIT-OTU-Miseq
  - Snakemake
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# snakemake-workflow-configuration

## Summary

Configure and launch the CDSnake Snakefile so CD-HIT-OTU-Miseq utilities run as a single reproducible Snakemake workflow that extracts annotated OTUs from paired-end 16S reads without a merging step.

## When to use

Use when paired-end 16S amplicon FASTQ files must be processed end-to-end into an annotated OTU table and read merging is to be avoided in favor of separate R1/R2 clustering with CD-HIT-OTU-Miseq.

## When NOT to use

- Input is already merged/concatenated reads or an existing OTU/feature table — the skill's CD-HIT-OTU-Miseq scheme requires unmerged paired-end FASTQ files
- Reads have no clearly demarcated good-quality region (e.g. no plateau of quality ~25-30 before a sudden drop), making p/q cropping to 200/180 bp inappropriate
- The 16S database is not reformatted so that sequences with more specific annotations appear at the beginning of the file, as CD-HIT-OTU-Miseq installation step 4 requires

## Inputs

- Folder of unzipped paired-end 16S FASTQ files (forward/reverse named with '_R1_'/'_R2_' or '_1'/'_2' suffixes)
- Annotated 16S reference database in FASTA form reformatted per CD-HIT-OTU-Miseq installation step 4 (e.g. Greengenes or SILVA)
- Edited Snakefile placed in the input folder

## Outputs

- OTU count table for all pooled samples at pooled/OTU.txt
- Clustering results for pooled samples in the 'pooled' subfolder of the input folder

## How to apply

Edit the Snakefile in the input folder containing unzipped paired-end FASTQ files (filenames carrying '_R1_'/'_R2_' or trailing '_1'/'_2' before '.fastq') and the reformatted annotated 16S reference database (Greengenes or SILVA) in FASTA. Set INPUT_FOLDER, CDHIT (path to the CD-HIT folder installed per CD-HIT-OTU-Miseq installation steps 1-4), DATABASE, and TRIMMOMATIC (path to trimmomatic.jar). Tune p and q to the lengths of the decent-quality parts of R1 and R2 (defaults 200 bp and 180 bp, chosen from quality profiles such as a fastqc report showing quality ~30 with a sudden drop), and set THREADS and MINLEN so Trimmomatic discards cropped reads shorter than 100 nucleotides. Install Snakemake, cd into the input folder, and launch with the single 'snakemake' command, using standard Snakemake parameters (e.g. core count) to modify general behavior. Clustering results for pooled samples are written to the 'pooled' folder, with the OTU count table for all samples in pooled/OTU.txt.

## Related tools

- **Snakemake** (Workflow engine that wraps CD-HIT-OTU-Miseq utilities for one-command, reproducible launch of the whole pipeline) — github.com/ydkondratenko/cdsnake
- **CD-HIT-OTU-Miseq** (Underlying utility suite performing separate 99% R1/R2 clustering and 97% annotated OTU clustering without read merging) — github.com/ydkondratenko/cdsnake
- **Trimmomatic** (Crops R1/R2 to good-quality parts (p=200, q=180 default) and filters reads shorter than MINLEN 100 nucleotides)
- **CD-HIT** (Provides cd-hit-est used to cluster filtered reads at 99% identity for chimera removal)

## Examples

```
snakemake --cores 8
```

## Evaluation signals

- pooled/OTU.txt exists inside the input folder after the 'snakemake' run completes
- Reported OTU counts per sample are comparable to the number of strains in the analyzed mock community (e.g. 57 in the Balanced community)
- Snakemake reports a fully executed DAG with no errors, confirming all CD-HIT-OTU-Miseq steps ran consecutively
- Input FASTQ files are unzipped with '.fastq' extension and carry '_R1_'/'_R2_' (or '_1'/'_2') markers, matching Snakefile name conventions
- pooled/ folder contains clustering results for the pooled samples

## Limitations

- Snakefile must be hand-edited for each starting condition (INPUT_FOLDER, CDHIT, DATABASE, TRIMMOMATIC, p, q, THREADS, MINLEN) — no automatic configuration
- Reads must be unzipped before pipeline usage; zipped FASTQ inputs are not supported
- p and q defaults (200/180 bp) assume quality generally above 25 with no sudden drops; data with different quality profiles require re-tuning via fastqc reports
- CDSnake is generally slower than DADA2/deblur pipelines, though it outputs higher total counts preserving more information from raw data

## Evidence

- [readme] Place your paired-end 16S reads in a folder. Reads should be unzipped prior to pipeline usage (should have an extension “.fastq”).: "Place your paired-end 16S reads in a folder. Reads should be unzipped prior to pipeline usage (should have an extension “.fastq”)."
- [methods] Default lengths of “good” parts for R1 and R2 are set to 200 and 180 bp correspondingly.: "Default lengths of “good” parts for R1 and R2 are set to 200 and 180 bp correspondingly."
- [readme] Results of clustering for pooled samples will be stored in folder “pooled” in input folder. OTU table for all samples will be saved in file “OTU.txt” in “pooled” folder.: "Results of clustering for pooled samples will be stored in folder “pooled” in input folder. OTU table for all samples will be saved in file “OTU.txt” in “pooled” folder."
- [abstract] Usage of pipeline make 16S analysis easier due to one-command launch and helps to yield reproducible results.: "Usage of pipeline make 16S analysis easier due to one-command launch and helps to yield reproducible results."
- [readme] Trimmomatic will try to use THREADS number of threads and will discard cropped reads if they become shorter than 100 nucleotides.: "Trimmomatic will try to use THREADS number of threads and will discard cropped reads if they become shorter than 100 nucleotides."
