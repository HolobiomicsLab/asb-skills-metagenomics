---
name: confusion-matrix-metric-computation
description: Use when benchmarking a protein-type prediction method (e.g. DiSCo Dsr-dependent
  dissimilatory sulphur metabolism assignments) against baseline assignments on genomes
  with curated reference labels, to quantify precision, recall, accuracy, balanced
  accuracy and FDR.
when_to_use_negative:
- When no curated reference labels exist for the input genomes — the contingency matrix
  cannot be built without ground-truth assignments
- When the goal is de novo identification of sulphur-metabolism proteins rather than
  benchmarking a predictor against reference assignments
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0621
tools:
- name: DiSCo
  role: Predictor under evaluation; its filtered hitlist supplies the predictions
    scored in the contingency matrix
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: blastp 2.10.0
  role: Generates the rBBH and best-hit blast baseline assignments compared against
    DiSCo's metrics
- name: hmmsearch
  role: HMMER3 search engine invoked by DiSCo.pl to score proteins against the DiSCo
    HMM library prior to filtering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:b7264eba28376cd463609b970b4b08e46121d8e24075fae01973c342d47a744d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - blastp
  - blastp 2.10.0
  - hmmsearch
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# confusion-matrix-metric-computation

## Summary

Build a per-method contingency matrix from predicted versus curated reference labels and compute precision, recall, accuracy, balanced accuracy and false discovery rate to benchmark protein-function classifiers such as DiSCo against rBBH and best-hit blast baselines.

## When to use

Use when benchmarking a protein-type prediction method (e.g. DiSCo Dsr-dependent dissimilatory sulphur metabolism assignments) against baseline assignments on genomes with curated reference labels, to quantify precision, recall, accuracy, balanced accuracy and FDR.

## When NOT to use

- When no curated reference labels exist for the input genomes — the contingency matrix cannot be built without ground-truth assignments
- When the goal is de novo identification of sulphur-metabolism proteins rather than benchmarking a predictor against reference assignments

## Inputs

- Per-genome curated reference labels for Dsr-dependent dissimilatory sulphur metabolism proteins
- DiSCo filtered hitlist (per-genome DiSCo.pl output with sequence identifier, HMM name, predicted enzyme type, E-value, accuracy)
- rBBH and best-hit blast assignment lists derived from blastp 2.10.0 runs on the same genomes
- Contingency matrix definition with paper-specific true/false positive classes

## Outputs

- Per-method per-genome contingency matrix (true positives, false positives, true negatives, false negatives)
- Precision (PR), recall (RC), accuracy (AC), balanced accuracy (BA) and false discovery rate (FDR) per method
- Comparison table of metrics for DiSCo, rBBH and best-hit blast mirroring Table S7

## How to apply

For each genome, treat the curated reference labels as ground truth and each method's predictions (DiSCo filtered hitlist, rBBH, best-hit blast) as predictions, applying the paper's true/false positive definitions to fill the contingency matrix. Run DiSCo.pl against the DiSCo HMM library via hmmsearch, retaining only hits that pass the hmmer-specific conditional and independent E-value ≤10−10, accuracy ≥0.5, and bias one order of magnitude lower than score, with the best-hit assignment strategy. Derive the blast baselines with blastp 2.10.0 using identity ≥25% and E-value ≤10−10 thresholds. From each contingency matrix compute PR (precision), RC (recall), AC (accuracy), BA (balanced accuracy) and FDR per method and assemble a comparison table mirroring Table S7. The rationale is that metric-based benchmarking distinguishes true positives from false positives, validating that DiSCo attains the reported minimal accuracy of 0.98 on complete genomes.

## Related tools

- **DiSCo** (Predictor under evaluation; its filtered hitlist supplies the predictions scored in the contingency matrix) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **blastp 2.10.0** (Generates the rBBH and best-hit blast baseline assignments compared against DiSCo's metrics)
- **hmmsearch** (HMMER3 search engine invoked by DiSCo.pl to score proteins against the DiSCo HMM library prior to filtering)

## Examples

```
perl DiSCo.pl -i genome.faa -p genome_hits -n 4
```

## Evaluation signals

- Reproduced DiSCo metrics match Table S7 within tolerance across both datasets
- DiSCo attains the reported minimal accuracy of 0.98 for complete genomes (0.93 for metagenomic records benchmarked against genomic records)
- DiSCo outperforms rBBH and best-hit blast in the distinction of true positives over false positives in the contingency matrix
- All scored DiSCo hits satisfy the filter cutoffs: conditional and independent E-value ≤10−10, accuracy ≥0.5, bias one order of magnitude lower than score
- Blast baselines use the stated thresholds (identity ≥25%, E-value ≤10−10) before metric computation

## Limitations

- Evaluation depends on curated reference labels; the discussion notes no explicit benchmark dataset accessions are given for the 0.98/0.93 accuracy evaluation
- Phylogeny- or marker-based alternatives can produce incorrect assignments due to micro-organisms with a reductive-type Dsr-system that are in vivo dSDM, complicating ground-truth definition
- Benchmark is specific to Dsr-dependent dissimilatory sulphur metabolism; metrics do not transfer directly to other metabolic prediction tasks

## Evidence

- [intro] DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [discussion] minimal accuracy for complete genomes: "with a minimal accuracy of 0.98 for complete genomes and 0.93 for metagenomic records"
- [intro] blastp thresholds: "mapped to their respective genome using blastp (version 2.10.0, default parameters) [75], with an identity of ≥25"
- [readme] DiSCo run command: "perl DiSCo.pl -i example.faa  #runs DiSCo with default settings"
