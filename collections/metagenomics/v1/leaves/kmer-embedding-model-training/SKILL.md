---
name: kmer-embedding-model-training
description: Use when metagenomic reads (fastq/fasta) must be classified at species/genus
  rank by a deep learning model, or when choosing the k-mer length K for a k-mer embedding
  classifier.
when_to_use_negative:
- Input is already a feature/abundance table or assembled contigs — this skill classifies
  raw short sequencing reads via k-mer embeddings, not profiles
- Reads come from long-read platforms or have error profiles strongly different from
  the training data — the article states retraining would be required to adapt
- The goal is to add new species to an existing DeepMicrobes model without retraining
  — the article notes adding new species requires retraining the entire deep neural
  network
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3474
tools:
- name: DeepMicrobes
  role: Framework providing TFRecord conversion, k-mer embedding DNN training, GPU
    submission, and prediction routines used for each K variant
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: TensorFlow
  role: Backend for training the embedding-based recurrent self-attention DNN and
    the TFRecord data format
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:1e61d432014304b1f847fcfd4b0b81618f6ea0b92e9804d93abcec05b4c0ccf2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/MicrobeLab/DeepMicrobes
  repo_url: https://github.com/MicrobeLab/DeepMicrobes
  tools:
  - DeepMicrobes
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# kmer-embedding-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train the DeepMicrobes embedding-based recurrent self-attention DNN (Embed + LSTM + Attention) on simulated metagenomic reads and evaluate how the k-mer embedding length K affects read-level taxonomic classification, with 12-mers the validated optimum for gut-genome-scale training.

## When to use

Use when metagenomic reads (fastq/fasta) must be classified at species/genus rank by a deep learning model, or when choosing the k-mer length K for a k-mer embedding classifier.

## When NOT to use

- Input is already a feature/abundance table or assembled contigs — this skill classifies raw short sequencing reads via k-mer embeddings, not profiles
- Reads come from long-read platforms or have error profiles strongly different from the training data — the article states retraining would be required to adapt
- The goal is to add new species to an existing DeepMicrobes model without retraining — the article notes adding new species requires retraining the entire deep neural network

## Inputs

- fastq/fasta sequencing reads simulated from gut-species reference genomes (e.g. 2,505-species repertoire or 3,269 MAGs)
- k-mer vocabulary files for TFRecord conversion (merged k-mers, complementary k-mers sharing embeddings)
- Training labels embedded in sequence IDs of the genome collections
- Hyperparameter search space (Supplementary Table S8)

## Outputs

- Trained Embed + LSTM + Attention model parameters per k-mer length K
- Read-level predictions on held-out simulated reads
- Read-level precision and recall per K, assembled into a K-ablation comparison table/figure (cf. Supplementary Figures S7/S11, Tables S9–S12)

## How to apply

Simulate sequencing reads from the gut-species reference genomes (e.g. the 2,505-species gut bacterial repertoire or 3,269 MAGs) and convert the fastq/fasta reads to TFRecord format with the DeepMicrobes conversion utility, using the merged k-mer vocabulary so complementary k-mers share an embedding vector. For each k-mer size K in {8, 9, 10, 11, 12}, train the Embed + LSTM + Attention model with the DeepMicrobes training routine, submitting jobs to GPUs as documented, adopting the hyperparameters from the paper's search space (Supplementary Table S8). Run prediction on held-out simulated reads and compute read-level precision and recall per K. The decisive decision point is K: 12-mers must markedly outperform 8- to 11-mer variants (Supplementary Figures S7/S11), consistent with the finding that k-mer embedding significantly boosts model performance. Note the vocabulary size scales with K (Table S7), and the paper warns the 12-mer choice is optimized for thousands-of-genomes data volumes — for other scenarios try K in 6–12 bp.

## Related tools

- **DeepMicrobes** (Framework providing TFRecord conversion, k-mer embedding DNN training, GPU submission, and prediction routines used for each K variant) — https://github.com/MicrobeLab/DeepMicrobes
- **TensorFlow** (Backend for training the embedding-based recurrent self-attention DNN and the TFRecord data format)

## Examples

```
python DeepMicrobes/scripts/tfrecord.py --input reads.fq --kmer 12 --merged --output reads.tfrecord; python DeepMicrobes/scripts/train.py --model Embed+LSTM+Attention --kmer 12 --tfrecord reads.tfrecord
```

## Evaluation signals

- The 12-mer model must achieve markedly higher read-level precision and recall than 8-, 9-, 10-, and 11-mer variants, matching Supplementary Figures S7 and 11
- TFRecord conversion must use the merged k-mer vocabulary (complementary k-mers represented as the same embedding vector)
- Precision/recall computed on reads simulated from MAGs excluded from training (Tables S9–S12) should reproduce published values for the Embed + LSTM + Attention architecture
- Vocabulary size per K should match the relationship reported in Table S7 (effect of k-mer length on vocabulary size)
- LSTM-based architecture should outperform CNN variants under the same training regime

## Limitations

- Adding new species requires retraining the entire deep neural network
- The 12-mer length is optimized for typical data volumes of thousands of genomes; other scenarios (including long reads or different error profiles) may require trying different k-mer lengths (6–12 bp) and retraining
- Retraining might be required to adapt to reads whose lengths and error profiles are strongly different (e.g. long-read sequencing)

## Evidence

- [results] The final best architecture using 12-mers performs much better than the variants using 8-mers to 11-mers, as confirmed in Supplementary Figures S7 and 11.: [evidence span withheld — anchor status: not_found]
- [readme] The vocabulary of k-mers used in TFRecord conversion. The pre-trained DeepMicrobes models use 12-mers.: [evidence span withheld — anchor status: external_doc]
- [readme] Complementary k-mers are represented as the same embedding vector. Such representation requires merged k-mers when converting fasta/fastq sequences to TFRecord.: [evidence span withheld — anchor status: external_doc]
- [discussion] the k-mer length we used in this study is optimized for typical data volume of thousands of genomes: [evidence span withheld — anchor status: not_found]
