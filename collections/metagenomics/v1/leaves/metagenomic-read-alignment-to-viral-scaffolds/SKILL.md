---
name: metagenomic-read-alignment-to-viral-scaffolds
description: Use when predicted viral scaffolds (e.g., from VIBRANT or VirSorter)
  must be binned from metagenomes and per-sample coverage tables are needed as input
  to vRhyme's coverage variance / effect-size binning network.
when_to_use_negative:
- The goal is binning bacterial, archaeal, or eukaryotic genomes — vRhyme is built
  for viral sequences/scaffolds and 'is not meant to bin microbes'
- A combined coverage table already exists (use vRhyme -c input; note coverage outlier
  removal is disabled for -c input)
- Only a single input sample is available and robust coverage variance comparisons
  are required — vRhyme 'performs optimally with multiple input samples'
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: vRhyme
  role: Orchestrates the alignment, coverage-table assembly, masking, outlier removal,
    and Cohen's d effect-size computation
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Bowtie2
  role: Default aligner for mapping reads to viral scaffolds with '--no-unal --no-discordant'
  repo: http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- name: BWA
  role: Alternative aligner run with 'mem' via the --aligner flag
  repo: https://github.com/lh3/bwa
- name: Samtools
  role: Converts SAM to sorted/indexed BAM; required whenever coverage input other
    than -c is used
  repo: http://www.htslib.org/
- name: PySam
  role: Parses alignments to compute read identity (exact matches over aligned length)
    and enforce the 0.97 --read_id threshold
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:39d22ba6db4c27c15c15ff6b23ee144a0a0f3ac489e31fcc532006dfdd00372d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/AnantharamanLab/vRhyme
  repo_url: https://github.com/AnantharamanLab/vRhyme
  tools:
  - BWA
  - Bowtie2
  - PySam
  - Python
  - Samtools
  - vRhyme
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# metagenomic-read-alignment-to-viral-scaffolds

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align metagenomic reads (paired, unpaired, or interleaved FASTQ) to predicted viral scaffold FASTAs with Bowtie2 or BWA-mem and convert the alignments into a per-scaffold, per-sample coverage table of averages and standard deviations, masking terminal scaffold coverage. This table is the input for vRhyme's Cohen's d coverage effect-size comparisons between scaffold pairs.

## When to use

Use when predicted viral scaffolds (e.g., from VIBRANT or VirSorter) must be binned from metagenomes and per-sample coverage tables are needed as input to vRhyme's coverage variance / effect-size binning network.

## When NOT to use

- The goal is binning bacterial, archaeal, or eukaryotic genomes — vRhyme is built for viral sequences/scaffolds and 'is not meant to bin microbes'
- A combined coverage table already exists (use vRhyme -c input; note coverage outlier removal is disabled for -c input)
- Only a single input sample is available and robust coverage variance comparisons are required — vRhyme 'performs optimally with multiple input samples'

## Inputs

- Viral scaffold FASTA (typically VIBRANT/VirSorter predictions)
- Paired-end reads (_1.fastq and _2.fastq), unpaired reads, or interleaved reads
- Optional precomputed coverage table supplied via the -c flag

## Outputs

- One (sample).coverage.tsv per sample with average and standard deviation of coverage per scaffold
- Combined vRhyme_coverage_values.tsv in the vRhyme_coverage_files folder
- Sorted and indexed BAM files per sample
- vRhyme_machine_distances.tsv with one distance row per scaffold pair

## How to apply

Build a Bowtie2 or BWA index at the -i location for the input viral scaffold FASTA (creating it if absent), then align paired (_1/_2.fastq), unpaired, or interleaved reads using Bowtie2 with '--no-unal --no-discordant' (default aligner) or BWA 'mem'. Convert SAM to sorted, indexed BAM with Samtools. Parse alignments with Pysam, retaining only reads meeting the --read_id minimum of 0.97 (97%) identity, computed as exact matches over aligned read length (aligned length minus gaps and mismatches). Mask coverage within the terminal --mask 150 bases of each scaffold end and optionally remove coverage outliers ≥4.0 standard deviations above the average (--outliers; off for -c coverage-table input). Write one (sample).coverage.tsv per sample plus the combined vRhyme_coverage_values.tsv in the vRhyme_coverage_files folder; downstream, compute Cohen's d coverage effect sizes per scaffold pair and sample, apply the --penalty_w 0.20 penalty weight against preset thresholds with at most --penalty_n 10 penalties, and normalize to d' distances written to vRhyme_machine_distances.tsv.

## Related tools

- **vRhyme** (Orchestrates the alignment, coverage-table assembly, masking, outlier removal, and Cohen's d effect-size computation) — https://github.com/AnantharamanLab/vRhyme
- **Bowtie2** (Default aligner for mapping reads to viral scaffolds with '--no-unal --no-discordant') — http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- **BWA** (Alternative aligner run with 'mem' via the --aligner flag) — https://github.com/lh3/bwa
- **Samtools** (Converts SAM to sorted/indexed BAM; required whenever coverage input other than -c is used) — http://www.htslib.org/
- **PySam** (Parses alignments to compute read identity (exact matches over aligned length) and enforce the 0.97 --read_id threshold) — https://github.com/pysam-developers/pysam

## Examples

```
vRhyme -i viral_scaffolds.fna -r reads_1.fastq -v reads_2.fastq -o out_vRhyme -t 15 --aligner 2 --read_id 0.97 --mask 150 --penalty_w 0.20
```

## Evaluation signals

- vRhyme_machine_distances.tsv and vRhyme_coverage_values.tsv exist as tab-separated tables
- vRhyme_coverage_values.tsv has one coverage column per input sample and matches the -c input format
- vRhyme_machine_distances.tsv has one distance row per scaffold pair
- Retained reads satisfy the --read_id 0.97 minimum identity, and terminal 150 bases (--mask default) are excluded from coverage statistics
- test_vRhyme.py reports Success for Samtools, PySam, and the chosen aligner (Bowtie2 and/or BWA)

## Limitations

- vRhyme is built to run on viral sequences/scaffolds; performance on a whole metagenome 'has not been fully evaluated' and it is not meant to bin microbes
- Coverage outlier removal and circular scaffold removal were only added in v1.1.0 (March 2022); --penalty_n maximum is <30% of the number of input samples
- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome

## Evidence

- [methods] coverage table: [evidence span withheld — anchor status: not_found]
- [methods] alignment parameters: [evidence span withheld — anchor status: not_found]
- [methods] read identity filter: [evidence span withheld — anchor status: not_found]
