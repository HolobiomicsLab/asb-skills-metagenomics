---
name: architecture-variant-benchmarking
description: Use when choosing among candidate DNN architectures for classifying short
  metagenomic sequencing reads and you need head-to-head read-level precision/recall
  on held-out genomes to justify the selection.
when_to_use_negative:
- The task is abundance estimation or genus-level profiling of mock communities rather
  than architecture selection by read-level precision/recall
- Reads have lengths and error profiles strongly different from the training data
  (e.g. long-read sequencing), where retraining is required and benchmarked short-read
  architectures may not transfer
- Reference database curation output (e.g. incorporating new MAGs into Kraken/Kaiju
  databases) is the goal rather than training and comparing neural architectures
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3474
tools:
- name: DeepMicrobes
  role: Framework providing the DNN architecture implementations, TFRecord conversion
    utility, training and prediction workflows used to benchmark architecture variants
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: MicrobeLab/DeepMicrobes-data
  role: Source of simulated training reads (2505 genomes), held-out MAG test reads
    (3269 MAGs), vocabulary, and supplementary precision/recall tables S9-S12
  repo: https://github.com/MicrobeLab/DeepMicrobes-data
- name: R
  role: R scripts used to generate benchmarking figures from per-architecture precision/recall
    results
  repo: https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:1ba2c81a5ad1ae9287153c2d04f696fc8ff1c2f3054b39c9ff6580202ff5621f
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
  - MicrobeLab/DeepMicrobes-data
  - R
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# architecture-variant-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multiple deep neural network architectures (one-hot CNN, hybrid, seq2species, embedding baseline, LSTM, and self-attention variants) on identical training and held-out test reads to select the best architecture for a sequence-classification task, as done for DeepMicrobes species-level metagenomic read classification.

## When to use

Use when choosing among candidate DNN architectures for classifying short metagenomic sequencing reads and you need head-to-head read-level precision/recall on held-out genomes to justify the selection.

## When NOT to use

- The task is abundance estimation or genus-level profiling of mock communities rather than architecture selection by read-level precision/recall
- Reads have lengths and error profiles strongly different from the training data (e.g. long-read sequencing), where retraining is required and benchmarked short-read architectures may not transfer
- Reference database curation output (e.g. incorporating new MAGs into Kraken/Kaiju databases) is the goal rather than training and comparing neural architectures

## Inputs

- Simulated training reads from 2505 human gut reference genomes (FASTQ/FASTA, labels in sequence IDs)
- Held-out test reads simulated from 3269 gut-derived MAGs excluded from training (FASTQ/FASTA)
- Merged 12-mer k-mer vocabulary for TFRecord conversion
- TFRecord-formatted training and test sets

## Outputs

- Trained model checkpoints for each DNN architecture variant
- Per-architecture read-level precision and recall on held-out MAG reads (cf. Tables S9-S10)
- Selected best-performing architecture (Embed + LSTM + Attention for DeepMicrobes)

## How to apply

Convert FASTQ/FASTA sequence files to TFRecord format with the DeepMicrobes tfrecord conversion utility, using the merged k-mer vocabulary (12-mers for the pre-trained models). Train each architecture variant (one-hot CNN, hybrid, seq2species, embedding baseline, LSTM, and the k-mer embedding + BiLSTM + self-attention model) on TFRecords simulated from the 2505 human gut reference genomes, submitting jobs to GPUs per the DeepMicrobes GPU guide. Run prediction with each trained model on variable-length reads held out from 3269 gut-derived MAGs excluded from training, then aggregate read-level precision and recall per architecture. Select the architecture with the highest read-level scores; in the source study the Embed + LSTM + Attention model won with mean precision 0.942 and mean recall 0.428, confirming that k-mer embedding boosts performance and LSTM surpasses CNN.

## Related tools

- **DeepMicrobes** (Framework providing the DNN architecture implementations, TFRecord conversion utility, training and prediction workflows used to benchmark architecture variants) — https://github.com/MicrobeLab/DeepMicrobes
- **MicrobeLab/DeepMicrobes-data** (Source of simulated training reads (2505 genomes), held-out MAG test reads (3269 MAGs), vocabulary, and supplementary precision/recall tables S9-S12) — https://github.com/MicrobeLab/DeepMicrobes-data
- **R** (R scripts used to generate benchmarking figures from per-architecture precision/recall results) — https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts

## Examples

```
python deepmicrobes/convert_seq2tfrecord.py --input reads.fa --vocabulary vocab/merged_12mers.txt --output train.tfrecord
```

## Evaluation signals

- The embedding + BiLSTM + self-attention architecture attains the highest read-level precision (mean 0.942) and recall (mean 0.428) across variants, matching Figure 2 / Tables S9-S10
- All architecture variants are trained and evaluated on identical TFRecord training data and the same held-out 3269-MAG test reads
- Read-level precision and recall are reported per architecture on variable-length reads excluded from training
- The LSTM-based variants outperform the one-hot CNN, consistent with the article's finding that LSTM surpasses CNN and k-mer embedding boosts performance
- Hyperparameters for each model match the search space and selections documented in Table S8

## Limitations

- The k-mer length (12-mers for pre-trained models) is optimized for typical data volume of thousands of genomes; other scenarios require trying different k-mer lengths (6-12 bp)
- Adding new species requires retraining the entire deep neural network
- Retraining might be required to adapt to reads whose lengths and error profiles are strongly different (e.g. long-read sequencing)
- Benchmark uses simulated reads from gut-derived MAGs; relative recall (0.428) is far below precision, so overall sensitivity on novel organisms remains limited

## Evidence

- [results] The embedding-based recurrent self-attention model achieves higher read-level precision (mean = 0.942) and recall (mean = 0.428) than the second-best LSTM variant: [evidence span withheld — anchor status: not_found]
- [discussion] k-mer embedding boosts performance: "We found that k-mer embedding significantly boosts model performance."
- [discussion] LSTM surpasses CNN: "Our finding that LSTM surpasses CNN highlights the importance of order and context of oligonucleotides in taxonomic classification."
- [readme] Read-level precision of different model architectures on variable-length reads simulated from 3,269 MAGs excluded from training: [evidence span withheld — anchor status: external_doc]
- [methods] How to submit to GPUs documented in the repo: "* [How to submit to GPUs](https://github.com/MicrobeLab/DeepMicrobes/blob/master/document/gpu.md)"
