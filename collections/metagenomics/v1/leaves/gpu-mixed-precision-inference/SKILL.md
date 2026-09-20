---
name: gpu-mixed-precision-inference
description: Use when classifying 16S rRNA FASTA sequences (full-length ~1,500 bp
  or in-silico V3–V4 ~420 bp amplicons) with a released DeepTaxa checkpoint on a CUDA
  GPU, where per-rank predictions and accuracy/F1/ECE metrics are required.
when_to_use_negative:
- Input is FASTQ or otherwise requires base-level quality scores — DeepTaxa operates
  on FASTA-formatted sequences only and does not incorporate quality information
- Classifying 18S rRNA or shotgun metagenomic data — the framework was trained and
  evaluated only on 16S rRNA from Greengenes2 2024.09
- No CUDA GPU available for the sequence set — CPU inference of the 75.8 M-parameter
  model will be impractically slow for large test sets
tools:
- name: DeepTaxa
  role: Hybrid CNN-BERT classifier run via `deeptaxa predict` under PyTorch mixed
    precision to generate per-rank softmax predictions from the released checkpoint
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: DNABERT-2
  role: BPE tokenizer used to tokenize each 16S sequence into subword tokens (fixed
    512-token length) before model inference
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:0a3381edf6a170dfd2743f26ab23eb2aaacf94a8f6fbf772295f32801791659a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gpu-mixed-precision-inference

## Summary

Runs DeepTaxa taxonomic classification inference on CUDA GPUs under PyTorch mixed precision, tokenizing 16S rRNA sequences with the DNABERT-2 BPE tokenizer and emitting per-rank softmax predictions and top-1 labels across the seven Linnean ranks.

## When to use

Use when classifying 16S rRNA FASTA sequences (full-length ~1,500 bp or in-silico V3–V4 ~420 bp amplicons) with a released DeepTaxa checkpoint on a CUDA GPU, where per-rank predictions and accuracy/F1/ECE metrics are required.

## When NOT to use

- Input is FASTQ or otherwise requires base-level quality scores — DeepTaxa operates on FASTA-formatted sequences only and does not incorporate quality information
- Classifying 18S rRNA or shotgun metagenomic data — the framework was trained and evaluated only on 16S rRNA from Greengenes2 2024.09
- No CUDA GPU available for the sequence set — CPU inference of the 75.8 M-parameter model will be impractically slow for large test sets

## Inputs

- 16S rRNA sequences in FASTA format (e.g., gg_2024_09_testing.fna.gz, or in-silico V3–V4 ~420 bp amplicon FASTA)
- DeepTaxa checkpoint file (.pt, e.g., deeptaxa-v3v4-v2.pt or deeptaxa-full-length-v2.pt from the Hugging Face repository)
- Ground-truth taxonomy TSV (e.g., gg_2024_09_testing.tsv.gz) when evaluation metrics are needed

## Outputs

- Per-rank softmax probability vectors for the seven Linnean ranks (domain through species)
- Top-1 predicted taxonomic labels at each rank
- Per-rank accuracy, F1, precision, recall, and ECE metrics (JSON, e.g., written to v3v4_species_accuracy)

## How to apply

Download the appropriate checkpoint (e.g., deeptaxa-v3v4-v2.pt, 75.8 M parameters, 8,347-species vocabulary) from the Hugging Face systems-genomics-lab/deeptaxa repository. Tokenize each input sequence with the DNABERT-2 BPE tokenizer, truncating or padding to 512 tokens, then run `deeptaxa predict` with the HybridCNNBERTClassifier under PyTorch mixed precision on GPU. The model performs a single forward pass to produce per-rank softmax probabilities for all seven ranks (domain through species); take the argmax at each rank as the top-1 label. If ground-truth taxonomy TSV files are supplied, the tool also computes per-rank top-1 accuracy, F1, and expected calibration error. Validation against Greengenes2 2024.09 test amplicons should recover ~87.54% species accuracy (seed-42 single model; ~87.55% reported) and F1 near 0.8592 with ECE ~0.0278.

## Related tools

- **DeepTaxa** (Hybrid CNN-BERT classifier run via `deeptaxa predict` under PyTorch mixed precision to generate per-rank softmax predictions from the released checkpoint) — https://github.com/systems-genomics-lab/deeptaxa
- **DNABERT-2** (BPE tokenizer used to tokenize each 16S sequence into subword tokens (fixed 512-token length) before model inference)

## Examples

```
deeptaxa predict --fasta-file your_sequences.fna --checkpoint ../deeptaxa-data/models/deeptaxa-v3v4-v2.pt --output-dir ../deeptaxa-outputs/predictions
```

## Evaluation signals

- Species-level top-1 accuracy on the Greengenes2 2024.09 V3–V4 test set reproduces the reported ~87.54–87.55% from the dedicated V3–V4 checkpoint
- Species F1 ≈ 0.8592 and species ECE ≈ 0.0278 for the seed-42 V3–V4 checkpoint
- Full-length checkpoint reproduces ~92.95% species accuracy (5-seed mean) on the 69,335-sequence test set with ECE 0.0242
- Output JSON contains predictions for all seven ranks, not just species
- Higher cross-rank monotonicity: domain/class F1 ≥ 0.99 and accuracy decreases smoothly toward finer ranks

## Limitations

- Performance degrades for test species dissimilar to the training set (species F1 drops from 0.95 to 0.45 as training-set similarity decreases), and unseen species labels cannot be predicted — 'Unseen' labels are absent from the training vocabulary
- Results are database-specific: performance across SILVA, RDP, or independent environmental/clinical datasets remains to be tested
- PyTorch checkpoint files use pickle-based serialization; download only from the official Hugging Face repository to mitigate arbitrary-code-execution risk
- Single-model results vary slightly by seed, hardware, and package versions; use the 5-seed soft-vote ensemble for the best calibrated accuracy

## Evidence

- [abstract] V3V4 checkpoint and expected result: "a dedicated V3–V4 amplicon checkpoint reaches 87.55% species accuracy from an approximately 420 bp window"
- [methods] Training/inference regime: "The model was trained using the AdamW optimizer (Loshchilov and Hutter 2019) with hyperparameters set to β1=0.9, β2=0.999, ε=1×10−8"
- [methods] Tokenization detail: "Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens"
- [readme] Checkpoint download and scale: [evidence span withheld — anchor status: external_doc]
- [readme] GPU hardware expectations: [evidence span withheld — anchor status: external_doc]
