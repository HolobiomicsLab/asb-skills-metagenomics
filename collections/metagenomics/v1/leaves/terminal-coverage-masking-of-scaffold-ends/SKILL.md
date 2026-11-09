---
name: terminal-coverage-masking-of-scaffold-ends
description: Use when computing per-scaffold read coverage tables for viral scaffold
  binning from BAM/CRAM alignments, where terminal coverage on scaffold ends should
  be excluded before pairwise coverage effect-size comparisons.
when_to_use_negative:
- 'Input is already a pre-computed coverage table via the -c flag: outlier removal
  (and by extension vRhyme''s internal coverage recalculation) is turned off for -c
  coverage-table input'
- 'The goal is binning microbial (non-viral) genomes: vRhyme is explicitly not meant
  to bin microbes and performance on whole metagenomes is not fully evaluated'
- Scaffolds are shorter than twice the --mask window (e.g. 2 x 150 bases), where masking
  would remove most of the scaffold's usable coverage signal
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: vRhyme
  role: 'Implements the coverage-processing layer: parses alignments, applies the
    --mask terminal window (default 150 bases) and --read_id 0.97 identity filter,
    builds per-scaffold coverage tables, and computes Cohen''s d effect sizes'
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Bowtie2
  role: Default aligner producing the SAM/BAM alignments (with '--no-unal --no-discordant')
    whose per-base coverage is masked at scaffold termini
  repo: http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- name: BWA
  role: Alternative aligner ('mem') whose alignments feed the same coverage table
    construction and terminal masking
  repo: https://github.com/lh3/bwa
- name: Samtools
  role: Converts generated SAM files to BAM and sorts/indexes them before Pysam parsing
  repo: http://www.htslib.org/
- name: PySam
  role: Parses BAM alignments and computes per-base coverage, applying the terminal
    --mask and --read_id 0.97 identity filters
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:119f7dbb3e17b4cf1d47cd7371533e4b5ef5a71c0f81d42663de16d74cce9ebe
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# Terminal coverage masking of scaffold ends

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A coverage-processing step in vRhyme that zeroes out coverage accumulated in the terminal --mask bases (default 150) at each end of every input viral scaffold, so that coverage effect-size comparisons between scaffold pairs are not biased by edge-mapping artifacts before the Cohen's d / d' distances reach the machine learning layer.

## When to use

Use when computing per-scaffold read coverage tables for viral scaffold binning from BAM/CRAM alignments, where terminal coverage on scaffold ends should be excluded before pairwise coverage effect-size comparisons.

## When NOT to use

- Input is already a pre-computed coverage table via the -c flag: outlier removal (and by extension vRhyme's internal coverage recalculation) is turned off for -c coverage-table input
- The goal is binning microbial (non-viral) genomes: vRhyme is explicitly not meant to bin microbes and performance on whole metagenomes is not fully evaluated
- Scaffolds are shorter than twice the --mask window (e.g. 2 x 150 bases), where masking would remove most of the scaffold's usable coverage signal

## Inputs

- Sorted BAM/CRAM alignment files of reads mapped to a viral scaffold FASTA (Bowtie2 '--no-unal --no-discordant' or BWA 'mem')
- Viral scaffold FASTA (predicted viruses, e.g. from VIBRANT or VirSorter) from which scaffold lengths determine the masked terminal windows
- Sample read files: paired _1/_2.fastq, unpaired, or interleaved FASTQ

## Outputs

- Per-sample coverage tables: one (sample).coverage.tsv per sample with per-scaffold average and standard deviation of coverage after terminal masking
- Combined coverage table vRhyme_coverage_values.tsv (equivalent to -c input format) in the vRhyme_coverage_files folder
- Downstream vRhyme_machine_distances.tsv with one Cohen's d-derived d' distance row per scaffold pair

## How to apply

After aligning paired (_1/_2.fastq), unpaired, or interleaved reads to the viral scaffold FASTA with Bowtie2 ('--no-unal --no-discordant') or BWA 'mem', parse the sorted BAM files with Pysam, retaining only reads meeting the --read_id minimum of 0.97 identity (exact matches over aligned length minus gaps and mismatches). While tallying per-base coverage, exclude (mask) coverage values falling within the terminal --mask 150 bases at each end of every scaffold. Optionally also remove coverage outliers at least 4.0 standard deviations above the average coverage (--outliers; not applicable when coverage tables are supplied via -c). Write one (sample).coverage.tsv per sample plus the combined vRhyme_coverage_values.tsv in the vRhyme_coverage_files folder, then feed these averages and standard deviations per scaffold per sample into Cohen's d effect-size computation with --penalty_w 0.20 and --penalty_n 10 penalties to obtain d' distances in vRhyme_machine_distances.tsv. Masking terminal bases is the rationale: end-of-contig mapping artifacts would otherwise inflate coverage variance and distort scaffold-pair distances used by the supervised machine learning binning.

## Related tools

- **vRhyme** (Implements the coverage-processing layer: parses alignments, applies the --mask terminal window (default 150 bases) and --read_id 0.97 identity filter, builds per-scaffold coverage tables, and computes Cohen's d effect sizes) — https://github.com/AnantharamanLab/vRhyme
- **Bowtie2** (Default aligner producing the SAM/BAM alignments (with '--no-unal --no-discordant') whose per-base coverage is masked at scaffold termini) — http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- **BWA** (Alternative aligner ('mem') whose alignments feed the same coverage table construction and terminal masking) — https://github.com/lh3/bwa
- **Samtools** (Converts generated SAM files to BAM and sorts/indexes them before Pysam parsing) — http://www.htslib.org/
- **PySam** (Parses BAM alignments and computes per-base coverage, applying the terminal --mask and --read_id 0.97 identity filters) — https://github.com/pysam-developers/pysam

## Examples

```
vRhyme -i viral_scaffolds.fna -r reads -m 5 --mask 150 --read_id 0.97 -o vRhyme_out
```

## Evaluation signals

- Per-scaffold coverage averages and standard deviations in vRhyme_coverage_values.tsv should reflect no contribution from the first and last --mask (150) bases of each scaffold
- Per-sample (sample).coverage.tsv files plus the combined vRhyme_coverage_values.tsv exist in the vRhyme_coverage_files folder as tab-separated tables with one coverage column per sample
- Only alignments with identity >= 0.97 (exact matches over aligned length minus gaps and mismatches) contribute to the coverage tallies
- The combined coverage table matches the -c input format accepted by vRhyme
- vRhyme_machine_distances.tsv exists as a tab-separated table with one distance row per scaffold pair, confirming the masked coverage table fed the Cohen's d / d' computation

## Limitations

- Coverage outlier removal (--outliers, threshold 4.0 standard deviations above average) is disabled when coverage tables are supplied via -c, so masking-and-outlier behavior cannot be combined with pre-computed tables
- vRhyme performs optimally with multiple input samples for more robust coverage variance comparisons; single-sample runs have less stable coverage statistics
- vRhyme is built to run on viral sequences/scaffolds; performance on an entire metagenome as input has not been fully evaluated

## Evidence

- [methods] Terminal masking threshold and pipeline position: [evidence span withheld — anchor status: not_found]
- [methods] Coverage table construction from alignments: [evidence span withheld — anchor status: not_found]
- [methods] Effect-size step consuming the masked coverage table: [evidence span withheld — anchor status: not_found]
