---
name: bayesian-map-model-selection
description: Use when you have Mykrobe predictor JSON output for an M. tuberculosis
  sample and need to reproduce or audit the susceptible/minor/major genotype logic
  behind each resistance call.
when_to_use_negative:
- Analysing M. tuberculosis direct from sputum, where a general mixture model rather
  than simple major/minor clonal models is needed
- The goal is per-drug frequency thresholds for M. tuberculosis — the paper states
  there was not enough MTBC data to estimate them, so only the single 10% threshold
  is defensible
- Input is not Mykrobe JSON output with k-mer depth fields (e.g. already-parsed resistance
  calls), leaving no depths to fit the Poisson models
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Mykrobe predictor
  role: Generates the JSON k-mer depth output and reference genotype calls against
    which the recomputed MAP genotype calls are compared
  repo: https://github.com/Mykrobe-tools/mykrobe
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:c6bf1610a1f9f9288064ab115614b352fa8f3a576536f84e873d8e8196bb9fc0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe predictor
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Bayesian MAP model selection for genotyping

## Summary

Re-derives Mykrobe predictor's per-mutation-site genotype calls by comparing Poisson likelihoods of observed reference/alternate k-mer depths under three clonal models — susceptible, minor resistant (10% frequency), and major resistant (75% frequency) — and selecting the maximum a posteriori model.

## When to use

Use when you have Mykrobe predictor JSON output for an M. tuberculosis sample and need to reproduce or audit the susceptible/minor/major genotype logic behind each resistance call.

## When NOT to use

- Analysing M. tuberculosis direct from sputum, where a general mixture model rather than simple major/minor clonal models is needed
- The goal is per-drug frequency thresholds for M. tuberculosis — the paper states there was not enough MTBC data to estimate them, so only the single 10% threshold is defensible
- Input is not Mykrobe JSON output with k-mer depth fields (e.g. already-parsed resistance calls), leaving no depths to fit the Poisson models

## Inputs

- Mykrobe predictor JSON output (--format ) for an MTBC sample, containing per-mutation-site reference and alternate k-mer depths
- Mykrobe resistance panel defining mutation sites and genetic regions (e.g. the 201901 or 202010 panel)

## Outputs

- Per-site susceptible/minor/major genotype calls derived by MAP model selection
- genotype_concordance.csv with site-by-site concordance/discordance versus Mykrobe predictor calls, per drug and genetic region

## How to apply

Load the Mykrobe predictor JSON output for a training-set (MTBC_A1) sample and extract per-mutation-site k-mer depths for the reference and alternate alleles. For each site, compute expected k-mer count likelihoods under three Poisson models: pure susceptible, a minor resistant clone at 10% frequency, and a major resistant clone at 75% frequency, then pick the model with the maximum a posteriori (MAP) probability. Call a site resistant whenever a resistant genotype (minor or major) wins, applying the single 10% resistance-calling threshold adopted for M. tuberculosis because the training data did not support per-drug threshold estimation. Compare recomputed calls site-by-site against Mykrobe predictor's emitted calls, tabulating concordance/discordance per drug and genetic region.

## Related tools

- **Mykrobe predictor** (Generates the JSON k-mer depth output and reference genotype calls against which the recomputed MAP genotype calls are compared) — https://github.com/Mykrobe-tools/mykrobe

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out. --format  --seq reads.fq.gz
```

## Evaluation signals

- Recomputed MAP genotype calls match Mykrobe predictor's site-by-site calls, with concordance/discordance counts tabulated per drug and genetic region
- Every resistant phenotype call coincides with a minor or major model win under the single 10% threshold used for M. tuberculosis
- Poisson expected k-mer counts use the clonal frequencies 0.10 (minor) and 0.75 (major) at each mutation site
- genotype_concordance.csv contains both per-site calls and a per-drug concordance summary

## Limitations

- For M. tuberculosis, a single 10% threshold was used for all drugs because per-drug thresholds could not be estimated from the imperfect phenotyping data
- The implementation models only major/minor clones; a more general mixture model is suspected to be of value for direct-from-sputum M. tuberculosis analysis
- MTBC sensitivity is limited (82.6%) by poorly predictive known mutations, e.g. pyrazinamide is excluded entirely

## Evidence

- [abstract] Minor alleles improve detection of extremely drug-resistant strains: "We give evidence that minor alleles improve detection of extremely drug-resistant strains, and demonstrate feasibility of the use of emerging single-molecule"
- [discussion] Single 10% threshold rationale: "For M. tuberculosis, we did not have enough data to estimate per-drug thresholds, and we knew that the phenotyping data was imperfect"
- [discussion] Major/minor clonal model limitation: "incorporating a more general model of mixtures, rather than simply major/minor clones, will be of value when analysing M. tuberculosis samples direct from sputum"
- [readme] Generating the JSON input for genotyping: "mykrobe predict --sample my_sample_name \
  --species tb \
  --output out. \
  --format  \
  --seq reads.fq.gz"
