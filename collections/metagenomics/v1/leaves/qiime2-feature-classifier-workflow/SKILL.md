---
name: qiime2-feature-classifier-workflow
description: Use when full-length or near-full-length SSU rRNA sequences (e.g. 97%-clustered
  reconstructions in SSU_sequences.fasta) need taxonomic lineages plus per-sequence
  relative abundance from metagenomic reads.
when_to_use_negative:
- When classifying raw short-length Illumina reads directly — the article shows this
  'resulted in a high rate of false positives while reconstructed sequences were correctly
  assigned'
- When turnaround speed on very large datasets is the priority — RiboTaxa 'is not
  as fast as short-read classifiers (i.e. Kraken2, Centrifuge)'
- When the input is already a classified feature table or taxonomy file rather than
  reconstructed SSU sequences awaiting classification
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: QIIME 2 Sklearn classifier
  role: Classify the full-length reconstructed SSU sequences at a 0.7 confidence threshold
    using the trained SILVA 138 classifier
  repo: https://docs.qiime2.org/2020.8/plugins/available/feature-classifier/classify-sklearn/
- name: BBMap
  role: Map trimmed metagenomic short reads back onto reconstructed SSU sequences
    to count assigned reads per sequence for relative abundance
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: RiboTaxa
  role: Pipeline wrapper whose scripts convert sequences to QIIME 2 artifacts, run
    classify-sklearn in the RiboTaxa_py36 environment, export and parse taxonomy,
    and build the abundance TSV
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:8cfbed94cd187e9b6f5fb51fc04d85cd7aa3e8b9a06d997045c4e6327ac5d56b
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
  - QIIME 2 Sklearn classifier
  - RiboTaxa
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# qiime2-feature-classifier-workflow

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Taxonomic classification of reconstructed full-length SSU rRNA gene sequences using QIIME 2's sklearn feature-classifier with a pre-trained SILVA 138 classifier, followed by read-mapping-based abundance estimation into a combined SSU_taxonomy_abundance.tsv table.

## When to use

Use when full-length or near-full-length SSU rRNA sequences (e.g. 97%-clustered reconstructions in SSU_sequences.fasta) need taxonomic lineages plus per-sequence relative abundance from metagenomic reads.

## When NOT to use

- When classifying raw short-length Illumina reads directly — the article shows this 'resulted in a high rate of false positives while reconstructed sequences were correctly assigned'
- When turnaround speed on very large datasets is the priority — RiboTaxa 'is not as fast as short-read classifiers (i.e. Kraken2, Centrifuge)'
- When the input is already a classified feature table or taxonomy file rather than reconstructed SSU sequences awaiting classification

## Inputs

- Reconstructed full-length/nearly full-length SSU rRNA gene sequences in FASTA (metagenome_SSU_sequences.fasta)
- Pre-trained QIIME 2 classifier artifact (silva-138-99-nb-classifier.qza, SILVA 138 reference from the QIIME 2 Data resources)
- Trimmed metagenomic short reads (FASTQ) for BBMap read mapping to estimate per-sequence abundance

## Outputs

- QIIME 2 classification artifact (metagenome_renamed_16S18S_recons_qiime2_taxonomy.qza)
- Combined taxonomy/abundance table metagenome_SSU_taxonomy_abundance.tsv with columns Sequence_ID, Domain, Phylum, Class, Order, Family, Genus, Species, Confidence, Length(bp), Assigned reads, Relative_Abundance(%)

## How to apply

Convert the reconstructed SSU FASTA (metagenome_SSU_sequences.fasta) into a QIIME 2 artifact, then run QIIME 2's sklearn feature-classifier with the pre-trained SILVA 138 reference classifier (silva-138-99-nb-classifier.qza) at a confidence threshold of 0.7 for limiting taxonomic depth, using read-batch mode 'auto' (20,000 sequences per batch; set batch=1 when less than 16 GB RAM is available). Export the classification artifact and parse taxonomy ranks from domain to species together with the per-sequence confidence value. Separately, map trimmed metagenomic short reads back onto the reconstructed SSU sequences with BBMap, count assigned reads per sequence, and compute relative abundance as assigned reads divided by the sum of all assigned reads times 100, deliberately without 16S rRNA gene copy-number correction. Write the combined taxonomy-and-abundance results to SSU_taxonomy_abundance.tsv (metagenome_SSU_taxonomy_abundance.tsv) with the twelve named columns Sequence_ID, Domain, Phylum, Class, Order, Family, Genus, Species, Confidence, Length(bp), Assigned reads, and Relative_Abundance(%).

## Related tools

- **QIIME 2 Sklearn classifier** (Classify the full-length reconstructed SSU sequences at a 0.7 confidence threshold using the trained SILVA 138 classifier) — https://docs.qiime2.org/2020.8/plugins/available/feature-classifier/classify-sklearn/
- **BBMap** (Map trimmed metagenomic short reads back onto reconstructed SSU sequences to count assigned reads per sequence for relative abundance) — https://jgi.doe.gov/data-and-tools/bbtools/
- **RiboTaxa** (Pipeline wrapper whose scripts convert sequences to QIIME 2 artifacts, run classify-sklearn in the RiboTaxa_py36 environment, export and parse taxonomy, and build the abundance TSV) — https://github.com/oschakoory/RiboTaxa

## Examples

```
qiime feature-classifier classify-sklearn --i-classifier silva-138-99-nb-classifier.qza --i-reads metagenome_SSU_sequences_qiime2.qza --p-confidence 0.7 --p-reads-per-batch auto --o-classification metagenome_renamed_16S18S_recons_qiime2_taxonomy.qza
```

## Evaluation signals

- The output TSV contains all twelve named columns: Sequence_ID, Domain, Phylum, Class, Order, Family, Genus, Species, Confidence, Length(bp), Assigned reads, Relative_Abundance(%)
- Every reconstructed sequence carries a taxonomy lineage with a Confidence value at or above the 0.7 threshold
- Relative abundances are percentages in the 0-100 range that sum to approximately 100 across all reconstructed sequences
- Each reconstructed sequence ID in the TSV matches an entry in the input SSU_sequences.fasta, with a plausible Length(bp) value
- The QIIME 2 run completed without memory failure — batch mode 'auto' (20,000 sequences per batch) was used, or batch=1 was set when less than 16 GB RAM was available

## Limitations

- No 16S rRNA gene copy-number normalization is applied, so relative abundances reflect mapped read counts only
- Species-level assignment can fail where 16S rRNA conservation is high (e.g. certain genera in Enterobacteriaceae and Lachnospiraceae), so classification may stop at genus level
- Some diversity may be artificial in silico diversity created during full-length sequence reconstruction, and occasional small indel errors can occur in reconstructed sequences (though these rarely affect classification)
- RiboTaxa's classification step is slower than short-read classifiers such as Kraken2 and Centrifuge

## Evidence

- [methods] the database used is the trained classifier SILVA 138 reference sequence downloaded from the Data resources of Qiime2: [evidence span withheld — anchor status: not_found]
- [other] RiboTaxa reports the 97%-clustered reconstructed SSU sequences in SSU_sequences.fasta, and the taxonomic affiliation and relative abundances of each reconstructed sequence in SSU_taxonomy_abundance.tsv.: [evidence span withheld — anchor status: not_found]
