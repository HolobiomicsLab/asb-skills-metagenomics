---
name: misclassification-count-quantification
description: Use when benchmarking taxonomic classifiers for false-positive behavior
  on reads whose true source species is missing from every tool's reference database
  (e.g. AQ<60%, ANI<95% novelty cutoffs).
when_to_use_negative:
- The true source species IS present in the tools' reference databases — use a precision/recall
  benchmark on training-excluded genomes (e.g. the 3,269 MAGs) instead
- The goal is genus-level or abundance-estimation comparison rather than species-level
  false-positive counting
- Input data are real metagenomes (e.g. the IBD/iHMP dataset) where ground-truth species
  labels per read are unavailable
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: DeepMicrobes
  role: Deep learning species classifier under evaluation; expected to produce far
    fewer false positives on absent-species reads
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: Kraken
  role: k-mer/LCA baseline classifier whose false positives are tallied for comparison
- name: Kraken 2
  role: k-mer baseline classifier whose false positives are tallied for comparison
- name: CLARK
  role: k-mer/alignment baseline classifier whose false positives are tallied for
    comparison
- name: CLARK-S
  role: sensitive k-mer/alignment baseline classifier whose false positives are tallied
    for comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:2b0698d21d0a9ea18707dbe86722ba21a97090312c1fc65d4de9a8a8c3ce5d3a
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# misclassification-count-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies false positives of taxonomic classifiers on reads simulated from species absent from all reference databases, tallying misclassified reads and misidentified species per tool. It isolates database-independence of error rates, as in the DeepMicrobes Figure 4 benchmark.

## When to use

Use when benchmarking taxonomic classifiers for false-positive behavior on reads whose true source species is missing from every tool's reference database (e.g. AQ<60%, ANI<95% novelty cutoffs).

## When NOT to use

- The true source species IS present in the tools' reference databases — use a precision/recall benchmark on training-excluded genomes (e.g. the 3,269 MAGs) instead
- The goal is genus-level or abundance-estimation comparison rather than species-level false-positive counting
- Input data are real metagenomes (e.g. the IBD/iHMP dataset) where ground-truth species labels per read are unavailable

## Inputs

- 121 genome assemblies whose species are absent from all reference databases (e.g. from PRJNA348753), with AQ/ANI values versus closest database relatives
- Simulated short reads at 1x coverage per genome
- Standard reference databases for each classifier (DeepMicrobes, Kraken, Kraken 2, CLARK, CLARK-S)

## Outputs

- Per-tool counts of misclassified reads on absent-species data
- Per-tool counts of misidentified species
- Comparison table (e.g. figure4_misclassification_results.csv; cf. supplementary Table S6)
- Grouped comparison plot reproducing the Figure 4 layout

## How to apply

From a genome set such as PRJNA348753, select genomes whose closest database relatives fall below novelty cutoffs (AQ<60% and ANI<95%) so that no tool's database contains the true source species. Simulate short sequencing reads at 1x coverage per genome with a short-read simulator, then classify the reads with each classifier (DeepMicrobes, Kraken, Kraken 2, CLARK, CLARK-S) using its standard database. For each tool tally two error counts: the number of misclassified reads and the number of misidentified species. Tabulate the five-classifier comparison (e.g. into figure4_misclassification_results.csv) and render a grouped comparison plot reproducing the Figure 4 layout; the expected result is that DeepMicrobes produces far fewer false positives regardless of each genome's similarity to its closest database relative.

## Related tools

- **DeepMicrobes** (Deep learning species classifier under evaluation; expected to produce far fewer false positives on absent-species reads) — https://github.com/MicrobeLab/DeepMicrobes
- **Kraken** (k-mer/LCA baseline classifier whose false positives are tallied for comparison)
- **Kraken 2** (k-mer baseline classifier whose false positives are tallied for comparison)
- **CLARK** (k-mer/alignment baseline classifier whose false positives are tallied for comparison)
- **CLARK-S** (sensitive k-mer/alignment baseline classifier whose false positives are tallied for comparison)

## Evaluation signals

- Every genome selected satisfies both novelty cutoffs: AQ<60% and ANI<95% versus its closest database relative, so no tool's database contains the true source species
- Each genome contributes reads at exactly 1x coverage from the short-read simulator
- The output table contains exactly two error metrics (misclassified reads, misidentified species) for each of the five classifiers
- DeepMicrobes shows far fewer misclassified reads and misidentified species than Kraken, Kraken 2, CLARK and CLARK-S, consistent with published Figure 4 / Table S6
- The false-positive pattern holds across strata of similarity to the closest database relative (e.g. query aligned 10-60%)

## Limitations

- Adding new species to DeepMicrobes requires retraining the entire deep neural network, so absent-species behavior reflects the trained repertoire
- k-mer length (12-mers in pre-trained models) was optimized for typical data volume of thousands of genomes; other scenarios may require k-mer lengths of 6-12 bp
- Results depend on the simulated reads' length and error profile; retraining might be required for reads with strongly different characteristics (e.g. long reads)

## Evidence

- [results] the species model of DeepMicrobes produced far fewer false positives (misclassified reads and misidentified species) than Kraken, Kraken 2, CLARK and CLARK-S: [evidence span withheld — anchor status: not_found]
- [methods] select the 121 genomes whose closest database relatives fall below the novelty cutoffs (AQ<60% and ANI<95%): [evidence span withheld — anchor status: not_found]
- [supplementary] Table S6 False positive species classification of different tools on data sets generated by simulating 1x coverage reads using 121 genomes: [evidence span withheld — anchor status: external_doc]
- [readme] Reads simulated from species absent from reference databases: [evidence span withheld — anchor status: external_doc]
