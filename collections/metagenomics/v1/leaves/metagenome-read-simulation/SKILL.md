---
name: metagenome-read-simulation
description: Use when benchmarking metagenomic gene prediction or sequence-analysis
  tools where nucleotide-level ground-truth gene coordinates are needed, i.e. reads
  must be simulated from annotated genomes at known error rates.
when_to_use_negative:
- The input is real sequencing data (e.g. real 454 human gut reads) with no simulated
  ground truth — simulation-based scoring is not applicable
- You need to evaluate only assembly or binning, not gene prediction accuracy against
  known gene coordinates
- Your target sequencing platform or error rate falls outside the tested space (Sanger/454/Illumina
  at 0–2% error), so results will not be comparable to the reported grid
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: Glimmer-MG
  role: Gene prediction system whose accuracy on the simulated read sets is measured;
    consumes error-prone reads and predicts indels to track coding frames
  repo: https://github.com/davek44/Glimmer-MG
- name: FragGeneScan
  role: Comparator gene predictor run on the identical simulated read sets under the
    same error conditions
- name: MetaGeneMark
  role: Comparator gene predictor evaluated at each read-type/error-rate combination
- name: MetaGeneAnnotator
  role: Comparator gene predictor run on the same error-free read set in the Table
    1 experiment
- name: FlowSim
  role: Read simulator generating simulated 454 metagenome reads from reference GenBank
    genomes with a Pareto abundance distribution and the 454 homopolymer-run error
    model
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:38a806de4adf5a72212de0297d7c1a172ca0b18b3196e8ba2f8198ae8aa80e6b
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
  - 454 Life Sciences
  - FlowSim
  - FragGeneScan
  - Glimmer-MG
  - MetaGeneAnnotator
  - MetaGeneMark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metagenome-read-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate metagenomic shotgun read sets from reference GenBank genomes using FlowSim under Sanger, 454, and Illumina read technologies, with controlled error rates from 0% to 2% including 454 homopolymer-run indel errors, to create ground-truth benchmarks for gene predictor sensitivity and precision.

## When to use

Use when benchmarking metagenomic gene prediction or sequence-analysis tools where nucleotide-level ground-truth gene coordinates are needed, i.e. reads must be simulated from annotated genomes at known error rates.

## When NOT to use

- The input is real sequencing data (e.g. real 454 human gut reads) with no simulated ground truth — simulation-based scoring is not applicable
- You need to evaluate only assembly or binning, not gene prediction accuracy against known gene coordinates
- Your target sequencing platform or error rate falls outside the tested space (Sanger/454/Illumina at 0–2% error), so results will not be comparable to the reported grid

## Inputs

- Reference GenBank genomes with annotated gene coordinates
- Chosen sequencing technology profile (Sanger, 454, Illumina) and target read lengths
- Error-rate grid (0% to 2%)
- Abundance distribution parameters (Pareto) for the simulated community

## Outputs

- Simulated read sets per read technology and error rate (e.g. FASTA/FASTQ metagenome libraries)
- Ground-truth annotated gene coordinates on each simulated read set
- Metrics table (e.g. table1_sensitivity_precision.csv, table2_sensitivity_precision.csv) with nucleotide-level sensitivity/precision per tool/read-type/error-rate combination

## How to apply

Simulate read libraries from reference GenBank genomes with a read simulator (FlowSim), covering Sanger, 454, and Illumina read technologies and error rates spanning 0% to 2%, with 454 simulations applying the error model in which low-quality base calls arise from homopolymer runs and indel errors are included. Draw abundances from a Pareto distribution so the simulated community resembles a real metagenome. Annotate the true gene coordinates on each simulated read set to serve as the ground-truth reference for scoring, since accuracy is then computed as nucleotide-level sensitivity and precision per tool, read type, and error rate. For tools such as Glimmer-MG, which predict insertion, deletion and stop codon-introducing substitutions to track coding frames, also record indel-prediction sensitivity and precision (reported as 23.2% and 63.8% at a 1.0% error rate on 454 reads). Rationale: error-free reads (Table 1) isolate accuracy on fragmented genes, while error-prone simulations (Table 2) test frame-shift tolerance, so simulations should span both regimes to replicate the paper's evaluation grid.

## Related tools

- **Glimmer-MG** (Gene prediction system whose accuracy on the simulated read sets is measured; consumes error-prone reads and predicts indels to track coding frames) — https://github.com/davek44/Glimmer-MG
- **FragGeneScan** (Comparator gene predictor run on the identical simulated read sets under the same error conditions)
- **MetaGeneMark** (Comparator gene predictor evaluated at each read-type/error-rate combination)
- **MetaGeneAnnotator** (Comparator gene predictor run on the same error-free read set in the Table 1 experiment)
- **FlowSim** (Read simulator generating simulated 454 metagenome reads from reference GenBank genomes with a Pareto abundance distribution and the 454 homopolymer-run error model)

## Evaluation signals

- Output CSV (e.g. table2_sensitivity_precision.csv) exists and contains exactly one row per tool/read-type/error-rate combination
- All sensitivity and precision values fall in [0, 100]
- On 454 reads at 1.0% error, indel-prediction sensitivity and precision match the reported 23.2% and 63.8% within tolerance
- Glimmer-MG on simulated 454 reads at 2.0% error reproduces 73.2% sensitivity and 54.7% precision
- Error-free simulation (0% error) reproduces the Table 1 pattern in which Glimmer-MG has the greatest sensitivity for every read length

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem (precision only 54.7% at 2.0% error on 454 reads)
- Glimmer-MG applied downstream of simulation is more computationally expensive than existing software
- Highly diverged organisms where the optimal model parameters are unclear present a challenge to metagenomics gene prediction
- No software version is stated for the simulators or compared gene finders, complicating exact reproduction

## Evidence

- [readme] Glimmer-MG can predict insertions and deletions in the sequence by branching into a different frame at low quality base calls such as homopolymer runs in 454 sequences.: [evidence span withheld — anchor status: external_doc]
