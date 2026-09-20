---
name: greengenes2-test-set-analysis
description: Use when you have DeepTaxa (or any 16S rRNA classifier) species predictions
  on a held-out test set plus the training FASTA, and you need to diagnose whether
  low species-level scores are driven by low similarity to training sequences rather
  than model failure.
when_to_use_negative:
- You only need overall test-set metrics (accuracy, F1, ECE) that DeepTaxa already
  reports; the full 5-seed table covers aggregate performance without stratification.
- The test sequences were not drawn from the same Greengenes2 2024.09 stratified split
  (20% of the 346,671 curated 16S sequences), since bucket-level F1 targets the article's
  specific held-out split.
- You lack the training FASTA to align against, so nearest-neighbor identity cannot
  be computed.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: DeepTaxa
  role: Supplies the per-sequence species-level predictions on the Greengenes2 test
    set that are stratified by identity bucket
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: BLAST
  role: Aligns each test sequence against the training pool to find the nearest training
    neighbor and record its percent nucleotide identity
- name: VSEARCH
  role: Alternative to BLAST for computing nearest-neighbor percent identity between
    test and training sequences
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:0bfdfd2ade7bc83b2217beb41a1822b50b6d54e2ac3452066e8fd1ecba8800b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - BLAST
  - DeepTaxa
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# greengenes2-test-set-analysis

## Summary

Stratifies the Greengenes2 2024.09 test-set species predictions from DeepTaxa by the percent nucleotide identity of each test sequence to its nearest training-set neighbor (>97%, 90-97%, <90%), then computes weighted species-level F1 per bucket to quantify how performance degrades as test sequences move away from the training distribution.

## When to use

Use when you have DeepTaxa (or any 16S rRNA classifier) species predictions on a held-out test set plus the training FASTA, and you need to diagnose whether low species-level scores are driven by low similarity to training sequences rather than model failure.

## When NOT to use

- You only need overall test-set metrics (accuracy, F1, ECE) that DeepTaxa already reports; the full 5-seed table covers aggregate performance without stratification.
- The test sequences were not drawn from the same Greengenes2 2024.09 stratified split (20% of the 346,671 curated 16S sequences), since bucket-level F1 targets the article's specific held-out split.
- You lack the training FASTA to align against, so nearest-neighbor identity cannot be computed.

## Inputs

- Greengenes2 2024.09 training FASTA (gg_2024_09_training.fna.gz, 277,336 sequences)
- Greengenes2 2024.09 testing FASTA (gg_2024_09_testing.fna.gz, 69,335 sequences)
- Per-sequence species-level predictions from DeepTaxa on the test set
- Taxonomy label TSV (gg_2024_09_testing.tsv.gz) for ground-truth species labels

## Outputs

- similarity_stratified_f1.csv: species-level weighted F1 per similarity bucket with sequence counts per bucket
- Per-test-sequence mapping of nearest training neighbor percent identity and bucket assignment

## How to apply

Load the Greengenes2 2024.09 training FASTA (gg_2024_09_training.fna.gz, 277,336 sequences) and test FASTA (gg_2024_09_testing.fna.gz, 69,335 sequences), and the per-sequence species predictions from DeepTaxa (produced via `deeptaxa predict` with the full-length checkpoint). Align every test sequence against the training pool with BLAST (or VSEARCH) to identify the nearest training neighbor and its percent nucleotide identity. Bin each test sequence into one of three similarity buckets: >97%, 90-97%, or <90% identity to its nearest training sequence. Join the bucket assignments with the per-sequence species predictions and compute weighted species-level F1 per bucket (e.g. with scikit-learn's f1_score using per-sequence species labels), then write a summary table with bucket sequence counts (similarity_stratified_f1.csv). The rationale is that DeepTaxa's reported degradation 'with decreasing training-set similarity (species F1 from 0.95 to 0.45)' can only be verified by conditioning on identity to the training pool.

## Related tools

- **DeepTaxa** (Supplies the per-sequence species-level predictions on the Greengenes2 test set that are stratified by identity bucket) — https://github.com/systems-genomics-lab/deeptaxa
- **BLAST** (Aligns each test sequence against the training pool to find the nearest training neighbor and record its percent nucleotide identity)
- **VSEARCH** (Alternative to BLAST for computing nearest-neighbor percent identity between test and training sequences)

## Examples

```
blastn -query gg_2024_09_testing.fna -db gg_2024_09_training -outfmt "6 qseqid pident" -max_target_seqs 1 -num_threads 8
```

## Evaluation signals

- Bucket sequence counts sum to the full 69,335 test sequences with no unassigned records.
- Reproduces the reported smooth degradation: species F1 ~0.95 in the >97% bucket, ~0.66 in the 90-97% bucket, and ~0.45 in the <90% bucket.
- F1 values must decrease monotonically from the >97% to the <90% bucket.
- Per-bucket weighted F1 is bounded in [0,1] and weighted by bucket size consistent with scikit-learn weighted averaging.
- Each test sequence has exactly one nearest-neighbor identity value from BLAST/VSEARCH with a non-empty hit.

## Limitations

- Evaluated only on the Greengenes2 2024.09 split; performance across other reference databases (SILVA, RDP) and independent environmental or clinical datasets remains untested.
- Results depend on the single stratified train/validation/test split used by the paper rather than k-fold cross-validation.
- Species labels absent from the training vocabulary ('Unseen') are inherently penalized in the low-similarity buckets; seen vs unseen status should be tracked to interpret bucket F1.

## Evidence

- [abstract] smooth degradation finding: "Performance degrades smoothly with decreasing training-set similarity (species F1 from 0.95 to 0.45)"
- [methods] train/test split sizes: "The dataset was partitioned into training (277 336 sequences, 80%) and test (69 335 sequences, 20%) sets using stratified sampling"
- [methods] seen/unseen labels caveat: "We classified sequences into two categories: “Seen” (species label present in the training vocabulary) and “Unseen” (species label absent"
- [readme] README data layout: [evidence span withheld — anchor status: external_doc]
