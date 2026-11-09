---
name: multi-sample-coverage-profile-processing
description: Use when assembling metagenomic short reads into contigs and you need
  sample-wise read coverage per contig as input features for a binner such as CH-Bin.
when_to_use_negative:
- You already have a per-sample coverage TSV for the contigs and only need to feed
  it into the binner
- Only contigs are available with no reads/BAM files to map, so coverage cannot be
  recomputed
- Binning long reads directly (e.g. long-read pipelines such as MetaBCC-LR) rather
  than short-read-assembled contigs
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: CoverM
  role: Computes sample-wise read coverage of assembled contigs by read mapping; output
    TSV is the coverage feature input for binning
  repo: https://github.com/wwood/CoverM
- name: CH-Bin
  role: Consumes the CoverM coverage TSV together with oligonucleotide composition
    vectors to perform convex hull distance based binning
  repo: https://github.com/kdsuneraavinash/CH-Bin
- name: minimap2
  role: One of the read mappers CoverM uses internally to align reads to contigs during
    coverage calculation
- name: SAMTools
  role: Dependency of CoverM used in read mapping/coverage computation (v1.9 required
    when compiling from source)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.compbiolchem.2022.107734
    title: 'CH-Bin: A convex hull based approach for binning metagenomic contigs'
  source_content_hash: sha256:dfe8436897730dc52f6dbfec9395843962e12fce8080e72dcaf24d5069052260
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kdsuneraavinash/CH-Bin
  repo_url: https://github.com/wwood/CoverM
  tools:
  - Busy Bee Web
  - CH-Bin
  - CoverM
  - MaxBin2
  - MetaBAT2
  - Minimap2
  - SAMTools
  - SolidBin
  - metaSPAdes
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# multi-sample-coverage-profile-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes sample-wise read coverage profiles for metagenomic contigs with CoverM, producing the per-contig, per-sample coverage feature matrix that CH-Bin combines with oligonucleotide composition vectors for binning.

## When to use

Use when assembling metagenomic short reads into contigs and you need sample-wise read coverage per contig as input features for a binner such as CH-Bin.

## When NOT to use

- You already have a per-sample coverage TSV for the contigs and only need to feed it into the binner
- Only contigs are available with no reads/BAM files to map, so coverage cannot be recomputed
- Binning long reads directly (e.g. long-read pipelines such as MetaBCC-LR) rather than short-read-assembled contigs

## Inputs

- Multi-FASTA file of assembled metagenomic contigs (e.g. metaSPAdes output)
- Paired-end FASTQ reads, one pair per metagenomic sample (or BAM files sorted by reference)
- Optional pre-built mapper index from `coverm makedb` (e.g. minimap2-sr .mmi)

## Outputs

- Sample-wise contig coverage matrix in TSV format (rows = contigs, columns = samples) consumed by CH-Bin

## How to apply

After assembling reads into contigs (e.g. with metaSPAdes), use CoverM to map reads from each sample back to the contigs and emit a sample-wise coverage TSV with one row per contig. CoverM can be run directly (bioconda: `conda install coverm`), from a pre-compiled binary, or via `coverm makedb` to pre-generate a reusable mapper index (e.g. minimap2-sr) when multiple samples share the same reference. In the CH-Bin workflow the resulting coverage matrix is combined with O-dimensional composition feature vectors generated from oligonucleotide (k = 5, 6) frequencies, and short contigs are removed by the ContigLengthFilterBp threshold (default 1000 bp) before convex hull distance based binning. Validate that every contig in the binning input has a coverage entry for every sample.

## Related tools

- **CoverM** (Computes sample-wise read coverage of assembled contigs by read mapping; output TSV is the coverage feature input for binning) — https://github.com/wwood/CoverM
- **CH-Bin** (Consumes the CoverM coverage TSV together with oligonucleotide composition vectors to perform convex hull distance based binning) — https://github.com/kdsuneraavinash/CH-Bin
- **minimap2** (One of the read mappers CoverM uses internally to align reads to contigs during coverage calculation)
- **SAMTools** (Dependency of CoverM used in read mapping/coverage computation (v1.9 required when compiling from source))

## Examples

```
coverm contig -r contigs.fna -1 sample1_1.fq -2 sample1_2.fq --methods mean
```

## Evaluation signals

- Coverage TSV contains one row per contig in the (length-filtered) input FASTA and exactly one column per sample, with no missing values
- All coverage values are non-negative and finite; per-sample distributions are consistent with read depth expectations
- Contigs shorter than the ContigLengthFilterBp threshold (default 1000 bp) are excluded from both coverage table and binning input
- Downstream CH-Bin binning on Sim50 with the k = 5, 6 composition setting reproduces the reported F1 score of 96.91%
- AMBER evaluation of the resulting bins yields non-zero purity, completeness, and percentage-binned values for every evaluated dataset

## Limitations

- The CH-Bin repository provides no worked examples, so exact CLI wiring of the coverage TSV must be inferred from config/default.ini
- CH-Bin only supports Linux, and building requires cmake, build-essential, python3-dev, venv, and libboost
- CoverM compiled from source needs non-Rust dependencies (samtools v1.9, plus a mapper such as minimap2 v2.21 or strobealign v0.14.0) installed separately
- Coverage features alone are insufficient — CH-Bin also requires single-copy marker gene analysis (FragGeneScan, HMMER, marker.hmm) to estimate the number of bins

## Evidence

- [methods] Initially, we assemble short reads into longer contigs using a metagenomic assembler: [evidence span withheld — anchor status: not_found]
- [readme] CoverM aims to be a configurable, easy to use and fast DNA read coverage and relative abundance calculator focused on metagenomics applications.: [evidence span withheld — anchor status: external_doc]
