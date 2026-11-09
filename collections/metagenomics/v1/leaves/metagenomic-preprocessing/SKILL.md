---
name: metagenomic-preprocessing
description: Use when raw shotgun metagenomic FASTQs must be cleaned before functional
  annotation — removing human/host reads, PCR or optical duplicates, and low-quality
  or too-short reads prior to gene mapping.
when_to_use_negative:
- Input data are already post-quality-control (e.g. HMP data files are post-HMP quality
  control, so minimal reads are discarded) — applying heavy filtering adds little
  value
- Reads still contain sequencing adapters — MetaLAFFA expects adapter-removed FASTQs
  as input
- The goal is host genome analysis rather than microbial functional annotation — discarding
  host reads is destructive here
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: Bowtie 2
  role: Aligns reads to the hs37d5 host reference index; reads with match qualities
    above the threshold are discarded
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Trimmomatic
  role: Performs quality trimming and filtering using the MAXINFO criterion, removing
    low-quality end bases and dropping short reads
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: MarkDuplicates (PICARD)
  role: Identifies artificial (PCR/optical) duplicate reads that are then discarded
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: MetaLAFFA
  role: Snakemake pipeline that orchestrates the quality-control phase and generates
    per-step summary statistics
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Conda
  role: Installs MetaLAFFA and all pre-processing dependencies (metalaffa environment)
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:e445e98b613853d387e7dcade74667a293bb15d878daa65d5dc9978a4203da04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - Bowtie 2
  - Conda
  - MarkDuplicates (PICARD)
  - MetaLAFFA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# metagenomic-preprocessing

## Summary

Quality-control pre-processing of raw shotgun metagenomic FASTQs as implemented in MetaLAFFA: host read filtering against the hs37d5 human reference with 1000 Genomes decoy sequences using Bowtie 2, duplicate read removal with PICARD MarkDuplicates, and quality trimming/filtering with Trimmomatic's MAXINFO criterion, with per-step summary statistics.

## When to use

Use when raw shotgun metagenomic FASTQs must be cleaned before functional annotation — removing human/host reads, PCR or optical duplicates, and low-quality or too-short reads prior to gene mapping.

## When NOT to use

- Input data are already post-quality-control (e.g. HMP data files are post-HMP quality control, so minimal reads are discarded) — applying heavy filtering adds little value
- Reads still contain sequencing adapters — MetaLAFFA expects adapter-removed FASTQs as input
- The goal is host genome analysis rather than microbial functional annotation — discarding host reads is destructive here

## Inputs

- Raw shotgun metagenomic FASTQ files (with sequencing adapters removed), gzipped and named per MetaLAFFA convention: <sample>.R1.fastq.gz, <sample>.R2.fastq.gz, <sample>.S.fastq.gz (e.g. HMP stool sample SRS017307)
- Bowtie 2 index of hs37d5 human reference plus 1000 Genomes Project decoy sequences, prepared via prepare_databases.py -hr

## Outputs

- Host-filtered, duplicate-filtered, quality-trimmed FASTQ files ready for read-to-gene mapping
- Per-sample summary statistics table: number of reads, average read length, and average base quality for input and filtered FASTQs, plus read counts removed per filtering step

## How to apply

Run host filtering first by aligning input FASTQs (e.g. <sample>.R1.fastq.gz / <sample>.R2.fastq.gz) to the hs37d5 human genome reference plus decoy sequences from the 1000 Genomes Project using Bowtie 2, then discard reads whose match qualities exceed the configured filtering threshold; substitute an alternative host reference index for non-human hosts. Next, identify artificial duplicates (PCR or optical) with MarkDuplicates from the PICARD toolset and remove them. Then use Trimmomatic with the MAXINFO trimming criterion to clip low-quality bases from read ends and drop reads falling below the length threshold. Finally, compute per-sample summary statistics (read counts, average read length, average base quality for input and filtered FASTQs) and record how many reads were removed at each step so pre-processing quality can be audited.

## Related tools

- **Bowtie 2** (Aligns reads to the hs37d5 host reference index; reads with match qualities above the threshold are discarded) — https://github.com/borenstein-lab/MetaLAFFA
- **Trimmomatic** (Performs quality trimming and filtering using the MAXINFO criterion, removing low-quality end bases and dropping short reads) — https://github.com/borenstein-lab/MetaLAFFA
- **MarkDuplicates (PICARD)** (Identifies artificial (PCR/optical) duplicate reads that are then discarded) — https://github.com/borenstein-lab/MetaLAFFA
- **MetaLAFFA** (Snakemake pipeline that orchestrates the quality-control phase and generates per-step summary statistics) — https://github.com/borenstein-lab/MetaLAFFA
- **Conda** (Installs MetaLAFFA and all pre-processing dependencies (metalaffa environment)) — https://github.com/borenstein-lab/MetaLAFFA

## Examples

```
conda create -n metalaffa metalaffa -c bioconda -c borenstein-lab; conda activate metalaffa; prepare_databases.py -hr -km -u
```

## Evaluation signals

- Host-filtered FASTQ files exist for each sample (e.g. SRS017307) after the quality-control phase
- Summary statistics table reports per-sample read counts removed during host filtering (input vs. filtered read numbers)
- Filtered FASTQs show reads discarded only when Bowtie 2 match quality exceeds the configured host-filtering threshold — no non-host reads lost
- Duplicate filtering removed reads flagged by MarkDuplicates as PCR or optical duplicates
- Post-trim reads pass the minimum length threshold and no longer carry low-quality terminal bases per Trimmomatic MAXINFO

## Limitations

- The default host database is human-specific (hs37d5 + 1000 Genomes decoys); other host organisms require preparing a substitute reference index
- On already quality-controlled data (e.g. post-HMP QC samples), minimal reads are discarded, so the QC phase adds limited benefit
- Preparing the reference databases is resource intensive: several hours, ~40GB RAM, and ~300GB disk (~150GB with cleanup via prepare_databases.py -c)
- MetaLAFFA is supported on Mac and Linux but not Windows

## Evidence

- [intro] Host read removal via Bowtie 2 mapping to hs37d5: [evidence span withheld — anchor status: not_found]
- [methods] Duplicate read filtering: "Duplicate read filtering aims to remove reads that erroneously appear in a sample multiple times due to artifacts of sequencing (PCR duplicates or optical duplicates)."
- [methods] Quality trimming criteria: "trimming bases with quality below a certain threshold from one or both ends of a read, as well as removing any reads that fall below a specific length threshold"
- [readme] Host filter database preparation: "`-hr`: Download and prepare the database of human reference and decoy sequences (used in the 1000 genomes project) for host filtering."
- [results] Post-HMP QC caveat: "Since these data files are post-HMP quality control, minimal reads were discarded from each sample during MetaLAFFA’s quality control phase."
