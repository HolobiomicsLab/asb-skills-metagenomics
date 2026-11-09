---
name: sensitivity-specificity-calculation
description: Use when you have per-sample resistance predictions (e.g. Mykrobe JSON
  calls) plus phenotype labels and must quantify false negatives/positives as sensitivity
  and specificity, per drug and overall.
when_to_use_negative:
- The predictions have not yet been parsed from Mykrobe JSON into a tabular report
  (run Mykrobe Parser first)
- No phenotypic reference labels exist for the samples — sensitivity/specificity cannot
  be defined without gold-standard calls
- A drug has fewer than ~10 resistant samples in the validation set, making false-negative
  rate estimates unreliable per the article's own criterion
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3303
- http://edamontology.org/topic_0622
tools:
- name: Mykrobe predictor
  role: Generates per-sample antibiotic resistance calls from raw sequence data for
    comparison against phenotypes
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe Parser
  role: Parses Mykrobe JSON results into output-report.csv, separating mutations into
    tidy genetic regions suitable for per-drug error tallying
  repo: https://github.com/phac-nml/mykrobe-parser
- name: Phoenix automated microbiology system (BD Biosciences)
  role: Phenotypic gold-standard (with BSAC disc test) used to label validation samples
    as resistant/susceptible
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:7650cbe48c30023216bd924861c3b5a2d27a1af7f15c6377d79bdca89dc8af8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe Parser
  - Mykrobe predictor
  - Phoenix automated microbiology system (BD Biosciences)
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3303
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# sensitivity-specificity-calculation

## Summary

Compute per-drug and overall sensitivity/specificity of Mykrobe predictor's resistance calls by comparing parsed JSON predictions against phenotypic references (BSAC disc test, Phoenix), as done for S. aureus (99.1%/99.6% across 12 antibiotics) and M. tuberculosis (82.6%/98.5%).

## When to use

Use when you have per-sample resistance predictions (e.g. Mykrobe JSON calls) plus phenotype labels and must quantify false negatives/positives as sensitivity and specificity, per drug and overall.

## When NOT to use

- The predictions have not yet been parsed from Mykrobe JSON into a tabular report (run Mykrobe Parser first)
- No phenotypic reference labels exist for the samples — sensitivity/specificity cannot be defined without gold-standard calls
- A drug has fewer than ~10 resistant samples in the validation set, making false-negative rate estimates unreliable per the article's own criterion

## Inputs

- Per-sample Mykrobe predictor JSON output files (resistance calls per antibiotic)
- Phenotypic gold-standard labels (BSAC disc test and Phoenix automated microbiology system results)
- Validation isolate read sets (e.g. St_B1 S. aureus isolates, St_B2 CoNS, Myc_Retro M. tuberculosis samples)

## Outputs

- Per-drug error table (false negatives, false positives, FNR/FPR per antibiotic, e.g. st_b1_per_drug_error_table.csv)
- Overall sensitivity and specificity across the antibiotic panel (e.g. 99.1%/99.6% for S. aureus across 12 antibiotics)

## How to apply

Run Mykrobe predictor with '--format ' and '--panel 201901' on each isolate read set, then parse the JSON files with Mykrobe Parser ('Rscript mykrobe_parser.R -d PATH/TO/Mykrobe__files') into output-report.csv with mutations separated into tidy genetic regions. Compare each per-drug prediction in output-report.csv against the phenotypic reference — for the S. aureus validation set St_B1, phenotyping used a BSAC disc test and the Phoenix automated microbiology system (BD Biosciences) — flagging false negatives and false positives per antibiotic. Aggregate FN/FP counts per drug to compute per-drug error rates, then overall sensitivity and specificity across the panel (e.g. 12 antibiotics for S. aureus, reproducing 99.1%/99.6% with methicillin FPR 0.0%, penicillin FPR 11.7%, and ciprofloxacin FNR 4.6%). Only estimate false-negative rates where there were sufficiently many (>10) resistant samples in the validation set, as resistant-phenotype counts are limiting.

## Related tools

- **Mykrobe predictor** (Generates per-sample antibiotic resistance calls from raw sequence data for comparison against phenotypes) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe Parser** (Parses Mykrobe JSON results into output-report.csv, separating mutations into tidy genetic regions suitable for per-drug error tallying) — https://github.com/phac-nml/mykrobe-parser
- **Phoenix automated microbiology system (BD Biosciences)** (Phenotypic gold-standard (with BSAC disc test) used to label validation samples as resistant/susceptible)

## Examples

```
Rscript PATH/TO/R/mykrobe_parser.R -d PATH/TO/Mykrobe__files
```

## Evaluation signals

- Overall S. aureus sensitivity/specificity reproduces 99.1%/99.6% across 12 antibiotics on validation set St_B1
- Per-drug checks: methicillin FPR = 0.0%, penicillin FPR = 11.7%, ciprofloxacin FNR = 4.6%
- output-report.csv exists with per-drug prediction columns and mutations separated into individual genetic regions in tidy format
- Failed gene predictions are marked 'failed' with genetic regions set to 'NA' rather than silently scored as correct/incorrect
- False-negative rates are only reported for drugs with >10 resistant samples in the validation set

## Limitations

- M. tuberculosis sensitivity is low (82.6% across all drugs) versus traditional DST, and pyrazinamide is completely excluded since known mutations are poorly predictive
- Number of samples with resistant phenotypes was limiting; false-negative rates were only estimated where >10 resistant samples existed in the validation set
- Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes, affecting the scored sample set
- If prediction for one gene fails in Mykrobe, Mykrobe parser sets all '[antimicrobial]_prediction' columns to 'failed' and genetic regions to 'NA'

## Evidence

- [abstract] S. aureus headline metrics: "For S. aureus, the error rates of our method are comparable to gold-standard phenotypic methods, with sensitivity/specificity of 99.1%/99.6% across 12 antibiotics"
- [discussion] Phenotypic reference methods: "all validation samples were phenotyped using two methods: a BSAC disc test15 and the Phoenix automated microbiology system (BD Biosciences)"
- [discussion] Minimum resistant-sample threshold for FNR estimation: "the number of samples with resistant phenotypes was limiting, and we only estimated false-negative rates where there were sufficiently many (>10) resistant samples"
- [readme] Failed-gene handling in parser output: "If prediction for one gene fails in Mykrobe, Mykrobe parser will set all "[antimicrobial]_prediction" columns to "failed" and all genetic regions to "NA"."
- [readme] Required Mykrobe arguments for parser compatibility: "To be compatible with Mykrobe Parser, Mykrobe must be run using the following arguments: --format  ... --panel 201901"
