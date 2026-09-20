---
name: hybrid-cnn-bert-variant-training
description: Use when you need to attribute DeepTaxa's species-level performance to
  its components by training ablation variants (CNN pathway only, BERT pathway only,
  or DNABERT-2 BPE tokenization replaced by one-hot encodings) and comparing species
  F1 to the full hybrid baseline.
when_to_use_negative:
- The goal is benchmarking DeepTaxa against external tools (DADA2, QIIME 2, Kraken2,
  SINTAX) rather than attributing performance to internal architecture components
- The working data are in-silico V3–V4 or V4 amplicons rather than the full-length
  16S split — the published ablation was run on the Greengenes2 full-length training
  set only
- Only predictions from already-published checkpoints are needed; no retraining or
  component-removal experiment is required
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_3307
tools:
- name: DeepTaxa
  role: CLI framework used to train each architecture variant (HybridCNNBERTClassifier,
    CNNClassifier, BERTClassifier) under the compact configuration and to evaluate
    them on the held-out test set
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: DNABERT-2
  role: BPE tokenizer whose vocabulary is used to tokenize 16S sequences (truncated/padded
    to 512 tokens); replaced by raw one-hot nucleotide encodings in the one-hot CNN
    ablation variant
- name: PyTorch
  role: Underlying training engine providing the AdamW optimizer, mixed-precision
    training, and pickle-based checkpoint serialization used for all variants
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:667790a9c02a994983a31b613c1121ef25968c0131d470b8e04a74350c85eac2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  - PyTorch
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# hybrid-cnn-bert-variant-training

## Summary

Trains DeepTaxa architecture ablation variants (CNN-only, BERT-only, one-hot CNN) under the compact hyperparameter configuration on the Greengenes2 2024.09 full-length 16S training set, then compares their species-level F1 on the shared 69,335-sequence held-out test set against the full HybridCNNBERT model to quantify each component's contribution.

## When to use

Use when you need to attribute DeepTaxa's species-level performance to its components by training ablation variants (CNN pathway only, BERT pathway only, or DNABERT-2 BPE tokenization replaced by one-hot encodings) and comparing species F1 to the full hybrid baseline.

## When NOT to use

- The goal is benchmarking DeepTaxa against external tools (DADA2, QIIME 2, Kraken2, SINTAX) rather than attributing performance to internal architecture components
- The working data are in-silico V3–V4 or V4 amplicons rather than the full-length 16S split — the published ablation was run on the Greengenes2 full-length training set only
- Only predictions from already-published checkpoints are needed; no retraining or component-removal experiment is required

## Inputs

- gg_2024_09_training.fna.gz — 277,336 full-length 16S rRNA sequences (~1,500 bp) from Hugging Face systems-genomics-lab/greengenes
- gg_2024_09_training.tsv.gz — hierarchical taxonomy labels (domain through species) for the training split
- Architecture variant selector (HybridCNNBERTClassifier, CNNClassifier, or BERTClassifier) and compact hyperparameter configuration (Table S1), seed 42
- gg_2024_09_testing.fna.gz / gg_2024_09_testing.tsv.gz — 69,335-sequence stratified held-out test set shared by all variants

## Outputs

- Per-epoch model checkpoints (checkpoints/deeptaxa_<uuid>_epoch<N>.pt) with weights, optimizer state, scheduler state, and label encoders for each trained variant
- Per-epoch metrics JSON (metrics/deeptaxa_<uuid>_epoch<N>.json) with validation loss, accuracy, F1, precision, and recall at each of the seven ranks
- ablation_results.csv with species-level F1 per variant for comparison against the reported targets (full 0.9203, CNN-only 0.8931, BERT-only 0.0849, one-hot CNN 0.8428)

## How to apply

Download the Greengenes2 2024.09 curated FASTA/TAX split (277,336 training and 69,335 test sequences) from the Hugging Face Greengenes dataset repository, then train each variant with the same compact configuration and seed 42: AdamW (β1=0.9, β2=0.999, ε=1e-8, weight decay 0.01), batch size 64, dropout 0.2, peak LR 5.0e-4 with 10% linear warmup and linear decay, mixed precision, cross-entropy loss with uniform per-rank weights, 10 epochs. For the CNN-only variant, remove the BERT pathway and attach the seven classification heads directly to the CNN output; for the BERT-only variant, attach the heads to the [CLS] representation of a randomly initialized 4-layer, 7-head, 896-hidden transformer with a 3584-dim feedforward; for the one-hot CNN, replace DNABERT-2 BPE tokenization with raw one-hot nucleotide encodings while keeping the CNN pathway unchanged. Evaluate every variant on the identical 69,335-sequence stratified test set and compute species-level top-1 accuracy and F1, then compare against the reported targets (full hybrid 0.9203 at seed 42; CNN-only 0.8931, BERT-only 0.0849, one-hot CNN 0.8428 per Fig. 4) and record the per-variant values in ablation_results.csv.

## Related tools

- **DeepTaxa** (CLI framework used to train each architecture variant (HybridCNNBERTClassifier, CNNClassifier, BERTClassifier) under the compact configuration and to evaluate them on the held-out test set) — https://github.com/systems-genomics-lab/deeptaxa
- **DNABERT-2** (BPE tokenizer whose vocabulary is used to tokenize 16S sequences (truncated/padded to 512 tokens); replaced by raw one-hot nucleotide encodings in the one-hot CNN ablation variant)
- **PyTorch** (Underlying training engine providing the AdamW optimizer, mixed-precision training, and pickle-based checkpoint serialization used for all variants)

## Examples

```
deeptaxa train --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_training.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_training.tsv.gz --model-type hybridcnnbert --output-dir ../deeptaxa-outputs/
```

## Evaluation signals

- Species F1 of each variant falls near the reported targets on the same 69,335-sequence test set: full hybrid ≈0.9203 (seed 42), CNN-only ≈0.8931, BERT-only ≈0.0849, one-hot CNN ≈0.8428
- All variants are trained on the identical 277,336-sequence training split and evaluated on the identical 69,335-sequence stratified test set — any split mismatch invalidates the comparison
- Repeated runs at different seeds show cross-seed species F1 standard deviation ≤0.0008 at every rank, so a substantially higher deviation indicates a misconfigured run
- The compact-config full-hybrid species F1 (~0.92) should closely match the published 5-seed mean of 0.9212–0.9213; a large gap suggests the compact configuration was not applied
- Training produces the expected per-epoch metrics JSON (validation loss, accuracy, F1 at each rank) and checkpoints per epoch, with loss decreasing across the 10 epochs

## Limitations

- The BERT-only variant performs drastically worse at the species rank (F1 ≈0.0849) because the transformer is trained from scratch — it is provided mainly for ablation, not as a usable classifier
- The ablation was run only under the compact hyperparameter configuration (Table S1) with a single stratified train/test split and three training seeds rather than k-fold cross-validation
- Each training run takes ~1 h 20 m on an NVIDIA RTX 4090 (2 h 35 m on an A40), and exact results depend on seed, hardware, and package versions

## Evidence

- [methods] Ablation under compact config: "we trained four variants of DeepTaxa under the compact hyperparameter configuration (Table S1) on the same Greengenes2 2024.09 training set"
- [methods] Optimizer settings: "The model was trained using the AdamW optimizer (Loshchilov and Hutter 2019) with hyperparameters set to β1=0.9, β2=0.999, ε=1×10−8, and a weight decay of 0.01."
- [results] Learning-rate schedule: "The learning rate followed the linear warmup-and-decay schedule described in Methods, with a peak 5.0×10−4 at the end of warmup"
- [methods] Stratified split sizes: "The dataset was partitioned into training (277 336 sequences, 80%) and test (69 335 sequences, 20%) sets using stratified sampling to preserve label distributions."
- [readme] Ablation variants in README: [evidence span withheld — anchor status: not_found]
- [readme] Hybrid default architecture: [evidence span withheld — anchor status: not_found]
