---
name: 16s-amplicon-quality-filtering
description: Use when paired-end 16S FASTQ reads show good base quality over an initial
  stretch (e.g. ~200 bp R1 / 180 bp R2) followed by a sudden quality drop, and reads
  must be trimmed and length-filtered before OTU clustering.
when_to_use_negative:
- Input is not paired-end 16S amplicon FASTQ data (e.g. shotgun metagenomic or already-merged
  reads), since the downstream CD-HIT-OTU-Miseq scheme relies on separate R1/R2 clustering
  without merging
- Reads lack a clear high-quality prefix with a sudden quality drop that can be captured
  by fixed p/q lengths, making fixed-length cropping inappropriate
- A feature/OTU table already exists and no raw-read trimming is required
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3168
tools:
- name: Trimmomatic
  role: Crops R1/R2 reads to their 'good' quality parts (p=200, q=180 by default)
    and discards cropped reads shorter than MINLEN=100 nucleotides
  repo: https://github.com/ydkondratenko/cdsnake
- name: Snakemake
  role: Workflow engine that launches the trimming step with configured THREADS and
    MINLEN settings via the edited Snakefile
  repo: https://github.com/ydkondratenko/cdsnake
- name: CDSnake
  role: Pipeline that wraps Trimmomatic and passes filtered reads to CD-HIT-OTU-Miseq
    clustering
  repo: https://github.com/ydkondratenko/cdsnake
- name: FastQC
  role: Quality assessment used to choose the p parameter (length of the decent-quality
    prefix of R1 reads)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:bd75faa9e14d5d65ad2de3fd2cad00ad97ee7782b13828215016238365c39749
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ydkondratenko/cdsnake
  tools:
  - CD-HIT
  - CD-HIT-OTU-Miseq
  - CDSnake
  - FastQC
  - Snakemake
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# 16s-amplicon-quality-filtering

## Summary

Cropping paired-end 16S amplicon reads to their high-quality prefixes and quality-filtering them with Trimmomatic, as configured in the CDSnake pipeline, so that only reliable bases enter the downstream CD-HIT-OTU-Miseq clustering.

## When to use

Use when paired-end 16S FASTQ reads show good base quality over an initial stretch (e.g. ~200 bp R1 / 180 bp R2) followed by a sudden quality drop, and reads must be trimmed and length-filtered before OTU clustering.

## When NOT to use

- Input is not paired-end 16S amplicon FASTQ data (e.g. shotgun metagenomic or already-merged reads), since the downstream CD-HIT-OTU-Miseq scheme relies on separate R1/R2 clustering without merging
- Reads lack a clear high-quality prefix with a sudden quality drop that can be captured by fixed p/q lengths, making fixed-length cropping inappropriate
- A feature/OTU table already exists and no raw-read trimming is required

## Inputs

- Unzipped paired-end 16S FASTQ files (forward and reverse, named with '_R1_'/'_R2_' or '_1'/'_2' before '.fastq')
- trimmomatic.jar (path specified via TRIMMOMATIC parameter)
- Per-read quality reports (e.g. FastQC) for choosing p and q prefix lengths

## Outputs

- Cropped and quality-filtered R1/R2 FASTQ reads (>=100 nt after cropping) ready for 99% cd-hit-est clustering

## How to apply

Place unzipped paired-end FASTQ files (forward/reverse named with '_R1_'/'_R2_' or ending '_1'/'_2') in the input folder and point the CDSnake Snakefile's TRIMMOMATIC parameter at trimmomatic.jar. Determine the 'good' quality prefix lengths by inspecting per-base quality reports (e.g. FastQC) and set the p and q parameters (defaults 200 bp for R1 and 180 bp for R2, where quality stays generally above 25 with no sudden drop). Run Trimmomatic with the configured THREADS setting and a MINLEN of 100 nucleotides so that any cropped read shorter than 100 nt is discarded entirely. This cropping-and-filtering step precedes separate 99% cd-hit-est clustering of R1 and R2 reads, so over-trimming would reduce overlap information while under-trimming would let low-quality tails degrade cluster voting.

## Related tools

- **Trimmomatic** (Crops R1/R2 reads to their 'good' quality parts (p=200, q=180 by default) and discards cropped reads shorter than MINLEN=100 nucleotides) — https://github.com/ydkondratenko/cdsnake
- **Snakemake** (Workflow engine that launches the trimming step with configured THREADS and MINLEN settings via the edited Snakefile) — https://github.com/ydkondratenko/cdsnake
- **CDSnake** (Pipeline that wraps Trimmomatic and passes filtered reads to CD-HIT-OTU-Miseq clustering) — https://github.com/ydkondratenko/cdsnake
- **FastQC** (Quality assessment used to choose the p parameter (length of the decent-quality prefix of R1 reads))

## Examples

```
snakemake
```

## Evaluation signals

- No retained read is shorter than 100 nucleotides after cropping (MINLEN filter enforced)
- Cropped read lengths do not exceed the chosen p (R1) and q (R2) 'good quality' prefix lengths
- Per-base quality of retained regions stays generally above 25 with no sudden quality drops
- Downstream pooled/OTU.txt is produced and OTU counts per sample are comparable to the number of strains in the analyzed mock community
- Pipeline completes with the single 'snakemake' command and writes clustering results to the 'pooled' folder

## Limitations

- Default p=200 and q=180 are starting points; users must inspect read quality (e.g. with fastqc) and adjust p/q per dataset, otherwise trimming may be too aggressive or too lax
- Reads must be unzipped '.fastq' with '_R1_'/'_R2_' or '_1'/'_2' naming; other naming conventions are not picked up
- Trimmomatic only attempts to use THREADS threads; performance depends on the configured core count, and software versions are not stated in the article

## Evidence

- [methods] Crop R1/R2 reads to their 'good' quality parts (default 200 bp for R1 and 180 bp for R2, quality generally above 25 with no sudden drops) and quality-filter them with Trimmomatic: [evidence span withheld — anchor status: not_found]
- [readme] Parameters THREADS and MINLEN specify settings for trimmomatic, which is used for data filtering. Trimmomatic will try to use THREADS number of threads and will discard cropped reads if they become shorter than 100 nucleotides.: "Trimmomatic will try to use THREADS number of threads and will discard cropped reads if they become shorter than 100 nucleotides."
- [readme] you can have the quality of first 220 nucleotides of your R1 reads of about 30 with a sudden drop of quality after 220 nucleotide. In this case you should set value of p parameter equal to 220.: "you can have the quality of first 220 nucleotides of your R1 reads of about 30 with a sudden drop of quality after 220 nucleotide"
- [readme] Place your paired-end 16S reads in a folder. Reads should be unzipped prior to pipeline usage (should have an extension “.fastq”).: "Reads should be unzipped prior to pipeline usage (should have an extension “.fastq”). Forward and reverse reads should have “_R1_” and “_R2_” in filenames."
- [methods] Filtered sequences are clustered at 99% using cd-hit-est utility to discard chimeric reads.: "Filtered sequences are clustered at 99% using cd-hit-est utility to discard chimeric reads."
