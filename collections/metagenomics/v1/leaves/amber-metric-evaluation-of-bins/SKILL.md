---
name: amber-metric-evaluation-of-bins
description: Use when binning outputs from competing binners (e.g., CH-Bin, MaxBin2,
  MetaBAT2, SolidBin) on datasets with known reference genomes (Sim50, Sim100, CAMI2
  UG, Sharon) must be compared using purity/completeness/F1 metrics.
when_to_use_negative:
- No reference genomes are available for the dataset, so no AMBER ground truth can
  be constructed
- Input is raw contig feature data (coverage TSV / composition vectors) rather than
  completed bin assignments from a binner
- Contigs shorter than the 1000 bp ContigLengthFilterBp threshold dominate the dataset,
  since AMBER evaluation in this workflow only covers the filtered long-contig set
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: AMBER
  role: Computes bp-based purity, completeness, F1, accuracy and percentage binned
    for each tool's bins against the ground truth
- name: Minimap2
  role: Aligns assembled contigs to reference genomes to build the AMBER ground truth
- name: SAMTools
  role: Filters Minimap2 alignments, retaining only contigs with the highest mapping
    quality for ground truth
- name: CH-Bin
  role: Primary binner whose binning output (k=5,6 setting, 75th percentile ScmSelectPercentile)
    is evaluated and compared
  repo: https://github.com/kdsuneraavinash/CH-Bin
- name: MaxBin2
  role: Baseline binner run with default parameters on the same inputs for benchmarking
- name: MetaBAT2
  role: Baseline binner run with -m 1500 on the same inputs for benchmarking
- name: SolidBin
  role: Baseline binner run in 'naive' mode with default parameters on the same inputs
    for benchmarking
  repo: https://github.com/sufforest/SolidBin
- name: CoverM
  role: Produces the sample-wise read coverage TSV inputs consumed by the binners
    whose outputs are evaluated
  repo: https://github.com/wwood/CoverM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.compbiolchem.2022.107734
    title: 'CH-Bin: A convex hull based approach for binning metagenomic contigs'
  source_content_hash: sha256:94d2997f0d534019fb72dcb2d15b9a4d184f914278928d2aa06a28c3e9bd7379
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
  - CH-Bin
  - CoverM
  - MaxBin2
  - MetaBAT2
  - Minimap2
  - SAMTools
  - SolidBin
  - metaSPAdes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# amber-metric-evaluation-of-bins

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantitatively evaluate metagenomic binning outputs with AMBER by aligning contigs to reference genomes, building a ground-truth mapping, and extracting base-pair-based purity, completeness, F1, accuracy, percentage binned, and bin counts per tool per dataset.

## When to use

Use when binning outputs from competing binners (e.g., CH-Bin, MaxBin2, MetaBAT2, SolidBin) on datasets with known reference genomes (Sim50, Sim100, CAMI2 UG, Sharon) must be compared using purity/completeness/F1 metrics.

## When NOT to use

- No reference genomes are available for the dataset, so no AMBER ground truth can be constructed
- Input is raw contig feature data (coverage TSV / composition vectors) rather than completed bin assignments from a binner
- Contigs shorter than the 1000 bp ContigLengthFilterBp threshold dominate the dataset, since AMBER evaluation in this workflow only covers the filtered long-contig set

## Inputs

- Binner output bin files (contig-to-bin assignments) for each evaluated tool and dataset
- metaSPAdes-assembled contigs in multi-FASTA format
- Reference genomes for the dataset (simMC+/simHC+ derived Sim50/Sim100, CAMI2 UG, Sharon infant gut metagenome)
- Contig-to-reference alignments produced by Minimap2 for ground-truth construction

## Outputs

- AMBER ground-truth files per dataset built from highest-mapping-quality alignments
- Per-tool AMBER metrics: average purity (bp), average completeness (bp), F1 score (bp), accuracy (bp), percentage binned (bp), number of bins identified
- Aggregated comparison table amber_evaluation_results.csv

## How to apply

Build the AMBER ground truth for each dataset by aligning assembled contigs (multi-FASTA, e.g., metaSPAdes output) to their reference genomes with Minimap2 and filtering alignments with SAMTools, retaining only contigs with the highest mapping quality. Collect each binner's output bins (CH-Bin with k=5,6 and default ContigLengthFilterBp of 1000 bp; MaxBin2 and SolidBin 'naive' mode with defaults; MetaBAT2 with -m 1500) on identical inputs. Run AMBER per tool per dataset to extract average purity (bp), average completeness (bp), F1 score (bp), accuracy (bp), percentage binned (bp) and number of bins identified. Aggregate all metric values per dataset and tool into a single comparison table written to amber_evaluation_results.csv. Rationale: bp-weighted metrics reward correctly assigning contig bases rather than contig counts, matching the paper's reporting style.

## Related tools

- **AMBER** (Computes bp-based purity, completeness, F1, accuracy and percentage binned for each tool's bins against the ground truth)
- **Minimap2** (Aligns assembled contigs to reference genomes to build the AMBER ground truth)
- **SAMTools** (Filters Minimap2 alignments, retaining only contigs with the highest mapping quality for ground truth)
- **CH-Bin** (Primary binner whose binning output (k=5,6 setting, 75th percentile ScmSelectPercentile) is evaluated and compared) — https://github.com/kdsuneraavinash/CH-Bin
- **MaxBin2** (Baseline binner run with default parameters on the same inputs for benchmarking)
- **MetaBAT2** (Baseline binner run with -m 1500 on the same inputs for benchmarking)
- **SolidBin** (Baseline binner run in 'naive' mode with default parameters on the same inputs for benchmarking) — https://github.com/sufforest/SolidBin
- **CoverM** (Produces the sample-wise read coverage TSV inputs consumed by the binners whose outputs are evaluated) — https://github.com/wwood/CoverM

## Evaluation signals

- amber_evaluation_results.csv contains one row per dataset x tool with all six metrics: purity (bp), completeness (bp), F1 (bp), accuracy (bp), percentage binned (bp), and number of bins
- On Sim50, CH-Bin with k = 5, 6 attains an F1 score of 96.91%, the best among the evaluated tools
- On the large and complex datasets (Sim50, Sim100, CAMI2 UG), CH-Bin generally attains the highest purity, completeness and F1 among all evaluated tools
- AMBER ground truth contains only contigs with the highest mapping quality alignments (Minimap2 + SAMTools filtering)
- Metric values are base-pair weighted, consistent with the (bp) reporting convention

## Limitations

- Ground truth requires reference genomes, so the evaluation is only valid for simulated datasets (Sim5-Sim100, CAMI2 UG) or datasets with known species composition
- Contigs below the 1000 bp ContigLengthFilterBp threshold are excluded, so metrics do not reflect binning of short contigs
- Retaining only highest-mapping-quality alignments discards ambiguously mapped contigs from the ground truth, which can bias metric estimates
- The number of bins reported depends on CH-Bin's ScmSelectPercentile (75th percentile) choice, affecting completeness/F1 comparisons against tools using different bin-count estimation

## Evidence

- [results] Sim50 best F1 validation: "For the Sim50, CH-Bin has shown the best performance with the setting k = 5, 6 demonstrating an F1 score of 96.91 %"
- [methods] Benchmark datasets: "Total of six simulated datasets named Sim5, Sim10, Sim20, Sim50, Sim100 and CAMI2 UG were used for testing and evaluation of CH-Bin."
- [methods] Contig length filter: "ContigLengthFilterBp | Threshold to filter the short contigs."
