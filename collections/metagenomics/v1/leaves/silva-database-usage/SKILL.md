---
name: silva-database-usage
description: Use when setting up reference databases for SSU rRNA analysis of metagenomic
  Illumina data — indexing SILVA for SortMeRNA/EMIRGE or selecting a trained classifier
  (SILVA 138, Greengenes, UNITE, RDP, GTDB) for QIIME 2 classification.
when_to_use_negative:
- The analysis targets non-SSU rRNA genes or protein-coding markers — SILVA SSU and
  the trained SSU classifier are specific to 16S/18S reconstruction and classification
- Rapid direct classification of raw short reads is needed (e.g. with Kraken2 or Centrifuge)
  rather than full-length SSU reconstruction — RiboTaxa is not as fast as short-read
  classifiers
- Copy-number-corrected absolute abundances are required — RiboTaxa explicitly chooses
  not to apply 16S rRNA gene copy number correction
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0084
tools:
- name: SortMeRNA
  role: Consumes the indexed SILVA SSU 138.1 database to filter 16S/18S reads from
    trimmed metagenomic files
- name: EMIRGE
  role: Requires its own indexed SILVA database to reconstruct full-length SSU rRNA
    sequences from filtered reads
- name: QIIME 2 Sklearn classifier
  role: Applies the pre-trained SILVA 138 Naive Bayes classifier (confidence threshold
    0.7) to classify reconstructed SSU sequences
  repo: https://docs.qiime2.org/2020.8/plugins/available/feature-classifier/classify-sklearn/
- name: RiboTaxa
  role: Pipeline that packages SILVA indexing, read filtering, SSU reconstruction,
    and SILVA-138-based classification into one workflow
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:395fc9342fa2bf1661017637ad201abba523a30f4805edb9b6ab0c24da82b062
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  tools:
  - EMIRGE
  - QIIME 2 Sklearn classifier
  - RiboTaxa
  - SortMeRNA
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# silva-database-usage

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preparing and using the SILVA SSU rRNA reference database throughout the RiboTaxa pipeline: indexed SILVA SSU 138.1 for SortMeRNA read filtering and EMIRGE/MetaRib reconstruction, plus the pre-trained SILVA 138 QIIME 2 Naive Bayes classifier for taxonomic assignment of reconstructed full-length SSU sequences.

## When to use

Use when setting up reference databases for SSU rRNA analysis of metagenomic Illumina data — indexing SILVA for SortMeRNA/EMIRGE or selecting a trained classifier (SILVA 138, Greengenes, UNITE, RDP, GTDB) for QIIME 2 classification.

## When NOT to use

- The analysis targets non-SSU rRNA genes or protein-coding markers — SILVA SSU and the trained SSU classifier are specific to 16S/18S reconstruction and classification
- Rapid direct classification of raw short reads is needed (e.g. with Kraken2 or Centrifuge) rather than full-length SSU reconstruction — RiboTaxa is not as fast as short-read classifiers
- Copy-number-corrected absolute abundances are required — RiboTaxa explicitly chooses not to apply 16S rRNA gene copy number correction

## Inputs

- Metagenomic Illumina reads (single-end or paired-end, .fastq or .fastq.gz)
- SILVA SSU 138.1 reference database (or indexed SILVA SSU 138.1 provided for RiboTaxa)
- Trained SILVA 138 QIIME 2 classifier artifact (.qza), e.g. from the QIIME 2 Data resources
- indexDB_arguments.conf config file (to index a custom database)

## Outputs

- Indexed SILVA databases for SortMeRNA and EMIRGE
- Filtered 16S/18S read files from SortMeRNA
- Reconstructed full-length SSU rRNA sequences (SSU_sequences.fasta)
- QIIME 2 taxonomy artifact (metagenome_renamed_16S18S_recons_qiime2_taxonomy.qza)
- metagenome_SSU_taxonomy_abundance.tsv with Domain-to-Species ranks, Confidence, Assigned reads, Relative_Abundance(%)

## How to apply

Download SILVA SSU 138.1 and the RiboTaxa-provided indexed version (SortMeRNA and EMIRGE each require their own indexed database), or index your own database by filling the indexDB_arguments.conf config file. For taxonomic classification, use the trained SILVA 138 reference classifier downloaded from the Data resources of QIIME 2 with the classify-sklearn plugin at a confidence threshold of 0.7 for limiting taxonomic depth. The NR99 version used for classification contains 510,508 SSU rRNA gene sequences. If switching references (RDP, Greengenes, UNITE, GTDB), download an already-trained classifier or train your own — reference choice must be consistent between read filtering, reconstruction, and classification steps.

## Related tools

- **SortMeRNA** (Consumes the indexed SILVA SSU 138.1 database to filter 16S/18S reads from trimmed metagenomic files)
- **EMIRGE** (Requires its own indexed SILVA database to reconstruct full-length SSU rRNA sequences from filtered reads)
- **QIIME 2 Sklearn classifier** (Applies the pre-trained SILVA 138 Naive Bayes classifier (confidence threshold 0.7) to classify reconstructed SSU sequences) — https://docs.qiime2.org/2020.8/plugins/available/feature-classifier/classify-sklearn/
- **RiboTaxa** (Pipeline that packages SILVA indexing, read filtering, SSU reconstruction, and SILVA-138-based classification into one workflow) — https://github.com/oschakoory/RiboTaxa

## Examples

```
git clone https://github.com/oschakoory/RiboTaxa.git; cd RiboTaxa; bash conda_virt_env.sh
```

## Evaluation signals

- SortMeRNA and EMIRGE both load their indexed SILVA databases without errors (each tool needs its own index)
- Every reconstructed SSU sequence receives a taxonomy lineage with a Confidence value at or above the 0.7 threshold
- The output TSV reports assignments against the SILVA reference containing 510,508 SSU rRNA gene sequences, with all twelve named columns present
- Reference provenance is consistent: indexed SILVA SSU 138.1 used for filtering/reconstruction and the trained SILVA 138 classifier used in QIIME 2
- Custom databases (Greengenes, UNITE, RDP, GTDB) are only used with an already-trained or newly trained classifier, not a mismatched one

## Limitations

- No chimera detection is included, so chimeric database sequences could carry over into SSU reconstructions if reads map across the full length of the chimera
- High conservation of 16S rRNA sequence prevents species-level discrimination for some genera (e.g. Enterobacteriaceae, Lachnospiraceae), so classification may stop at genus level
- Using alternative databases (Greengenes, UNITE, RDP, GTDB) requires downloading an already-trained classifier or training your own, adding setup burden

## Evidence

- [methods] the database used is the trained classifier SILVA 138 reference sequence downloaded from the Data resources of Qiime2: [evidence span withheld — anchor status: not_found]
