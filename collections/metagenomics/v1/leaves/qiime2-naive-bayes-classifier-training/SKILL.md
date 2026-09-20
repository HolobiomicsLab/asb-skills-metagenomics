---
name: qiime2-naive-bayes-classifier-training
description: Use when benchmarking 16S rRNA taxonomic classifiers on a common reference
  database (e.g. Greengenes2 2024.09 FASTA) and a held-out test set, to obtain a naive-Bayes
  baseline comparable to DADA2, SINTAX, and Kraken 2.
when_to_use_negative:
- The goal is training or applying DeepTaxa itself rather than establishing a classical
  naive-Bayes baseline
- Input sequences are V3–V4/V4 amplicons from a different reference database (e.g.
  SILVA or RDP) that the Greengenes2 2024.09-trained classifier does not cover
- You need alignment-based or k-mer classification (Kraken 2, SINTAX, BLAST) rather
  than QIIME 2's sklearn naive Bayesian plugin
edam_operation: http://edamontology.org/operation_0463
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: QIIME 2
  role: Hosts the q2-feature-classifier sklearn naive Bayesian plugin that is trained
    on Greengenes2 2024.09 reference sequences and used to classify the shared test
    set (v2024.5.0; species accuracy 85.01%)
- name: DeepTaxa
  role: Hybrid CNN-BERT classifier against which the QIIME 2 baseline is compared
    on the same 69,335-sequence test set
  repo: https://github.com/systems-genomics-lab/deeptaxa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:87b6e373deddf0546ee3dc001155d1b2734e3832b081f93d0c4d8305f39e794c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DeepTaxa
  - Kraken 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_0463
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# qiime2-naive-bayes-classifier-training

## Summary

Trains a QIIME 2 (v2024.5.0) q2-feature-classifier sklearn naive Bayesian classifier on Greengenes2 2024.09 reference sequences and uses it to benchmark species-level 16S rRNA taxonomy assignment, which achieved 85.01% species accuracy on the shared 69,335-sequence held-out test set in the DeepTaxa study.

## When to use

Use when benchmarking 16S rRNA taxonomic classifiers on a common reference database (e.g. Greengenes2 2024.09 FASTA) and a held-out test set, to obtain a naive-Bayes baseline comparable to DADA2, SINTAX, and Kraken 2.

## When NOT to use

- The goal is training or applying DeepTaxa itself rather than establishing a classical naive-Bayes baseline
- Input sequences are V3–V4/V4 amplicons from a different reference database (e.g. SILVA or RDP) that the Greengenes2 2024.09-trained classifier does not cover
- You need alignment-based or k-mer classification (Kraken 2, SINTAX, BLAST) rather than QIIME 2's sklearn naive Bayesian plugin

## Inputs

- Greengenes2 2024.09 training/reference FASTA (reference sequences for classifier training)
- Held-out test FASTA of 69,335 full-length Greengenes2 2024.09 16S rRNA sequences
- Greengenes2 2024.09 ground-truth taxonomy labels for the test set

## Outputs

- Trained QIIME 2 naive Bayesian classifier artifact (.qza)
- Per-sequence taxonomy assignments on the 69,335-sequence test set
- Per-rank top-1 accuracy table (species-level benchmark accuracy ~85.01%) and wall-clock classification time, e.g. in benchmark_species_accuracy.csv

## How to apply

Train the q2-feature-classifier sklearn naive Bayesian plugin on the Greengenes2 2024.09 reference sequences (QIIME 2 version 2024.5.0), producing a trained classifier artifact. Classify the full-length held-out test FASTA (69,335 sequences from Greengenes2 2024.09) with the trained classifier, then compute per-rank top-1 accuracy on the same shared test set used by the other tools so results are directly comparable. Record wall-clock classification time alongside accuracy. Validation compares the resulting species-level accuracy against the reported 85.01%, allowing only a small reproduction tolerance around that figure.

## Related tools

- **QIIME 2** (Hosts the q2-feature-classifier sklearn naive Bayesian plugin that is trained on Greengenes2 2024.09 reference sequences and used to classify the shared test set (v2024.5.0; species accuracy 85.01%))
- **DeepTaxa** (Hybrid CNN-BERT classifier against which the QIIME 2 baseline is compared on the same 69,335-sequence test set) — https://github.com/systems-genomics-lab/deeptaxa

## Examples

```
qiime feature-classifier fit-classifier-naive-bayes --i-reference-reads gg_2024_09_training.fna --o-classifier gg2_2024_09_nb_classifier.qza
```

## Evaluation signals

- Species-level top-1 accuracy on the 69,335-sequence Greengenes2 2024.09 test set reproduces 85.01% within a small tolerance
- Classifier is trained exclusively on the Greengenes2 2024.09 reference FASTA and evaluated on the same held-out test set used by all four benchmark tools
- Per-rank accuracies are tabulated (e.g. benchmark_species_accuracy.csv) covering DADA2, QIIME 2, SINTAX, and Kraken 2, with Kraken 2 lowest of the four
- Wall-clock classification time is recorded for the QIIME 2 run
- QIIME 2 runs with version 2024.5.0 and the q2-feature-classifier sklearn naive Bayesian plugin

## Limitations

- The naive Bayesian baseline (85.01%) underperforms DeepTaxa's 92.96% species-level accuracy on the same held-out test set
- Results are specific to the Greengenes2 2024.09 full-length 16S setting; performance across other reference databases such as SILVA and RDP was not tested
- Classification is reference-based and can only detect species already represented in the reference catalog

## Evidence

- [methods] QIIME 2 naive Bayesian training and benchmark accuracy: [evidence span withheld — anchor status: not_found]
- [abstract] Baseline comparison against DeepTaxa: "DeepTaxa exceeds DADA2 (90.05%) and QIIME 2 (85.01%) at the species rank on the same held-out test set"
- [methods] Benchmarking protocol on the shared test set: "we conducted a benchmarking study comparing its performance against widely used taxonomic classification tools, including DADA2 (Callahan et al. 2016), QIIME 2, Kraken2, and SINTAX"
- [methods] Shared held-out test set size: "The dataset was partitioned into training (277 336 sequences, 80%) and test (69 335 sequences, 20%) sets using stratified sampling to preserve label distributions."
