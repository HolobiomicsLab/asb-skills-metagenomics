---
name: 16s-rrna-taxonomic-classification
description: Use when 16S rRNA FASTA sequences (full-length ~1,500 bp or in-silico
  V3–V4 ~420 bp amplicons) need hierarchical taxonomic assignment from domain to species
  against the Greengenes2 2024.09 reference.
when_to_use_negative:
- Input is a feature/abundance table or ASV matrix rather than raw 16S FASTA sequences
  — DeepTaxa operates on FASTA-formatted sequences only
- Sequences are not 16S rRNA (e.g., 18S rRNA or shotgun metagenomic data) — the paper
  states performance on other marker genes and modalities remains untested
- Base-level quality scores are essential to the analysis — the model does not incorporate
  quality scores and takes FASTA-only input
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: DeepTaxa
  role: 'Primary classifier: HybridCNNBERTClassifier loaded from a released checkpoint
    and run via the deeptaxa predict command to assign taxonomy at all seven ranks
    in one forward pass'
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: DNABERT-2
  role: BPE tokenizer that converts each 16S sequence into tokens truncated/padded
    to 512 tokens before model input
  repo: https://github.com/systems-genomics-lab/deeptaxa
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_001
  - task_002
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:9f56840f6c67b61914a134ba4ffc41cf2d75815e80c0095ca111843b5061026a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  - Kraken 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# 16s-rrna-taxonomic-classification

## Summary

Classifies 16S rRNA gene sequences into all seven Linnean ranks (domain through species) in a single forward pass using DeepTaxa's HybridCNNBERTClassifier, which fuses a multiscale CNN with a from-scratch transformer on the DNABERT-2 BPE vocabulary, reaching 92.96% species-level accuracy on the Greengenes2 2024.09 test set.

## When to use

Use when 16S rRNA FASTA sequences (full-length ~1,500 bp or in-silico V3–V4 ~420 bp amplicons) need hierarchical taxonomic assignment from domain to species against the Greengenes2 2024.09 reference.

## When NOT to use

- Input is a feature/abundance table or ASV matrix rather than raw 16S FASTA sequences — DeepTaxa operates on FASTA-formatted sequences only
- Sequences are not 16S rRNA (e.g., 18S rRNA or shotgun metagenomic data) — the paper states performance on other marker genes and modalities remains untested
- Base-level quality scores are essential to the analysis — the model does not incorporate quality scores and takes FASTA-only input

## Inputs

- 16S rRNA sequence FASTA (e.g., gg_2024_09_testing.fna.gz with 69,335 full-length sequences, or an in-silico V3–V4 ~420 bp amplicon FASTA)
- Taxonomy label TSV matching the FASTA (e.g., gg_2024_09_testing.tsv.gz) when evaluation metrics are desired
- Pre-trained DeepTaxa PyTorch checkpoint (.pt) from huggingface.co/systems-genomics-lab/deeptaxa, e.g., deeptaxa-full-length-v2.pt or deeptaxa-v3v4-v2.pt

## Outputs

- Per-rank top-1 taxonomic predictions for each input sequence (domain through species)
- Per-rank metrics (top-1 accuracy, weighted F1, ECE with 10 confidence bins) written to CSV, e.g., species_metrics.csv or v3v4_species_accuracy.csv
- Softmax probability distributions per rank usable as prediction-uncertainty (entropy) measures or for seed-ensemble averaging

## How to apply

Download the appropriate checkpoint from huggingface.co/systems-genomics-lab/deeptaxa (deeptaxa-full-length-v2.pt, 76.4 M parameters, for ~1,500 bp sequences; deeptaxa-v3v4-v2.pt, 75.8 M parameters with an 8,347-species label vocabulary, for ~420 bp V3–V4 amplicons). Tokenize each FASTA sequence with the DNABERT-2 BPE tokenizer, truncating or padding to a fixed length of 512 tokens. Run `deeptaxa predict` with the HybridCNNBERTClassifier on GPU under PyTorch mixed precision, applying softmax at temperature T = 1.0 to the seven rank classifier heads to get per-rank top-1 labels. When ground-truth taxonomy is supplied (e.g., gg_2024_09_testing.tsv.gz for the 69,335-sequence held-out test set), the tool computes per-rank top-1 accuracy, weighted F1, and expected calibration error with 10 equal-width confidence bins. For best accuracy and calibration on new data, download all five seed checkpoints for a region (seed 42, 123, 456, 789, 1011) and average their per-rank softmax probabilities via scripts/ensemble_predict.py before taking the argmax.

## Related tools

- **DeepTaxa** (Primary classifier: HybridCNNBERTClassifier loaded from a released checkpoint and run via the deeptaxa predict command to assign taxonomy at all seven ranks in one forward pass) — https://github.com/systems-genomics-lab/deeptaxa
- **DNABERT-2** (BPE tokenizer that converts each 16S sequence into tokens truncated/padded to 512 tokens before model input) — https://github.com/systems-genomics-lab/deeptaxa

## Examples

```
deeptaxa predict --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_testing.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_testing.tsv.gz --checkpoint ../deeptaxa-data/models/deeptaxa-full-length-v2.pt --output-dir ../deeptaxa-outputs/evaluation
```

## Evaluation signals

- On the 69,335-sequence Greengenes2 2024.09 held-out test set, species-level top-1 accuracy must land near 92.96% (paper 3-seed mean) or 92.95% (README v2 5-seed mean) within a small tolerance
- Species weighted F1 must be ~0.9212 (paper) / 0.9208 (README), with cross-seed standard deviation ≤0.0008 F1 at every rank
- Species ECE must reproduce ~0.0242 with 10 equal-width confidence bins
- With the dedicated V3–V4 checkpoint on ~420 bp amplicons, species accuracy must reproduce ~87.55% (paper) / 87.54% (README), F1 ~0.8592
- All 69,335 test sequences must receive predictions at every rank; per-rank F1 above 0.99 from domain through class

## Limitations

- DeepTaxa operates on FASTA-formatted sequences and does not incorporate base-level quality scores, which may matter for noisy amplicon datasets
- Trained and evaluated primarily on Greengenes2; performance on SILVA, RDP, or independent environmental/clinical datasets remains untested
- Performance degrades with decreasing training-set similarity: species F1 falls from 0.95 (high-similarity bucket) to 0.45 (low bucket), and unseen species labels show a monotonic accuracy decline from 99.8% (domain) to 48.9% (genus)
- Single stratified 80/20 train/test split with multiple seeds rather than k-fold cross-validation

## Evidence

- [abstract] DeepTaxa achieves species-level accuracy of 92.96% and F1 of 0.9212 (3-seed mean; cross-seed standard deviation ≤0.0008 F1 at every rank): "DeepTaxa achieves species-level accuracy of 92.96% and F1 of 0.9212 (3-seed mean; cross-seed standard deviation ≤0.0008 F1 at every rank)"
- [methods] Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens: "Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens"
- [methods] The dataset was partitioned into training (277 336 sequences, 80%) and test (69 335 sequences, 20%) sets using stratified sampling: "The dataset was partitioned into training (277 336 sequences, 80%) and test (69 335 sequences, 20%) sets using stratified sampling"
- [readme] DeepTaxa is a deep learning framework for hierarchical taxonomic classification of 16S rRNA gene sequences.: [evidence span withheld — anchor status: external_doc]
- [discussion] DeepTaxa currently operates on FASTA-formatted sequences and does not incorporate base-level quality scores: "DeepTaxa currently operates on FASTA-formatted sequences and does not incorporate base-level quality scores"
