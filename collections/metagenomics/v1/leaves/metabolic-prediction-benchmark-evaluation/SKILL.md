---
name: metabolic-prediction-benchmark-evaluation
description: Use when a sequence-based metabolic assignment tool must be validated
  against reference-labelled genomes, and comparative performance versus blast/reciprocal--hit
  baselines needs to be quantified.
when_to_use_negative:
- Input is raw (meta)genomic sequence needing assembly or gene calling rather than
  protein FASTA assignment
- The task is de novo discovery of pathway diversity rather than evaluating a predictor's
  classification performance against curated labels
- Metagenomic records lacking curated reference labels, since the 0.98 accuracy figure
  applies specifically to the complete-genomes benchmark
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0654
tools:
- name: DiSCo
  role: HMM-based predictor under evaluation; runs FASTA protein input against the
    DiSCo HMM library and produces the classified hitlist
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: hmmsearch 3.1b2
  role: HMMER3 search engine invoked by DiSCo.pl to score input proteins against the
    curated HMM models
- name: blastp 2.10.0
  role: Generates the rBBH and best-hit blast baseline assignments (identity ≥25%,
    E-value ≤10−10) used for comparison
- name: Perl 5
  role: Implementation language of DiSCo.pl and the filter script applying the model-specific
    cutoffs and best-hit strategy
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:7868bd466f4e0f2d047452bf844e07ce3f9b7458443d297ef6802bda06b49b0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - Perl 5
  - blastp
  - blastp 2.10.0
  - hmmsearch 3.1b2
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabolic-prediction-benchmark-evaluation

## Summary

Benchmark a metabolic functional predictor (e.g. DiSCo) against rBBH and best-hit blast baselines by building per-genome contingency matrices and computing precision, recall, accuracy, balanced accuracy and false discovery rate against curated reference labels.

## When to use

Use when a sequence-based metabolic assignment tool must be validated against reference-labelled genomes, and comparative performance versus blast/reciprocal-best-hit baselines needs to be quantified.

## When NOT to use

- Input is raw (meta)genomic sequence needing assembly or gene calling rather than protein FASTA assignment
- The task is de novo discovery of pathway diversity rather than evaluating a predictor's classification performance against curated labels
- Metagenomic records lacking curated reference labels, since the 0.98 accuracy figure applies specifically to the complete-genomes benchmark

## Inputs

- Complete genomic protein FASTA sets for 4825 bacteria and 253 archaea NCBI assemblies (Table S2)
- Curated reference labels for Dsr-dependent dissimilatory sulphur metabolism proteins
- DiSCo HMM library and filter script
- blastp 2.10.0 all-versus-all hit results for rBBH and best-hit baselines

## Outputs

- Filtered DiSCo hitlist per genome with protein family, predicted enzyme type and DsrC-type notes
- Per-genome contingency matrices for DiSCo, rBBH and best-hit blast
- Comparison table of precision, recall, accuracy, balanced accuracy and FDR per method (mirroring Table S7)

## How to apply

Download the complete genomic protein sets for the 4825 bacteria and 253 archaea NCBI assemblies and run DiSCo.pl (-i <protein FASTA>) against the DiSCo HMM library, keeping hits that pass the hmmer-specific conditional and independent E-value ≤10−10, accuracy ≥0.5, and bias cutoff, with the best-hit assignment strategy producing the filtered hitlist. Run blastp 2.10.0 (default parameters, identity ≥25%, E-value ≤10−10) on the same dataset to derive the reciprocal best blast hit (rBBH) and best-hit blast reference assignments. Build a contingency matrix per genome by comparing each method's predictions against the curated reference labels using the paper's true/false positive definitions, then compute PR, RC, AC, BA and FDR for all three methods. Assemble the metrics into a single comparison table mirroring Table S7; success is judged by DiSCo matching the reported minimal accuracy of 0.98 for complete genomes (0.93 for metagenomic records) and outperforming both blast-based baselines in the distinction of true positives over false positives.

## Related tools

- **DiSCo** (HMM-based predictor under evaluation; runs FASTA protein input against the DiSCo HMM library and produces the classified hitlist) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **hmmsearch 3.1b2** (HMMER3 search engine invoked by DiSCo.pl to score input proteins against the curated HMM models)
- **blastp 2.10.0** (Generates the rBBH and best-hit blast baseline assignments (identity ≥25%, E-value ≤10−10) used for comparison)
- **Perl 5** (Implementation language of DiSCo.pl and the filter script applying the model-specific cutoffs and best-hit strategy) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo

## Examples

```
perl DiSCo.pl -i genome.faa -p genome -t thresholds.txt
```

## Evaluation signals

- Reproduced metrics match Table S7 within tolerance for DiSCo, rBBH and best-hit blast
- DiSCo attains the reported minimal accuracy of 0.98 for complete genomes (and 0.93 for metagenomic records benchmarked against genomic records)
- All accepted DiSCo hits fulfil the hmmer-specific conditional and independent E-value ≤10−10, accuracy ≥0.5, and bias-cutoff thresholds
- DiSCo outperforms both baselines in the distinction of true positives over false positives on both datasets
- Contingency matrices are built consistently per genome using the paper's true/false positive definitions

## Limitations

- Benchmark performance figures depend on curated reference labels; no explicit benchmark dataset accessions beyond NCBI assemblies are provided for the 0.98/0.93 accuracy evaluation
- Distinction between paralogous families is hampered by the small size (~100 amino acids) of some marker proteins such as DsrEFH
- Single-marker approaches (e.g. DsrC-type assignment alone) can lead to incorrect assignments
- No software version number is stated for DiSCo in the discussion/repository context

## Evidence

- [intro] Publicly available complete genomic assemblies of 4825 bacteria and 253 archaea were downloaded from NCBI: "Publicly available complete genomic assemblies of 4825 bacteria and 253 archaea were downloaded from NCBI"
- [intro] DiSCo hit cutoffs: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [discussion] Reported benchmark accuracies: "with a minimal accuracy of 0.98 for complete genomes and 0.93 for metagenomic records"
- [methods] DiSCo.pl first step: [evidence span withheld — anchor status: not_found]
- [readme] README usage example: "perl DiSCo.pl -i example.faa  #runs DiSCo with default settings"
