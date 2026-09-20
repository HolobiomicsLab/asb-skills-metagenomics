---
name: species-level-f1-computation
description: Use when evaluating a taxonomic classifier (e.g., DeepTaxa, DADA2, QIIME
  2, SINTAX, Kraken 2) on labeled 16S rRNA test sequences and you need species-rank
  F1, especially stratified by similarity (>97%, 90–97%, <90%) to the training set.
when_to_use_negative:
- Sequences lack ground-truth species labels (F1 requires reference taxonomy, e.g.,
  gg_2024_09_testing.tsv.gz)
- The analysis goal is genus- or family-rank evaluation only — this skill targets
  the species rank, where degradation from 0.95 to 0.45 F1 is most pronounced
- Test sequences are not from the same reference framework as the training pool (e.g.,
  SILVA or RDP sequences), since similarity bucketing against a Greengenes2 training
  set would not be meaningful
edam_operation: http://edamontology.org/operation_0478
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: DeepTaxa
  role: Generates per-sequence species-level predictions on the test FASTA that are
    scored with F1
  repo: github.com/systems-genomics-lab/deeptaxa
- name: BLAST
  role: Aligns test sequences against the training FASTA to find nearest-neighbor
    percent identity for bucketing
- name: VSEARCH
  role: Alternative to BLAST for nearest-neighbor identity search against the training
    set
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:8e594ab6f685b160228dc0af6612129109c9d4dbb05c9f6cf88eb272dad874be
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - DeepTaxa
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0478
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# species-level-f1-computation

## Summary

Computes species-level F1-score for 16S rRNA taxonomic classifiers, either globally on the held-out Greengenes2 2024.09 test set or stratified by nearest-neighbor sequence identity buckets, to quantify how classification performance degrades with decreasing similarity to the training pool.

## When to use

Use when evaluating a taxonomic classifier (e.g., DeepTaxa, DADA2, QIIME 2, SINTAX, Kraken 2) on labeled 16S rRNA test sequences and you need species-rank F1, especially stratified by similarity (>97%, 90–97%, <90%) to the training set.

## When NOT to use

- Sequences lack ground-truth species labels (F1 requires reference taxonomy, e.g., gg_2024_09_testing.tsv.gz)
- The analysis goal is genus- or family-rank evaluation only — this skill targets the species rank, where degradation from 0.95 to 0.45 F1 is most pronounced
- Test sequences are not from the same reference framework as the training pool (e.g., SILVA or RDP sequences), since similarity bucketing against a Greengenes2 training set would not be meaningful

## Inputs

- Greengenes2 2024.09 training FASTA (gg_2024_09_training.fna.gz, 277,336 sequences)
- Greengenes2 2024.09 test FASTA (gg_2024_09_testing.fna.gz, 69,335 sequences)
- Greengenes2 2024.09 test taxonomy labels (gg_2024_09_testing.tsv.gz)
- Per-sequence species-level predictions from a taxonomic classifier (e.g., DeepTaxa output directory)
- BLAST or VSEARCH nearest-neighbor percent-identity assignments for test sequences

## Outputs

- Global species-level F1-score (and optionally accuracy/ECE) on the held-out test set
- Bucket-level weighted species F1 values with sequence counts per similarity bucket (>97%, 90–97%, <90%)
- Summary table similarity_stratified_f1.csv

## How to apply

Load the Greengenes2 2024.09 training (277,336 sequences) and test (69,335 sequences) FASTA files from the Hugging Face systems-genomics-lab/greengenes dataset, plus per-sequence species predictions from the classifier (produced, e.g., by `deeptaxa predict` with `--taxonomy-file` labels). To stratify, align every test sequence against the training set with BLAST or VSEARCH to find the nearest training neighbor and record its percent nucleotide identity; assign each test sequence to one of three buckets: >97%, 90–97%, or <90% identity. Join bucket assignments with the per-sequence species predictions and compute weighted species-level F1 per bucket using scikit-learn/pandas. Assemble bucket-level F1 values with sequence counts into a summary table (similarity_stratified_f1.csv). The rationale is that global F1 (e.g., DeepTaxa's 0.9212 species F1 on Greengenes2) masks smooth degradation in performance on sequences dissimilar to training data (species F1 from ~0.95 in the >97% bucket to ~0.45 in the <90% bucket, ~0.66 in 90–97%).

## Related tools

- **DeepTaxa** (Generates per-sequence species-level predictions on the test FASTA that are scored with F1) — github.com/systems-genomics-lab/deeptaxa
- **BLAST** (Aligns test sequences against the training FASTA to find nearest-neighbor percent identity for bucketing)
- **VSEARCH** (Alternative to BLAST for nearest-neighbor identity search against the training set)

## Examples

```
deeptaxa predict --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_testing.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_testing.tsv.gz --checkpoint ../deeptaxa-data/models/deeptaxa-full-length-v2.pt --output-dir ../deeptaxa-outputs/evaluation
```

## Evaluation signals

- Global species F1 on the 69,335-sequence test set should reproduce ~0.9212 (DeepTaxa 3-seed mean); cross-seed std ≤0.0008 F1
- Bucket-level F1 should show smooth degradation: ~0.95 in the >97% identity bucket, ~0.66 in the 90–97% bucket, and ~0.45 in the <90% bucket
- Every test sequence must be assigned to exactly one bucket and buckets must jointly cover all 69,335 test sequences (sequence counts per bucket sum to the test-set size)
- similarity_stratified_f1.csv must contain one row per bucket with F1 and sequence-count columns populated
- Weighted F1 computed with scikit-learn must use true labels from gg_2024_09_testing.tsv.gz joined one-to-one on sequence IDs

## Limitations

- Evaluation is confined to a single stratified train/test split (80/20) of Greengenes2 2024.09 with three training seeds rather than k-fold cross-validation
- Results are specific to Greengenes2; performance on other reference databases (SILVA, RDP) or independent environmental/clinical datasets remains untested
- Nearest-neighbor identity bucketing depends on the chosen alignment tool (BLAST vs VSEARCH) and its parameters, which may shift sequences across bucket boundaries
- The 'Unseen' species labels absent from the training vocabulary cannot be correctly predicted and can depress bucket-level F1 in low-similarity buckets

## Evidence

- [abstract] Performance degrades smoothly with decreasing training-set similarity: "Performance degrades smoothly with decreasing training-set similarity (species F1 from 0.95 to 0.45)"
- [methods] Dataset split sizes: "The dataset was partitioned into training (277 336 sequences, 80%) and test (69 335 sequences, 20%) sets using stratified sampling to preserve label distributions."
- [abstract] Species-level accuracy and F1: "DeepTaxa achieves species-level accuracy of 92.96% and F1 of 0.9212 (3-seed mean; cross-seed standard deviation ≤0.0008 F1 at every rank)"
- [readme] Evaluation command from README: [evidence span withheld — anchor status: not_found]
- [readme] Test-set size confirmation in README: [evidence span withheld — anchor status: external_doc]
