---
name: resistance-genotype-concordance-assessment
description: Use when you have Mykrobe predictor JSON output for an M. tuberculosis
  sample and need to independently re-derive susceptible/minor/major genotype calls
  and measure per-site concordance with Mykrobe's emitted calls.
when_to_use_negative:
- Input is already a parsed Mykrobe parser report CSV (output-report.csv); this skill
  needs the raw per-site JSON k-mer depths, not aggregated per-drug predictions
- The goal is resistance prediction for a poorly characterized species or direct-from-sputum
  mixtures, where the major/minor two-clone model is inadequate and per-drug thresholds
  cannot be estimated
- Working with S. aureus samples, where per-drug thresholds were estimated from data
  rather than the fixed 10% M. tuberculosis threshold
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Mykrobe predictor
  role: Generates the JSON resistance-genotyping output (per-site k-mer depths and
    genotype calls) that is recomputed and compared against
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: mykrobe-parser
  role: R script that parses Myk tuberculosis-specific results from the JSON files
    into tidy per-genetic-region output, an alternative consumption path for the same
    JSON
  repo: https://github.com/phac-nml/mykrobe-parser
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:6097434292e1b1c5b4f6f4d85cf83b546d79098d95b4f7ddcba9108a8c5c7e5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe predictor
  - mykrobe-parser
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# resistance-genotype-concordance-assessment

## Summary

Recomputes per-site susceptible/minor/major resistance genotype calls from Mykrobe predictor JSON k-mer depths using a three-model MAP Poisson framework, then benchmarks them site-by-site against Mykrobe predictor's own calls for an MTBC training-set sample, yielding a genotype_concordance.csv summary.

## When to use

Use when you have Mykrobe predictor JSON output for an M. tuberculosis sample and need to independently re-derive susceptible/minor/major genotype calls and measure per-site concordance with Mykrobe's emitted calls.

## When NOT to use

- Input is already a parsed Mykrobe parser report CSV (output-report.csv); this skill needs the raw per-site JSON k-mer depths, not aggregated per-drug predictions
- The goal is resistance prediction for a poorly characterized species or direct-from-sputum mixtures, where the major/minor two-clone model is inadequate and per-drug thresholds cannot be estimated
- Working with S. aureus samples, where per-drug thresholds were estimated from data rather than the fixed 10% M. tuberculosis threshold

## Inputs

- Mykrobe predictor JSON output file (per-mutation-site k-mer depths for reference and alternate alleles)
- Training-set M. tuberculosis sample from MTBC_A1 (FASTQ reads sequenced on Illumina HiSeq 2000 or MiSeq)
- Resistance-calling frequency threshold (10% for M. tuberculosis)

## Outputs

- genotype_concordance.csv with per-site susceptible/minor/major calls and per-drug/per-genetic-region concordance summary
- Per-site MAP genotype assignments (susceptible / minor resistant at 10% / major resistant at 75%)

## How to apply

Load the Mykrobe predictor JSON output for a training-set (MTBC_A1, 338 MTBC clinical isolates) sample and extract per-mutation-site k-mer depths for reference and alternate alleles. For each mutation site, compute expected k-mer count likelihoods under three Poisson models: pure clonal susceptible, minor resistant clone at 10% frequency, and major resistant clone at 75% frequency, then assign the susceptible/minor/major genotype via maximum a posteriori (MAP) selection. Call a site resistant when the minor or major model wins under the single 10% resistance-calling threshold adopted for M. tuberculosis, chosen because there was insufficient data to estimate per-drug thresholds. Compare recomputed calls site-by-site against Mykrobe predictor's calls for the same sample, tabulating concordance/discordance per drug and genetic region, and write per-site calls plus the concordance summary to genotype_concordance.csv. The resistance panel and JSON format must match what the tool emits (e.g. 201901/202010 panels with --format , as Mykrobe Parser only consumes  files).

## Related tools

- **Mykrobe predictor** (Generates the JSON resistance-genotyping output (per-site k-mer depths and genotype calls) that is recomputed and compared against) — https://github.com/Mykrobe-tools/mykrobe
- **mykrobe-parser** (R script that parses Myk tuberculosis-specific results from the JSON files into tidy per-genetic-region output, an alternative consumption path for the same JSON) — https://github.com/phac-nml/mykrobe-parser

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out. --format  --seq reads.fq.gz
```

## Evaluation signals

- Site-by-site comparison of recomputed susceptible/minor/major calls against Mykrobe predictor's emitted calls yields high concordance, with discordances tabulated per drug and genetic region
- genotype_concordance.csv exists and contains a per-site call row for every mutation site in the sample's JSON output
- Every MAP call is one of the three admissible classes: susceptible, minor resistant (10%), or major resistant (75%)
- All resistance calls derive from the single 10% threshold; no per-drug thresholds appear for M. tuberculosis samples
- Sample-level QC holds: non-tuberculous mycobacteria detections removed before concordance assessment

## Limitations

- For M. tuberculosis there was not enough data to estimate per-drug thresholds, so a single 10% threshold was used for all drugs despite imperfect phenotyping data
- The implementation models only major/minor clones; a more general mixture model is suspected to be needed for direct-from-sputum M. tuberculosis analysis
- If prediction for one gene fails in Mykrobe, downstream parsed outputs mark predictions as 'failed' and genetic regions as 'NA'
- Overall M. tuberculosis sensitivity is low (82.6%) and pyrazinamide is excluded because known mutations are poorly predictive

## Evidence

- [discussion] Single fixed threshold rationale: "For M. tuberculosis, we did not have enough data to estimate per-drug thresholds, and we knew that the phenotyping data was imperfect"
- [discussion] Training set used: "MTBC_A1 (338 MTBC clinical isolates, Supplementary Data 6)"
- [discussion] Mixture-model limitation: "we suspect that incorporating a more general model of mixtures, rather than simply major/minor clones, will be of value when analysing M. tuberculosis samples direct from sputum"
- [readme] JSON format requirement: [evidence span withheld — anchor status: not_found]
- [readme] QC sample removal: "Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes."
