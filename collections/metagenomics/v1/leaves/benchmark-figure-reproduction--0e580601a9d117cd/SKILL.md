---
name: benchmark-figure-reproduction
description: Use when reproducing or re-evaluating a published classifier benchmark
  figure — specifically false-positive behavior on reads simulated from species absent
  from reference databases — using the DeepMicrobes Figure 4 design.
when_to_use_negative:
- The source species ARE present in every classifier's database — the benchmark's
  premise (novelty cutoffs AQ<60%, ANI<95%) is violated, so the false-positive comparison
  is meaningless
- The goal is to reproduce the mock-community abundance profiles or the IBD species
  profiles rather than the absent-species false-positive Figure 4 benchmark
- Evaluating genus-level or abundance-estimation performance, which this figure-level
  skill does not cover (it tallies species-level misclassification only)
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3473
tools:
- name: DeepMicrobes
  role: Deep learning (Embed + LSTM + Attention) species classifier under test on
    absent-species reads
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: Kraken
  role: k-mer LCA baseline classifier benchmarked for false positives
- name: Kraken 2
  role: k-mer LCA baseline classifier benchmarked for false positives
- name: CLARK
  role: k-mer/alignment baseline classifier benchmarked for false positives
- name: CLARK-S
  role: sensitive k-mer/alignment baseline classifier benchmarked for false positives
- name: Mash
  role: Genome-distance screening (with mashdiff.sh) to identify genomes below the
    ANI/alignment novelty cutoffs
- name: R
  role: Scripts to render the Figure 4 grouped comparison plot from the results CSV
  repo: https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:ab4fd515948c7166b130371b94befa8f0578d6acc1358d1e9505f6eaaab97529
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
  - Kraken
  - Kraken 2
  - Mash
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# benchmark-figure-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the DeepMicrobes Figure 4 absent-species false-positive benchmark: reads simulated from 121 genomes whose species are absent from all reference databases are classified by DeepMicrobes, Kraken, Kraken 2, CLARK and CLARK-S, and misclassified reads/misidentified species are tabulated and plotted into figure4_misclassification_results.csv.

## When to use

Use when reproducing or re-evaluating a published classifier benchmark figure — specifically false-positive behavior on reads simulated from species absent from reference databases — using the DeepMicrobes Figure 4 design.

## When NOT to use

- The source species ARE present in every classifier's database — the benchmark's premise (novelty cutoffs AQ<60%, ANI<95%) is violated, so the false-positive comparison is meaningless
- The goal is to reproduce the mock-community abundance profiles or the IBD species profiles rather than the absent-species false-positive Figure 4 benchmark
- Evaluating genus-level or abundance-estimation performance, which this figure-level skill does not cover (it tallies species-level misclassification only)

## Inputs

- 121 genome FASTA files from the PRJNA375587/PRJNA348753 gut MAG set whose closest relatives fall below AQ<60% and ANI<95%
- Standard reference databases for Kraken, Kraken 2, CLARK and CLARK-S
- Pre-trained DeepMicrobes species model and 12-mer vocabulary from github.com/MicrobeLab/DeepMicrobes
- Mash reference sketch (.msh) and reference genome database for computing ANI/alignment fraction with mashdiff.sh

## Outputs

- figure4_misclassification_results.csv with per-tool counts of misclassified reads and misidentified species
- Grouped comparison plot reproducing the Figure 4 layout
- Per-genome Mash/dnadiff metrics (ANI, % query aligned) used to justify the 121-genome selection

## How to apply

From the PRJNA348753 genome set, select the 121 genomes whose closest database relatives fall below the novelty cutoffs (query aligned fraction 10-60%, i.e. AQ<60%, and ANI<95%) so that no tool's database contains the true source species. Simulate short sequencing reads at 1x coverage per genome with a short-read simulator, then classify them with DeepMicrobes and the k-mer/alignment baselines Kraken, Kraken 2, CLARK and CLARK-S, each with its standard database (command lines for all benchmarked tools are provided in the DeepMicrobes-data scripts). Tally per tool the number of misclassified reads and the number of misidentified species, following the design of Table S6. Tabulate the five-tool comparison into figure4_misclassification_results.csv and render a grouped plot reproducing the Figure 4 layout with the provided R scripts. Acceptance requires DeepMicrobes to show far fewer false positives than the four baselines regardless of similarity to the closest database relative.

## Related tools

- **DeepMicrobes** (Deep learning (Embed + LSTM + Attention) species classifier under test on absent-species reads) — https://github.com/MicrobeLab/DeepMicrobes
- **Kraken** (k-mer LCA baseline classifier benchmarked for false positives)
- **Kraken 2** (k-mer LCA baseline classifier benchmarked for false positives)
- **CLARK** (k-mer/alignment baseline classifier benchmarked for false positives)
- **CLARK-S** (sensitive k-mer/alignment baseline classifier benchmarked for false positives)
- **Mash** (Genome-distance screening (with mashdiff.sh) to identify genomes below the ANI/alignment novelty cutoffs)
- **R** (Scripts to render the Figure 4 grouped comparison plot from the results CSV) — https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts

## Examples

```
mashdiff.sh -i genome_folder/ -r reference.msh -s refseq -p gut
```

## Evaluation signals

- The 121 selected genomes all fall below the novelty cutoffs (AQ<60% and ANI<95%) relative to every reference database, matching the Table S6 condition (query aligned = 10-60%, ANI < 95%)
- Reads are simulated at exactly 1x coverage per genome; read counts scale with genome length
- All five classifiers (DeepMicrobes, Kraken, Kraken 2, CLARK, CLARK-S) run on the identical simulated read set with their standard databases
- Tabulated results show DeepMicrobes producing far fewer misclassified reads and misidentified species than the four baselines, consistent with the published Figure 4 finding
- Output CSV matches the figure4_misclassification_results.csv schema with per-tool misclassification counts

## Limitations

- Adding new species to DeepMicrobes requires retraining the entire deep neural network, so new absent species cannot simply be added to its database like the k-mer/alignment baselines
- The k-mer length (12-mers in the pre-trained models) was optimized for thousands of genomes; other scenarios require trying different k-mer lengths (6-12 bp)
- Reads are short simulated reads at 1x coverage; retraining might be required to adapt to reads with strongly different lengths and error profiles (e.g. long-read sequencing)
- Table S6 restricts the selection to genomes with query aligned = 10-60% and ANI < 95%, so genomes outside this similarity band are not covered by this benchmark

## Evidence

- [readme] False positive species classification of different tools on data sets generated by simulating 1x coverage reads using 121 genomes from species not included in RefSeq database: [evidence span withheld — anchor status: external_doc]
- [readme] Reads simulated from species absent from reference databases: [evidence span withheld — anchor status: external_doc]
- [readme] Command lines used to run all the taxonomic classification tools benchmarked in the paper: [evidence span withheld — anchor status: external_doc]
- [supplementary] the R scripts used to generate figures for benchmarking are available at https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts: "the R scripts used to generate figures for benchmarking are available at https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts"
- [readme] Query name / Reference name / Ref length / % Ref covered / Query length / % Query aligned / ANI / Mash distance: [evidence span withheld — anchor status: not_found]
