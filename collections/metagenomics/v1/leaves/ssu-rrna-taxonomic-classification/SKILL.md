---
name: ssu-rrna-taxonomic-classification
description: Use when reconstructed full-length SSU rRNA sequences (e.g. metagenome_SSU_sequences.fasta
  from RiboTaxa's EMIRGE/MetaRib reconstruction) must be taxonomically assigned down
  to species level with per-sequence confidence and relative abundance.
when_to_use_negative:
- Input is raw short Illumina reads to be classified directly without prior SSU reconstruction
  — the article shows inferring taxa directly to short reads results in a high rate
  of false positives
- Only rapid approximate profiling is needed and runtime dominates — the article notes
  RiboTaxa is not as fast as short-read classifiers (Kraken2, Centrifuge)
- Chimera detection or 16S rRNA gene copy-number normalization are mandatory requirements
  — neither was included in this workflow
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: QIIME 2 Sklearn classifier
  role: Primary classifier of full-length reconstructed SSU sequences with the trained
    SILVA 138 reference classifier at confidence 0.7
  repo: https://docs.qiime2.org/2020.8/plugins/available/feature-classifier/classify-sklearn/
- name: BBMap
  role: Maps trimmed metagenomic short reads back onto reconstructed SSU sequences
    to count assigned reads per sequence for relative abundance
- name: Kraken2
  role: Benchmarked comparator classifier in the seven-classifier precision/recall/F-measure
    evaluation
- name: Centrifuge
  role: Benchmarked comparator classifier in the seven-classifier evaluation
- name: SPINGO
  role: Benchmarked species-level classifier for meta-genomic amplicons in the evaluation
  repo: https://github.com/GuyAllard/SPINGO
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:e27a587583176898aab974cd85257abd0cff31bc3e3c1c75f48992e1d29ee127
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBMap
  - BLCA
  - Centrifuge
  - Kraken2
  - QIIME 2 Sklearn classifier
  - SPINGO
  - mothur
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# ssu-rrna-taxonomic-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Taxonomic classification of reconstructed full-length SSU rRNA gene sequences from metagenomics data, using QIIME 2's sklearn classifier against a SILVA 138 reference classifier at a confidence threshold of 0.7, optionally benchmarked against seven classifiers to compute precision/recall/F-measure at each rank.

## When to use

Use when reconstructed full-length SSU rRNA sequences (e.g. metagenome_SSU_sequences.fasta from RiboTaxa's EMIRGE/MetaRib reconstruction) must be taxonomically assigned down to species level with per-sequence confidence and relative abundance.

## When NOT to use

- Input is raw short Illumina reads to be classified directly without prior SSU reconstruction — the article shows inferring taxa directly to short reads results in a high rate of false positives
- Only rapid approximate profiling is needed and runtime dominates — the article notes RiboTaxa is not as fast as short-read classifiers (Kraken2, Centrifuge)
- Chimera detection or 16S rRNA gene copy-number normalization are mandatory requirements — neither was included in this workflow

## Inputs

- Reconstructed/clustered SSU rRNA FASTA (metagenome_SSU_sequences.fasta from EMIRGE/MetaRib reconstruction)
- Trimmed metagenomic short reads (FASTQ, single- or paired-end, optionally fastq.gz) for abundance mapping
- Trained SILVA 138 QIIME 2 reference classifier (silva-138-99-nb-classifier.qza)
- SILVA SSU 138.1 NR99 database (for benchmarked classifiers)
- Mock/synthetic community expected taxonomy (for benchmark evaluation)

## Outputs

- SSU_taxonomy_abundance.tsv / metagenome_SSU_taxonomy_abundance.tsv with columns Sequence_ID, Domain, Phylum, Class, Order, Family, Genus, Species, Confidence, Length(bp), Assigned reads, Relative_Abundance(%)
- Exported QIIME 2 taxonomy artifact (e.g. metagenome_renamed_16S18S_recons_qiime2_taxonomy.qza)
- benchmark_precision_recall_fmeasure.tsv with one precision/recall/F-measure row per classifier per taxonomic level

## How to apply

Convert the reconstructed SSU FASTA (e.g. metagenome_SSU_sequences.fasta) to a QIIME 2 artifact and run the sklearn feature-classifier with the trained SILVA 138 reference classifier (silva-138-99-nb-classifier.qza) at a confidence threshold of 0.7 for limiting taxonomic depth, using batch mode 'auto' (20,000 sequences/batch; batch=1 if <16 GB RAM). Parse the exported taxonomy artifact to extract ranks from domain to species with per-sequence confidence, then map trimmed metagenomic short reads back onto the reconstructed SSU sequences with BBMap to compute relative abundance as assigned reads per sequence divided by the total assigned reads x 100, without 16S rRNA gene copy-number correction. Emit a combined table such as metagenome_SSU_taxonomy_abundance.tsv with Sequence_ID, Domain, Phylum, Class, Order, Family, Genus, Species, Confidence, Length(bp), Assigned reads, and Relative_Abundance(%). For benchmarking, run Centrifuge, Kraken2, mothur, RDP Classifier, SPINGO, BLCA and QIIME 2's classify-sklearn against SILVA SSU 138.1 NR99, bin assignments as true/false positives/negatives at each rank, and compute precision, recall and F-measure per classifier per level.

## Related tools

- **QIIME 2 Sklearn classifier** (Primary classifier of full-length reconstructed SSU sequences with the trained SILVA 138 reference classifier at confidence 0.7) — https://docs.qiime2.org/2020.8/plugins/available/feature-classifier/classify-sklearn/
- **BBMap** (Maps trimmed metagenomic short reads back onto reconstructed SSU sequences to count assigned reads per sequence for relative abundance)
- **Kraken2** (Benchmarked comparator classifier in the seven-classifier precision/recall/F-measure evaluation)
- **Centrifuge** (Benchmarked comparator classifier in the seven-classifier evaluation)
- **SPINGO** (Benchmarked species-level classifier for meta-genomic amplicons in the evaluation) — https://github.com/GuyAllard/SPINGO

## Examples

```
qiime feature-classifier classify-sklearn --i-classifier silva-138-99-nb-classifier.qza --i-reads metagenome_SSU_sequences_qiime2.qza --p-confidence 0.7 --o-classification taxonomy.qza
```

## Evaluation signals

- Output TSV contains all twelve named columns (Sequence_ID through Relative_Abundance(%)) and every reconstructed sequence has a lineage with Confidence at or above the 0.7 threshold
- Relative abundances are percentages in the 0-100 range summing to ~100 across all reconstructed sequences
- benchmark_precision_recall_fmeasure.tsv contains one precision/recall/F-measure row per classifier per taxonomic level (domain to species)
- Reproduced metrics match article-reported values: QIIME 2 Sklearn species-level F-measure ~91.9% on reconstructed sequences and RiboTaxa 100% genus / 83.7% species F-measure with no false positives
- Taxonomic depth is limited by the 0.7 confidence threshold — sequences with low confidence stop at higher ranks (e.g. genus level) rather than being force-assigned to species

## Limitations

- Chimera detection was not included in the reconstruction workflow, and chimeric database sequences could carry over into SSU reconstructions if reads map across the full length of the chimera
- No 16S rRNA gene copy number normalization is applied, so relative abundances are not copy-number corrected
- RiboTaxa is not as fast as short-read classifiers (Kraken2, Centrifuge)
- High conservation of 16S rRNA sequence cannot discriminate all species — some sequences stop at genus level, and artificial in silico diversity from full-length reconstruction or small indel errors cannot be fully excluded
- 16S rRNA sequences will never provide a perfect representation of bacterial species diversity

## Evidence

- [methods] the database used is the trained classifier SILVA 138 reference sequence downloaded from the Data resources of Qiime2: [evidence span withheld — anchor status: not_found]
