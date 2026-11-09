---
name: allele-frequency-threshold-classification
description: Use when a resistance-calling pipeline must distinguish susceptible,
  low-frequency (minor), and high-frequency (major) resistant alleles from k-mer depths
  at mutation sites, and when a frequency threshold for calling resistance must be
  applied or justified.
when_to_use_negative:
- When a general mixture model is required for direct-from-sputum M. tuberculosis
  samples — the paper notes the current implementation only models major/minor clones
  and 'a more general model of mixtures' is needed
- When per-drug frequency thresholds must be data-driven for M. tuberculosis — the
  paper states there was insufficient TB data to estimate per-drug thresholds, so
  only a single 10% threshold was adopted
- When the sample has already been called resistant/susceptible by Mykrobe and only
  tabular reporting is needed — Mykrobe Parser already emits per-drug predictions
  and genetic-region columns
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2830
- http://edamontology.org/topic_3673
tools:
- name: Mykrobe predictor
  role: Produces the JSON output containing per-mutation k-mer depths from which the
    three-model MAP genotyping and frequency-threshold calls are recomputed and compared
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe Parser
  role: Parses Mykrobe JSON results into tidy per-genetic-region tables with per-antimicrobial
    prediction columns for downstream reporting
  repo: https://github.com/phac-nml/mykrobe-parser
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:5d59c776f2b8596089b6832551271eb765c00fa7f467870bc18e5c4999a9e861
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe Parser
  - Mykrobe predictor
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2830
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# allele-frequency-threshold-classification

## Summary

Classifies each resistance mutation/gene site as clonal susceptible, minor resistant, or major resistant via MAP over Poisson k-mer-depth models, then converts a resistant genotype into a phenotype call using an allele-frequency threshold (a single 10% threshold was adopted for M. tuberculosis).

## When to use

Use when a resistance-calling pipeline must distinguish susceptible, low-frequency (minor), and high-frequency (major) resistant alleles from k-mer depths at mutation sites, and when a frequency threshold for calling resistance must be applied or justified.

## When NOT to use

- When a general mixture model is required for direct-from-sputum M. tuberculosis samples — the paper notes the current implementation only models major/minor clones and 'a more general model of mixtures' is needed
- When per-drug frequency thresholds must be data-driven for M. tuberculosis — the paper states there was insufficient TB data to estimate per-drug thresholds, so only a single 10% threshold was adopted
- When the sample has already been called resistant/susceptible by Mykrobe and only tabular reporting is needed — Mykrobe Parser already emits per-drug predictions and genetic-region columns

## Inputs

- Mykrobe predictor JSON output with per-mutation-site k-mer depths for reference and alternate alleles
- Sequencing reads (e.g., Illumina FASTQ) processed by Mykrobe predict for M. tuberculosis or S. aureus
- Resistance panel (e.g., the tb '201901' or '202010' panels) defining the mutation/gene sites to genotype
- Training-set phenotype data (e.g., MTBC_A1 or St_A1) used to estimate resistance-calling frequency thresholds

## Outputs

- Per-site susceptible/minor/major genotype calls (MAP model assignment)
- Per-site resistant/susceptible phenotype calls under the frequency threshold
- Site-by-site concordance/discordance versus Mykrobe predictor calls, tabulated per drug and genetic region (e.g., genotype_concordance.csv)

## How to apply

For each mutation site, extract reference and alternate allele k-mer depths from the sample (e.g., a Mykrobe predictor JSON output for an MTBC_A1 training-set isolate) and compute expected k-mer count likelihoods under three Poisson models: pure susceptible clone, minor resistant clone at 10% frequency, and major resistant clone at 75% frequency. Select the maximum a posteriori (MAP) model at each site to assign a susceptible/minor/major genotype. Call the site resistant whenever a resistant genotype (minor or major) is selected under the single 10% resistance-calling threshold adopted for M. tuberculosis, since the article states there was not enough TB data to estimate per-drug thresholds and phenotyping was imperfect. Recompute genotype calls site-by-site and compare against Mykrobe predictor's own calls (per drug and genetic region) to verify concordance before reporting per-site calls and a concordance summary. For S. aureus, per-drug thresholds can be estimated from the training set (e.g., St_A1, 532 clinical isolates) rather than fixed at 10%.

## Related tools

- **Mykrobe predictor** (Produces the JSON output containing per-mutation k-mer depths from which the three-model MAP genotyping and frequency-threshold calls are recomputed and compared) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe Parser** (Parses Mykrobe JSON results into tidy per-genetic-region tables with per-antimicrobial prediction columns for downstream reporting) — https://github.com/phac-nml/mykrobe-parser

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out. --format  --seq reads.fq.gz
```

## Evaluation signals

- Recomputed MAP genotype calls match Mykrobe predictor's emitted calls site-by-site; tabulate concordance/discordance per drug and genetic region
- Sites called resistant have an alternate-allele k-mer depth consistent with at least ~10% of total depth at that site (the adopted M. tuberculosis resistance-calling threshold)
- Each site resolves to exactly one of the three models (pure susceptible Poisson, minor at 10%, major at 75%) with a defined MAP probability
- Genes failing prediction are surfaced as 'failed' with genetic regions 'NA' rather than silently dropped
- Sensitivity/specificity of the derived phenotype calls against phenotype ground truth fall in the reported ranges (e.g., 82.6%/98.5% for M. tuberculosis on the n=1,609 validation set)

## Limitations

- For M. tuberculosis, not enough data existed to estimate per-drug thresholds, so a single 10% threshold was used for all drugs despite imperfect phenotyping
- The model only handles major/minor clones; a more general mixture model is needed for direct-from-sputum TB analysis
- Overall M. tuberculosis sensitivity is low (82.6%) and pyrazinamide is excluded entirely because known mutations are poorly predictive
- Whether low-frequency (minor) alleles affect patient outcome remains an open question, despite generally failing to cause in vitro resistance

## Evidence

- [methods] After the three-category genotyping step (clonal susceptible, minor resistant, major resistant), the algorithm predicts a resistant phenotype whenever a resistance allele is present at a sufficiently high frequency, with the 'sufficiently high' frequency threshold estimated from the training set.: [evidence span withheld — anchor status: not_found]
- [discussion] For M. tuberculosis, we did not have enough data to estimate per-drug thresholds, and we knew that the phenotyping data was imperfect: "For M. tuberculosis, we did not have enough data to estimate per-drug thresholds, and we knew that the phenotyping data was imperfect"
- [discussion] we suspect that incorporating a more general model of mixtures, rather than simply major/minor clones, will be of value when analysing M. tuberculosis samples direct from sputum: "incorporating a more general model of mixtures, rather than simply major/minor clones, will be of value when analysing M. tuberculosis samples direct from sputum"
- [discussion] our sensitivity for M. tuberculosis is low (82.6% across all drugs) compared with traditional DST, and completely excludes the first-line drug pyrazinamide since known mutations are poorly predictive: "our sensitivity for M. tuberculosis is low (82.6% across all drugs) compared with traditional DST, and completely excludes the first-line drug pyrazinamide"
- [readme] Mykrobe Parser only works with  files.: [evidence span withheld — anchor status: not_found]
