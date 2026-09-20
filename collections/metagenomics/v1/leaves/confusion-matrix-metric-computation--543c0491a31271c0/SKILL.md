---
name: confusion-matrix-metric-computation
description: Use when evaluating genome-based resistance predictions (e.g. Mykrobe
  JSON calls) against phenotypic DST labels by building per-drug confusion matrices
  and computing overall and per-drug sensitivity/specificity on a validation set.
when_to_use_negative:
- Estimating false-negative rates for drugs with <=10 resistant samples in the validation
  set — the study only estimates these where sufficient resistant phenotypes exist
- Evaluating pyrazinamide resistance in M. tuberculosis — known mutations are poorly
  predictive so the drug is completely excluded from the panel
- Estimating per-drug prediction thresholds for M. tuberculosis — the study lacked
  enough data and used a single 10% threshold for all drugs with imperfect phenotyping
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Mykrobe predictor
  role: Produces per-isolate JSON resistance predictions (--format json) that are
    converted to per-drug calls for the confusion matrices
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe Parser
  role: R script that parses Mykrobe M. tuberculosis JSON results into tidy CSV with
    per-antimicrobial prediction columns and genetic regions used for tabulation
  repo: https://github.com/phac-nml/mykrobe-parser
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:b0c8604f29b93a1ce834d3b1d6d5ef70d5842d5be649bb79ef7871bab586cad7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe Parser
  - Mykrobe predictor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# confusion-matrix-metric-computation

## Summary

Tabulate Mykrobe predictor's per-drug resistant/susceptible calls against phenotypic DST labels in per-drug confusion matrices to compute overall and per-drug sensitivity/specificity for M. tuberculosis resistance prediction, benchmarked against KvarQ.

## When to use

Use when evaluating genome-based resistance predictions (e.g. Mykrobe JSON calls) against phenotypic DST labels by building per-drug confusion matrices and computing overall and per-drug sensitivity/specificity on a validation set.

## When NOT to use

- Estimating false-negative rates for drugs with <=10 resistant samples in the validation set — the study only estimates these where sufficient resistant phenotypes exist
- Evaluating pyrazinamide resistance in M. tuberculosis — known mutations are poorly predictive so the drug is completely excluded from the panel
- Estimating per-drug prediction thresholds for M. tuberculosis — the study lacked enough data and used a single 10% threshold for all drugs with imperfect phenotyping

## Inputs

- Mykrobe predictor JSON output files (one per isolate, --format json)
- Phenotypic DST resistance labels per isolate and drug (from Supplementary Data, ENA/NCBI SRA accessions)
- Mykrobe speciation data (output-jsondata.csv: phylo_group, species, lineage) for non-tuberculous mycobacteria QC

## Outputs

- Per-drug confusion matrices (Mykrobe resistant/susceptible calls vs phenotypic DST labels)
- validation_metrics.csv with overall sensitivity/specificity, per-drug sensitivities, and KvarQ comparison values
- Mykrobe Parser output-report.csv with per-antimicrobial prediction columns (resistant/susceptible/failed) and genetic regions

## How to apply

Run Mykrobe predictor on each validation isolate with the M. tuberculosis resistance panel using --format json so each isolate yields a JSON of resistance predictions. Apply QC: remove samples in which Mykrobe detects non-tuberculous mycobacteria, and exclude samples exceeding the 40 confident heterozygous SNP contamination threshold. Convert each JSON prediction to per-drug resistant/susceptible calls and tabulate them against phenotypic DST labels in per-drug confusion matrices (use Mykrobe Parser to separate mutations into genetic regions in a tidy CSV). Compute overall sensitivity/specificity across all drugs and per-drug sensitivities for rifampicin, isoniazid and ethambutol; restrict false-negative rate estimates to drugs with >10 resistant samples because resistant phenotype counts are limiting. Compare per-drug sensitivities side by side with the KvarQ reference values (90.8%, 83.2%, 76.3%) and save the metrics table.

## Related tools

- **Mykrobe predictor** (Produces per-isolate JSON resistance predictions (--format json) that are converted to per-drug calls for the confusion matrices) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe Parser** (R script that parses Mykrobe M. tuberculosis JSON results into tidy CSV with per-antimicrobial prediction columns and genetic regions used for tabulation) — https://github.com/phac-nml/mykrobe-parser

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out.json --format json --seq reads.fq.gz
```

## Evaluation signals

- Reproduced overall sensitivity/specificity on the n=1,609 MTBC_B validation set must equal the reported 82.6%/98.5%
- Per-drug sensitivities must match 93.7% (rifampicin), 84.3% (isoniazid) and 71.6% (ethambutol), alongside KvarQ values of 90.8%, 83.2% and 76.3%
- QC must remove samples Mykrobe detects as non-tuberculous mycobacteria and samples exceeding the 40-confident-heterozygous-SNP contamination threshold
- Any gene whose Mykrobe prediction failed must appear as 'failed' in all [antimicrobial]_prediction columns with genetic regions set to 'NA'
- Only M. tuberculosis panel drugs with >10 resistant samples carry estimated false-negative rates

## Limitations

- Overall M. tuberculosis sensitivity is low (82.6% across all drugs) compared with traditional DST, probably due to limited understanding of the underlying genetic mechanisms
- Pyrazinamide is completely excluded because known mutations are poorly predictive
- A single 10% threshold was used for all M. tuberculosis drugs since per-drug thresholds could not be estimated from available data
- Only 18%/44% of training/validation samples were phenotyped for at least one quinolone or injectable, so second-drug findings need replication with larger data sets

## Evidence

- [abstract] overall_mtbc_metrics: "For M. tuberculosis, our method predicts resistance with sensitivity/specificity of 82.6%/98.5% (independent validation set, n=1,609)"
- [discussion] fnr_resistant_sample_threshold: "the number of samples with resistant phenotypes was limiting, and we only estimated false-negative rates where there were sufficiently many (>10) resistant samples"
- [discussion] specificity_context: "For M. tuberculosis, specificity is high (98.5%) and sensitivity of 82.6% matches the line-probe assays from which our resistance panel was constructed"
- [readme] ntm_qc_removal: "Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes."
