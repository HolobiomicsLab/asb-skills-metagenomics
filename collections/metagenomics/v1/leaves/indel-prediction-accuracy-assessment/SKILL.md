---
name: indel-prediction-accuracy-assessment
description: Use when benchmarking gene-prediction tools on raw error-prone metagenomic
  reads (e.g. 454 homopolymer indels) and you need nucleotide-level sensitivity/precision,
  including indel-position accuracy, against known truth.
when_to_use_negative:
- Reads are error-free or already error-corrected/assembled — indel modeling is only
  exercised on error-prone sequences
- The task is organism classification or binning rather than gene-prediction accuracy
  benchmarking
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0080
tools:
- name: Glimmer-MG
  role: Gene finder under test; predicts insertions, deletions, and stop-codon-introducing
    substitutions to track coding frames in error-prone reads
  repo: https://github.com/davek44/Glimmer-MG
- name: FragGeneScan
  role: Comparator gene finder for indel detection accuracy on the same read sets
- name: MetaGeneMark
  role: Comparator gene finder run under the same error conditions
- name: Phymm
  role: Phylogenetic classifier that parameterizes Glimmer-MG prediction models before
    benchmarking
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:83710fe6c8855ec40604bd10b8fd0a134d0e19709b37bcbe8c698d1e5bd4b55f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/davek44/Glimmer-MG
  repo_url: https://github.com/davek44/Glimmer-MG
  tools:
  - FragGeneScan
  - Glimmer-MG
  - MetaGeneMark
  - Phymm
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# indel-prediction-accuracy-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the accuracy with which metagenomic gene finders (Glimmer-MG, FragGeneScan, MetaGeneMark) predict insertion/deletion sequencing errors at the nucleotide level on simulated error-prone reads, using nucleotide-level sensitivity and precision against ground-truth gene coordinates.

## When to use

Use when benchmarking gene-prediction tools on raw error-prone metagenomic reads (e.g. 454 homopolymer indels) and you need nucleotide-level sensitivity/precision, including indel-position accuracy, against known truth.

## When NOT to use

- Reads are error-free or already error-corrected/assembled — indel modeling is only exercised on error-prone sequences
- The task is organism classification or binning rather than gene-prediction accuracy benchmarking

## Inputs

- Simulated metagenomic read sets (454, Sanger, Illumina) at error rates 0%-2%, with 454 libraries containing homopolymer-run indel errors
- Ground-truth gene coordinate annotations on each simulated read set
- Reference genomes used by the read simulator

## Outputs

- Nucleotide-level sensitivity and precision per tool per read-type/error-rate combination
- Indel-prediction sensitivity and precision (e.g. 23.2% and 63.8% at 1.0% error on 454 reads)
- Table 2 replica grid, e.g. table2_sensitivity_precision.csv with one row per tool/read-type/error-rate combination

## How to apply

Simulate read sets (454, Sanger, Illumina) from reference genomes with error rates spanning 0% to 2%, ensuring 454 simulations include homopolymer-run indel errors, and annotate true gene coordinates on each read as ground truth. Run Glimmer-MG (phylogenetic-classification-parameterized and cluster-retrained modes, per manual.pdf) plus FragGeneScan and MetaGeneMark on identical read sets under the same error conditions. Compute nucleotide-level sensitivity and precision per tool/read-type/error-rate combination; Glimmer-MG can branch into a different coding frame at low-quality base calls to track indels, so verify indel-prediction sensitivity and precision explicitly — the paper's reference point is 23.2% sensitivity and 63.8% precision at a 1.0% error rate on 454 reads. Assemble the metrics grid into a Table 2 replica (e.g. table2_sensitivity_precision.csv) with one row per tool/read-type/error-rate combination.

## Related tools

- **Glimmer-MG** (Gene finder under test; predicts insertions, deletions, and stop-codon-introducing substitutions to track coding frames in error-prone reads) — https://github.com/davek44/Glimmer-MG
- **FragGeneScan** (Comparator gene finder for indel detection accuracy on the same read sets)
- **MetaGeneMark** (Comparator gene finder run under the same error conditions)
- **Phymm** (Phylogenetic classifier that parameterizes Glimmer-MG prediction models before benchmarking)

## Examples

```
glimmer-mg --manual manual.pdf run phylogenetic-and-cluster-modes on 454_sim_1pct.fa; then score indel sensitivity/precision against true gene coordinates and write table2_sensitivity_precision.csv
```

## Evaluation signals

- Output CSV exists and contains one row per tool/read-type/error-rate combination
- 454 1.0%-error indel sensitivity and precision match the reported 23.2% and 63.8% within tolerance
- Sensitivity/precision degrade monotonically (or near-monotonically) as error rate rises from 0% to 2%
- Glimmer-MG sensitivity/precision exceed FragGeneScan across read lengths and error rates (e.g. ~5.8%/6.0% relative gains at 454 1.0% error)
- Precision at 2.0% error on 454 reads stays near the reported 54.7% (sensitivity 73.2%) rather than implausibly high values

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem — precision drops sharply at high error rates (54.7% at 2.0% error on 454 reads)
- Glimmer-MG is more computationally expensive than existing software
- Highly diverged organisms where optimal model parameters are unclear present a challenge
- No software version stated for Glimmer-MG or compared tools

## Evidence

- [readme] Glimmer-MG can predict insertions and deletions in the sequence by branching into a different frame at low quality base calls such as homopolymer runs in 454 sequences.: [evidence span withheld — anchor status: external_doc]
