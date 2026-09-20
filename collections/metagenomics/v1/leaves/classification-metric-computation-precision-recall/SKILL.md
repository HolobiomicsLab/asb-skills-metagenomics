---
name: classification-metric-computation-precision-recall
description: Use when a new protein/sequence classifier must be benchmarked on a curated
  test set against baseline methods, and per-method and per-protein precision/recall/accuracy/FDR
  values are needed to demonstrate ity.
when_to_use_negative:
- The goal is to run DiSCo as a predictor on new genomes rather than to evaluate it
  — no curated ground-truth annotations are available, so TP/FP/TN/FN cannot be defined
- Input is raw (meta)genomic nucleotide data needing gene calling; this skill assumes
  protein FASTA inputs already produced
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0621
tools:
- name: DiSCo
  role: Classifier under test; emits filtered hitlists with classification and predicted
    enzyme type used to score against curated annotations
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: blastp 2.10.0
  role: 'Comparator baseline: simple best-hit blast searches on the same test set
    with identity ≥25% and E-value ≤10−10 thresholds'
- name: hmmsearch 3.1b2
  role: HMMER3 search engine executed by DiSCo.pl against the DiSCo HMM library to
    produce hits for classification
  repo: http://hmmer.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:87a75e9d4b1a910086ee017252cc061c8903e49905d7816f062947ef86d1b423
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - blastp
  - blastp 2.10.0
  - hmmsearch 3.1b2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# classification-metric-computation-precision-recall

## Summary

Benchmark a sequence-classification method such as DiSCo against comparator approaches (rBBH, best-hit blastp) by constructing TP/FP/TN/FN contingency matrices and computing precision, recall, accuracy, balanced accuracy and false discovery rate, including per-protein metrics for DsrC, DsrD, DsrEFH, DsrL and DsrO.

## When to use

Use when a new protein/sequence classifier must be benchmarked on a curated test set against baseline methods, and per-method and per-protein precision/recall/accuracy/FDR values are needed to demonstrate superiority.

## When NOT to use

- The goal is to run DiSCo as a predictor on new genomes rather than to evaluate it — no curated ground-truth annotations are available, so TP/FP/TN/FN cannot be defined
- Input is raw (meta)genomic nucleotide data needing gene calling; this skill assumes protein FASTA inputs already produced

## Inputs

- Per-genome protein FASTA files from the 1187-genome independent test set (Table S3)
- Curated protein-type annotations for the test set (ground truth labels)
- DiSCo filtered hitlists (tab-separated) from DiSCo.pl with default cutoffs
- blastp 2.10.0 best-hit and rBBH hit lists with identity ≥25% and E-value ≤10−10

## Outputs

- Contingency matrices (TP/FP/TN/FN) per classification method
- Benchmark table (e.g. benchmark_metrics.tsv) with PR, RC, AC, BA and FDR per method and per protein family (DsrC, DsrD, DsrEFH, DsrL, DsrO)

## How to apply

Run the classifier under test (DiSCo.pl -i per-genome FASTA, invoking hmmsearch 3.1b2 with DiSCo hits fulfilling hmmer conditional and independent E-value ≤10−10, accuracy ≥0.5, and bias one order of magnitude lower than score) and the comparators (blastp 2.10.0 best-hit and reciprocal best blast hit, using identity ≥25% and E-value ≤10−10 thresholds) on the same independent test set. Build contingency matrices (TP/FP/TN/FN) per method against the curated protein-type annotations, following the paper's definitions of positives and negatives. Compute precision (PR), recall (RC), accuracy (AC), balanced accuracy (BA) and false discovery rate (FDR) per method, and additionally per protein family (DsrC, DsrD, DsrEFH, DsrL, DsrO) to expose family-specific differences such as the DsrO FDR of 0.06 for DiSCo versus 0.8 for simple blast. Tabulate results (e.g. benchmark_metrics.tsv) and validate against the paper's reported benchmark, expecting minimal accuracy of 0.98 for complete genomes.

## Related tools

- **DiSCo** (Classifier under test; emits filtered hitlists with classification and predicted enzyme type used to score against curated annotations) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **blastp 2.10.0** (Comparator baseline: simple best-hit blast searches on the same test set with identity ≥25% and E-value ≤10−10 thresholds)
- **hmmsearch 3.1b2** (HMMER3 search engine executed by DiSCo.pl against the DiSCo HMM library to produce hits for classification) — http://hmmer.org/

## Examples

```
perl DiSCo.pl -i test_genome.faa -p test_genome -s 0 -d ./results/
```

## Evaluation signals

- Computed PR/RC/AC/BA/FDR values reproduce the paper's reported benchmark comparison for complete genomes and metagenomic records
- Minimal accuracy reaches 0.98 for complete genomes (0.93 for metagenomic records) as reported in the discussion
- Per-protein FDRs match the reported ordering, e.g. DsrO FDR 0.06 for DiSCo versus 0.6 for rBBH and 0.8 for simple blast
- DiSCo hitlists only contain hits fulfilling the hmmer cutoffs (conditional and independent E-value ≤10−10, accuracy ≥0.5, bias one order of magnitude lower than score)
- Contingency matrices sum to the number of curated test-set annotations per protein family and method

## Limitations

- Benchmarks depend on curated protein-type annotations; the discussion section notes no explicit benchmark dataset accessions are given for the 0.98/0.93 accuracy evaluation
- blast-based comparators can misassign paralogous families because some marker proteins (e.g. DsrEFH, ~100 amino acids) are small, limiting distinction between paralogous families
- Metrics were computed on the 1187-genome independent test set of complete genomes; metagenomic-record performance (0.93 minimal accuracy) is lower

## Evidence

- [results] DiSCo outperformed both rBBH and simple blast approaches in distinguishing true positives from false positives across the datasets, e.g. simple blast's false discovery rate for DsrO was 0.8 versus 0.6 for rBBH and 0.06 for DiSCo.: [evidence span withheld — anchor status: not_found]
- [discussion] with a minimal accuracy of 0.98 for complete genomes and 0.93 for metagenomic records: "with a minimal accuracy of 0.98 for complete genomes and 0.93 for metagenomic records"
- [intro] DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [intro] with an identity of ≥25% and an E-value of ≤10−10 as thresholds: [evidence span withheld — anchor status: not_found]
- [readme] This tool runs a set of FASTA formatted protein sequences against the curated set of hidden Markov models (HMMs).: "This tool runs a set of FASTA formatted protein sequences against the curated set of hidden Markov models (HMMs)."
