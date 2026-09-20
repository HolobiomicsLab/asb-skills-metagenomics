---
name: contingency-matrix-construction
description: Use when benchmarking a protein-type predictor (e.g. DiSCo HMM hits)
  against comparator similarity-search methods on a curated test set and you need
  TP/FP/TN/FN matrices and derived PR/RC/AC/BA/FDR metrics.
when_to_use_negative:
- Input is already a labelled feature/prediction table — go straight to metric computation
  instead of re-running similarity searches
- Goal is de novo discovery of Dsr-pathway organisms in new (meta)genomes rather than
  benchmarking against curated annotations
- No curated ground-truth protein-type annotations exist for the test genomes, so
  TP/FP labels cannot be defined
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: DiSCo
  role: 'Method under test: hmmsearch 3.1b2 against the DiSCo HMM library with the
    filter script''s best-hit strategy produces the predictions scored in the contingency
    matrices'
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: blastp 2.10.0
  role: 'Comparator method: simple best-hit blast searches on the same test set, hits
    filtered at identity ≥25% and E-value ≤10−10'
- name: hmmsearch 3.1b2
  role: Search engine invoked by DiSCo.pl against the DiSCo HMM library to generate
    candidate hits for filtering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:4986e8c4301f8fb528827e51146ca7a341c5b69efe18619e5c873ad6b0f1e17a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - blastp
  - blastp 2.10.0
  - hmmsearch 3.1b2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# contingency-matrix-construction

## Summary

Constructs TP/FP/TN/FN contingency matrices for sequence-based protein-type predictions (DiSCo vs. blast best-hit vs. rBBH) against curated Dsr-protein annotations, then derives precision, recall, accuracy, balanced accuracy and false discovery rate per method and per protein (DsrC, DsrD, DsrEFH, DsrL, DsrO).

## When to use

Use when benchmarking a protein-type predictor (e.g. DiSCo HMM hits) against comparator similarity-search methods on a curated test set and you need TP/FP/TN/FN matrices and derived PR/RC/AC/BA/FDR metrics.

## When NOT to use

- Input is already a labelled feature/prediction table — go straight to metric computation instead of re-running similarity searches
- Goal is de novo discovery of Dsr-pathway organisms in new (meta)genomes rather than benchmarking against curated annotations
- No curated ground-truth protein-type annotations exist for the test genomes, so TP/FP labels cannot be defined

## Inputs

- Per-genome FASTA protein files of the 1187-genome independent test set (Table S3)
- Curated protein-type annotations (ground truth) for DsrC, DsrD, DsrEFH, DsrL, DsrO
- DiSCo.pl filtered hitlists (tab-separated) with E-value, score, bias, accuracy columns
- blastp best-hit and rBBH hit lists filtered at identity ≥25% and E-value ≤10−10

## Outputs

- Contingency matrices (TP/FP/TN/FN) per method against curated annotations
- benchmark_metrics.tsv with precision, recall, accuracy, balanced accuracy and FDR per method and per protein type

## How to apply

Run each method on the same 1187-genome independent test set per-genome FASTA proteomes: DiSCo.pl (-i) with hmmsearch 3.1b2 enforcing the conditional and independent E-value ≤10−10, accuracy ≥0.5 and bias one order of magnitude lower than score, plus the filter script's best-hit strategy; blastp 2.10.0 best-hit and reciprocal best blast hit (rBBH) with identity ≥25% and E-value ≤10−10. Map each method's predictions to the curated protein-type annotations and build a contingency matrix (TP/FP/TN/FN) per method following the paper's TP/FP definitions. Compute precision, recall, accuracy, balanced accuracy and false discovery rate per method, and per-protein metrics for DsrC, DsrD, DsrEFH, DsrL and DsrO, writing results to benchmark_metrics.tsv. Validate by checking that the numbers reproduce the reported benchmark pattern (e.g. DsrO FDR of 0.06 for DiSCo versus 0.6 rBBH and 0.8 blast) and the minimal accuracy of 0.98 for complete genomes.

## Related tools

- **DiSCo** (Method under test: hmmsearch 3.1b2 against the DiSCo HMM library with the filter script's best-hit strategy produces the predictions scored in the contingency matrices) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **blastp 2.10.0** (Comparator method: simple best-hit blast searches on the same test set, hits filtered at identity ≥25% and E-value ≤10−10)
- **hmmsearch 3.1b2** (Search engine invoked by DiSCo.pl against the DiSCo HMM library to generate candidate hits for filtering)

## Examples

```
perl DiSCo.pl -i test_genome.faa -s 0
```

## Evaluation signals

- DiSCo must show lower FDR than both comparators, e.g. DsrO FDR ≈0.06 vs 0.6 (rBBH) and 0.8 (simple blast)
- Minimal accuracy of 0.98 for complete genomes (0.93 for metagenomic records) must be reproduced
- All DiSCo hits entering the matrix must pass conditional and independent E-value ≤10−10, accuracy ≥0.5, and bias one order of magnitude lower than score
- All blast/rBBH hits entering the matrix must satisfy identity ≥25% and E-value ≤10−10
- Per-method TP+FN counts must equal the curated annotation counts per protein type (contingency consistency check)

## Limitations

- Validation requires the curated protein-type annotations of the 1187-genome test set (Table S3); without them TP/FP labels are undefined
- The 0.98/0.93 accuracy benchmark lacks explicit named dataset accessions in the discussion section, so exact dataset provenance must be traced via supplementary materials
- DiSCo's classification depends on model-specific user-defined cut-offs (model, score, E-value table), which affect the resulting contingency counts

## Evidence

- [intro] DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [intro] blast hit thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] benchmark accuracy: "with a minimal accuracy of 0.98 for complete genomes and 0.93 for metagenomic records"
- [readme] DiSCo.pl usage: "This tool runs a set of FASTA formatted protein sequences against the curated set of hidden Markov models (HMMs)."
