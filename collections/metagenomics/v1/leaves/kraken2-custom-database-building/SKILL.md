---
name: kraken2-custom-database-building
description: Use when a 16S rRNA benchmark requires taxonomic assignment by a k-mer-based
  classifier that must be constrained to the same reference database (e.g., Greengenes2
  2024.09) and the same test sequences (69,335 held-out Greengenes2 reads) as the
  other tools being compared.
when_to_use_negative:
- When the reference space must extend beyond Greengenes2 2024.09 — the custom database
  is restricted to that FASTA, so sequences from species absent from it will be mis-
  or unclassified
- When the goal is DeepTaxa-style seven-rank hierarchical classification (domain through
  species) with calibration metrics — Kraken 2 provides per-read k-mer classification,
  not calibrated per-rank probabilities
- When input sequences are short, error-prone amplicons rather than the full-length
  (~1,500 bp) 16S sequences the benchmark used
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0659
tools:
- name: Kraken 2
  role: k-mer-based taxonomic classifier; a custom database is built from Greengenes2
    2024.09 sequences via kraken2-build and used to classify the 69,335-sequence held-out
    test set (v2.1.3)
- name: DeepTaxa
  role: 'the hybrid CNN-BERT classifier against which Kraken 2''s species-level accuracy
    is compared on the same test set (Kraken 2: 28.05% vs DeepTaxa: 92.96%)'
  repo: https://github.com/systems-genomics-lab/deeptaxa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:0b5eff0c609ece1b1ffa5995708d4d5eac02b1dbe27573f516fda5048b30e304
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DeepTaxa
  - Kraken 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# kraken2-custom-database-building

## Summary

Build a custom Kraken 2 v2.1.3 database from the Greengenes2 2024.09 reference FASTA via kraken2-build so that the k-mer-based classifier can be benchmarked against DADA2, QIIME 2, SINTAX, and DeepTaxa on the same held-out test set.

## When to use

Use when a 16S rRNA benchmark requires taxonomic assignment by a k-mer-based classifier that must be constrained to the same reference database (e.g., Greengenes2 2024.09) and the same test sequences (69,335 held-out Greengenes2 reads) as the other tools being compared.

## When NOT to use

- When the reference space must extend beyond Greengenes2 2024.09 — the custom database is restricted to that FASTA, so sequences from species absent from it will be mis- or unclassified
- When the goal is DeepTaxa-style seven-rank hierarchical classification (domain through species) with calibration metrics — Kraken 2 provides per-read k-mer classification, not calibrated per-rank probabilities
- When input sequences are short, error-prone amplicons rather than the full-length (~1,500 bp) 16S sequences the benchmark used

## Inputs

- Greengenes2 2024.09 reference FASTA (346,671 curated 16S rRNA sequences; 277,336-sequence training split used as reference)
- Held-out Greengenes2 2024.09 test FASTA (69,335 full-length sequences, ~1,500 bp)
- Greengenes2 taxonomy labels for the test set (gg_2024_09_testing.tsv.gz) for accuracy computation

## Outputs

- Custom Kraken 2 database directory built via kraken2-build from Greengenes2 2024.09 sequences
- Per-sequence Kraken 2 classification output for the 69,335-sequence test set
- Species-level (per-rank) top-1 accuracy for Kraken 2 (benchmark value: 28.05%) recorded in benchmark_species_accuracy.csv
- Wall-clock classification time for Kraken 2

## How to apply

First, obtain the Greengenes2 2024.09 reference FASTA (346,671 curated 16S rRNA gene sequences; the DeepTaxa split uses 277,336 training sequences for the reference and 69,335 held-out test sequences for classification). Build a fresh Kraken 2 v2.1.3 database directory with kraken2-build, adding the Greengenes2 FASTA as the only library source so classification is restricted to the same label space as DADA2, QIIME 2, and SINTAX. Classify the 69,335-sequence test set with kraken2 and record both per-rank top-1 accuracy and wall-clock classification time so the tool is directly comparable on the shared test set. When tabulating (e.g., in benchmark_species_accuracy.csv), treat the resulting Kraken 2 species accuracy as the lowest expected among the four benchmark tools (reported: 28.05%), far below DeepTaxa's 92.96%.

## Related tools

- **Kraken 2** (k-mer-based taxonomic classifier; a custom database is built from Greengenes2 2024.09 sequences via kraken2-build and used to classify the 69,335-sequence held-out test set (v2.1.3))
- **DeepTaxa** (the hybrid CNN-BERT classifier against which Kraken 2's species-level accuracy is compared on the same test set (Kraken 2: 28.05% vs DeepTaxa: 92.96%)) — https://github.com/systems-genomics-lab/deeptaxa

## Examples

```
kraken2-build --db gg2_2024_09_db --add-to-library gg_2024_09_training.fna && kraken2 --db gg2_2024_09_db --output kraken2_test_results.txt gg_2024_09_testing.fna
```

## Evaluation signals

- Kraken 2 species-level top-1 accuracy on the shared 69,335-sequence Greengenes2 test set falls within a small reproduction tolerance of 28.05%
- Kraken 2 is the lowest-scoring of the four benchmark tools (DADA2 90.05%, QIIME 2 85.01%, SINTAX 73.13%, Kraken 2 28.05%)
- Accuracy is computed on the same 69,335-sequence test set used for all other tools (no sequence leakage between reference and test FASTAs)
- Both per-rank accuracy and wall-clock classification time are recorded in benchmark_species_accuracy.csv
- The kraken2-build step completes with the Greengenes2 FASTA as the sole library source (no NCBI default contamination)

## Limitations

- As a k-mer-based classifier constrained to a single reference FASTA, Kraken 2 achieved only 28.05% species-level accuracy in this benchmark — far below DeepTaxa's 92.96% — and many reads fall below classification confidence
- Results are specific to full-length Greengenes2 2024.09 16S sequences; performance on SILVA, RDP, or independent environmental/clinical datasets is not established by this benchmark
- A custom database lacks the NCBI genomic context of the standard Kraken 2 database, so absolute accuracies here are not comparable to published Kraken 2 runs on native databases

## Evidence

- [methods] Kraken 2 version and database build: "Kraken2 version 2.1.3 with a custom database built from Greengenes2 2024.09 sequences via kraken2-build"
- [abstract] DeepTaxa advantage over k-mer classifiers: [evidence span withheld — anchor status: not_found]
- [abstract] Kraken 2 classifier type: "with larger gains over the k-mer-based classifiers SINTAX and Kraken 2"
- [methods] Benchmark test set composition: "The dataset was partitioned into training (277 336 sequences, 80%) and test (69 335 sequences, 20%) sets using stratified sampling to preserve label distributions."
