---
name: nearest-neighbor-sequence-identity-search
description: 'Use when you need to explain a classifier''s per-sequence errors by
  distance to training data: bucket held-out 16S sequences by nearest-neighbor percent
  identity (>97%, 90–97%, <90%) to stratify F1 degradation.'
when_to_use_negative:
- You need classifier accuracy on the full held-out test set rather than identity-stratified
  breakdowns — the pooled `deeptaxa predict --evaluate` metrics already cover this
- The sequences are not 16S rRNA gene sequences drawn from the same reference universe
  as the training pool (e.g. 18S or shotgun metagenomic data), since identity to the
  Greengenes2 training set would be uninformative
- You want database-wide similarity search for annotation (e.g. MEGAN-style assignment)
  rather than a single nearest-neighbor identity value per query for evaluation stratification
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3473
tools:
- name: BLAST
  role: Aligns each test 16S sequence against the Greengenes2 training FASTA to identify
    the nearest training neighbor and its percent nucleotide identity
- name: VSEARCH
  role: Alternative to BLAST for the alignment-based nearest-neighbor identity search
- name: DeepTaxa
  role: Produces the per-sequence species predictions that are joined with the identity
    buckets for stratified F1 computation
  repo: https://github.com/systems-genomics-lab/deeptaxa
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:52acff6cd6ea5998c8dfbe22d85d4fce83c5739ab9aaedf257c5d8ecf4de8848
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
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# nearest-neighbor-sequence-identity-search

## Summary

Aligns each test sequence against the training pool with BLAST or VSEARCH to find its nearest training neighbor and record percent nucleotide identity, enabling similarity-stratified evaluation of taxonomic classifier performance (e.g. DeepTaxa species F1 from 0.95 at >97% identity down to 0.45 at <90%).

## When to use

Use when you need to explain a classifier's per-sequence errors by distance to training data: bucket held-out 16S sequences by nearest-neighbor percent identity (>97%, 90–97%, <90%) to stratify F1 degradation.

## When NOT to use

- You need classifier accuracy on the full held-out test set rather than identity-stratified breakdowns — the pooled `deeptaxa predict --evaluate` metrics already cover this
- The sequences are not 16S rRNA gene sequences drawn from the same reference universe as the training pool (e.g. 18S or shotgun metagenomic data), since identity to the Greengenes2 training set would be uninformative
- You want database-wide similarity search for annotation (e.g. MEGAN-style assignment) rather than a single nearest-neighbor identity value per query for evaluation stratification

## Inputs

- Training FASTA: gg_2024_09_training.fna.gz (277,336 Greengenes2 2024.09 sequences)
- Test FASTA: gg_2024_09_testing.fna.gz (69,335 Greengenes2 2024.09 sequences)
- Per-sequence test-set species predictions from DeepTaxa (e.g. the output directory of `deeptaxa predict`)
- Greengenes2 test taxonomy labels (gg_2024_09_testing.tsv.gz) for computing per-bucket F1

## Outputs

- Nearest-neighbor percent identity per test sequence
- Similarity-bucket assignment per test sequence (>97%, 90–97%, <90%)
- similarity_stratified_f1.csv: bucket-level weighted species F1 with sequence counts per bucket

## How to apply

Load the Greengenes2 2024.09 training FASTA (277,336 sequences) and test FASTA (69,335 sequences), then align every test sequence against the training pool using BLAST (or VSEARCH, which the article lists among alignment-based tools) and record the percent nucleotide identity of the nearest training neighbor. Assign each test sequence to one of three similarity buckets: >97%, 90–97%, or <90% identity. Join these bucket assignments with the per-sequence species predictions from DeepTaxa (e.g. from `deeptaxa predict` on gg_2024_09_testing.fna.gz) and compute weighted species-level F1 per bucket with scikit-learn/pandas. The rationale is that DeepTaxa's reported finding — smooth degradation of species F1 from ~0.95 to ~0.45 — is only observable when test sequences are stratified by alignment identity to the training set rather than evaluated as a single pooled metric.

## Related tools

- **BLAST** (Aligns each test 16S sequence against the Greengenes2 training FASTA to identify the nearest training neighbor and its percent nucleotide identity)
- **VSEARCH** (Alternative to BLAST for the alignment-based nearest-neighbor identity search)
- **DeepTaxa** (Produces the per-sequence species predictions that are joined with the identity buckets for stratified F1 computation) — https://github.com/systems-genomics-lab/deeptaxa

## Examples

```
deeptaxa predict --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_testing.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_testing.tsv.gz --checkpoint ../deeptaxa-data/models/deeptaxa-full-length-v2.pt --output-dir ../deeptaxa-outputs/evaluation
```

## Evaluation signals

- Bucket-level weighted species F1 reproduces the reported smooth degradation: ~0.95 in the >97% bucket, ~0.66 in the 90–97% bucket, and ~0.45 in the <90% bucket
- Every test sequence (all 69,335) receives exactly one similarity-bucket assignment with no unaligned or missing records
- Each bucket's sequence counts sum to the 69,335-sequence test set size
- similarity_stratified_f1.csv is written with per-bucket F1 and sequence counts, and F1 values are monotonically decreasing across the identity buckets

## Limitations

- Alignment-based nearest-neighbor identity to the training pool measures only sequence similarity, not label novelty; the article's separate Seen vs Unseen species-label analysis addresses label vocabulary rather than identity
- The analysis depends on a single stratified 80/20 train/test split (277,336 / 69,335 sequences); the article notes no k-fold cross-validation was used, so bucket-level F1 is split-dependent
- DeepTaxa was trained and evaluated primarily on Greengenes2; the identity-stratified degradation pattern may differ on SILVA, RDP, or independent environmental/clinical datasets

## Evidence

- [abstract] Performance degrades smoothly with decreasing training-set similarity: "Performance degrades smoothly with decreasing training-set similarity (species F1 from 0.95 to 0.45)"
- [methods] Stratified train/test partitioning of Greengenes2: "The dataset was partitioned into training (277 336 sequences, 80%) and test (69 335 sequences, 20%) sets using stratified sampling"
- [intro] VSEARCH as an alternative alignment tool: "Other alignment-based approaches, such as MMseqs2 (Steinegger and Söding 2017), Minimap2 (Li 2018), and VSEARCH (Rognes et al. 2016)"
- [readme] Test FASTA and evaluation entry point in README: [evidence span withheld — anchor status: not_found]
