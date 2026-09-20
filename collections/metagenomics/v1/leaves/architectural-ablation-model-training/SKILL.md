---
name: architectural-ablation-model-training
description: Use when you need to isolate the contribution of individual model pathways
  (CNN vs transformer vs input tokenization) by training variant architectures on
  the same data/schedule and comparing species-level top-1 accuracy and F1.
when_to_use_negative:
- Your goal is the best possible species classification accuracy rather than attribution
  — use the full HybridCNNBERT model or the 5-seed soft-vote ensemble instead of ablation
  variants
- You changed more than one variable (data, split, epochs, seed) relative to the full-model
  run — F1 deltas would no longer isolate the architectural effect
- You need classification of non-16S marker genes (e.g., 18S rRNA or shotgun metagenomic
  data) — DeepTaxa was trained and evaluated primarily on Greengenes2 16S rRNA
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DeepTaxa
  role: Framework providing the HybridCNNBERTClassifier, CNNClassifier, and BERTClassifier
    architectures and the train/evaluate CLI used to run ablation variants under the
    compact configuration
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: DNABERT-2
  role: BPE tokenizer whose vocabulary the transformer pathway is trained from scratch
    on; replaced by raw one-hot nucleotide encodings in the one-hot CNN ablation variant
- name: PyTorch
  role: Deep learning backend for training the ablation variants with AdamW, mixed
    precision, and cross-entropy loss
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:3f091a487afc5dde06df405ab6245492df42d5394f4744705b7b0d91443ac465
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  - PyTorch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# architectural-ablation-model-training

## Summary

Trains architectural ablation variants of DeepTaxa (CNN-only, BERT-only, one-hot CNN) alongside the full HybridCNNBERT model under the identical compact hyperparameter configuration on Greengenes2 2024.09, to quantify each pathway's contribution to species-level F1.

## When to use

Use when you need to isolate the contribution of individual model pathways (CNN vs transformer vs input tokenization) by training variant architectures on the same data/schedule and comparing species-level top-1 accuracy and F1.

## When NOT to use

- Your goal is the best possible species classification accuracy rather than attribution — use the full HybridCNNBERT model or the 5-seed soft-vote ensemble instead of ablation variants
- You changed more than one variable (data, split, epochs, seed) relative to the full-model run — F1 deltas would no longer isolate the architectural effect
- You need classification of non-16S marker genes (e.g., 18S rRNA or shotgun metagenomic data) — DeepTaxa was trained and evaluated primarily on Greengenes2 16S rRNA

## Inputs

- Greengenes2 2024.09 training FASTA + taxonomy TSV (gg_2024_09_training.fna.gz / gg_2024_09_training.tsv.gz, 277,336 sequences)
- Greengenes2 2024.09 held-out test set (gg_2024_09_testing.fna.gz / gg_2024_09_testing.tsv.gz, 69,335 sequences)
- Ablation variant architecture choice (CNN-only, BERT-only, or one-hot CNN)
- Compact hyperparameter configuration (AdamW betas 0.9/0.999, eps 1e-8, weight decay 0.01, batch 64, dropout 0.2, 10% warmup, 10 epochs, seed 42)

## Outputs

- Trained ablation checkpoint(s) (checkpoints/deeptaxa_<uuid>_epoch<N>.pt with weights, optimizer state, scheduler state, label encoders)
- Per-epoch validation metrics JSON (loss, accuracy, F1, precision, recall at each rank)
- Species-level top-1 accuracy and F1 per ablation variant on the 69,335-sequence test set
- ablation_results.csv recording per-variant species F1 vs reported targets

## How to apply

Download the curated Greengenes2 2024.09 dataset (346,671 sequences; 277,336 train / 69,335 stratified test) from the Hugging Face Greengenes dataset repository. Train each ablation variant — CNN-only (BERT pathway removed, seven classification heads attached directly to the CNN output), BERT-only (CNN pathway removed, heads on the [CLS] token of a randomly initialized 4-layer, 7-head, 896-hidden transformer with 3584-dim FFN), and one-hot CNN (DNABERT-2 BPE tokenization replaced with raw one-hot nucleotide encodings) — using the compact configuration: AdamW (beta1=0.9, beta2=0.999, eps=1e-8, weight decay 0.01), batch size 64, dropout 0.2, 10% warmup with linear warmup-and-decay schedule peaking at 5.0e-4, mixed precision, cross-entropy loss with uniform per-rank weights, 10 epochs, seed 42. Evaluate each variant on the same 69,335-sequence held-out test set, computing species-level top-1 accuracy and F1, and compare against the reported targets (full hybrid 0.9203, CNN-only 0.8931, BERT-only 0.0849, one-hot CNN 0.8428). The decision point is keeping all other variables (data, split, schedule, seed) fixed so F1 deltas are attributable solely to architecture.

## Related tools

- **DeepTaxa** (Framework providing the HybridCNNBERTClassifier, CNNClassifier, and BERTClassifier architectures and the train/evaluate CLI used to run ablation variants under the compact configuration) — https://github.com/systems-genomics-lab/deeptaxa
- **DNABERT-2** (BPE tokenizer whose vocabulary the transformer pathway is trained from scratch on; replaced by raw one-hot nucleotide encodings in the one-hot CNN ablation variant)
- **PyTorch** (Deep learning backend for training the ablation variants with AdamW, mixed precision, and cross-entropy loss)

## Examples

```
deeptaxa train --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_training.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_training.tsv.gz --model-type cnn --output-dir ../deeptaxa-outputs/
```

## Evaluation signals

- Species F1 of the CNN-only variant should land near the reported target 0.8931, BERT-only near 0.0849, and one-hot CNN near 0.8428 (Fig. 4 targets)
- Full hybrid reference under the same compact configuration and seed 42 should reproduce species F1 ≈ 0.9203
- All variants must be evaluated on the identical 69,335-sequence held-out test set with the same stratified split
- Per-epoch metrics JSON files should show 10 epochs completed with AdamW hyperparameters (beta1=0.9, beta2=0.999, eps=1e-8, weight decay 0.01) and batch size 64
- Cross-seed standard deviation of the full model should stay ≤0.0008 F1 at every rank, indicating the comparison is stable

## Limitations

- BERT-only ablation uses a randomly initialized transformer, so its very low species F1 (0.0849) reflects training from scratch without pretraining rather than an upper bound on transformer capacity
- Ablations were run under the compact hyperparameter configuration (Table S1), not the full tuning pipeline (Optuna TPE + LR grid search), so absolute values may differ from the published checkpoint
- Results depend on the single stratified train/test split and training seed; the paper notes k-fold cross-validation was not used
- Exact numbers still depend on seed, hardware, and package versions per the README

## Evidence

- [methods] Architectural ablation via compact-configuration variant training: "we trained four variants of DeepTaxa under the compact hyperparameter configuration (Table S1) on the same Greengenes2 2024.09 training set"
- [methods] The model was trained using the AdamW optimizer (Loshchilov and Hutter 2019) with hyperparameters set to β1=0.9, β2=0.999, ε=1×10−8, and a weight decay of 0.01.: "The model was trained using the AdamW optimizer (Loshchilov and Hutter 2019) with hyperparameters set to β1=0.9, β2=0.999, ε=1×10−8, and a weight decay of 0.01."
- [readme] CNN vs BERT pathway roles for ablation: [evidence span withheld — anchor status: external_doc]
- [readme] Compact training configuration: [evidence span withheld — anchor status: external_doc]
