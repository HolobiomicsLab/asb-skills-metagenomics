---
name: multi-classifier-performance-comparison
description: Use when you need to select a taxonomic classifier by benchmarking several
  (e.g. Centrifuge, Kraken2, SPINGO, BLCA, QIIME 2 Sklearn) on sequences with a known
  ground-truth taxonomy, such as a mock community.
when_to_use_negative:
- Input is raw short Illumina reads classified directly without prior SSU reconstruction
  — the article shows this yields a high rate of false positives, so benchmarking
  classifiers on unassembled reads answers a different question
- There is no ground-truth community composition (no mock/synthetic community), so
  true positives/false positives cannot be defined
- The goal is runtime/throughput comparison rather than assignment accuracy — RiboTaxa
  is admittedly slower than Kraken2/Centrifuge and this skill measures precision/recall/F-measure,
  not speed
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Centrifuge
  role: Benchmarked general-purpose taxonomic classifier run against SILVA SSU 138.1
    NR99 on reconstructed SSU sequences
- name: Kraken2
  role: Benchmarked short-read taxonomic classifier compared against the other classifiers
    on reconstructed sequences
- name: SPINGO
  role: Benchmarked species-level classifier for microbial amplicon sequences
  repo: https://github.com/GuyAllard/SPINGO
- name: BLCA
  role: Benchmarked BLAST-based taxonomic classifier for 16S sequences
- name: QIIME 2 Sklearn classifier
  role: Benchmarked classifier (classify-sklearn) run with the SILVA 138 trained classifier
    and a 0.7 confidence threshold for limiting taxonomic depth; it achieved the highest
    species-level score (91.9%)
  repo: https://github.com/oschakoory/RiboTaxa
- name: mothur
  role: Benchmarked 16S rRNA analysis classifier using the SILVA SSU 138.1 NR99 database
- name: RDP Classifier
  role: Benchmarked Bayesian 16S taxonomic classifier using the SILVA reference database
- name: VSEARCH
  role: Used for taxonomic identification of assembled/reconstructed SSU sequences
    within the RiboTaxa pipeline
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:4beb91fde25323473f341c32deeb932e6098ca5ed68259728658efec74c79f56
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# multi-classifier-performance-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multiple taxonomic classifiers (Centrifuge, Kraken2, mothur, RDP Classifier, SPINGO, BLCA, QIIME 2's classify-sklearn) against a known mock/synthetic community by classifying reconstructed SSU rRNA sequences and computing precision, recall and F-measure per taxonomic level, to identify which classifier gives the most accurate species-level assignment.

## When to use

Use when you need to select a taxonomic classifier by benchmarking several (e.g. Centrifuge, Kraken2, SPINGO, BLCA, QIIME 2 Sklearn) on sequences with a known ground-truth taxonomy, such as a mock community.

## When NOT to use

- Input is raw short Illumina reads classified directly without prior SSU reconstruction — the article shows this yields a high rate of false positives, so benchmarking classifiers on unassembled reads answers a different question
- There is no ground-truth community composition (no mock/synthetic community), so true positives/false positives cannot be defined
- The goal is runtime/throughput comparison rather than assignment accuracy — RiboTaxa is admittedly slower than Kraken2/Centrifuge and this skill measures precision/recall/F-measure, not speed

## Inputs

- Reconstructed/clustered SSU rRNA sequences in FASTA format (e.g. metagenome_SSU_sequences.fasta from EMIRGE/MetaRib)
- 16S rRNA reference sequences from the microbial mock community (ground-truth taxonomy)
- SILVA SSU 138.1 NR99 reference database (510,508 SSU rRNA gene sequences)
- Trained SILVA 138 reference classifier for QIIME 2 classify-sklearn (downloaded from the QIIME 2 Data resources)

## Outputs

- Per-classifier taxonomic assignments of reconstructed SSU sequences at domain to species level
- Benchmark table of precision, recall and F-measure per classifier per taxonomic level (benchmark_precision_recall_fmeasure.tsv), reproducing Figure 4A/4B

## How to apply

Load the reconstructed SSU rRNA FASTA (e.g. metagenome_SSU_sequences.fasta from EMIRGE/MetaRib reconstruction) and the 16S reference sequences of the mock community, then run each classifier against the SILVA SSU 138.1 NR99 database; for QIIME 2's classify-sklearn use the SILVA 138 trained classifier with a confidence threshold of 0.7 for limiting taxonomic depth. Assign each classified sequence to the expected mock community taxonomy and bin assignments as true positives, false positives or false negatives at domain, phylum, class, order, family, genus and species levels. Compute precision, recall and F-measure at each taxonomic level for every classifier (e.g. Sklearn species-level F-measure of 91.9%) and compile the metrics into a single comparison table reproducing the article's Figure 4A/4B. Use the table to judge which classifier resolves down to species level with the fewest false positives, as in the article where RiboTaxa's Sklearn classifier reached 100% and 83.7% F-measure at genus and species level with no false positives.

## Related tools

- **Centrifuge** (Benchmarked general-purpose taxonomic classifier run against SILVA SSU 138.1 NR99 on reconstructed SSU sequences)
- **Kraken2** (Benchmarked short-read taxonomic classifier compared against the other classifiers on reconstructed sequences)
- **SPINGO** (Benchmarked species-level classifier for microbial amplicon sequences) — https://github.com/GuyAllard/SPINGO
- **BLCA** (Benchmarked BLAST-based taxonomic classifier for 16S sequences)
- **QIIME 2 Sklearn classifier** (Benchmarked classifier (classify-sklearn) run with the SILVA 138 trained classifier and a 0.7 confidence threshold for limiting taxonomic depth; it achieved the highest species-level score (91.9%)) — https://github.com/oschakoory/RiboTaxa
- **mothur** (Benchmarked 16S rRNA analysis classifier using the SILVA SSU 138.1 NR99 database)
- **RDP Classifier** (Benchmarked Bayesian 16S taxonomic classifier using the SILVA reference database)
- **VSEARCH** (Used for taxonomic identification of assembled/reconstructed SSU sequences within the RiboTaxa pipeline) — https://github.com/oschakoory/RiboTaxa

## Examples

```
qiime feature-classifier classify-sklearn --i-classifier silva-138-classifier.qza --i-reads metagenome_SSU_sequences.qza --p-confidence 0.7 --o-classification reconstructed_SSU_taxonomy.qza
```

## Evaluation signals

- Output TSV exists and contains one precision/recall/F-measure row per classifier per taxonomic level (domain, phylum, class, order, family, genus, species)
- Reproduced metrics match article-reported values, e.g. QIIME 2 Sklearn species-level F-measure of 91.9% (close to the 97% harmonic mean obtained with 16S rRNA reference sequences)
- RiboTaxa's Sklearn classifier reproduces 100% genus-level and 83.7% species-level F-measure with zero false positives on the mock community
- Every classified sequence can be labelled true positive, false positive or false negative at each taxonomic level against the mock community composition, with no unaccounted assignments
- Classifier rankings in the comparison table are consistent with the article's conclusion that reconstructed sequences were correctly assigned while direct short-read classification inferred taxa with a high rate of false positives

## Limitations

- Direct classification of short-length Illumina reads yields a high rate of false positives; benchmarking on reconstructed full-length sequences is required for fair comparison
- High conservation of 16S rRNA sequence cannot discriminate all species (e.g. within Enterobacteriaceae, Lachnospiraceae), so some sequences cannot be assigned at species level and stop at genus level
- 16S rRNA sequences will never provide a perfect representation of bacterial species diversity, and artificial in silico diversity created during full-length sequence reconstruction cannot be excluded

## Evidence

- [results] QIIME 2's Sklearn classifier had the highest species-level score (91.9%) on the reconstructed sequences, close to the 97% harmonic mean: [evidence span withheld — anchor status: not_found]
- [abstract] RiboTaxa achieved F-measure of 100% and 83.7% at genus and species level, respectively, with no false positive detection: [evidence span withheld — anchor status: not_found]
