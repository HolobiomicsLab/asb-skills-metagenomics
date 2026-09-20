---
name: checkpoint-based-model-inference
description: Use when you need to classify 16S rRNA sequences into seven taxonomic
  ranks (Domain–Species) with an existing published DeepTaxa checkpoint, without retraining,
  e.g. on the 69,335-sequence Greengenes2 2024.09 test FASTA.
when_to_use_negative:
- The checkpoint was trained on full-length Greengenes2 16S but your input is short
  V3–V4 or V4 amplicons — use the matching region-specific checkpoint (deeptaxa-v3v4-v2.pt
  ~420 bp, deeptaxa-v4-v2.pt ~253 bp) instead
- Sequences carry species labels absent from the training vocabulary ('Unseen' labels)
  — species-rank accuracy metrics will not be comparable to the published Seen/Unseen
  stratified evaluation
- You need to train or fine-tune rather than infer (e.g. new reference database) —
  use deeptaxa train instead of predict
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3474
tools:
- name: DeepTaxa
  role: deeptaxa predict CLI runs inference with the HybridCNNBERTClassifier checkpoint,
    computing per-rank accuracy, F1, and ECE when labels are supplied
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: DNABERT-2
  role: BPE tokenizer used to encode input 16S sequences to a fixed 512-token length
    before the classifier forward pass
- name: Greengenes2 2024.09 test set
  role: Held-out 69,335-sequence test FASTA with taxonomy TSV used to validate checkpoint
    inference against published metrics
  repo: https://huggingface.co/datasets/systems-genomics-lab/greengenes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:d55f91fcd5f309ebe4d6db5b0517dba87c5306c9e80652b7cb56b6ba94ec8538
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  - Greengenes2 2024.09 test set
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# checkpoint-based-model-inference

## Summary

Run taxonomic inference from a published pre-trained DeepTaxa checkpoint (e.g. deeptaxa-full-length-v2.pt) instead of retraining: download the checkpoint from Hugging Face, classify 16S rRNA sequences at all seven ranks with HybridCNNBERTClassifier, and evaluate against known labels when available.

## When to use

Use when you need to classify 16S rRNA sequences into seven taxonomic ranks (Domain–Species) with an existing published DeepTaxa checkpoint, without retraining, e.g. on the 69,335-sequence Greengenes2 2024.09 test FASTA.

## When NOT to use

- The checkpoint was trained on full-length Greengenes2 16S but your input is short V3–V4 or V4 amplicons — use the matching region-specific checkpoint (deeptaxa-v3v4-v2.pt ~420 bp, deeptaxa-v4-v2.pt ~253 bp) instead
- Sequences carry species labels absent from the training vocabulary ('Unseen' labels) — species-rank accuracy metrics will not be comparable to the published Seen/Unseen stratified evaluation
- You need to train or fine-tune rather than infer (e.g. new reference database) — use deeptaxa train instead of predict

## Inputs

- 16S rRNA sequence FASTA (e.g. gg_2024_09_testing.fna.gz, 69,335 sequences)
- Optional taxonomy label TSV matching the FASTA (e.g. gg_2024_09_testing.tsv.gz) for evaluation
- Pre-trained checkpoint from Hugging Face (e.g. deeptaxa-full-length-v2.pt, deeptaxa-v3v4-v2.pt, deeptaxa-v4-v2.pt)

## Outputs

- Per-sequence taxonomic assignments at all seven ranks (Domain through Species)
- Per-rank top-1 accuracy, weighted F1-score, and expected calibration error (ECE, 10 bins) when labels are provided
- Species metrics file (e.g. species_metrics.csv) and prediction outputs in the specified output directory
- Softmax distributions / entropy as prediction uncertainty

## How to apply

Download the pre-trained checkpoint from huggingface.co/systems-genomics-lab/deeptaxa (e.g. deeptaxa-full-length-v2.pt, a copy of the seed-42 checkpoint, 76.4 M parameters) into a sibling deeptaxa-data/models directory outside the codebase. Tokenize each input sequence with the DNABERT-2 BPE tokenizer, truncating or padding to a fixed length of 512 tokens, then run the deeptaxa predict module on a GPU with mixed precision, applying softmax at temperature T=1.0 across the seven per-rank classifier heads of the HybridCNNBERTClassifier. When a taxonomy label file accompanies the FASTA (e.g. gg_2024_09_testing.tsv.gz), deeptaxa predict adds per-rank accuracy, weighted F1, and expected calibration error (10 equal-width confidence bins) to the outputs. Verify correctness by checking that species-level metrics fall near the published values: species accuracy ~92.95–92.96%, species F1 ~0.9212–0.9213, and species ECE ~0.0242 (species std 0.0004 F1 across seeds). For better accuracy and calibration, average per-rank softmax probabilities across all five seed checkpoints (seeds 42, 123, 456, 789, 1011) via scripts/ensemble_predict.py and take the argmax at each rank (ensemble species F1 0.9330 for full-length).

## Related tools

- **DeepTaxa** (deeptaxa predict CLI runs inference with the HybridCNNBERTClassifier checkpoint, computing per-rank accuracy, F1, and ECE when labels are supplied) — https://github.com/systems-genomics-lab/deeptaxa
- **DNABERT-2** (BPE tokenizer used to encode input 16S sequences to a fixed 512-token length before the classifier forward pass)
- **Greengenes2 2024.09 test set** (Held-out 69,335-sequence test FASTA with taxonomy TSV used to validate checkpoint inference against published metrics) — https://huggingface.co/datasets/systems-genomics-lab/greengenes

## Examples

```
deeptaxa predict --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_testing.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_testing.tsv.gz --checkpoint ../deeptaxa-data/models/deeptaxa-full-length-v2.pt --output-dir ../deeptaxa-outputs/evaluation
```

## Evaluation signals

- Species-level top-1 accuracy within a small tolerance of the published 92.96% (paper, 3-seed mean) or 92.95% (README v2, 5-seed mean)
- Species weighted F1 close to 0.9212 (paper) / 0.9213 (README single-seed); cross-seed std ≤0.0008 F1 at every rank
- Species ECE ≈ 0.0242 (10 equal-width confidence bins)
- F1 above 0.99 from domain through class ranks; genus accuracy ~96.87%, family ~98.61%
- Checkpoint self-identification: each checkpoint records its seed in a 'seed' field; deeptaxa describe reports model metadata matching the downloaded file

## Limitations

- Full-length checkpoints should not be expected to reach species accuracy on short amplicon windows: the dedicated V3–V4 checkpoint reaches 87.55% species accuracy from an ~420 bp window and V4 82.84% from ~253 bp
- Performance degrades smoothly with decreasing training-set similarity (species F1 from 0.95 to 0.45) and for 'Unseen' species labels absent from the training vocabulary
- Checkpoint files use PyTorch pickle-based serialization and should only be downloaded from the official Hugging Face repository
- DeepTaxa operates on FASTA-formatted sequences only and does not incorporate base-level quality scores

## Evidence

- [abstract] DeepTaxa achieves species-level accuracy of 92.96% and F1 of 0.9212 (3-seed mean; cross-seed standard deviation ≤0.0008 F1 at every rank): "DeepTaxa achieves species-level accuracy of 92.96% and F1 of 0.9212 (3-seed mean; cross-seed standard deviation ≤0.0008 F1 at every rank)"
- [methods] Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens: "Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens"
- [readme] deeptaxa predict with fasta and taxonomy files adds per-rank accuracy, F1, ECE to the output: "deeptaxa predict \
  --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_testing.fna.gz \
  --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_testing.tsv.gz \
  --checkpoint"
- [readme] README checkpoint note on pickle serialization and provenance: [evidence span withheld — anchor status: external_doc]
- [readme] Checkpoint seeds are self-identifying for ensembling: [evidence span withheld — anchor status: external_doc]
