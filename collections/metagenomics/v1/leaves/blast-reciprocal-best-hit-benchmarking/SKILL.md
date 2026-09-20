---
name: blast-reciprocal-best-hit-benchmarking
description: Use when evaluating whether an HMM-based protein assignment tool (e.g.
  DiSCo on Dsr proteins) outperforms blast-based baselines on curated genome or metagenome
  test sets.
when_to_use_negative:
- When the goal is de novo discovery or classification of sulphur-metabolism proteins
  rather than benchmarking a predictor against blast baselines.
- When no curated reference labels exist, since contingency matrices and precision/recall
  cannot be computed without ground truth.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_0080
tools:
- name: DiSCo
  role: HMM-based classifier being benchmarked; run as DiSCo.pl -i <protein FASTA>
    against the DiSCo HMM library
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: blastp 2.10.0
  role: Baseline method; used with default parameters and identity ≥25%, E-value ≤10−10
    thresholds to derive rBBH and best-hit blast assignments
- name: hmmsearch 3.1b2
  role: Sequence search engine invoked by DiSCo.pl against the DiSCo HMM library
  repo: http://hmmer.org/
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:6fce6c80db4243c4c136cec5ad4ae8b95f0431b3de3985b6fb2c1a9ee43eeb4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - blastp
  - blastp 2.10.0
  - hmmsearch 3.1b2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0623
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# blast-reciprocal-best-hit-benchmarking

## Summary

Benchmark a protein classifier such as DiSCo against reciprocal best blast hit (rBBH) and simple best-hit blast baselines by computing precision, recall, accuracy, balanced accuracy and false discovery rate from per-genome contingency matrices.

## When to use

Use when evaluating whether an HMM-based protein assignment tool (e.g. DiSCo on Dsr proteins) outperforms blast-based baselines on curated genome or metagenome test sets.

## When NOT to use

- When the goal is de novo discovery or classification of sulphur-metabolism proteins rather than benchmarking a predictor against blast baselines.
- When no curated reference labels exist, since contingency matrices and precision/recall cannot be computed without ground truth.

## Inputs

- Per-genome protein FASTA files from 4825 complete bacterial and 253 archaeal NCBI assemblies (complete genomes dataset, Table S2)
- 1187-genome independent test set protein FASTA files (Table S3)
- Curated reference annotations of protein types (ground-truth labels)

## Outputs

- Filtered DiSCo hitlists (tab-separated) per genome
- rBBH and best-hit blast assignment tables
- Contingency matrices (TP/FP/TN/FN) per method per genome
- Comparison table of PR, RC, AC, BA, FDR per method (mirroring Table S7), optionally written to benchmark_metrics.tsv

## How to apply

Run each method on the same per-genome protein FASTA sets: DiSCo.pl via hmmsearch with hmmer-specific conditional and independent E-value ≤10−10, accuracy ≥0.5 and bias one order of magnitude lower than score, plus best-hit assignment; and blastp 2.10.0 with identity ≥25% and E-value ≤10−10 thresholds to derive rBBH and best-hit blast reference assignments. For each genome, build a contingency matrix (TP/FP/TN/FN) comparing method predictions to curated reference labels using the paper's true/false positive definitions. From these compute precision (PR), recall (RC), accuracy (AC), balanced accuracy (BA) and false discovery rate (FDR) per method and per protein type (DsrC, DsrD, DsrEFH, DsrL, DsrO). Assemble the metrics into a comparison table mirroring Table S7 and validate against reported values, e.g. DiSCo's minimal accuracy of 0.98 for complete genomes.

## Related tools

- **DiSCo** (HMM-based classifier being benchmarked; run as DiSCo.pl -i <protein FASTA> against the DiSCo HMM library) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **blastp 2.10.0** (Baseline method; used with default parameters and identity ≥25%, E-value ≤10−10 thresholds to derive rBBH and best-hit blast assignments)
- **hmmsearch 3.1b2** (Sequence search engine invoked by DiSCo.pl against the DiSCo HMM library) — http://hmmer.org/

## Examples

```
perl DiSCo.pl -i example.faa && blastp -query example.faa -db ref_proteomes -outfmt 6 -evalue 1e-10
```

## Evaluation signals

- Reproduced metrics match Table S7 within tolerance for DiSCo, rBBH and best-hit blast.
- DiSCo attains the reported minimal accuracy of 0.98 on complete genomes (0.93 for metagenomic records when benchmarked against genomic records).
- Per-protein false discovery rates reproduce paper values, e.g. DsrO FDR of 0.06 for DiSCo versus 0.6 for rBBH and 0.8 for simple blast.
- Contingency matrices sum consistently: TP+FP+TN+FN equals the number of curated reference proteins per genome.
- All blast hits used respect the identity ≥25% and E-value ≤10−10 thresholds; all DiSCo hits respect E-value ≤10−10, accuracy ≥0.5 and bias cutoffs.

## Limitations

- Blast-based baselines such as simple best-hit blast show high false discovery rates (e.g. 0.8 for DsrO), so the comparison is only meaningful with curated ground truth.
- The distinction between paralogous families is hampered by the small size (~100 amino acids) of some marker proteins like DsrEFH, which can complicate labeling.
- Model thresholds and curated references are tied to known lineages using the Dsr-dependent dissimilatory sulphur pathway; results may not generalize outside this protein families' scope.

## Evidence

- [results] DiSCo outperformed both the rBBH and simple blast approaches in the distinction of true positives over false positives across both datasets: [evidence span withheld — anchor status: not_found]
- [intro] blast thresholds: [evidence span withheld — anchor status: not_found]
- [intro] DiSCo hit filters: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [discussion] minimal accuracy: "with a minimal accuracy of 0.98 for complete genomes and 0.93 for metagenomic records"
- [readme] README usage: "This tool runs a set of FASTA formatted protein sequences against the curated set of hidden Markov models (HMMs)."
