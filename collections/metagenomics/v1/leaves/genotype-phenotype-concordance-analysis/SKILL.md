---
name: genotype-phenotype-concordance-analysis
description: Use when you have per-sample predicted resistance calls in JSON from
  Mykrobe predictor plus phenotypic DST labels and need sensitivity/specificity per
  antibiotic, or when reproducing error tables like St_B1's 99.1%/99.6% or minor-allele
  sensitivity deltas.
when_to_use_negative:
- When only one method (prediction or phenotype) is available — concordance metrics
  need both per-sample calls and DST labels
- When per-drug error rates are requested for a drug with fewer than ~10 phenotypically
  resistant validation samples — false-negative rates were only estimated where >10
  resistant samples existed
- When the goal is pyrazinamide prediction from known mutations — the article states
  known mutations are poorly predictive and the drug is excluded
edam_operation: http://edamontology.org/operation_3282
tools:
- name: Mykrobe predictor
  role: Generates per-sample JSON resistance predictions from raw sequencing reads
    that are compared against phenotypic labels
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe Parser
  role: R script that parses Mykrobe JSON into tidy per-drug output-report.csv with
    mutations separated into individual genetic regions
  repo: https://github.com/phac-nml/mykrobe-parser
- name: Phoenix automated microbiology system (BD Biosciences)
  role: Phenotypic gold-standard reference used to derive validation resistance labels
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:0b7058fa3f328d018c5b01720822bc79e14160109a57b78b93c323ccbb50316e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe Parser
  - Mykrobe predictor
  - Phoenix automated microbiology system (BD Biosciences)
  edam_operation: http://edamontology.org/operation_3282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Genotype-phenotype concordance analysis

## Summary

Computes per-drug sensitivity (false-negative rate) and specificity (false-positive rate) by comparing sequence-based resistance predictions (Mykrobe predictor JSON calls) against gold-standard phenotypic DST labels (BSAC disc test, Phoenix, MGIT), as done for S. aureus across 12 antibiotics and for M. tuberculosis second-line drugs.

## When to use

Use when you have per-sample predicted resistance calls in JSON from Mykrobe predictor plus phenotypic DST labels and need sensitivity/specificity per antibiotic, or when reproducing error tables like St_B1's 99.1%/99.6% or minor-allele sensitivity deltas.

## When NOT to use

- When only one method (prediction or phenotype) is available — concordance metrics need both per-sample calls and DST labels
- When per-drug error rates are requested for a drug with fewer than ~10 phenotypically resistant validation samples — false-negative rates were only estimated where >10 resistant samples existed
- When the goal is pyrazinamide prediction from known mutations — the article states known mutations are poorly predictive and the drug is excluded

## Inputs

- Per-sample Mykrobe predictor JSON resistance predictions ('--format ')
- FASTQ/sequence read sets from ENA/SRA (e.g. 471 St_B1 S. aureus isolates or MTBC_B second-line-drug validation samples)
- Phenotypic DST reference labels: BSAC disc test and Phoenix for S. aureus; MGIT-based phenotypic DST from Supplementary Data 1-11 for M. tuberculosis

## Outputs

- Per-drug error table (FN/FP counts, sensitivity and specificity per antibiotic, e.g. st_b1_per_drug_error_table.csv)
- output-report.csv from Mykrobe Parser with per-drug predictions and mutations separated into individual genetic regions
- minor_allele_sensitivity_deltas.csv with baseline vs minor-allele-enabled sensitivity per drug

## How to apply

Run Mykrobe predictor per isolate with '--format ' (and '--panel 201901' or '202010' for the parser-compatible workflow) on each FASTQ read set, then either parse JSON with Mykrobe Parser via Conda-installed R ('conda env create -f mykrobe_parser_installation.yml'; 'Rscript mykrobe_parser.R -d ...') to obtain tidy per-drug output-report.csv, or convert JSON directly to resistance calls. Match each per-drug prediction against phenotypic labels (BSAC disc test / Phoenix for S. aureus; MGIT-based DST for M. tuberculosis) to flag false negatives and false positives. Compute sensitivity as true positives / all phenotypically resistant samples and specificity analogously, aggregating FN/FP counts per antibiotic into a per-drug error table (e.g. st_b1_per_drug_error_table.csv). For minor-allele experiments, run the same comparison twice (minor-allele calls off vs on with the single 10% allele-frequency threshold) and tabulate sensitivity deltas per drug (capreomycin +9.1 pp, amikacin +11.8 pp, ofloxacin +23.0 pp).

## Related tools

- **Mykrobe predictor** (Generates per-sample JSON resistance predictions from raw sequencing reads that are compared against phenotypic labels) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe Parser** (R script that parses Mykrobe JSON into tidy per-drug output-report.csv with mutations separated into individual genetic regions) — https://github.com/phac-nml/mykrobe-parser
- **Phoenix automated microbiology system (BD Biosciences)** (Phenotypic gold-standard reference used to derive validation resistance labels)

## Examples

```
Rscript PATH/TO/R/mykrobe_parser.R -d PATH/TO/Mykrobe__files
```

## Evaluation signals

- S. aureus overall sensitivity/specificity should reproduce 99.1%/99.6% across 12 antibiotics on St_B1 (n≈471)
- Sub-metrics should match: methicillin false-positive rate 0.0%, penicillin FPR 11.7%, ciprofloxacin FNR 4.6%
- MTBC_B minor-allele deltas should match: capreomycin 74.5→83.6%, amikacin 78.0→89.8%, ofloxacin 38.5→61.5% (N=13)
- M. tuberculosis validation should yield sensitivity/specificity of 82.6%/98.5% (n=1,609)
- output-report.csv should be a tidy table with per-drug prediction columns; genes whose Mykrobe prediction fails must show '[antimicrobial]_prediction' = 'failed' with genetic regions 'NA'

## Limitations

- M. tuberculosis sensitivity is low (82.6%) and pyrazinamide is completely excluded because known mutations are poorly predictive
- A single 10% minor-allele threshold was applied to all drugs because there was insufficient M. tuberculosis data to estimate per-drug thresholds
- The minor-allele finding for second-line drugs needs replication with larger data sets; only 18%/44% of training/validation samples were phenotyped for at least one quinolone or injectable
- False-negative rates were only estimated where >10 resistant samples existed in the validation set; contaminated samples were removed (6 training, 20 validation; >40 confident heterozygous SNPs threshold)

## Evidence

- [abstract] sensitivity/specificity of 99.1%/99.6% across 12 antibiotics: "For S. aureus, the error rates of our method are comparable to gold-standard phenotypic methods, with sensitivity/specificity of 99.1%/99.6% across 12 antibiotics"
- [discussion] Phenotyping reference methods: "all validation samples were phenotyped using two methods: a BSAC disc test15 and the Phoenix automated microbiology system (BD Biosciences)"
- [discussion] FN-rate estimation constraint: "the number of samples with resistant phenotypes was limiting, and we only estimated false-negative rates where there were sufficiently many (>10) resistant samples"
- [readme] JSON format requirement: [evidence span withheld — anchor status: not_found]
- [readme] Failed-gene handling: "If prediction for one gene fails in Mykrobe, Mykrobe parser will set all "[antimicrobial]_prediction" columns to "failed" and all genetic regions to "NA"."
