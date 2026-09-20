---
name: short-read-simulation-from-reference-genomes
description: Use when reads must be simulated from genomes whose true species are
  absent from every classifier database, in order to measure false-positive misclassification
  of taxonomic classifiers as a function of similarity to the closest database relative.
when_to_use_negative:
- The true source species IS present in one or more classifier databases — assignments
  are then not unambiguous false positives and this absent-species design is invalid
- The goal is read-level precision/recall on known taxa rather than false positives;
  use the 3,269 MAG benchmark reads excluded from training instead
- Reads have strongly different lengths or error profiles (e.g. long reads); the paper
  notes retraining might be required to adapt to such data
tools:
- name: DeepMicrobes
  role: Deep-learning species classifier under benchmark; its false positives on absent-species
    reads are compared against baselines
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: Kraken
  role: k-mer/LCA baseline classifier scored for misclassified reads and misidentified
    species
- name: Kraken 2
  role: k-mer baseline classifier scored alongside Kraken, CLARK and CLARK-S
- name: CLARK
  role: Alignment/k-mer baseline classifier in the five-way false-positive comparison
- name: CLARK-S
  role: Spaced-variant baseline classifier in the five-way false-positive comparison
- name: DeepMicrobes-data
  role: Provides benchmark sequences ('Reads simulated from species absent from reference
    databases'), command lines used to run all benchmarked tools, and R scripts for
    figures
  repo: https://github.com/MicrobeLab/DeepMicrobes-data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:15281e52a4083bfcb063533f30ca04749be496b9bb73823fcbcbf001ebc32eb8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/MicrobeLab/DeepMicrobes
  repo_url: https://github.com/MicrobeLab/DeepMicrobes
  tools:
  - CLARK
  - CLARK-S
  - DeepMicrobes
  - DeepMicrobes-data
  - Kraken
  - Kraken 2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# short-read-simulation-from-reference-genomes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate short sequencing reads at 1x coverage from genomes whose true species are absent from all reference databases (AQ<60%, ANI<95% to closest relative), then benchmark false-positive species misclassification across DeepMicrobes, Kraken, Kraken 2, CLARK and CLARK-S.

## When to use

Use when reads must be simulated from genomes whose true species are absent from every classifier database, in order to measure false-positive misclassification of taxonomic classifiers as a function of similarity to the closest database relative.

## When NOT to use

- The true source species IS present in one or more classifier databases — assignments are then not unambiguous false positives and this absent-species design is invalid
- The goal is read-level precision/recall on known taxa rather than false positives; use the 3,269 MAG benchmark reads excluded from training instead
- Reads have strongly different lengths or error profiles (e.g. long reads); the paper notes retraining might be required to adapt to such data

## Inputs

- 121 genome assemblies from PRJNA348753 whose closest database relatives satisfy AQ<60% and ANI<95% (FASTA)
- Standard classifier databases for DeepMicrobes, Kraken, Kraken 2, CLARK and CLARK-S
- Short-read simulator and NCBI taxonomy for assigning closest relatives and misidentified species names

## Outputs

- Simulated short reads at 1x coverage per absent-species genome (FASTQ)
- Per-tool counts of misclassified reads and misidentified species, stratified by similarity to closest database relative
- figure4_misclassification_results.csv
- Grouped comparison plot reproducing the Figure 4 layout (cf. Table S6)

## How to apply

From the PRJNA348753 genome set, select the 121 genomes whose closest database relatives fall below the novelty cutoffs (AQ<60% and ANI<95%) so that no tool's database contains the true source species. Simulate short sequencing reads at 1x coverage per genome with a short-read simulator, then classify each read set with DeepMicrobes and the k-mer/alignment-based baselines Kraken, Kraken 2, CLARK and CLARK-S, each using its standard database. Because the true source species is absent from all databases, any species-level assignment is by definition a false positive; tally per tool the number of misclassified reads and the number of misidentified species, stratified by similarity of each genome to its closest database relative (e.g. query aligned 10-60%, ANI<95% as in Table S6). Tabulate the comparison into figure4_misclassification_results.csv and render a grouped plot reproducing the Figure 4 layout.

## Related tools

- **DeepMicrobes** (Deep-learning species classifier under benchmark; its false positives on absent-species reads are compared against baselines) — https://github.com/MicrobeLab/DeepMicrobes
- **Kraken** (k-mer/LCA baseline classifier scored for misclassified reads and misidentified species)
- **Kraken 2** (k-mer baseline classifier scored alongside Kraken, CLARK and CLARK-S)
- **CLARK** (Alignment/k-mer baseline classifier in the five-way false-positive comparison)
- **CLARK-S** (Spaced-variant baseline classifier in the five-way false-positive comparison)
- **DeepMicrobes-data** (Provides benchmark sequences ('Reads simulated from species absent from reference databases'), command lines used to run all benchmarked tools, and R scripts for figures) — https://github.com/MicrobeLab/DeepMicrobes-data

## Evaluation signals

- Tabulated results show DeepMicrobes producing far fewer misclassified reads and misidentified species than Kraken, Kraken 2, CLARK and CLARK-S, consistent with the published Figure 4 finding
- The DeepMicrobes advantage holds across all similarity strata of each genome to its closest database relative (independent of AQ/ANI bin)
- Every read is simulated from a species absent from all databases (AQ<60%, ANI<95%), so any species-level assignment must be counted as a false positive
- Coverage is 1x per genome across all 121 genomes, matching Table S6 (query aligned 10-60%, ANI<95%)
- Output CSV (figure4_misclassification_results.csv) covers exactly the five classifiers with both misclassified-read and misidentified-species columns

## Limitations

- Results depend on the 12-mer vocabulary of the pre-trained DeepMicrobes models; the k-mer length was optimized for thousands of genomes and other scenarios should try 6-12 bp
- Adding new species to DeepMicrobes requires retraining the entire deep neural network, so this benchmark only evaluates the fixed trained species model
- Simulation-based false positives may not reflect real sequencing error profiles; findings are specific to short reads

## Evidence

- [readme] Reads simulated from species absent from reference databases: [evidence span withheld — anchor status: not_found]
- [readme] Table S6 false positives from 1x coverage simulation of 121 absent genomes: [evidence span withheld — anchor status: external_doc]
- [other] Genome selection cutoffs AQ<60% and ANI<95% from PRJNA348753: [evidence span withheld — anchor status: not_found]
- [other] Finding: DeepMicrobes far fewer false positives than the four baselines: [evidence span withheld — anchor status: not_found]
