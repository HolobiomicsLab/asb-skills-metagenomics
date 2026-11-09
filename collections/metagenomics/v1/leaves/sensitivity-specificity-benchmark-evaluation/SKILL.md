---
name: sensitivity-specificity-benchmark-evaluation
description: Use when you have per-isolate resistant/susceptible predictions plus
  phenotypic DST labels on an independent validation set and need overall and per-drug
  sensitivity/specificity, e.g. reproducing Mykrobe's 82.6%/98.5% MTBC_B result.
when_to_use_negative:
- You lack phenotypic DST ground truth for the isolates, so no confusion matrix against
  a reference can be built
- A drug has ≤10 resistant samples in the validation set — the paper only estimated
  false-negative rates where there were sufficiently many (>10) resistant samples
- 'Pyrazinamide evaluation: the paper completely excluded pyrazinamide since known
  mutations are poorly predictive, so benchmarks should not include it'
edam_operation: http://edamontology.org/operation_3482
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_2815
tools:
- name: Mykrobe predictor
  role: Generates per-sample JSON resistance predictions from raw sequence data for
    each validation isolate, which are converted into per-drug calls for the confusion
    matrices
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe Parser
  role: R script that parses Mykrobe's M. tuberculosis-specific JSON results into
    tidy per-genetic-region prediction tables used for metric computation; removes
    non-tuberculous mycobacteria detections for QC
  repo: https://github.com/phac-nml/mykrobe-parser
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:aaf4b020bba0f96e62ac417f2b133c68a0250743f10c45e873247821cae63c46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe Parser
  - Mykrobe predictor
  edam_operation: http://edamontology.org/operation_3482
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_2815
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# sensitivity-specificity-benchmark-evaluation

## Summary

Benchmark a genotype-based resistance predictor by tabulating per-drug predictions against phenotypic DST labels in confusion matrices and computing overall and per-drug sensitivity/specificity, as done for Mykrobe predictor on the 1,609-sample MTBC_B M. tuberculosis validation set (82.6%/98.5%) with a side-by-side KvarQ comparison.

## When to use

Use when you have per-isolate resistant/susceptible predictions plus phenotypic DST labels on an independent validation set and need overall and per-drug sensitivity/specificity, e.g. reproducing Mykrobe's 82.6%/98.5% MTBC_B result.

## When NOT to use

- You lack phenotypic DST ground truth for the isolates, so no confusion matrix against a reference can be built
- A drug has ≤10 resistant samples in the validation set — the paper only estimated false-negative rates where there were sufficiently many (>10) resistant samples
- Pyrazinamide evaluation: the paper completely excluded pyrazinamide since known mutations are poorly predictive, so benchmarks should not include it

## Inputs

- Per-isolate sequencing reads (FASTQ) for the 1,609 MTBC_B validation isolates from ENA/NCBI SRA
- Phenotypic DST resistance labels per isolate and drug (Supplementary Data accessions)
- Mykrobe predictor JSON output files per sample
- KvarQ per-drug sensitivity values for comparison (90.8% rifampicin, 83.2% isoniazid, 76.3% ethambutol)

## Outputs

- Per-drug confusion matrices (predicted vs phenotypic DST)
- Overall sensitivity/specificity across all drugs (expected 82.6%/98.5%)
- Per-drug sensitivities for first-line drugs (rifampicin 93.7%, isoniazid 84.3%, ethambutol 71.6%)
- validation_metrics.csv with overall, per-drug and KvarQ-comparison metrics

## How to apply

Run the predictor (Mykrobe predictor with --format  and the M. tuberculosis resistance panel) on each of the 1,609 MTBC_B validation isolates, then apply the paper's QC filters: remove samples in which Mykrobe detects non-tuberculous mycobacteria and exclude samples exceeding the 40-confident-heterozygous-SNP contamination threshold. Convert each JSON prediction into per-drug resistant/susceptible calls and tabulate them against the phenotypic DST labels from Supplementary Data in per-drug confusion matrices. Compute overall sensitivity/specificity across all drugs (target 82.6%/98.5%) and per-drug sensitivities for rifampicin (93.7%), isoniazid (84.3%) and ethambutol (71.6%), restricting false-negative rate estimates to drugs with >10 resistant samples. For the comparator, record KvarQ's per-drug sensitivities (90.8%, 83.2%, 76.3%) side by side and save all metrics to validation_metrics.csv.

## Related tools

- **Mykrobe predictor** (Generates per-sample JSON resistance predictions from raw sequence data for each validation isolate, which are converted into per-drug calls for the confusion matrices) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe Parser** (R script that parses Mykrobe's M. tuberculosis-specific JSON results into tidy per-genetic-region prediction tables used for metric computation; removes non-tuberculous mycobacteria detections for QC) — https://github.com/phac-nml/mykrobe-parser

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out. --format  --seq reads.fq.gz
```

## Evaluation signals

- Overall sensitivity/specificity on the n=1,609 validation set equals the reported 82.6%/98.5%
- Per-drug sensitivities match 93.7% (rifampicin), 84.3% (isoniazid) and 71.6% (ethambutol)
- QC removes samples detected as non-tuberculous mycobacteria and those exceeding the 40-heterozygous-SNP contamination threshold (paper removed 20 contaminated samples from the validation set)
- False-negative rate estimates are restricted to drugs with >10 resistant samples in the validation set
- KvarQ comparator values recorded side by side: 90.8%, 83.2%, 76.3% for rifampicin, isoniazid, ethambutol

## Limitations

- M. tuberculosis sensitivity is low (82.6%) compared with traditional DST and the panel completely excludes pyrazinamide since known mutations are poorly predictive
- Number of samples with resistant phenotypes was limiting; false-negative rates were only estimated where >10 resistant samples existed
- Not enough M. tuberculosis data to estimate per-drug thresholds — a single 10% threshold was used for all drugs
- Specificity of 98.5% matches line-probe assays but is below gold-standard DST performance

## Evidence

- [abstract] MTBC_B validation performance: "For M. tuberculosis, our method predicts resistance with sensitivity/specificity of 82.6%/98.5% (independent validation set, n=1,609)"
- [discussion] contamination threshold: "We used this threshold of 40 heterozygous sites to determine whether a sample was contaminated by an unrelated strain"
- [discussion] per-drug threshold limitation: "For M. tuberculosis, we did not have enough data to estimate per-drug thresholds, and we knew that the phenotyping data was imperfect"
- [readme] QC removal of NTM detections: "Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes."
- [readme] JSON format requirement: "--format 
        * Stores results a . file instead of a text file. Mykrobe Parser only works with  files."
