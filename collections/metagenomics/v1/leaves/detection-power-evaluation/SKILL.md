---
name: detection-power-evaluation
description: Use when estimating the minimum detectable minor-population frequency
  for a genotype-based resistance or species caller on Illumina data, e.g. benchmarking
  mecA or plasmid-borne tetracycline/erythromycin detection in in silico S. aureus/CoNS
  mixtures.
when_to_use_negative:
- The caller being evaluated has no JSON output mode and cannot be run in --format
  json, so per-sample structured parsing is impossible
- The question is per-isolate resistance prediction accuracy (sensitivity/specificity
  against a phenotype gold standard, e.g. the n=470 S. aureus validation set) rather
  than minor-population detection power in mixtures
- Input is a single isolate rather than pooled/simulated mixtures — detection power
  as a function of minor frequency is undefined
tools:
- name: Mykrobe predictor
  role: Run on every simulated mixture (with --format json) to call species (phylo_group,
    species, lineage) and resistance determinants (mecA, tetracycline, erythromycin
    loci)
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe Parser
  role: R script to parse the Mykrobe JSON files, separating mutations into genetic
    regions and extracting speciation data (phylo_group, species, lineage) into tidy
    CSV outputs
  repo: https://github.com/phac-nml/mykrobe-parser
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:f19dc25589a9cf8a6907612abe2d1a09afb7f458ccc41e95cc1e990b763df69d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe Parser
  - Mykrobe predictor
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# detection-power-evaluation

## Summary

Quantifies how reliably Mykrobe predictor detects low-frequency resistant populations in simulated mixed infections, by pooling FASTQ reads from St_B1 S. aureus and St_B2 CoNS samples across a controlled frequency gradient and computing the fraction of correct calls per frequency.

## When to use

Use when estimating the minimum detectable minor-population frequency for a genotype-based resistance or species caller on Illumina data, e.g. benchmarking mecA or plasmid-borne tetracycline/erythromycin detection in in silico S. aureus/CoNS mixtures.

## When NOT to use

- The caller being evaluated has no JSON output mode and cannot be run in --format json, so per-sample structured parsing is impossible
- The question is per-isolate resistance prediction accuracy (sensitivity/specificity against a phenotype gold standard, e.g. the n=470 S. aureus validation set) rather than minor-population detection power in mixtures
- Input is a single isolate rather than pooled/simulated mixtures — detection power as a function of minor frequency is undefined

## Inputs

- FASTQ (Illumina) reads for 450 St_B1 S. aureus samples with >=100x mean sequencing depth of coverage
- FASTQ reads for 18 St_B2 CoNS samples (9 S. epidermidis and 9 S. haemolyticus)
- Mixture frequency design points: 30 frequencies from 0.5% to 20% (single-pair) and ratios 1:99 to 99:1 (pairwise)

## Outputs

- Mykrobe predictor JSON output files (one per simulated mixture)
- Extracted species calls (phylo_group, species, lineage) and resistance determinant calls (mecA, tetracycline, erythromycin)
- Per-frequency detection power estimates aggregated in detection_power_summary.csv

## How to apply

Download 450 St_B1 S. aureus samples (each with at least 100x mean depth of coverage) and 18 St_B2 CoNS samples (9 S. epidermidis and 9 S. haemolyticus) from ENA/SRA. Build 540 single-pair S. aureus/CoNS in silico mixtures by pooling FASTQ reads at 30 frequencies from 0.5 to 20%, and 27,000 pairwise St_B1 mixtures at ratios spanning 1:99 to 99:1. Run Mykrobe predictor with --format json on every mixture, then parse the JSON outputs to extract species calls (phylo_group, species, lineage) and resistance determinant calls (mecA, tetracycline, erythromycin loci), e.g. via the R-based Mykrobe Parser. At each mixture frequency, compute detection power as the fraction of mixtures receiving the correct species or resistance call, and aggregate the results into a summary table such as detection_power_summary.csv. The rationale is that plasmid-borne resistance genes are present in multiple copies per cell, so the minimum detectable frequency differs by locus.

## Related tools

- **Mykrobe predictor** (Run on every simulated mixture (with --format json) to call species (phylo_group, species, lineage) and resistance determinants (mecA, tetracycline, erythromycin loci)) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe Parser** (R script to parse the Mykrobe JSON files, separating mutations into genetic regions and extracting speciation data (phylo_group, species, lineage) into tidy CSV outputs) — https://github.com/phac-nml/mykrobe-parser

## Examples

```
mykrobe predict --sample mixture_001 --species saureus --output out.json --format json --seq reads.fq.gz
```

## Evaluation signals

- Replicate the expected anchors: CoNS species detection power of 100% and mecA detection power of 0.33 at 1% minor frequency
- At 2% minor frequency, detection power should reach 94% for tetracycline and 100% for erythromycin resistance genes on multi-copy plasmids
- Verify that the mixture design contains the full frequency grid: 30 frequencies from 0.5% to 20% for single-pair mixtures and ratios spanning 1:99 to 99:1 for pairwise mixtures
- Check that all Mykrobe runs were executed with --format json and that the parsed outputs contain phylo_group, species, and lineage fields
- Confirm the final aggregated table (detection_power_summary.csv) reports detection power as the fraction of correct calls per mixture frequency

## Limitations

- Detection power is estimated only on simulated two-population mixtures of S. aureus and CoNS; the model handles major/minor clones, and the article notes that incorporating a more general model of mixtures would be needed for direct-from-sputum M. tuberculosis analysis
- Estimates depend on the sequencing depth cutoff (only St_B1 samples with >=100x mean coverage were used); shallower input data will shift the minimum detectable frequency
- The aggregated power table mixes distinct readouts (species vs mecA vs plasmid-borne loci), so detection power is locus-specific and must not be generalized across determinants

## Evidence

- [discussion] In silico mixture design over a frequency gradient: "We then made in silico mixtures of these pairs with the CoNS samples at 30 frequencies ranging from 0.5 to 20%"
- [discussion] High-depth parent samples for mixture construction: "We took 450 samples from the S. aureus data set St_B1 that all had at least 100 × mean sequencing depth of coverage across the genome"
- [discussion] CoNS parent samples (9 S. epidermidis, 9 S. haemolyticus): "We took 18 CoNS samples from the St_B2 (Supplementary Data 4) data set (9 S. epidermidis and 9 S. haemolyticus)"
- [readme] JSON output requirement for downstream parsing: "Stores results a .json file instead of a text file. Mykrobe Parser only works with json files."
- [readme] Speciation fields extracted by the parser: "Stores Mykrobe's speciation data such as "phylo_group", "species", and "lineage" and their associated depths and percentages of coverage."
