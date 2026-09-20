---
name: precision-recall-fmeasure-calculation-per-taxonomic-rank
description: Use when comparing taxonomic classifiers (e.g. Centrifuge, Kraken2, mothur,
  RDP, SPINGO, BLCA, QIIME 2 Sklearn) on sequences with a known ground-truth community
  composition and you need rank-wise precision/recall/F-measure.
when_to_use_negative:
- Classification is being run directly on short-length Illumina reads without a ground-truth
  mock community, so no true/false positive binning against expected taxonomy is possible
- Sequences cannot be assigned at species level due to near-identical 16S rRNA genes
  (e.g. some Enterobacteriaceae, Lachnospiraceae genera), so species-level metrics
  would be misleading
- Input is real environmental metagenomics data with unknown composition rather than
  a synthetic/mock community with expected taxonomy
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: QIIME 2 Sklearn classifier
  role: classify-sklearn classifier run against the trained SILVA 138 reference classifier
    with confidence threshold 0.7; also the top-performing classifier (91.9% species-level
    F-measure) in the benchmark
  repo: https://github.com/oschakoory/RiboTaxa
- name: Centrifuge
  role: Benchmarked general-purpose taxonomic classifier compared rank-wise against
    RiboTaxa
- name: Kraken2
  role: Benchmarked short-read taxonomic classifier; fast but produced more false
    positives than RiboTaxa
- name: SPINGO
  role: Benchmarked species-level classifier for microbial amplicon sequences
  repo: https://github.com/GuyAllard/SPINGO
- name: VSEARCH
  role: Used for taxonomic identification of assembled/reconstructed SSU sequences
    within RiboTaxa
- name: mothur
  role: Benchmarked taxonomic classifier evaluated at each taxonomic rank
- name: RDP Classifier
  role: Benchmarked Bayesian taxonomic classifier evaluated at each taxonomic rank
- name: BLCA
  role: Benchmarked BLAST-based taxonomic classifier evaluated at each taxonomic rank
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:66c46306f2998892f3a5f95056c7ef8333400ebc9a3b98b79a77d0e8ba316c6f
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
  - RDP Classifier
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
yes_rate: 0.9166666666666666
n_claims: 12
---

# precision-recall-fmeasure-calculation-per-taxonomic-rank

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark protocol for evaluating taxonomic classifiers by assigning reconstructed SSU rRNA sequences to expected mock-community taxonomy and computing precision, recall and F-measure at domain-through-species ranks. Used in RiboTaxa to show QIIME 2's Sklearn classifier reached 91.9% species-level F-measure and RiboTaxa 100%/83.7% at genus/species level.

## When to use

Use when comparing taxonomic classifiers (e.g. Centrifuge, Kraken2, mothur, RDP, SPINGO, BLCA, QIIME 2 Sklearn) on sequences with a known ground-truth community composition and you need rank-wise precision/recall/F-measure.

## When NOT to use

- Classification is being run directly on short-length Illumina reads without a ground-truth mock community, so no true/false positive binning against expected taxonomy is possible
- Sequences cannot be assigned at species level due to near-identical 16S rRNA genes (e.g. some Enterobacteriaceae, Lachnospiraceae genera), so species-level metrics would be misleading
- Input is real environmental metagenomics data with unknown composition rather than a synthetic/mock community with expected taxonomy

## Inputs

- Reconstructed/clustered SSU rRNA FASTA (e.g. metagenome_SSU_sequences.fasta from EMIRGE/MetaRib)
- 16S rRNA reference sequences with known taxonomy of the mock/synthetic community (ground truth)
- Taxonomic classification outputs from each benchmarked classifier (Centrifuge, Kraken2, mothur, RDP Classifier, SPINGO, BLCA, QIIME 2 classify-sklearn)
- SILVA SSU 138.1 NR99 database and QIIME 2-trained SILVA 138 reference classifier

## Outputs

- benchmark_precision_recall_fmeasure.tsv with one precision/recall/F-measure row per classifier per taxonomic level
- Comparison table reproducing Figure 4A/4B of classifier performance across taxonomic ranks

## How to apply

Load the reconstructed/clustered SSU rRNA FASTA (e.g. metagenome_SSU_sequences.fasta from EMIRGE/MetaRib reconstruction) together with the 16S reference sequences of the mock community as ground truth. Run each classifier against the SILVA SSU 138.1 NR99 database, using the trained SILVA 138 reference classifier for QIIME 2's classify-sklearn with a confidence threshold of 0.7 for limiting taxonomic depth. Assign each classified sequence to the expected mock/synthetic community taxonomy and bin assignments as true positives, false positives or false negatives at domain, phylum, class, order, family, genus and species levels. Compute precision, recall and F-measure per classifier per rank (e.g. Sklearn species-level F-measure of 91.9%; RiboTaxa's Sklearn classifier 100% genus and 83.7% species with no false positives), then compile all metrics into a single comparison table reproducing Figure 4A/4B.

## Related tools

- **QIIME 2 Sklearn classifier** (classify-sklearn classifier run against the trained SILVA 138 reference classifier with confidence threshold 0.7; also the top-performing classifier (91.9% species-level F-measure) in the benchmark) — https://github.com/oschakoory/RiboTaxa
- **Centrifuge** (Benchmarked general-purpose taxonomic classifier compared rank-wise against RiboTaxa)
- **Kraken2** (Benchmarked short-read taxonomic classifier; fast but produced more false positives than RiboTaxa)
- **SPINGO** (Benchmarked species-level classifier for microbial amplicon sequences) — https://github.com/GuyAllard/SPINGO
- **VSEARCH** (Used for taxonomic identification of assembled/reconstructed SSU sequences within RiboTaxa)
- **mothur** (Benchmarked taxonomic classifier evaluated at each taxonomic rank)
- **RDP Classifier** (Benchmarked Bayesian taxonomic classifier evaluated at each taxonomic rank)
- **BLCA** (Benchmarked BLAST-based taxonomic classifier evaluated at each taxonomic rank)

## Examples

```
qiime feature-classifier classify-sklearn --i-classifier silva-138-classifier.qza --i-reads metagenome_SSU_sequences.qza --p-confidence 0.7 --o-classification reconstructed_SSU_taxonomy.qza
```

## Evaluation signals

- benchmark_precision_recall_fmeasure.tsv exists and contains one precision/recall/F-measure row per classifier per taxonomic level (domain through species)
- Reproduced metrics match article-reported values: QIIME 2 Sklearn F-measure 91.9% at species level; RiboTaxa 100% F-measure at genus and 83.7% at species level
- RiboTaxa's Sklearn classification shows no false positives at genus and species level on the mock community
- Sklearn species-level score (91.9%) is close to the 97% harmonic mean obtained using the 16S rRNA reference sequences of the mock community
- Every classified sequence is binned as exactly one of true positive, false positive or false negative at each of the seven taxonomic levels

## Limitations

- RiboTaxa cannot assign certain sequences at species level due to near-identical 16S rRNA genes and stops at genus level, capping achievable species-level recall
- High conservation of 16S rRNA sequence cannot discriminate all species (e.g. Enterobacteriaceae, Lachnospiraceae), so species-level metrics understate biological resolution
- Artificial in silico diversity from full-length sequence reconstruction cannot be excluded, and occasional small indel errors may occur in reconstructed sequences
- RiboTaxa is slower than short-read classifiers (Kraken2, Centrifuge), which matters when benchmarking throughput alongside accuracy

## Evidence

- [results] QIIME 2's Sklearn classifier had the highest species-level score (91.9%) on the reconstructed sequences: [evidence span withheld — anchor status: not_found]
- [abstract] RiboTaxa F-measure 100% genus, 83.7% species: [evidence span withheld — anchor status: not_found]
- [readme] RiboTaxa pipeline classifies reconstructed SSU sequences with QIIME2 sklearn: [evidence span withheld — anchor status: not_found]
