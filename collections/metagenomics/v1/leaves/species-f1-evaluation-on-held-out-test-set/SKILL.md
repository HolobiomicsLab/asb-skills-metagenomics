---
name: species-f1-evaluation-on-held-out-test-set
description: Use when reproducing or validating the architectural ablation species-F1
  table (CNN-only 0.8931, BERT-only 0.0849, one-hot CNN 0.8428) or when quantifying
  how removing the BERT pathway affects species-level classification on Greengenes2
  2024.09.
when_to_use_negative:
- Input sequences are not 16S rRNA or not from the Greengenes2 2024.09-style stratified
  80/20 split — the reported ablation targets are specific to this test distribution
- The goal is comparing DeepTaxa against external classifiers (DADA2, QIIME 2, Kraken2,
  SINTAX) rather than internal architectural ablations
- Evaluating a model on training data or the validation split — the skill requires
  the untouched 69,335-sequence held-out test set
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3474
tools:
- name: DeepTaxa
  role: Framework providing the CNNClassifier, BERTClassifier, and HybridCNNBERTClassifier
    variants, the CLI predict/evaluate mode with per-rank accuracy/F1, and the training
    entry point for ablation runs
  repo: github.com/systems-genomics-lab/deeptaxa
- name: DNABERT-2
  role: BPE tokenizer whose vocabulary is used for sequence tokenization (512 tokens);
    removed and replaced by one-hot encodings in the one-hot CNN ablation
- name: PyTorch
  role: Underlying deep-learning framework for training the ablation variants with
    mixed precision and AdamW and for serializing the checkpoints
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:80623757c85d324d7d435f0c49a532f25f54a50010e4106d4358e7e165a5eaf5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNABERT-2
  - DeepTaxa
  - PyTorch
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# species-f1-evaluation-on-held-out-test-set

## Summary

Evaluates DeepTaxa model variants (full HybridCNNBERT, CNN-only, BERT-only, one-hot CNN) on the 69,335-sequence held-out Greengenes2 2024.09 test set and records species-level top-1 accuracy and F1 for comparison against the paper's ablation targets.

## When to use

Use when reproducing or validating the architectural ablation species-F1 table (CNN-only 0.8931, BERT-only 0.0849, one-hot CNN 0.8428) or when quantifying how removing the BERT pathway affects species-level classification on Greengenes2 2024.09.

## When NOT to use

- Input sequences are not 16S rRNA or not from the Greengenes2 2024.09-style stratified 80/20 split — the reported ablation targets are specific to this test distribution
- The goal is comparing DeepTaxa against external classifiers (DADA2, QIIME 2, Kraken2, SINTAX) rather than internal architectural ablations
- Evaluating a model on training data or the validation split — the skill requires the untouched 69,335-sequence held-out test set

## Inputs

- gg_2024_09_testing.fna.gz (69,335 held-out test sequences, Greengenes2 2024.09)
- gg_2024_09_testing.tsv.gz (taxonomy labels for the test set)
- Trained ablation checkpoints (CNN-only, BERT-only, one-hot CNN, full HybridCNNBERT) from the compact configuration

## Outputs

- ablation_results.csv with per-variant species-level top-1 accuracy and F1
- Comparison against reported targets: CNN-only 0.8931, BERT-only 0.0849, one-hot CNN 0.8428, full hybrid 0.9203 (seed 42)

## How to apply

Train each ablation variant under the compact configuration — AdamW (beta1=0.9, beta2=0.999, eps=1e-8, weight decay 0.01), batch size 64, dropout 0.2, 10% warmup, mixed precision, cross-entropy with uniform per-rank weights, 10 epochs, seed 42 — on the 277,336-sequence stratified training split of Greengenes2 2024.09. The CNN-only variant attaches seven classification heads directly to the CNN output; the BERT-only variant attaches heads to the [CLS] representation of a randomly initialized 4-layer, 7-head, 896-hidden transformer with 3584-dim feedforward; the one-hot CNN replaces DNABERT-2 BPE tokenization with raw one-hot nucleotide encodings. Run each trained checkpoint on the identical 69,335-sequence held-out test set (gg_2024_09_testing.fna.gz) and compute species-level top-1 accuracy and F1, then record values in ablation_results.csv and compare against the reported targets (full model 0.9203 at seed 42, CNN-only 0.8931, BERT-only 0.0849, one-hot CNN 0.8428 as in Fig. 4). The full hybrid model's fusion of CNN local motif features with BERT global context is expected to outperform any single-pathway ablation at the species rank.

## Related tools

- **DeepTaxa** (Framework providing the CNNClassifier, BERTClassifier, and HybridCNNBERTClassifier variants, the CLI predict/evaluate mode with per-rank accuracy/F1, and the training entry point for ablation runs) — github.com/systems-genomics-lab/deeptaxa
- **DNABERT-2** (BPE tokenizer whose vocabulary is used for sequence tokenization (512 tokens); removed and replaced by one-hot encodings in the one-hot CNN ablation)
- **PyTorch** (Underlying deep-learning framework for training the ablation variants with mixed precision and AdamW and for serializing the checkpoints)

## Examples

```
deeptaxa train --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_training.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_training.tsv.gz --model-type cnn --output-dir ../deeptaxa-outputs/
```

## Evaluation signals

- CNN-only species F1 reproduces 0.8931 and full HybridCNNBERT reproduces 0.9203 at seed 42 under the compact configuration
- BERT-only species F1 is drastically low (~0.0849), confirming a randomly initialized from-scratch transformer underperforms at the species rank
- One-hot CNN species F1 lands near 0.8428, showing BPE tokenization contributes above raw one-hot encodings
- All variants are evaluated on the identical 69,335-sequence test split with stratified label distributions preserved
- Species accuracy for the full model falls in the ~92.9–93.0% range consistent with published checkpoint metrics

## Limitations

- Results are seed- and hardware-dependent; the paper reports a single stratified split with three training seeds rather than k-fold cross-validation
- Compact configuration (10 epochs, reduced capacity) yields the ablation figures and may not match the full published checkpoint performance (F1 0.9212, 3-seed mean)
- The BERT-only pathway uses a randomly initialized transformer; its very low species F1 (0.0849) reflects insufficient training rather than a general property of transformers

## Evidence

- [methods] Ablation target figures from the source card: "we trained four variants of DeepTaxa under the compact hyperparameter configuration (Table S1) on the same Greengenes2 2024.09 training set"
- [methods] Test split size from the article: "The dataset was partitioned into training (277 336 sequences, 80%) and test (69 335 sequences, 20%) sets using stratified sampling to preserve label distributions."
- [readme] README description of CNN-only variant role: [evidence span withheld — anchor status: not_found]
- [readme] Compact training hyperparameters from the README: [evidence span withheld — anchor status: external_doc]
