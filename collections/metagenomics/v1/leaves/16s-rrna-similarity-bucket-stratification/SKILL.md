---
name: 16s-rrna-similarity-bucket-stratification
description: Use when you have per-sequence species predictions from a 16S rRNA classifier
  (e.g., DeepTaxa on the Greengenes2 2024.09 69,335-sequence test set) and need to
  show whether accuracy degrades for test sequences distant from the 277,336-sequence
  training pool.
when_to_use_negative:
- You only need overall species accuracy/F1 on the Greengenes2 test set (e.g., 92.95%
  accuracy / 0.9212 F1) — aggregate metrics require no similarity stratification.
- The classifier's training pool is unknown or disjoint from the reference used for
  the identity search, so 'nearest training neighbor' identity is undefined.
- Evaluating a different marker gene or modality (e.g., 18S rRNA or shotgun metagenomic
  data) for which no Greengenes2 training-pool identity relationship exists.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: BLAST
  role: Aligns each test sequence against the 277,336-sequence training FASTA to find
    the nearest training neighbor and record its percent nucleotide identity
- name: VSEARCH
  role: Alternative aligner to BLAST for the nearest-neighbor identity search against
    the training pool
- name: DeepTaxa
  role: Source of the per-sequence species-level predictions whose F1 is stratified
    by similarity bucket
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: scikit-learn/pandas
  role: Compute weighted species-level F1 per bucket after joining bucket labels with
    per-sequence predictions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:8ba86bff4dc2a89f544602fb034ef35555978e9341e1d68f9fdeb725478f8f53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - BLAST
  - DeepTaxa
  - VSEARCH
  - scikit-learn/pandas
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# 16s-rrna-similarity-bucket-stratification

## Summary

Stratifies 16S rRNA taxonomic-classifier performance by evolutionary novelty: each test sequence is BLAST/VSEARCH-aligned against the training FASTA to get nearest-neighbor percent identity, binned into >97%, 90–97%, and <90% buckets, and species-level weighted F1 is computed per bucket to expose degradation that a single aggregate F1 (e.g., DeepTaxa's 0.9212) hides.

## When to use

Use when you have per-sequence species predictions from a 16S rRNA classifier (e.g., DeepTaxa on the Greengenes2 2024.09 69,335-sequence test set) and need to show whether accuracy degrades for test sequences distant from the 277,336-sequence training pool.

## When NOT to use

- You only need overall species accuracy/F1 on the Greengenes2 test set (e.g., 92.95% accuracy / 0.9212 F1) — aggregate metrics require no similarity stratification.
- The classifier's training pool is unknown or disjoint from the reference used for the identity search, so 'nearest training neighbor' identity is undefined.
- Evaluating a different marker gene or modality (e.g., 18S rRNA or shotgun metagenomic data) for which no Greengenes2 training-pool identity relationship exists.

## Inputs

- Greengenes2 2024.09 training FASTA (gg_2024_09_training.fna.gz; 277,336 sequences, ~1,500 bp full-length 16S)
- Greengenes2 2024.09 test FASTA (gg_2024_09_testing.fna.gz; 69,335 sequences)
- Per-sequence species-level predictions from DeepTaxa (e.g., from 'deeptaxa predict --checkpoint deeptaxa-full-length-v2.pt')
- Test-set taxonomy labels (gg_2024_09_testing.tsv.gz) as ground truth for F1 computation

## Outputs

- similarity_stratified_f1.csv — bucket-level weighted species F1 with per-bucket sequence counts
- Per-test-sequence nearest-neighbor percent identity and bucket assignment (>97%, 90–97%, <90%)

## How to apply

Align every test sequence (gg_2024_09_testing.fna.gz, 69,335 sequences) against the training pool (gg_2024_09_training.fna.gz, 277,336 sequences) using BLAST (blastn) or VSEARCH in tabular outfmt, retaining the top hit's percent nucleotide identity as the nearest-neighbor similarity. Assign each test sequence to one of three similarity buckets: >97%, 90–97%, or <90% identity. Join the bucket labels to DeepTaxa's per-sequence species predictions and compute the weighted species-level F1-score per bucket with scikit-learn/pandas. Assemble bucket-level F1 values together with per-bucket sequence counts into similarity_stratified_f1.csv. The rationale is the paper's finding that performance 'degrades smoothly with decreasing training-set similarity', so the expected bucket profile is roughly 0.95 (>97%), 0.66 (90–97%), and 0.45 (<90%) — a decomposition of generalization that the aggregate 92.96% species accuracy cannot show.

## Related tools

- **BLAST** (Aligns each test sequence against the 277,336-sequence training FASTA to find the nearest training neighbor and record its percent nucleotide identity)
- **VSEARCH** (Alternative aligner to BLAST for the nearest-neighbor identity search against the training pool)
- **DeepTaxa** (Source of the per-sequence species-level predictions whose F1 is stratified by similarity bucket) — https://github.com/systems-genomics-lab/deeptaxa
- **scikit-learn/pandas** (Compute weighted species-level F1 per bucket after joining bucket labels with per-sequence predictions)

## Examples

```
blastn -query gg_2024_09_testing.fna.gz -db gg_2024_09_training -outfmt "6 qseqid pident" -max_target_seqs 1 -num_threads 8 > nearest_neighbor_identity.tsv
```

## Evaluation signals

- Bucket-level species F1 reproduces the reported smooth degradation: ~0.95 in the >97% bucket, ~0.66 in the 90–97% bucket, ~0.45 in the <90% bucket.
- Every one of the 69,335 test sequences receives exactly one bucket label, and per-bucket counts sum to the test-set size.
- Bucket boundaries match the defined thresholds: >97%, 90–97%, <90% percent nucleotide identity to the nearest training sequence.
- similarity_stratified_f1.csv contains bucket-level F1 values plus sequence counts per bucket, with no missing rows.
- Count-weighted combination of the three bucket F1 values is consistent with the overall species F1 (≈0.9212 full-length) reported for the same test set.

## Limitations

- Computationally heavy: requires aligning all 69,335 test sequences against a 277,336-sequence full-length training FASTA with BLAST or VSEARCH.
- Findings are specific to Greengenes2 2024.09 and DeepTaxa's full-length checkpoint; the article notes performance across other reference databases (SILVA, RDP) and independent datasets remains untested.
- Choice of aligner (BLAST vs VSEARCH) and top-hit tie-breaking can move boundary sequences across the 97% and 90% cutoffs, shifting bucket F1 values.

## Evidence

- [abstract] Performance degrades smoothly with decreasing training-set similarity (species F1 from 0.95 to 0.45): "Performance degrades smoothly with decreasing training-set similarity (species F1 from 0.95 to 0.45)"
- [methods] Stratified train/test partition sizes for Greengenes2 2024.09: "The dataset was partitioned into training (277 336 sequences, 80%) and test (69 335 sequences, 20%) sets using stratified sampling to preserve label distributions."
- [intro] VSEARCH named among alignment-based approaches usable for similarity search: "Other alignment-based approaches, such as MMseqs2 (Steinegger and Söding 2017), Minimap2 (Li 2018), and VSEARCH (Rognes et al. 2016)"
- [readme] Per-rank F1 output available from DeepTaxa evaluation for the bucket join: [evidence span withheld — anchor status: external_doc]
