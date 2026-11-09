---
name: sam-bam-alignment-processing-and-sorting
description: Use when raw read alignments (SAM files from Bowtie2 or BWA-mem against
  a viral scaffold FASTA) must be converted, sorted, indexed, and parsed before computing
  per-scaffold coverage statistics.
when_to_use_negative:
- Coverage input is already provided as a pre-computed coverage table via the -c flag,
  so read alignment and SAM/BAM processing are skipped (outlier removal is also disabled
  for -c input)
- The target is binning non-viral genomes or whole unfiltered metagenomes — vRhyme
  is explicitly not meant to bin microbes
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0102
tools:
- name: Bowtie2
  role: Default aligner producing SAM files from reads mapped to the viral scaffold
    FASTA using '--no-unal --no-discordant'
  repo: http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- name: BWA
  role: Alternative aligner run with 'mem' when --aligner selects it
  repo: https://github.com/lh3/bwa
- name: Samtools
  role: Converts SAM to BAM and sorts/indexes BAM files prior to coverage parsing
  repo: http://www.htslib.org/
- name: PySam
  role: Parses sorted BAM alignments to compute per-read identity and per-scaffold
    coverage
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:ca720debed4cbed60143f1ed7ba1f76481cc80be315f4569c6da3792525560e2
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# sam-bam-alignment-processing-and-sorting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts Bowtie2/BWA SAM alignments of metagenomic reads against viral scaffolds into sorted, indexed BAM files with Samtools, then parses them with PySam for vRhyme's coverage table. This is the prerequisite alignment-handling layer for coverage-variance-based viral scaffold binning.

## When to use

Use when raw read alignments (SAM files from Bowtie2 or BWA-mem against a viral scaffold FASTA) must be converted, sorted, indexed, and parsed before computing per-scaffold coverage statistics.

## When NOT to use

- Coverage input is already provided as a pre-computed coverage table via the -c flag, so read alignment and SAM/BAM processing are skipped (outlier removal is also disabled for -c input)
- The target is binning non-viral genomes or whole unfiltered metagenomes — vRhyme is explicitly not meant to bin microbes

## Inputs

- Paired (_1/_2.fastq), unpaired, or interleaved metagenomic reads (FASTQ)
- Input viral scaffold FASTA (e.g., predicted viral sequences from VIBRANT or VirSorter)
- SAM alignment files produced by Bowtie2 (--no-unal --no-discordant) or BWA mem

## Outputs

- Sorted, indexed BAM files per sample (generated from SAM via Samtools)
- Per-sample (sample).coverage.tsv and combined vRhyme_coverage_values.tsv in the vRhyme_coverage_files folder
- Filtered read alignments passing the 0.97 (--read_id) identity minimum used for coverage computation

## How to apply

Align paired (_1/_2.fastq), unpaired, or interleaved reads to the input viral scaffold FASTA with Bowtie2 ('--no-unal --no-discordant', the default aligner) or BWA run with 'mem', creating the Bowtie2/BWA index at the -i location if absent. Convert each generated SAM to BAM and sort/index it with Samtools so random-access queries are possible. Parse the sorted BAMs with PySam (>= 0.15.0), a lightweight wrapper of the samtools C-API, retaining only reads meeting the --read_id minimum of 0.97 identity (exact matches over aligned length, i.e., aligned length minus gaps and mismatches). These filtered alignments feed vRhyme's coverage table (vRhyme_coverage_values.tsv) after masking terminal --mask 150 bases and optional outlier removal (>=4.0 SD above mean, --outliers).

## Related tools

- **Bowtie2** (Default aligner producing SAM files from reads mapped to the viral scaffold FASTA using '--no-unal --no-discordant') — http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- **BWA** (Alternative aligner run with 'mem' when --aligner selects it) — https://github.com/lh3/bwa
- **Samtools** (Converts SAM to BAM and sorts/indexes BAM files prior to coverage parsing) — http://www.htslib.org/
- **PySam** (Parses sorted BAM alignments to compute per-read identity and per-scaffold coverage) — https://github.com/pysam-developers/pysam

## Examples

```
samtools view -b -o sample.bam sample.sam; samtools sort sample.bam -o sample.sorted.bam; samtools index sample.sorted.bam
```

## Evaluation signals

- vRhyme_coverage_values.tsv exists as a tab-separated table with one coverage column per input sample
- Each per-sample (sample).coverage.tsv in vRhyme_coverage_files matches the corresponding sample's rows in the combined table
- Downstream vRhyme_machine_distances.tsv exists with one distance row per scaffold pair, confirming alignments produced a usable coverage table
- aux/test_vRhyme.py reports 'Success' for Samtools (required for any coverage input besides -c) and for Bowtie2 and/or BWA when reads are input via -r/-v/-u
- Sorted BAMs are indexable and queryable (Samtools index completes without error), confirming correct SAM-to-BAM conversion

## Limitations

- Outlier removal is off when coverage is supplied via -c, so the alignment-processing path assumes raw reads when coverage refinement (masking plus ≥4.0 SD outlier removal) is desired
- vRhyme is built for viral sequences/scaffolds; performance on an entire metagenome has not been fully evaluated and it is not meant to bin microbes
- Bowtie2/BWA and Samtools are optional only if -c coverage input is used; otherwise they must be installed and pass the installation test

## Evidence

- [methods] Align paired (_1/_2.fastq), unpaired, or interleaved reads to the input viral scaffold FASTA using Bowtie2 ('--no-unal --no-discordant', default aligner) or BWA run with 'mem': [evidence span withheld — anchor status: not_found]
- [methods] Convert generated SAM files to BAM and sort/index with Samtools.: [evidence span withheld — anchor status: not_found]
- [readme] Pysam is a lightweight wrapper of the samtools C-API.: [evidence span withheld — anchor status: external_doc]
