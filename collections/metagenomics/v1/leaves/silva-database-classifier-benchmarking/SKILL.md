---
name: silva-database-classifier-benchmarking
description: Use when reconstructed or reference SSU rRNA sequences (e.g. 16S from
  a mock community) must be classified by several taxonomic classifiers against SILVA
  SSU 138.1 and their species-level performance compared quantitatively.
when_to_use_negative:
- Input reads are short Illumina reads rather than reconstructed full-length SSU sequences
  — the article showed direct classification of short reads infers taxa with a high
  rate of false positives
- The goal is taxonomic profiling/abundance estimation with RiboTaxa itself rather
  than benchmarking multiple classifiers head-to-head
- A non-SILVA database (RDP, Greengenes, GTDB, UNITE) is required without retraining
  or database preparation
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: Centrifuge
  role: General-purpose classifier benchmarked on reconstructed SSU sequences against
    SILVA SSU 138.1
- name: Kraken2
  role: Short-read classifier benchmarked as a comparison against RiboTaxa/Sklearn
    results
- name: SPINGO
  role: Species-level amplicon classifier included in the seven-way benchmark
  repo: https://github.com/GuyAllard/SPINGO
- name: QIIME 2 Sklearn classifier
  role: Classify full-length reconstructed SSU sequences using the trained SILVA 138
    classifier with confidence threshold 0.7; best species-level performer (91.9%
    F-measure)
  repo: https://github.com/oschakoory/RiboTaxa
- name: mothur
  role: Amplicon classifier benchmarked at each taxonomic level
- name: RDP Classifier
  role: Bayesian classifier benchmarked against the SILVA database
- name: BLCA
  role: Alignment-based classifier benchmarked on reconstructed SSU sequences
- name: VSEARCH
  role: Used in the RiboTaxa pipeline for taxonomic identification of assembled SSU
    sequences
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:2d18f43593a005813d4cf3ea556050da2602ca3b3da1f1a995e6929b86217a32
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  repo_url: https://github.com/GuyAllard/SPINGO
  tools:
  - BLCA
  - Centrifuge
  - Kraken2
  - QIIME 2 Sklearn classifier
  - RDP Classifier
  - SPINGO
  - VSEARCH
  - mothur
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# silva-database-classifier-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks multiple taxonomic classifiers (Centrifuge, Kraken2, mothur, RDP Classifier, SPINGO, BLCA, QIIME 2's Sklearn) against the SILVA SSU 138.1 NR99 database to classify reconstructed SSU rRNA sequences from a mock community and compare precision, recall and F-measure per taxonomic level.

## When to use

Use when reconstructed or reference SSU rRNA sequences (e.g. 16S from a mock community) must be classified by several taxonomic classifiers against SILVA SSU 138.1 and their species-level performance compared quantitatively.

## When NOT to use

- Input reads are short Illumina reads rather than reconstructed full-length SSU sequences — the article showed direct classification of short reads infers taxa with a high rate of false positives
- The goal is taxonomic profiling/abundance estimation with RiboTaxa itself rather than benchmarking multiple classifiers head-to-head
- A non-SILVA database (RDP, Greengenes, GTDB, UNITE) is required without retraining or database preparation

## Inputs

- Reconstructed/clustered SSU rRNA sequences in FASTA format (e.g. metagenome_SSU_sequences.fasta from EMIRGE/MetaRib)
- 16S rRNA reference sequences with known taxonomy from the microbial mock community
- SILVA SSU 138.1 NR99 database (510,508 SSU rRNA gene sequences)
- Trained SILVA 138 QIIME 2 reference classifier

## Outputs

- Per-classifier taxonomic assignments at domain to species level
- benchmark_precision_recall_fmeasure.tsv with precision, recall and F-measure per classifier per taxonomic level
- Comparison table reproducing Figure 4A/4B classifier benchmark

## How to apply

Load the reconstructed/clustered SSU rRNA FASTA (e.g. metagenome_SSU_sequences.fasta from EMIRGE/MetaRib reconstruction) together with the known 16S reference taxonomy of the mock community. Run each classifier — Centrifuge, Kraken2, mothur, RDP Classifier, SPINGO, BLCA and QIIME 2's classify-sklearn — against the SILVA SSU 138.1 NR99 database (510,508 SSU sequences); for the Sklearn classifier use the SILVA 138 trained classifier from the QIIME 2 Data resources with a confidence threshold of 0.7 to limit taxonomic depth. Bin each assignment as true positive, false positive or false negative at domain, phylum, class, order, family, genus and species levels versus the expected mock community taxonomy. Compute precision, recall and F-measure at each level per classifier and compile the results into a comparison table (benchmark_precision_recall_fmeasure.tsv) reproducing Figure 4A/4B. Rationale: the article showed reconstructed full-length sequences classify far more reliably than short reads, which yield high false-positive rates.

## Related tools

- **Centrifuge** (General-purpose classifier benchmarked on reconstructed SSU sequences against SILVA SSU 138.1)
- **Kraken2** (Short-read classifier benchmarked as a comparison against RiboTaxa/Sklearn results)
- **SPINGO** (Species-level amplicon classifier included in the seven-way benchmark) — https://github.com/GuyAllard/SPINGO
- **QIIME 2 Sklearn classifier** (Classify full-length reconstructed SSU sequences using the trained SILVA 138 classifier with confidence threshold 0.7; best species-level performer (91.9% F-measure)) — https://github.com/oschakoory/RiboTaxa
- **mothur** (Amplicon classifier benchmarked at each taxonomic level)
- **RDP Classifier** (Bayesian classifier benchmarked against the SILVA database)
- **BLCA** (Alignment-based classifier benchmarked on reconstructed SSU sequences)
- **VSEARCH** (Used in the RiboTaxa pipeline for taxonomic identification of assembled SSU sequences) — https://github.com/oschakoory/RiboTaxa

## Examples

```
qiime feature-classifier classify-sklearn --i-classifier silva-138-classifier.qza --i-reads metagenome_SSU_sequences.qza --p-confidence 0.7 --o-classification taxonomy.qza
```

## Evaluation signals

- Output table contains one precision/recall/F-measure row per classifier per taxonomic level (domain through species)
- Reproduced metrics match article-reported values: Sklearn species-level F-measure of 91.9%, close to the 97% harmonic mean obtained with mock community 16S references
- RiboTaxa/Sklearn comparison shows 100% F-measure at genus level and 83.7% at species level with no false positives
- QIIME 2 classify-sklearn runs used the trained SILVA 138 classifier with a confidence threshold of 0.7 limiting taxonomic depth
- Every classified sequence is assignable to a bin (true positive, false positive or false negative) relative to the expected mock community taxonomy

## Limitations

- RiboTaxa is not as fast as short-read classifiers such as Kraken2 and Centrifuge
- RiboTaxa cannot assign certain sequences at species level due to near-identical 16S rRNA genes (e.g. Enterobacteriaceae, Lachnospiraceae) and stops at genus level
- Artificial in silico diversity created during full-length sequence reconstruction cannot be excluded, and occasional small indel errors may occur in reconstructed sequences
- Direct classification of short-length Illumina reads yields a high rate of false positives, so benchmarking must use reconstructed sequences

## Evidence

- [methods] Trained QIIME2 SILVA classifier: [evidence span withheld — anchor status: not_found]
- [readme] README tool roles: [evidence span withheld — anchor status: not_found]
