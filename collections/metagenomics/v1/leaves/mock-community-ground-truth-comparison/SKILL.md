---
name: mock-community-ground-truth-comparison
description: Use when classifying reads or reconstructed SSU sequences from a mock/synthetic
  community with a known expected taxonomy, and you need per-rank precision/recall/F-measure
  to compare classifiers.
when_to_use_negative:
- The community composition is unknown (no ground truth), so TP/FP/FN binning against
  an expected taxonomy is impossible
- The input is raw short Illumina reads rather than reconstructed full-length SSU
  sequences — the article shows inferring taxa directly to short-length reads results
  in a high rate of false positives
- The goal is de novo novelty discovery rather than controlled benchmarking of classifier
  accuracy
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3050
tools:
- name: QIIME 2 Sklearn classifier
  role: Best-performing classifier under benchmark (91.9% species-level F-measure);
    run with trained SILVA 138 classifier and confidence threshold 0.7
- name: Centrifuge
  role: Benchmarked classifier compared against QIIME 2 Sklearn on reconstructed SSU
    sequences
- name: Kraken2
  role: Benchmarked classifier compared against QIIME 2 Sklearn on reconstructed SSU
    sequences
- name: SPINGO
  role: Benchmarked species-level classifier for mock community sequences
  repo: https://github.com/GuyAllard/SPINGO
- name: BLCA
  role: Benchmarked taxonomic classifier for reconstructed SSU sequences
- name: mothur
  role: Benchmarked taxonomic classifier for reconstructed SSU sequences
- name: RDP Classifier
  role: Benchmarked taxonomic classifier for reconstructed SSU sequences
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:ea7f6ac24abd5abcffe03ff492656c127a1d98f7dcb4987366c3ca5c2b394e3a
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
  - mothur
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mock-community-ground-truth-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks taxonomic classifiers (Centrifuge, Kraken2, mothur, RDP, SPINGO, BLCA, QIIME 2's Sklearn) on reconstructed full-length SSU rRNA sequences from a mock/synthetic community whose true taxonomy is known, by binning assignments as true/false positives and false negatives at each rank and computing precision, recall and F-measure.

## When to use

Use when classifying reads or reconstructed SSU sequences from a mock/synthetic community with a known expected taxonomy, and you need per-rank precision/recall/F-measure to compare classifiers.

## When NOT to use

- The community composition is unknown (no ground truth), so TP/FP/FN binning against an expected taxonomy is impossible
- The input is raw short Illumina reads rather than reconstructed full-length SSU sequences — the article shows inferring taxa directly to short-length reads results in a high rate of false positives
- The goal is de novo novelty discovery rather than controlled benchmarking of classifier accuracy

## Inputs

- Reconstructed/clustered SSU rRNA sequences in FASTA format (e.g. metagenome_SSU_sequences.fasta from EMIRGE/MetaRib)
- 16S rRNA reference sequences of the mock community with expected taxonomic assignments (ground truth)
- SILVA SSU 138.1 NR99 reference database for taxonomic classification
- Trained SILVA 138 reference classifier (QIIME 2 Data resources) for classify-sklearn

## Outputs

- benchmark_precision_recall_fmeasure.tsv with one precision/recall/F-measure row per classifier per taxonomic level
- Per-level true positive, false positive and false negative assignment counts for each classifier
- Comparison table reproducing Figure 4A/4B of the article

## How to apply

Load the reconstructed SSU rRNA FASTA (e.g. metagenome_SSU_sequences.fasta from EMIRGE/MetaRib reconstruction) together with the mock community's 16S rRNA reference sequences and expected taxonomy. Run each classifier against a common reference database — here SILVA SSU 138.1 NR99 (510,508 SSU sequences) — using the trained SILVA 138 QIIME 2 classifier with a confidence threshold of 0.7 to limit taxonomic depth for classify-sklearn. Assign every classified sequence to the expected mock community taxonomy and bin it as true positive, false positive or false negative at domain, phylum, class, order, family, genus and species levels. Compute precision, recall and their harmonic mean (F-measure) at each level for each classifier and compile one comparison row per classifier per level. Validate by checking the reproduced values against reported ones, e.g. QIIME 2 Sklearn species-level F-measure of 91.9% and RiboTaxa's Sklearn-based 100% (genus) and 83.7% (species) with no false positives.

## Related tools

- **QIIME 2 Sklearn classifier** (Best-performing classifier under benchmark (91.9% species-level F-measure); run with trained SILVA 138 classifier and confidence threshold 0.7)
- **Centrifuge** (Benchmarked classifier compared against QIIME 2 Sklearn on reconstructed SSU sequences)
- **Kraken2** (Benchmarked classifier compared against QIIME 2 Sklearn on reconstructed SSU sequences)
- **SPINGO** (Benchmarked species-level classifier for mock community sequences) — https://github.com/GuyAllard/SPINGO
- **BLCA** (Benchmarked taxonomic classifier for reconstructed SSU sequences)
- **mothur** (Benchmarked taxonomic classifier for reconstructed SSU sequences)
- **RDP Classifier** (Benchmarked taxonomic classifier for reconstructed SSU sequences)

## Examples

```
qiime feature-classifier classify-sklearn --i-classifier silva-138-classifier.qza --i-reads metagenome_SSU_sequences.qza --p-confidence 0.7 --o-classification taxonomy.tsv
```

## Evaluation signals

- benchmark_precision_recall_fmeasure.tsv exists and contains exactly one precision/recall/F-measure row per classifier per taxonomic level (domain through species)
- Reproduced metrics match article-reported values: QIIME 2 Sklearn species-level F-measure 91.9%, and 97% harmonic mean on the 16S mock community references
- RiboTaxa (Sklearn-based) reproduces 100% F-measure at genus and 83.7% at species level with zero false positives
- For every classifier and level, F-measure equals the harmonic mean of the reported precision and recall: F = 2PR/(P+R)
- TP + FN equals the number of expected mock community taxa at each rank, and TP + FP equals the number of assignments made

## Limitations

- High conservation of 16S rRNA sequence cannot discriminate all species (e.g. certain Enterobacteriaceae and Lachnospiraceae), so RiboTaxa stops at genus level for some sequences
- Direct classification of short-length Illumina reads yields a high rate of false positives; benchmarking must use reconstructed full-length sequences for fair comparison
- Artificial in silico diversity from full-length sequence reconstruction and occasional small indel errors cannot be excluded, and no 16S gene copy number normalization is applied
- RiboTaxa is slower than short-read classifiers (Kraken2, Centrifuge)

## Evidence

- [methods] trained SILVA 138 QIIME2 classifier: [evidence span withheld — anchor status: not_found]
