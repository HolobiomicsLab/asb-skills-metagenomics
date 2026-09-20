---
name: mock-community-benchmark-evaluation
description: Use when comparing taxonomic classification tools on simulated mock communities
  with known ground-truth abundance profiles, to quantify genus/species identification,
  abundance estimation accuracy, and read classification rate.
when_to_use_negative:
- Input is real-world metagenome data without ground-truth abundance profiles (e.g.
  the IBD/iHMP dataset), where precision/recall versus truth and L2 abundance distance
  cannot be computed
- Goal is classifying novel or long-read data whose error profiles and lengths differ
  from the training data — DeepMicrobes requires retraining for such reads
- The reference database must incorporate newly sequenced species without retraining
  — the current DNN framework requires retraining the entire network when new species
  are added
edam_operation: http://edamontology.org/operation_0392
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: DeepMicrobes
  role: Deep learning classifier under evaluation; converts fastq/fasta to TFRecord,
    predicts reads on GPU, and generates taxonomic profiles
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: Kraken
  role: k-mer/LCA-based comparator classifier run on the same mock-community reads
- name: Kaiju
  role: Protein-based comparator (second most sensitive tool, 68.27% mean read classification)
    run against a custom database of predicted proteins from the human gut bacterial
    repertoire
- name: R
  role: 'Benchmarking: R scripts aggregate metrics across tools/communities and generate
    the comparison figures'
  repo: https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:7961366f7a5828a40eb24dd9881b843013ad20f3390e89bd90086614e881a162
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
  edam_operation: http://edamontology.org/operation_0392
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# mock-community-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a taxonomic classifier against state-of-the-art tools (Kraken, Kraken 2, Centrifuge, CLARK, CLARK-S, Kaiju, DIAMOND-MEGAN, BLAST-MEGAN) on ten mock communities built from HBC gut isolate genomes, using ground-truth genus profiles to compute precision/recall, L2 abundance distance, and read classification rate.

## When to use

Use when comparing taxonomic classification tools on simulated mock communities with known ground-truth abundance profiles, to quantify genus/species identification, abundance estimation accuracy, and read classification rate.

## When NOT to use

- Input is real-world metagenome data without ground-truth abundance profiles (e.g. the IBD/iHMP dataset), where precision/recall versus truth and L2 abundance distance cannot be computed
- Goal is classifying novel or long-read data whose error profiles and lengths differ from the training data — DeepMicrobes requires retraining for such reads
- The reference database must incorporate newly sequenced species without retraining — the current DNN framework requires retraining the entire network when new species are added

## Inputs

- fastq/fasta sequencing reads of ten mock communities from HBC gastrointestinal isolates (ENA ERP105624, ERP012217)
- Ground-truth genus profiles of the ten mock communities (Table S4) and true relative abundances of the 14 shared species members (Table S5)
- Custom Kaiju database of predicted protein sequences from the complete bacterial repertoire of human gut
- Command lines and R benchmarking/figure scripts from MicrobeLab/DeepMicrobes-data

## Outputs

- Per-tool taxonomic profiles (per-genus abundances and read-level assignments) for each of the ten mock communities
- Community-level precision and recall at 0.01% and 0.0001% abundance cutoffs per tool
- L2 distances between predicted and ground-truth abundance profiles
- Read classification rates per tool (e.g. DeepMicrobes 89.40% vs Kaiju 68.27%)
- Final comparison table (benchmark_results.csv) and benchmark figures

## How to apply

Download sequencing reads for the ten simulated mock communities constructed from HBC isolate genomes (ENA accessions ERP105624 and ERP012217) and convert each fastq/fasta dataset to TFRecord with the DeepMicrobes conversion utility. Run DeepMicrobes prediction on GPU for each community and run the eight comparison classifiers on the same reads, generating per-tool taxonomic profiles (read-level assignments plus per-genus abundances) via the DeepMicrobes profiling pipeline. Compute community-level precision and recall at 0.01% and 0.0001% abundance cutoffs, L2 distances between predicted and ground-truth abundance profiles (ground truth in Table S4/S5), and per-tool read classification rates. Aggregate the metrics with the R benchmarking scripts from the DeepMicrobes-data repository and accept the reproduction if DeepMicrobes' read classification rate matches the reported 89.40% versus Kaiju's 68.27% and the precision/recall values reproduce Figure 3 at both abundance cutoffs.

## Related tools

- **DeepMicrobes** (Deep learning classifier under evaluation; converts fastq/fasta to TFRecord, predicts reads on GPU, and generates taxonomic profiles) — https://github.com/MicrobeLab/DeepMicrobes
- **Kraken** (k-mer/LCA-based comparator classifier run on the same mock-community reads)
- **Kaiju** (Protein-based comparator (second most sensitive tool, 68.27% mean read classification) run against a custom database of predicted proteins from the human gut bacterial repertoire)
- **R** (Benchmarking: R scripts aggregate metrics across tools/communities and generate the comparison figures) — https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts

## Evaluation signals

- DeepMicrobes' mean read classification rate across the ten mock communities reaches ~89.40%, versus ~68.27% for Kaiju (second best)
- Community-level precision and recall computed at both the 0.01% and 0.0001% abundance cutoffs reproduce the Figure 3 values
- L2 distances between predicted and ground-truth genus abundance profiles (Table S4/S5 truth) are comparable to or lower than the eight comparator tools
- Every tool is run on the identical reads of all ten mock communities, with profiles generated by the same profiling pipeline and metrics aggregated by the DeepMicrobes-data R scripts
- The final comparison table (benchmark_results.csv) contains per-tool metrics for all ten communities at both cutoffs

## Limitations

- Adding new species to the classifier requires retraining the entire deep neural network, so benchmarks cannot be extended to novel genomes without retraining
- The k-mer length (12-mers for pre-trained models) is optimized for typical data volumes of thousands of genomes; other scenarios should try k-mer lengths of 6-12 bp
- Reads whose lengths and error profiles differ strongly from training data (e.g. long-read sequencing) require retraining before benchmarking
- Comparator tools need custom databases built from the same genome set (e.g. Kaiju requires the custom predicted-protein database) for a fair comparison on shared species (Table S5: 14 species shared across nine tools' databases)

## Evidence

- [results] The DeepMicrobes genus model classified on average 89.40% of reads in the mock communities, higher than the second most sensitive tool, Kaiju, which classified on average 68.27% of reads.: [evidence span withheld — anchor status: not_found]
- [readme] Mock communities generated from gastrointestinal bacterial isolates: [evidence span withheld — anchor status: external_doc]
- [readme] Command lines used to run all the taxonomic classification tools benchmarked in the paper: [evidence span withheld — anchor status: external_doc]
