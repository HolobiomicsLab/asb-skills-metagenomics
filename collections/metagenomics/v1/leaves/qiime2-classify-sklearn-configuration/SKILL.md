---
name: qiime2-classify-sklearn-configuration
description: Use when reconstructed or clustered SSU rRNA FASTA sequences (e.g. EMIRGE/MetaRib
  output from metagenomics reads) must be assigned taxonomy down to genus/species
  level via a naive-Bayes trained classifier.
when_to_use_negative:
- Input is raw short Illumina reads rather than reconstructed full-length SSU sequences
  — direct classification of short reads yields a high rate of false positives
- A fast short-read classifier is required — RiboTaxa/QIIME 2 Sklearn is slower than
  Kraken2 or Centrifuge
- Sequences belong to genera with near-identical 16S rRNA genes (e.g. Enterobacteriaceae,
  Lachnospiraceae) where species-level assignment is not resolvable and classification
  will stop at genus level
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: QIIME 2 Sklearn classifier
  role: classify-sklearn assigns taxonomy to reconstructed full-length SSU rRNA sequences
    using the pre-trained SILVA 138 classifier with a 0.7 confidence threshold
  repo: https://github.com/oschakoory/RiboTaxa
- name: SPINGO
  role: benchmark comparator — species-level amplicon classifier run against the same
    reconstructed SSU sequences and SILVA SSU 138.1 NR99 database
  repo: https://github.com/GuyAllard/SPINGO
- name: VSEARCH
  role: used in RiboTaxa for taxonomic identification of assembled SSU sequences and
    clustering in the pipeline
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:f263eea7d16ea455273678884dcb3f9602f770b89841a3ffc58d1c446b8e4c80
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  repo_url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BLCA
  - Centrifuge
  - Kraken2
  - QIIME 2 Sklearn classifier
  - SPINGO
  - VSEARCH
  - mothur
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# qiime2-classify-sklearn-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configuration and application of QIIME 2's Sklearn feature classifier (classify-sklearn) to taxonomically classify full-length reconstructed SSU rRNA sequences, using the pre-trained SILVA 138 classifier with a confidence threshold of 0.7 to limit taxonomic depth.

## When to use

Use when reconstructed or clustered SSU rRNA FASTA sequences (e.g. EMIRGE/MetaRib output from metagenomics reads) must be assigned taxonomy down to genus/species level via a naive-Bayes trained classifier.

## When NOT to use

- Input is raw short Illumina reads rather than reconstructed full-length SSU sequences — direct classification of short reads yields a high rate of false positives
- A fast short-read classifier is required — RiboTaxa/QIIME 2 Sklearn is slower than Kraken2 or Centrifuge
- Sequences belong to genera with near-identical 16S rRNA genes (e.g. Enterobacteriaceae, Lachnospiraceae) where species-level assignment is not resolvable and classification will stop at genus level

## Inputs

- Reconstructed/clustered full-length SSU rRNA FASTA (metagenome_SSU_sequences.fasta from EMIRGE/MetaRib reconstruction)
- Pre-trained SILVA 138 reference classifier from QIIME 2 Data resources
- Expected mock/synthetic community taxonomy (for benchmark evaluation)

## Outputs

- Per-sequence taxonomic assignments (lineage strings down to the rank permitted by the 0.7 confidence threshold)
- Precision, recall and F-measure per classifier per taxonomic level (benchmark_precision_recall_fmeasure.tsv)

## How to apply

Run QIIME 2's classify-sklearn inside the RiboTaxa_py36 conda environment on the reconstructed SSU sequences (metagenome_SSU_sequences.fasta from EMIRGE/MetaRib), using the SILVA 138 reference classifier pre-trained and downloaded from the QIIME 2 Data resources; the same configuration is used when benchmarking against Centrifuge, Kraken2, mothur, RDP Classifier, SPINGO and BLCA. Apply the confidence threshold of 0.7 (default) to limit taxonomic depth so that assignments stop where the classifier confidence falls below the threshold, preventing over-specific species calls. In the mock-community benchmark this setup gave QIIME 2's Sklearn the highest species-level score (91.9% F-measure) on reconstructed sequences, close to the 97% harmonic mean obtained with 16S reference sequences, while within RiboTaxa it contributed to 100% genus and 83.7% species F-measure with no false positives. Validate by binning assignments into true/false positives/negatives at each rank and comparing precision, recall and F-measure against article-reported values.

## Related tools

- **QIIME 2 Sklearn classifier** (classify-sklearn assigns taxonomy to reconstructed full-length SSU rRNA sequences using the pre-trained SILVA 138 classifier with a 0.7 confidence threshold) — https://github.com/oschakoory/RiboTaxa
- **SPINGO** (benchmark comparator — species-level amplicon classifier run against the same reconstructed SSU sequences and SILVA SSU 138.1 NR99 database) — https://github.com/GuyAllard/SPINGO
- **VSEARCH** (used in RiboTaxa for taxonomic identification of assembled SSU sequences and clustering in the pipeline) — https://github.com/oschakoory/RiboTaxa

## Examples

```
conda activate RiboTaxa_py36 && qiime feature-classifier classify-sklearn --i-classifier silva-138-classifier.qza --i-reads metagenome_SSU_sequences.qza --p-confidence 0.7 --o-classification taxonomy.qza
```

## Evaluation signals

- Species-level F-measure of the Sklearn classifier on reconstructed mock-community sequences should reach ~91.9%, close to the 97% harmonic mean obtained with 16S rRNA reference sequences
- Within RiboTaxa, F-measure of 100% at genus level and 83.7% at species level with zero false positives
- benchmark_precision_recall_fmeasure.tsv exists with one precision/recall/F-measure row per classifier per taxonomic level (domain through species)
- No assignment exceeds the rank at which classifier confidence drops below the 0.7 threshold

## Limitations

- RiboTaxa/QIIME 2 Sklearn is not as fast as short-read classifiers (Kraken2, Centrifuge)
- High conservation of 16S rRNA sequence prevents species-level discrimination for certain genera (e.g. Enterobacteriaceae, Lachnospiraceae); classification stops at genus level
- RiboTaxa does not include chimera detection, so chimeric database sequences could carry over into SSU reconstructions, and small indel errors may occur in reconstructed sequences

## Evidence

- [methods] the database used is the trained classifier SILVA 138 reference sequence downloaded from the Data resources of Qiime2: [evidence span withheld — anchor status: not_found]
- [results] QIIME 2's Sklearn classifier had the highest species-level score (91.9%) on the reconstructed sequences: [evidence span withheld — anchor status: not_found]
