---
name: metagenomic-read-classification
description: Use when classifying fastq/fasta metagenomic reads at species or genus
  rank, especially gut microbiome reads from uncultured MAGs where k-mer/LCA tools
  miss species absent from reference databases.
when_to_use_negative:
- The input is long-read or single-cell data whose read lengths and error profiles
  differ strongly from the training data — the paper notes retraining would be required
  to adapt
- New genomes must be incorporated into an existing model without retraining — the
  framework requires retraining the entire deep neural network to add species
- The reads come from organisms outside the gut repertoire domain used for training,
  where k-mer length (12-mers, optimized for thousands of genomes; try 6-12 bp otherwise)
  may not generalize
edam_operation: http://edamontology.org/operation_0269
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3474
tools:
- name: DeepMicrobes
  role: Deep learning framework that converts reads to TFRecord, predicts taxonomy
    with an embedding+BiLSTM+self-attention model, and generates taxonomic profiles
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: Kraken
  role: k-mer/LCA-based classifier used as the state-of-the-art comparison tool in
    genus/species benchmarking
- name: Kaiju
  role: Protein-based classifier run on a custom database of proteins predicted from
    the 2505 gut genomes; used as the second-most-sensitive benchmark
- name: R
  role: Runs the benchmarking scripts (https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts)
    that generate comparison figures and tables
  repo: https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:f65e92a0751f8fc950a9111135eb3cefe7988376d993ea2c236b1bc7b5934f96
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
  - Kaiju
  - Kraken
  - R
  edam_operation: http://edamontology.org/operation_0269
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metagenomic-read-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Taxonomic classification of short shotgun metagenomic sequencing reads using DeepMicrobes, a deep learning framework trained on the complete bacterial repertoire of the human gut, and benchmarking it against k-mer/LCA-based (Kraken, Kraken 2, Centrifuge, CLARK) and protein-alignment-based (Kaiju, DIAMOND-MEGAN, BLAST-MEGAN) classifiers.

## When to use

Use when classifying fastq/fasta metagenomic reads at species or genus rank, especially gut microbiome reads from uncultured MAGs where k-mer/LCA tools miss species absent from reference databases.

## When NOT to use

- The input is long-read or single-cell data whose read lengths and error profiles differ strongly from the training data — the paper notes retraining would be required to adapt
- New genomes must be incorporated into an existing model without retraining — the framework requires retraining the entire deep neural network to add species
- The reads come from organisms outside the gut repertoire domain used for training, where k-mer length (12-mers, optimized for thousands of genomes; try 6-12 bp otherwise) may not generalize

## Inputs

- FASTQ/FASTA short shotgun metagenomic sequencing reads
- Trained DeepMicrobes species-level or genus-level LSTM DNN model
- k-mer vocabulary (12-mers) for TFRecord conversion
- Reference genome collections (e.g. 2505 human gut species genomes; 3269 gut-derived MAGs)

## Outputs

- Per-read taxonomic assignments (species/genus calls with confidence scores)
- Taxonomic abundance profiles (per-species/per-genus relative abundances)
- Benchmark tables of precision, recall, L2 abundance distance and read classification rate

## How to apply

Convert FASTQ/FASTA reads to TFRecord format using DeepMicrobes' conversion utility, remembering that complementary 12-mers share one embedding vector so 'merged' k-mers are required. Run prediction with the pre-trained embedding + BiLSTM + self-attention species or genus model, applying the documented confidence-threshold procedure to filter low-confidence calls. Aggregate per-read predictions into taxonomic profiles with the DeepMicrobes profiling pipeline, then benchmark precision/recall (e.g. at 0.01% and 0.0001% abundance cutoffs) and read classification rates against Kraken, Kaiju, CLARK, Centrifuge and MEGAN-based tools using the R scripts from DeepMicrobes-data. The model was trained on 2505 human gut reference genomes, and validation typically checks that DeepMicrobes outperforms the next-best tool (e.g. 89.40% vs Kaiju's 68.27% of mock-community reads classified at genus level).

## Related tools

- **DeepMicrobes** (Deep learning framework that converts reads to TFRecord, predicts taxonomy with an embedding+BiLSTM+self-attention model, and generates taxonomic profiles) — https://github.com/MicrobeLab/DeepMicrobes
- **Kraken** (k-mer/LCA-based classifier used as the state-of-the-art comparison tool in genus/species benchmarking)
- **Kaiju** (Protein-based classifier run on a custom database of proteins predicted from the 2505 gut genomes; used as the second-most-sensitive benchmark)
- **R** (Runs the benchmarking scripts (https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts) that generate comparison figures and tables) — https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts

## Evaluation signals

- Genus-level read classification rate on the ten HBC mock communities reaches ~89.40% for DeepMicrobes versus ~68.27% for Kaiju
- Architecture-selection benchmark on 3269 ERP108418 MAG reads yields mean precision 0.942 and mean recall 0.428 for the embedding+BiLSTM+self-attention model
- Precision/recall at 0.01% and 0.0001% abundance cutoffs reproduce the Figure 3 genus-level benchmark values
- TFRecord conversion used the merged 12-mer vocabulary so complementary k-mers share an embedding vector
- Species profiles on the IBD dataset reproduce the Table S14 LEfSe biomarkers (e.g. UMGS368 depleted in CD/UC) after Kruskal-Wallis + LDA effect size >2.0 filtering

## Limitations

- Adding new species requires retraining the entire deep neural network
- Retraining might be required to adapt to reads whose lengths and error profiles are strongly different (e.g. long-read sequencing)
- The k-mer length used is optimized for thousands of genomes; other scenarios should try different k-mer lengths (e.g. 6-12 bp)
- Fragmented MAGs degrade alignment-based and k-mer-based classifier performance, and incorporating MGS into reference databases for other tools is difficult and time-consuming

## Evidence

- [readme] complementary k-mers share embedding vector; 12-mers: [evidence span withheld — anchor status: external_doc]
