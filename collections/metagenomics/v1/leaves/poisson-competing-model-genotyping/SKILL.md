---
name: poisson-competing-model-genotyping
description: Use when you have per-mutation-site reference/alternate k-mer depths
  from a Mykrobe predictor JSON output (MTBC sample) and need to recompute or audit
  susceptible/minor/major genotype calls and drug resistance predictions.
when_to_use_negative:
- Input is a S. aureus sample where per-drug frequency thresholds exist and should
  be estimated from training data instead of using the single 10% M. tuberculosis
  threshold
- The sample is direct-from-sputum M. tuberculosis requiring a general mixture model
  beyond the major/minor clone formulation
- Mykrobe has already flagged the sample as non-tuberculous mycobacteria (removed
  for quality control) or a gene prediction failed, in which case predictions are
  'failed' and calls cannot be recomputed
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Mykrobe predictor
  role: Generates the JSON output with per-mutation-site k-mer depths that the three-model
    Poisson MAP genotyping reproduces; source of reference genotype calls for concordance
    comparison
  repo: https://github.com/iqbal-lab/Mykrobe-predictor
- name: Mykrobe Parser
  role: Parses Mykrobe JSON results for M. tuberculosis into per-genetic-region tidy
    tables, useful as a comparison/output format for the recomputed genotype calls
  repo: https://github.com/phac-nml/mykrobe-parser
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:ab50d84b751ff8085e4ae86d995fd0762b3105c5c0a64bdb84ac6d31840b5390
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iqbal-lab/Mykrobe-predictor
  tools:
  - Mykrobe Parser
  - Mykrobe predictor
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# poisson-competing-model-genotyping

## Summary

Classifies each resistance mutation site as susceptible, minor resistant, or major resistant by comparing Poisson k-mer depth likelihoods under three competing models and selecting the maximum a posteriori (MAP) genotype, then converting minor/major calls into resistance phenotypes at a 10% threshold. This reproduces the core genotyping logic of Mykrobe predictor (Bradley et al. 2015) for M. tuberculosis.

## When to use

Use when you have per-mutation-site reference/alternate k-mer depths from a Mykrobe predictor JSON output (MTBC sample) and need to recompute or audit susceptible/minor/major genotype calls and drug resistance predictions.

## When NOT to use

- Input is a S. aureus sample where per-drug frequency thresholds exist and should be estimated from training data instead of using the single 10% M. tuberculosis threshold
- The sample is direct-from-sputum M. tuberculosis requiring a general mixture model beyond the major/minor clone formulation
- Mykrobe has already flagged the sample as non-tuberculous mycobacteria (removed for quality control) or a gene prediction failed, in which case predictions are 'failed' and calls cannot be recomputed

## Inputs

- Mykrobe predictor JSON output file(s) for an MTBC sample (per-mutation-site k-mer depths for reference and alternate alleles)
- Resistance mutation/gene panel definitions with genetic region annotations (e.g. the 201901/202010 tb probe set)

## Outputs

- Per-site susceptible/minor/major genotype calls
- Per-drug resistance phenotype calls (resistant/susceptible) with 10% threshold applied
- Concordance/discordance table vs. Mykrobe predictor calls per drug and genetic region (genotype_concordance.csv)

## How to apply

Load the Mykrobe predictor JSON output for a training-set (e.g. MTBC_A1) sample and extract per-mutation-site k-mer depths for the reference and alternate alleles. For each site, compute expected k-mer count likelihoods under three Poisson models: pure susceptible, minor resistant clone at 10% frequency, and major resistant clone at 75% frequency. Assign the genotype by selecting the model with the maximum a posteriori (MAP) probability, and call the site resistant whenever a minor or major resistant genotype is selected, applying the single 10% resistance-calling threshold adopted for M. tuberculosis because per-drug thresholds could not be estimated. Finally, compare recomputed calls site-by-site against the Mykrobe predictor output per drug and genetic region, and write per-site calls plus a concordance summary to genotype_concordance.csv.

## Related tools

- **Mykrobe predictor** (Generates the JSON output with per-mutation-site k-mer depths that the three-model Poisson MAP genotyping reproduces; source of reference genotype calls for concordance comparison) — https://github.com/iqbal-lab/Mykrobe-predictor
- **Mykrobe Parser** (Parses Mykrobe JSON results for M. tuberculosis into per-genetic-region tidy tables, useful as a comparison/output format for the recomputed genotype calls) — https://github.com/phac-nml/mykrobe-parser

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out. --format  --seq reads.fq.gz
```

## Evaluation signals

- Site-by-site concordance between recomputed MAP genotype calls and the genotype calls emitted by Mykrobe predictor for the same sample, tabulated per drug and genetic region
- Each site is assigned exactly one of the three models (susceptible, minor at 10%, major at 75%) by maximum a posteriori probability — no unassigned sites
- Resistance is called only when the minor or major resistant model wins and the resistant allele frequency exceeds the single 10% threshold
- Per-site rows include reference and alternate k-mer depths extracted from the Mykrobe JSON, and the summary is written to genotype_concordance.csv
- Failed Mykrobe gene predictions propagate as 'failed' with genetic regions set to 'NA' rather than as genotype calls

## Limitations

- For M. tuberculosis there was not enough data to estimate per-drug thresholds, so a single 10% threshold was used for all drugs and the phenotyping data was known to be imperfect
- The model considers only major/minor clones; a more general mixture model is suspected to be needed for direct-from-sputum M. tuberculosis analysis
- The 10% and 75% model frequencies are parameters of the three-model approximation; low-frequency alleles' clinical relevance remains an open question

## Evidence

- [discussion] For M. tuberculosis, we did not have enough data to estimate per-drug thresholds, and we knew that the phenotyping data was imperfect: "For M. tuberculosis, we did not have enough data to estimate per-drug thresholds, and we knew that the phenotyping data was imperfect"
- [readme] It seperates mutations into indivudal genetic regions and presents them in a tidy data format compatible with reporting.: "It seperates mutations into indivudal genetic regions and presents them in a tidy data format compatible with reporting."
- [readme] Mykrobe Parser only works with  files.: [evidence span withheld — anchor status: not_found]
- [methods] If prediction for one gene fails in Mykrobe, Mykrobe parser will set all "[antimicrobial]_prediction" columns to "failed" and all genetic regions to "NA".: "If prediction for one gene fails in Mykrobe, Mykrobe parser will set all "[antimicrobial]_prediction" columns to "failed" and all genetic regions to "NA"."
- [discussion] Mykrobe predictor software is freely available (opensource) at www.github.com/iqbal-lab/Mykrobe-predictor: "The Mykrobe predictor software is freely available (opensource) at www.github.com/iqbal-lab/Mykrobe-predictor for non-commercial"
