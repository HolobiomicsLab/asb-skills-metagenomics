---
name: in-silico-read-mixture-simulation
description: Use when you need to quantify how reliably an AMP-based predictor (Mykrobe)
  detects a low-frequency contaminant or resistant subpopulation in NGS reads, e.g.
  down to 0.5–2% minor-population frequency.
when_to_use_negative:
- When you need to model within-sample mixtures of M. tuberculosis directly from sputum
  — the article notes the current implementation models only major/minor clones and
  a more general mixture model would be needed.
- When the goal is phenotypic resistance assessment rather than read-level detection
  of genetic determinants — simulated mixtures only calibrate genotypic detection
  power.
- 'When your samples lack sufficient depth: simulations were built from samples with
  at least 100 × mean sequencing depth, and power estimates will not transfer to low-coverage
  data.'
tools:
- name: Mykrobe predictor
  role: Runs on every simulated mixed-read FASTQ to produce species (phylo_group,
    species, lineage) and resistance determinant (mecA, tetracycline, erythromycin)
    calls, whose fraction of correctness defines detection power
  repo: https://github.com/Mykrobe-tools/mykrobe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:09fe30543522998994cab2a8ee8d26b70e6872775f23d444bcc78497bb123ea7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe predictor
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# In-silico read mixture simulation for detection-power calibration

## Summary

Pools FASTQ reads from real sequencing runs to construct simulated mixed infections, then runs Mykrobe predictor on each mixture to measure the detection power of the tool for minor-population species and resistance determinants at defined frequencies.

## When to use

Use when you need to quantify how reliably an AMP-based predictor (Mykrobe) detects a low-frequency contaminant or resistant subpopulation in NGS reads, e.g. down to 0.5–2% minor-population frequency.

## When NOT to use

- When you need to model within-sample mixtures of M. tuberculosis directly from sputum — the article notes the current implementation models only major/minor clones and a more general mixture model would be needed.
- When the goal is phenotypic resistance assessment rather than read-level detection of genetic determinants — simulated mixtures only calibrate genotypic detection power.
- When your samples lack sufficient depth: simulations were built from samples with at least 100 × mean sequencing depth, and power estimates will not transfer to low-coverage data.

## Inputs

- FASTQ sequencing files from real Illumina runs (e.g. St_B1 S. aureus samples with >=100x mean depth and St_B2 CoNS samples)
- A predefined set of mixture frequencies or ratios (e.g. 30 frequencies from 0.5 to 20%; ratios 1:99 to 99:1)

## Outputs

- Mykrobe predictor JSON output files (--format json) for every simulated mixture
- A detection-power summary table (e.g. detection_power_summary.csv) giving fraction of correct species/resistance calls at each mixture frequency

## How to apply

First select high-coverage real data: the article used 450 S. aureus samples from St_B1 (each with at least 100 × mean sequencing depth) and 18 CoNS samples from St_B2 (9 S. epidermidis and 9 S. haemolyticus). Pool the FASTQ reads of sample pairs to construct in silico mixtures — 540 single-pair S. aureus/CoNS mixtures at 30 CoNS frequencies ranging from 0.5 to 20%, and 27,000 pairwise S. aureus/S. aureus mixtures at ratios spanning 1:99 to 99:1. Run Mykrobe predictor on every simulated mixture with --format json so results are machine-parseable, then parse the JSON to extract species calls (phylo_group, species, lineage) and resistance determinant calls (mecA, tetracycline and erythromycin loci). Compute detection power at each frequency as the fraction of mixtures receiving the correct call; the article's benchmarks were 100% CoNS species detection, 0.33 for mecA at 1%, and 94% (tetracycline) / 100% (erythromycin) resistance detection at 2% minor frequency — the latter two reflect detection of resistance genes carried on multi-copy plasmids.

## Related tools

- **Mykrobe predictor** (Runs on every simulated mixed-read FASTQ to produce species (phylo_group, species, lineage) and resistance determinant (mecA, tetracycline, erythromycin) calls, whose fraction of correctness defines detection power) — https://github.com/Mykrobe-tools/mykrobe

## Examples

```
cat sAureus.fq.gz coNS.fq.gz > mixed.fq.gz; mykrobe predict --sample mixed --species sa --output out.json --format json --seq mixed.fq.gz
```

## Evaluation signals

- CoNS species detection power reaches 100% across the tested CoNS mixture frequencies (0.5–20%)
- mecA detection power is 0.33 at the 1% CoNS mixture frequency
- Tetracycline resistance detection power is 94% and erythromycin is 100% once the minor population reaches 2% frequency
- Every mixture yields a parseable JSON containing phylo_group, species, and lineage fields plus the queried resistance loci
- The mixture grid covers the stated design: 540 single-pair S. aureus/CoNS mixtures at 30 frequencies and 27,000 pairwise mixtures spanning 1:99 to 99:1

## Limitations

- High observed detection power for resistance genes (94–100% at 2%) applies to determinants on multi-copy plasmids; single-copy minor-allele detection is not directly benchmarked by this design.
- The current implementation models only major/minor clones; the article notes a more general mixture model will be needed when analysing M. tuberculosis samples direct from sputum.
- The simulation used S. aureus mixed with CoNS from St_B1/St_B2; power estimates may not generalise to other species pairs or to real clinical contamination with different coverage profiles.

## Evidence

- [discussion] Mixture construction design (frequencies): "We then made in silico mixtures of these pairs with the CoNS samples at 30 frequencies ranging from 0.5 to 20%"
- [discussion] High-coverage S. aureus input data: "We took 450 samples from the S. aureus data set St_B1 that all had at least 100 × mean sequencing depth of coverage"
- [discussion] CoNS contaminant samples: "We took 18 CoNS samples from the St_B2 (Supplementary Data 4) data set (9 S. epidermidis and 9 S. haemolyticus)"
- [abstract] Mykrobe accepts raw reads (needed to run on pooled FASTQ): "in a software package (‘Mykrobe predictor') that takes raw sequence data as input"
- [discussion] Major/minor-clone mixture model limitation: "incorporating a more general model of mixtures, rather than simply major/minor clones, will be of value when analysing M. tuberculosis"
- [readme] JSON output format requirement (from parser documentation): "Stores results a .json file instead of a text file. Mykrobe Parser only works with json files."
