---
name: coverage-cohen-d-effect-size-computation
description: Use when binning viral scaffolds with vRhyme and read coverage abundance
  must be converted into pairwise scaffold effect-size distances (vRhyme_machine_distances.tsv)
  from FASTQ reads or a pre-computed coverage table.
when_to_use_negative:
- Input is a whole metagenome rather than predicted viral sequences — vRhyme is 'not
  meant to bin microbes' and whole-metagenome performance 'has not been fully evaluated'
- Binning non-viral genomes or separating multiple inactive (non-replicating) prophages
  from the same host genome, which are noted failure modes
- Only a single input sample is available when the goal demands robust coverage variance
  comparisons — vRhyme 'performs optimally with multiple input samples'
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: vRhyme
  role: 'Orchestrates the coverage-processing layer: parses alignments, builds the
    coverage table, and computes Cohen''s d / d'' distances'
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Bowtie2
  role: Default aligner for mapping reads to the viral scaffold FASTA ('--no-unal
    --no-discordant'), generating SAM input for coverage
  repo: http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- name: BWA
  role: Alternative aligner run with 'mem' for read alignment against the scaffold
    index
  repo: https://github.com/lh3/bwa
- name: Samtools
  role: Converts SAM to sorted, indexed BAM for parsing (required when using any coverage
    input beside -c)
  repo: http://www.htslib.org/
- name: PySam
  role: Python API used to parse BAM alignments and filter reads by the 0.97 --read_id
    identity threshold
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:b6cc46af8f3eb47c21bb4013945877a38021d88511ad61c1965ac86a8f39d4f6
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# coverage-cohen-d-effect-size-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

vRhyme's coverage-processing layer converts read alignments against viral scaffold FASTAs into a per-sample, per-scaffold coverage table (average and standard deviation), then computes Cohen's d coverage effect sizes between every scaffold pair, normalized to d' distances that feed the downstream machine-learning network.

## When to use

Use when binning viral scaffolds with vRhyme and read coverage abundance must be converted into pairwise scaffold effect-size distances (vRhyme_machine_distances.tsv) from FASTQ reads or a pre-computed coverage table.

## When NOT to use

- Input is a whole metagenome rather than predicted viral sequences — vRhyme is 'not meant to bin microbes' and whole-metagenome performance 'has not been fully evaluated'
- Binning non-viral genomes or separating multiple inactive (non-replicating) prophages from the same host genome, which are noted failure modes
- Only a single input sample is available when the goal demands robust coverage variance comparisons — vRhyme 'performs optimally with multiple input samples'

## Inputs

- Paired (_1/_2.fastq), unpaired, or interleaved FASTQ reads from metagenome samples
- Viral scaffold FASTA (typically predicted viruses, e.g., from VIBRANT or VirSorter)
- Optional pre-computed per-scaffold coverage table supplied via the -c flag

## Outputs

- One (sample).coverage.tsv per input sample containing per-scaffold average and standard deviation of coverage
- Combined vRhyme_coverage_values.tsv (one coverage column per sample) in the vRhyme_coverage_files folder
- vRhyme_machine_distances.tsv with one normalized d' distance row per scaffold pair
- Sorted/indexed BAM alignment files per sample

## How to apply

Align paired (_1/_2.fastq), unpaired, or interleaved reads to the input viral scaffold FASTA using Bowtie2 with '--no-unal --no-discordant' (default aligner) or BWA run with 'mem', building the index at the -i location if absent, then convert SAM to sorted, indexed BAM with Samtools. Parse alignments with Pysam, retaining only reads meeting the --read_id minimum of 0.97 identity (exact matches over aligned read length, i.e., aligned length minus gaps and mismatches). Mask coverage within the terminal --mask 150 bases on each scaffold end and optionally remove coverage outliers >= 4.0 standard deviations above the average (--outliers; disabled for -c coverage-table input). Assemble one (sample).coverage.tsv per sample plus the combined vRhyme_coverage_values.tsv (equivalent to the -c format) in the vRhyme_coverage_files folder. For each scaffold pair and sample, compute Cohen's d effect sizes, apply the penalty weight --penalty_w 0.20 against preset thresholds with a maximum of --penalty_n 10 penalties, and normalize to obtain d' distances written to vRhyme_machine_distances.tsv for downstream supervised machine learning.

## Related tools

- **vRhyme** (Orchestrates the coverage-processing layer: parses alignments, builds the coverage table, and computes Cohen's d / d' distances) — https://github.com/AnantharamanLab/vRhyme
- **Bowtie2** (Default aligner for mapping reads to the viral scaffold FASTA ('--no-unal --no-discordant'), generating SAM input for coverage) — http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- **BWA** (Alternative aligner run with 'mem' for read alignment against the scaffold index) — https://github.com/lh3/bwa
- **Samtools** (Converts SAM to sorted, indexed BAM for parsing (required when using any coverage input beside -c)) — http://www.htslib.org/
- **PySam** (Python API used to parse BAM alignments and filter reads by the 0.97 --read_id identity threshold) — https://github.com/pysam-developers/pysam

## Examples

```
vRhyme -h
```

## Evaluation signals

- vRhyme_machine_distances.tsv exists as a tab-separated table with exactly one distance row per scaffold pair
- vRhyme_coverage_values.tsv has one coverage column per input sample and matches the -c coverage-table input format
- Only alignments passing the --read_id 0.97 identity threshold (exact matches over aligned length minus gaps and mismatches) contribute to coverage
- Coverage values within the terminal --mask 150 bases of each scaffold end are masked in the table
- Number of applied penalties does not exceed --penalty_n (10 by default; maximum is <30% of input samples in v1.1.0)

## Limitations

- Coverage outlier removal (--outliers) is disabled when coverage is supplied via the -c coverage-table input
- --penalty_n maximum is bounded to <30% of the number of input samples (v1.1.0 change)
- Performance depends on dataset construction; the optimal number of input coverage samples varies
- vRhyme is built for viral sequences/scaffolds; binning non-viral genomes is an anticipated limitation and inactive prophages from one host are hard to separate

## Evidence

- [intro] we present vRhyme, a software tool that incorporates supervised machine learning based classification of diverse sequence feature compositions as well as read coverage abundance effect size: "we present vRhyme, a software tool that incorporates supervised machine learning based classification of diverse sequence feature compositions as well as read coverage abundance effect size"
- [discussion] vRhyme performs optimally with multiple input samples for more robust coverage variance comparisons, though the optimal value depends on how the dataset or metagenome was constructed: "vRhyme performs optimally with multiple input samples for more robust coverage variance comparisons, though the optimal value depends on how the dataset or metagenome was constructed"
- [methods] Read alignment (Bowtie2, BWA): "Read alignment (Bowtie2, BWA)"
- [readme] vRhyme functions by utilizing coverage variance comparisons and supervised machine learning classification of sequence features to construct viral metagenome-assembled genomes (vMAGs).: "vRhyme functions by utilizing coverage variance comparisons and supervised machine learning classification of sequence features to construct viral metagenome-assembled genomes (vMAGs)."
