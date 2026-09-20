---
name: multi-tool-binning-benchmark-comparison
description: Use when a binning tool's relative performance on large, complex high-dimensional
  datasets must be quantified against baselines using standard AMBER metrics (purity,
  completeness, F1) on shared inputs.
when_to_use_negative:
- The goal is to evaluate bin refinement tools (Binning_refiner, GraphBin, GraphBin2,
  METAMVGL) rather than primary contig binners on common inputs
- Only small simulated datasets (Sim5, Sim10, Sim20) are of interest, where the large-dataset
  scaling behaviour being benchmarked is not exercised
- Reference genomes are unavailable, since AMBER ground truth requires aligning contigs
  to known references
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: CH-Bin
  role: Tool under test; convex-hull-distance binner run with k = 5, 6, 1000 bp contig
    filter and 75th-percentile ScmSelectPercentile
  repo: https://github.com/kdsuneraavinash/CH-Bin
- name: CoverM
  role: Computes sample-wise read coverage of assembled contigs, a shared input for
    all binners
  repo: https://github.com/wwood/CoverM
- name: MaxBin2
  role: Baseline binner run with default parameters
- name: MetaBAT2
  role: Baseline binner run with -m 1500
- name: SolidBin
  role: Baseline binner run in 'naive' mode with default parameters
  repo: https://github.com/sufforest/SolidBin
- name: BusyBee Web
  role: Baseline binner included in the comparison
- name: AMBER
  role: Produces ground truth and evaluates purity, completeness, F1, accuracy and
    percentage binned
- name: Minimap2
  role: Aligns contigs to reference genomes to build AMBER ground truth
- name: SAMTools
  role: Processes alignments, retaining contigs with highest mapping quality
- name: seq2vec
  role: Generates oligonucleotide composition feature vectors for CH-Bin
  repo: https://github.com/anuradhawick/seq2vec
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.compbiolchem.2022.107734
    title: 'CH-Bin: A convex hull based approach for binning metagenomic contigs'
  source_content_hash: sha256:4f37aa50424303f462c8170dda52942d68c548a3fb05c95a3f364f31d6e962b2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kdsuneraavinash/CH-Bin
  repo_url: https://github.com/kdsuneraavinash/CH-Bin
  tools:
  - AMBER
  - Busy Bee Web
  - BusyBee Web
  - CH-Bin
  - CoverM
  - MaxBin2
  - MetaBAT2
  - Minimap2
  - SAMTools
  - SolidBin
  - metaSPAdes
  - seq2vec
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.48
n_claims: 25
---

# multi-tool-binning-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a new metagenomic contig binner (e.g. CH-Bin) against state-of-the-art binners (MaxBin2, MetaBAT2, SolidBin, BusyBee Web) on the same assembled contigs and CoverM coverage files, scoring all outputs with AMBER across simulated (Sim50, Sim100, CAMI2 UG) and real (Sharon) datasets.

## When to use

Use when a binning tool's relative performance on large, complex high-dimensional datasets must be quantified against baselines using standard AMBER metrics (purity, completeness, F1) on shared inputs.

## When NOT to use

- The goal is to evaluate bin refinement tools (Binning_refiner, GraphBin, GraphBin2, METAMVGL) rather than primary contig binners on common inputs
- Only small simulated datasets (Sim5, Sim10, Sim20) are of interest, where the large-dataset scaling behaviour being benchmarked is not exercised
- Reference genomes are unavailable, since AMBER ground truth requires aligning contigs to known references

## Inputs

- metaSPAdes-assembled contigs (multi-FASTA) per dataset
- sample-wise read coverage TSV computed with CoverM
- Reference genome FASTA files for ground-truth construction
- Paired-end metagenomic read files (for CoverM mapping)

## Outputs

- Binning results per tool (bins with corresponding contigs)
- AMBER evaluation metrics: average purity (bp), average completeness (bp), F1 score (bp), accuracy (bp), percentage binned (bp), number of bins
- Aggregated comparison table (amber_evaluation_results.csv)

## How to apply

Assemble reads with metaSPAdes and compute sample-wise per-contig read coverage with CoverM to produce a multi-FASTA contig file plus a coverage TSV per dataset (Sim50, Sim100, CAMI2 UG, Sharon). Run the tool under test — CH-Bin with k = 5, 6 oligonucleotide setting, default ContigLengthFilterBp of 1000 bp and ScmSelectPercentile at the 75th percentile — alongside baselines: MaxBin2 and SolidBin ('naive' mode) with defaults, MetaBAT2 with -m 1500, and BusyBee Web. Build AMBER ground truth by aligning contigs to reference genomes with Minimap2 and SAMTools, retaining only contigs with the highest mapping quality. Evaluate every tool with AMBER, extracting average purity (bp), average completeness (bp), F1 score (bp), accuracy (bp), percentage binned (bp) and number of bins, then aggregate into a single comparison table per dataset and tool.

## Related tools

- **CH-Bin** (Tool under test; convex-hull-distance binner run with k = 5, 6, 1000 bp contig filter and 75th-percentile ScmSelectPercentile) — https://github.com/kdsuneraavinash/CH-Bin
- **CoverM** (Computes sample-wise read coverage of assembled contigs, a shared input for all binners) — https://github.com/wwood/CoverM
- **MaxBin2** (Baseline binner run with default parameters)
- **MetaBAT2** (Baseline binner run with -m 1500)
- **SolidBin** (Baseline binner run in 'naive' mode with default parameters) — https://github.com/sufforest/SolidBin
- **BusyBee Web** (Baseline binner included in the comparison)
- **AMBER** (Produces ground truth and evaluates purity, completeness, F1, accuracy and percentage binned)
- **Minimap2** (Aligns contigs to reference genomes to build AMBER ground truth)
- **SAMTools** (Processes alignments, retaining contigs with highest mapping quality)
- **seq2vec** (Generates oligonucleotide composition feature vectors for CH-Bin) — https://github.com/anuradhawick/seq2vec

## Examples

```
coverm contig -r assembly.fna -1 read1.fq -2 read2.fq --methods mean
```

## Evaluation signals

- CH-Bin (k = 5, 6) attains the highest F1 score among all tools on the large datasets, matching the reported Sim50 result of F1 = 96.91%
- AMBER reports all six metrics (purity bp, completeness bp, F1 bp, accuracy bp, % binned bp, number of bins) for every tool x dataset combination
- All binners receive identical inputs: the same metaSPAdes contigs and the same CoverM coverage TSV per dataset
- Ground truth is restricted to contigs with the highest Minimap2 mapping quality, so no low-confidence contigs contribute to AMBER scores
- Number of bins estimated by CH-Bin follows the 75th percentile of single-copy marker gene contig counts

## Limitations

- Traditional Euclidean and Manhattan distance metrics become unreliable in high dimensional space, motivating convex-hull distance instead
- The CH-Bin repository provides no examples and lacks a Zenodo/archival DOI; only a GitHub source link is available
- The repository CHANGELOG mentions qualitative effects only ('decreases binning slightly but drastically improves performance') without quantitative metrics

## Evidence

- [readme] CoverM role: [evidence span withheld — anchor status: external_doc]
