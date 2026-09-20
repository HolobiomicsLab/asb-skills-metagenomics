---
name: cross-method-diagnostic-comparison
description: Use when two genotype-based antibiotic-resistance prediction tools must
  be compared on the same isolates and phenotypic DST labels, e.g. per-drug sensitivity
  of Mykrobe vs. KvarQ on first-line TB drugs.
when_to_use_negative:
- Estimating false-negative rates for a drug with ≤10 resistant samples in the validation
  set — the article only estimated FNRs where '>10 resistant samples' were available
- Comparing methods for pyrazinamide — known mutations are poorly predictive and the
  drug was completely excluded from the M. tuberculosis panel
- Reproducing a comparator like SeqSphere whose gene-panel template was not publicly
  released, making the comparison non-reproducible
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: Mykrobe predictor
  role: Primary method under evaluation; predicts M. tuberculosis resistance per drug
    from raw reads, emitting JSON result files
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe Parser
  role: R script that parses the M. tuberculosis-specific Mykrobe JSON results into
    tidy CSV, removing samples Mykrobe detects as non-tuberculous mycobacteria
  repo: https://github.com/phac-nml/mykrobe-parser
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:690a38678938f47179b1e2c3cedc1c136cf3e8cbf6338b8df649665e84848936
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe Parser
  - Mykrobe predictor
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cross-method-diagnostic-comparison

## Summary

Head-to-head benchmarking of genotypic resistance-prediction methods (Mykrobe predictor vs. KvarQ) on the same independent validation set, computing per-drug sensitivities so the two tools' performances can be compared side by side.

## When to use

Use when two genotype-based antibiotic-resistance prediction tools must be compared on the same isolates and phenotypic DST labels, e.g. per-drug sensitivity of Mykrobe vs. KvarQ on first-line TB drugs.

## When NOT to use

- Estimating false-negative rates for a drug with ≤10 resistant samples in the validation set — the article only estimated FNRs where '>10 resistant samples' were available
- Comparing methods for pyrazinamide — known mutations are poorly predictive and the drug was completely excluded from the M. tuberculosis panel
- Reproducing a comparator like SeqSphere whose gene-panel template was not publicly released, making the comparison non-reproducible

## Inputs

- Sequencing reads (FASTQ) for the 1,609 MTBC_B validation isolates from ENA/NCBI SRA
- Phenotypic DST resistance labels per isolate and drug (Supplementary Data accession lists)
- Mykrobe predictor JSON outputs per isolate
- KvarQ per-drug sensitivity values for rifampicin (90.8%), isoniazid (83.2%) and ethambutol (76.3%)

## Outputs

- Per-drug confusion matrices (resistant/susceptible calls vs. phenotypic DST labels)
- Overall sensitivity/specificity metrics (target: 82.6%/98.5% on n=1,609)
- Per-drug sensitivity table comparing Mykrobe vs. KvarQ (validation_metrics.csv)

## How to apply

Run each tool (e.g. Mykrobe predictor with --format , and the KvarQ comparison panel) on the same validation isolates — here the n=1,609 MTBC_B set — and convert JSON predictions into per-drug resistant/susceptible calls tabulated against phenotypic DST labels in confusion matrices. Apply identical quality control to both arms: remove samples detected as non-tuberculous mycobacteria and samples exceeding the 40-confident-heterozygous-SNP contamination threshold. Compute overall sensitivity/specificity across all drugs (Mykrobe: 82.6%/98.5%) and per-drug sensitivities for rifampicin (93.7%), isoniazid (84.3%) and ethambutol (71.6%), restricting false-negative rate estimates to drugs with >10 resistant samples in the validation set. Record the comparator's per-drug sensitivities alongside (KvarQ: 90.8%, 83.2%, 76.3%) and save overall, per-drug and comparison metrics to a table such as validation_metrics.csv. The rationale is that restricted panel size and imperfect phenotyping limit sensitivity, so sensitivity differences between methods must be interpreted drug-by-drug rather than as a single headline number.

## Related tools

- **Mykrobe predictor** (Primary method under evaluation; predicts M. tuberculosis resistance per drug from raw reads, emitting JSON result files) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe Parser** (R script that parses the M. tuberculosis-specific Mykrobe JSON results into tidy CSV, removing samples Mykrobe detects as non-tuberculous mycobacteria) — https://github.com/phac-nml/mykrobe-parser

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out. --format  --seq reads.fq.gz
```

## Evaluation signals

- Reproduced overall sensitivity/specificity on MTBC_B (n=1,609) equals the reported 82.6%/98.5%
- Per-drug Mykrobe sensitivities match 93.7% (rifampicin), 84.3% (isoniazid) and 71.6% (ethambutol)
- KvarQ comparator sensitivities recorded as 90.8%, 83.2% and 76.3% for the same three first-line drugs
- QC applied consistently: samples flagged as non-tuberculous mycobacteria and samples with >40 confident heterozygous SNPs excluded from both methods' tallies
- Per-drug sensitivity estimates restricted to drugs with >10 resistant samples in the validation set

## Limitations

- Sensitivity for M. tuberculosis is low (82.6% across all drugs) compared with traditional DST, due to limited understanding of underlying genetic mechanisms
- Pyrazinamide is completely excluded since known mutations are poorly predictive
- Phenotyping data was imperfect and there was not enough data to estimate per-drug thresholds (a single 10% threshold was used for all drugs)
- Only 18%/44% of training/validation samples were phenotyped for quinolones or injectables, so minor-allele findings for second-line drugs need replication with larger data sets

## Evidence

- [abstract] Mykrobe vs KvarQ overall performance: "For M. tuberculosis, our method predicts resistance with sensitivity/specificity of 82.6%/98.5% (independent validation set, n=1,609)"
- [discussion] Comparison to line-probe assay performance: "For M. tuberculosis, specificity is high (98.5%) and sensitivity of 82.6% matches the line-probe assays from which our resistance panel was constructed"
- [discussion] False-negative rate restriction: "the number of samples with resistant phenotypes was limiting, and we only estimated false-negative rates where there were sufficiently many (>10) resistant samples"
- [readme] JSON format requirement: "--format 
        * Stores results a . file instead of a text file. Mykrobe Parser only works with  files."
- [readme] Species QC filter: "Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes."
