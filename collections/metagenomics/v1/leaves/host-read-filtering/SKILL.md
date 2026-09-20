---
name: host-read-filtering
description: Use when raw shotgun metagenomic FASTQs contain host contamination (e.g.
  human gut samples) and host-derived reads must be removed before gene mapping and
  functional annotation.
when_to_use_negative:
- Input data are not shotgun metagenomic reads (e.g. amplicon data or already host-filtered
  FASTQs)
- The host organism is not human and no alternative host reference database has been
  prepared — an appropriate host index must be substituted
- The goal is a later pipeline step such as duplicate read filtering or quality trimming,
  which are separate MetaLAFFA steps
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Bowtie 2
  role: Aligns input reads to the hs37d5 host index; reads identified as human (match
    quality above threshold) are discarded
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: MetaLAFFA
  role: Snakemake-based pipeline that orchestrates host read filtering as the first
    quality-control step and produces the summary statistics
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Conda
  role: Installation environment manager for MetaLAFFA and the prepare_databases.py
    script
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:6753dbc5ae387a3f1fc8036690a8026dbf5178123ce2c9008fccad34628be712
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - Bowtie 2
  - Conda
  - MetaLAFFA
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# host-read-filtering

## Summary

Quality-control step in MetaLAFFA that removes host reads from shotgun metagenomic FASTQs by aligning them to the hs37d5 human genome reference (with 1000 Genomes Project decoy sequences) using Bowtie 2 and discarding reads matching the host above a quality threshold.

## When to use

Use when raw shotgun metagenomic FASTQs contain host contamination (e.g. human gut samples) and host-derived reads must be removed before gene mapping and functional annotation.

## When NOT to use

- Input data are not shotgun metagenomic reads (e.g. amplicon data or already host-filtered FASTQs)
- The host organism is not human and no alternative host reference database has been prepared — an appropriate host index must be substituted
- The goal is a later pipeline step such as duplicate read filtering or quality trimming, which are separate MetaLAFFA steps

## Inputs

- Raw shotgun metagenomic FASTQ files (adapter-removed), gzipped, named <sample>.R1.fastq.gz / <sample>.R2.fastq.gz / <sample>.S.fastq.gz
- Bowtie 2 index of hs37d5 human reference with 1000 Genomes Project decoy sequences

## Outputs

- Host-filtered FASTQ files
- Per-sample summary statistics (number of reads, average read length, average base quality for input and filtered FASTQs; number of host reads removed)

## How to apply

Prepare the host database with `prepare_databases.py -hr`, which installs the hs37d5 human reference plus decoy sequences from the 1000 Genomes Project as a Bowtie 2 index. Align input FASTQs (named <sample>.R1.fastq.gz, <sample>.R2.fastq.gz, <sample>.S.fastq.gz) to this index with Bowtie 2, then discard reads whose match qualities exceed the filtering threshold. Write the surviving reads as host-filtered FASTQs and compute per-sample summary statistics (read counts, average read length, average base quality for input and filtered FASTQs, plus the number of host reads removed). Users can substitute an alternative host reference database to filter reads from non-human host organisms.

## Related tools

- **Bowtie 2** (Aligns input reads to the hs37d5 host index; reads identified as human (match quality above threshold) are discarded) — https://github.com/borenstein-lab/MetaLAFFA
- **MetaLAFFA** (Snakemake-based pipeline that orchestrates host read filtering as the first quality-control step and produces the summary statistics) — https://github.com/borenstein-lab/MetaLAFFA
- **Conda** (Installation environment manager for MetaLAFFA and the prepare_databases.py script)

## Examples

```
prepare_databases.py -hr
```

## Evaluation signals

- Host-filtered FASTQ exists for each sample after the step completes
- The summary table reports the number of host reads removed during host filtering (e.g. for HMP stool sample SRS017307)
- Read counts and average read length/base-quality statistics are present for both input and filtered FASTQs
- Surviving reads contain no alignments to the hs37d5 index above the match-quality threshold

## Limitations

- Default host reference is human (hs37d5 + decoys); non-human hosts require substituting an alternative host database
- Database preparation via prepare_databases.py is resource-intensive (several hours, ~40GB RAM, ~150–300GB disk depending on intermediate cleanup)
- Reads with weak/ambiguous host matches below the threshold may be retained
- MetaLAFFA is supported on Mac and Linux only, not Windows

## Evidence

- [intro] Mapping reads to hs37d5 with Bowtie 2: [evidence span withheld — anchor status: not_found]
- [methods] Host filtering rationale: "Host read filtering is usually performed by aligning reads to one or more reference genomes for the host species and removing reads with match qualities above a certain threshold."
- [readme] Database preparation flag: "`-hr`: Download and prepare the database of human reference and decoy sequences (used in the 1000 genomes project) for host filtering."
- [results] Validation of step: "Since these data files are post-HMP quality control, minimal reads were discarded from each sample during MetaLAFFA’s quality control phase."
