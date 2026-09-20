---
name: read-identity-filtered-coverage-quantification
description: Use when viral scaffolds (e.g., VIBRANT/VirSorter predictions) must be
  quantified from read alignment data to build the coverage table that vRhyme consumes
  for single- or multi-sample coverage effect size comparisons between scaffolds.
when_to_use_negative:
- Coverage is already supplied as a -c coverage table, in which case alignment and
  coverage outlier removal are skipped
- Input is a whole metagenome rather than predicted viral sequences, since performance
  on whole metagenomes has not been fully evaluated and vRhyme is not meant to bin
  microbes
- Binning non-viral (bacterial/archaeal) genomes, which is an anticipated limitation
  of vRhyme
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Bowtie2
  role: Default aligner for mapping reads to the viral scaffold FASTA using '--no-unal
    --no-discordant' to build coverage
  repo: http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- name: BWA
  role: Alternative aligner run with 'mem' when --aligner selects BWA; index created
    at the -i location if absent
  repo: https://github.com/lh3/bwa
- name: Samtools
  role: Converts generated SAM files to BAM and sorts/indexes them before Pysam parsing
- name: PySam
  role: Parses alignments and retains reads meeting the --read_id 0.97 identity minimum
    for coverage computation
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:d8c419159f174f38e1dadb825effbc425c96b70c8ea99d2e6f44bb6477a1da1e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/AnantharamanLab/vRhyme
  tools:
  - BWA
  - Bowtie2
  - PySam
  - Python
  - Samtools
  - vRhyme
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# read-identity-filtered-coverage-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies per-scaffold read coverage in vRhyme by aligning reads (Bowtie2/BWA), parsing BAM alignments with Pysam, retaining only reads with >=97% (--read_id 0.97) identity, and building an average-and-standard-deviation coverage table per scaffold per sample for downstream pairwise effect-size comparisons.

## When to use

Use when viral scaffolds (e.g., VIBRANT/VirSorter predictions) must be quantified from read alignment data to build the coverage table that vRhyme consumes for single- or multi-sample coverage effect size comparisons between scaffolds.

## When NOT to use

- Coverage is already supplied as a -c coverage table, in which case alignment and coverage outlier removal are skipped
- Input is a whole metagenome rather than predicted viral sequences, since performance on whole metagenomes has not been fully evaluated and vRhyme is not meant to bin microbes
- Binning non-viral (bacterial/archaeal) genomes, which is an anticipated limitation of vRhyme

## Inputs

- Viral scaffold FASTA (e.g., VIBRANT/VirSorter predictions from a metagenome)
- Paired (_1/_2.fastq), unpaired, or interleaved metagenomic reads
- Optional pre-computed coverage table (-c) in the vRhyme_coverage_values.tsv format

## Outputs

- Per-sample (sample).coverage.tsv files in the vRhyme_coverage_files folder
- Combined vRhyme_coverage_values.tsv with average and standard deviation coverage per scaffold per sample
- vRhyme_machine_distances.tsv of Cohen's d-derived normalized d' distances for machine learning

## How to apply

Align paired (_1/_2.fastq), unpaired, or interleaved reads to the viral scaffold FASTA with Bowtie2 using '--no-unal --no-discordant' (or BWA 'mem'), creating the Bowtie2/BWA index at the -i location if absent. Convert generated SAM files to sorted, indexed BAM with Samtools. Parse alignments with Pysam, retaining only reads meeting the --read_id minimum of 0.97 (97%) identity, computed as exact matches over aligned read length (aligned length minus gaps and mismatches). Mask coverage within the terminal --mask 150 bases on each scaffold end and optionally remove coverage outliers >=4.0 standard deviations above the average coverage (--outliers; off for -c coverage-table input). Assemble the per-sample per-scaffold table of average and standard deviation coverage, writing one (sample).coverage.tsv per sample plus the combined vRhyme_coverage_values.tsv in the vRhyme_coverage_files folder; these feed Cohen's d effect-size computation into vRhyme_machine_distances.tsv.

## Related tools

- **Bowtie2** (Default aligner for mapping reads to the viral scaffold FASTA using '--no-unal --no-discordant' to build coverage) — http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- **BWA** (Alternative aligner run with 'mem' when --aligner selects BWA; index created at the -i location if absent) — https://github.com/lh3/bwa
- **Samtools** (Converts generated SAM files to BAM and sorts/indexes them before Pysam parsing)
- **PySam** (Parses alignments and retains reads meeting the --read_id 0.97 identity minimum for coverage computation) — https://github.com/pysam-developers/pysam

## Examples

```
bowtie2 -x index -1 sample_1.fastq -2 sample_2.fastq --no-unal --no-discordant -S sample.sam
```

## Evaluation signals

- Every retained alignment has identity >= 0.97 (exact matches over aligned length minus gaps and mismatches)
- Coverage values within the terminal 150 bases (--mask) of each scaffold end are excluded from statistics
- vRhyme_coverage_values.tsv is a tab-separated table with one average and one standard deviation column per input sample
- vRhyme_machine_distances.tsv contains one distance row per scaffold pair and matches the -c coverage-table input format expectations
- Samtools reports Success in test_vRhyme.py when any coverage input other than -c is used

## Limitations

- Coverage outlier removal (--outliers, >=4.0 SD threshold) is disabled when coverage is supplied via -c
- vRhyme performs optimally with multiple input samples for more robust coverage variance comparisons, though the optimal number depends on dataset construction
- Pysam >= 0.15.0 is required; Samtools is mandatory whenever coverage input other than -c is used

## Evidence

- [methods] Parse alignments with Pysam, retaining only reads meeting the --read_id minimum of 0.97 (97%) identity, computed as exact matches over aligned read length: [evidence span withheld — anchor status: not_found]
- [methods] Mask coverage values within the terminal --mask 150 bases on each scaffold end, and optionally remove coverage outliers: [evidence span withheld — anchor status: not_found]
