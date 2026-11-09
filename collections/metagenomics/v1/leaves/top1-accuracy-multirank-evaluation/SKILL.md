---
name: top1-accuracy-multirank-evaluation
description: Use when a labeled 16S rRNA test FASTA with taxonomy TSV labels is available
  and per-rank top-1 accuracy, weighted F1, and calibration (ECE) of a classifier
  must be reproduced or benchmarked across all seven taxonomic ranks.
when_to_use_negative:
- Input sequences come from a different reference database (SILVA, RDP) or an independent
  environmental/clinical dataset — the checkpoint was trained and evaluated only on
  Greengenes2 2024.09 and cross-database performance remains untested
- The goal is training or hyperparameter tuning rather than evaluation of a fixed
  checkpoint on held-out data
- Input data lacks ground-truth taxonomic labels, so accuracy/F1/ECE cannot be computed
edam_operation: http://edamontology.org/operation_0394
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DeepTaxa
  role: Hybrid CNN-BERT classifier whose predict module computes per-rank accuracy,
    weighted F1, and ECE on labeled test FASTA/TSV inputs
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: DNABERT-2
  role: BPE tokenizer used to encode test sequences to a fixed 512-token length before
    inference
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:541b272aea92c3fea0ac54a606e35d0cb6fe14ac23f1f1dcced40e40ba75e613
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  edam_operation: http://edamontology.org/operation_0394
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# top1-accuracy-multirank-evaluation

## Summary

A skill for evaluating a hierarchical taxonomic classifier on a labeled held-out 16S rRNA test set by computing per-rank top-1 accuracy, weighted F1-score, and expected calibration error (10 equal-width bins) across all seven ranks, Domain through Species, as done for DeepTaxa on the Greengenes2 2024.09 test set of 69,335 sequences.

## When to use

Use when a labeled 16S rRNA test FASTA with taxonomy TSV labels is available and per-rank top-1 accuracy, weighted F1, and calibration (ECE) of a classifier must be reproduced or benchmarked across all seven taxonomic ranks.

## When NOT to use

- Input sequences come from a different reference database (SILVA, RDP) or an independent environmental/clinical dataset — the checkpoint was trained and evaluated only on Greengenes2 2024.09 and cross-database performance remains untested
- The goal is training or hyperparameter tuning rather than evaluation of a fixed checkpoint on held-out data
- Input data lacks ground-truth taxonomic labels, so accuracy/F1/ECE cannot be computed

## Inputs

- Held-out test FASTA of 16S rRNA sequences (gg_2024_09_testing.fna.gz, 69,335 sequences)
- Corresponding taxonomic labels TSV (gg_2024_09_testing.tsv.gz)
- Pre-trained per-rank classifier checkpoint (e.g., deeptaxa-full-length-v2.pt)
- GPU with mixed-precision inference environment

## Outputs

- species_metrics.csv / per-rank metrics report with top-1 accuracy, weighted F1, and ECE for Domain through Species
- Per-sequence predicted labels and softmax confidence distributions at each rank
- Prediction-uncertainty summary (softmax entropy)

## How to apply

Run the trained checkpoint in inference mode (deeptaxa predict module) on the held-out test FASTA (69,335 Greengenes2 2024.09 sequences) with the corresponding taxonomy TSV, tokenizing each sequence with the DNABERT-2 BPE tokenizer truncated/padded to 512 tokens and applying softmax at T=1.0 on the seven per-rank classifier heads. Supply the taxonomy file so the evaluate path emits per-rank accuracy, weighted F1, and ECE in addition to raw predictions. Compute expected calibration error using 10 equal-width confidence bins at each rank, and report softmax entropy as prediction uncertainty. Compare species-level top-1 accuracy, F1, and ECE against the published reference values (accuracy ~92.95–92.96%, species F1 ~0.9212/0.9208, species ECE ~0.0242) within a small tolerance; cross-seed standard deviation should stay at or below ~0.0008 F1 at every rank, so any larger deviation indicates an evaluation-pipeline error (wrong test split, tokenizer truncation, or checkpoint mismatch).

## Related tools

- **DeepTaxa** (Hybrid CNN-BERT classifier whose predict module computes per-rank accuracy, weighted F1, and ECE on labeled test FASTA/TSV inputs) — https://github.com/systems-genomics-lab/deeptaxa
- **DNABERT-2** (BPE tokenizer used to encode test sequences to a fixed 512-token length before inference)

## Examples

```
deeptaxa predict --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_testing.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_testing.tsv.gz --checkpoint ../deeptaxa-data/models/deeptaxa-full-length-v2.pt --output-dir ../deeptaxa-outputs/evaluation
```

## Evaluation signals

- Species-level top-1 accuracy lands within a small tolerance of the published 92.96% (article, 3-seed mean) or 92.95% (README, 5-seed mean)
- Species weighted F1 matches ~0.9212 (article) / 0.9208 (README), and F1 exceeds 0.99 from Domain through Class
- Species ECE equals ~0.0242 with 10 equal-width confidence bins
- Cross-seed standard deviation of species F1 is ≤0.0008, and no rank shows std above ~0.0008 F1
- Per-rank accuracy is monotonically decreasing from Domain (99.98%) toward Species (92.95%), matching the published ranking pattern

## Limitations

- Reported species metrics differ slightly between sources: 92.96% / F1 0.9212 (3-seed mean, article abstract) vs 92.95% / F1 92.08% (5-seed mean across seeds 42, 123, 456, 789, 1011, README); tolerate this spread when checking reproduction
- No k-fold cross-validation — results rest on a single stratified train/validation/test split with three training seeds
- Reference-based classification cannot assign species absent from the 16,909-species training label vocabulary (Unseen labels), which limits absolute species accuracy

## Evidence

- [abstract] published species-level reference metrics: "DeepTaxa achieves species-level accuracy of 92.96% and F1 of 0.9212 (3-seed mean; cross-seed standard deviation ≤0.0008 F1 at every rank)"
- [methods] test-set tokenization for inference: "Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens"
- [readme] README per-rank metric table scope: [evidence span withheld — anchor status: external_doc]
- [readme] README species-level metric values: [evidence span withheld — anchor status: external_doc]
