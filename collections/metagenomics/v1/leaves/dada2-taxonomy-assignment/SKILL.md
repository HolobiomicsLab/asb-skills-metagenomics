---
name: dada2-taxonomy-assignment
description: Use when reproducing or comparing species-level taxonomic classification
  benchmarks of full-length 16S rRNA sequences against DADA2, QIIME 2, SINTAX, Kraken
  2, or DeepTaxa on the same held-out reference test set.
when_to_use_negative:
- Sequences are short amplicons (e.g. V3–V4 or V4 regions) where the benchmark instead
  uses dedicated amplicon checkpoints (e.g. V3–V4 ~420 bp reaching 87.54% species
  accuracy) — full-length reference assumptions may not transfer
- Input is already a classified feature table or abundance matrix rather than raw
  reference FASTA sequences to be classified
- The goal is DeepTaxa training or inference rather than baseline benchmarking with
  DADA2
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: DADA2
  role: 'Taxonomic classifier: assignTaxonomy function run on the Greengenes2 2024.09
    training FASTA to classify the held-out test set (v1.30 via Bioconductor, R 4.3.3)'
- name: DeepTaxa
  role: Reference hybrid CNN-BERT classifier whose 92.96% species accuracy on the
    same test set provides the upper benchmark DADA2 is compared against
  repo: github.com/systems-genomics-lab/deeptaxa
- name: QIIME 2
  role: Co-benchmarked naive Bayes classifier (v2024.5.0 q2-feature-classifier, sklearn
    plugin) reported at 85.01% species accuracy
- name: SINTAX
  role: Co-benchmarked k-mer classifier (USEARCH v11.0.667, 0.8 bootstrap cutoff)
    reported at 73.13% species accuracy
- name: Kraken 2
  role: Co-benchmarked k-mer classifier (v2.1.3, custom Greengenes2 2024.09 database
    via kraken2-build) reported at 28.05% species accuracy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:b88c325bb508ee300d324d2ed6d460653f488f5b11308d8c01036b9f41b39712
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - DeepTaxa
  - Kraken 2
  - QIIME 2
  - SINTAX
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# dada2-taxonomy-assignment

## Summary

Assign full-length 16S rRNA taxonomy with DADA2 v1.30 assignTaxonomy (Bioconductor, R 4.3.3) against a Greengenes2 2024.09 training FASTA, as benchmarked in the DeepTaxa paper where DADA2 reached 90.05% species-level accuracy on the shared 69,335-sequence held-out test set.

## When to use

Use when reproducing or comparing species-level taxonomic classification benchmarks of full-length 16S rRNA sequences against DADA2, QIIME 2, SINTAX, Kraken 2, or DeepTaxa on the same held-out reference test set.

## When NOT to use

- Sequences are short amplicons (e.g. V3–V4 or V4 regions) where the benchmark instead uses dedicated amplicon checkpoints (e.g. V3–V4 ~420 bp reaching 87.54% species accuracy) — full-length reference assumptions may not transfer
- Input is already a classified feature table or abundance matrix rather than raw reference FASTA sequences to be classified
- The goal is DeepTaxa training or inference rather than baseline benchmarking with DADA2

## Inputs

- Greengenes2 2024.09 training FASTA (277,336 full-length 16S rRNA sequences, e.g. gg_2024_09_training.fna.gz)
- Greengenes2 2024.09 held-out test FASTA (69,335 full-length sequences, e.g. gg_2024_09_testing.fna.gz)
- Matching test taxonomy labels TSV (e.g. gg_2024_09_testing.tsv.gz) for accuracy computation

## Outputs

- Per-sequence taxonomic assignments from domain to species for the 69,335-sequence test set
- Per-rank top-1 accuracy table including the species-level figure (expected ~90.05%)
- Wall-clock classification time for DADA2
- benchmark_species_accuracy.csv comparing DADA2, QIIME 2, SINTAX, and Kraken 2

## How to apply

Load the Greengenes2 2024.09 training/reference FASTA (277,336 full-length sequences from the Hugging Face Greengenes2 dataset repository) and the held-out test FASTA (69,335 sequences). Run DADA2 v1.30 via Bioconductor in R 4.3.3, calling the assignTaxonomy function with the Greengenes2 2024.09 training FASTA as the reference taxonomy. Classify every test sequence and compute per-rank top-1 accuracy on the full 69,335-sequence shared test set, plus wall-clock classification time. The benchmark reports species-level accuracy of 90.05% for DADA2, below DeepTaxa's 92.96%; validate your run against this figure within a small reproduction tolerance before tabulating results (e.g., in benchmark_species_accuracy.csv).

## Related tools

- **DADA2** (Taxonomic classifier: assignTaxonomy function run on the Greengenes2 2024.09 training FASTA to classify the held-out test set (v1.30 via Bioconductor, R 4.3.3))
- **DeepTaxa** (Reference hybrid CNN-BERT classifier whose 92.96% species accuracy on the same test set provides the upper benchmark DADA2 is compared against) — github.com/systems-genomics-lab/deeptaxa
- **QIIME 2** (Co-benchmarked naive Bayes classifier (v2024.5.0 q2-feature-classifier, sklearn plugin) reported at 85.01% species accuracy)
- **SINTAX** (Co-benchmarked k-mer classifier (USEARCH v11.0.667, 0.8 bootstrap cutoff) reported at 73.13% species accuracy)
- **Kraken 2** (Co-benchmarked k-mer classifier (v2.1.3, custom Greengenes2 2024.09 database via kraken2-build) reported at 28.05% species accuracy)

## Examples

```
Rscript -e 'library(dada2); tax <- assignTaxonomy(seqs, "gg_2024_09_training.fna.gz", minBoot=0, tryRC=TRUE, outputBootstraps=TRUE)'
```

## Evaluation signals

- Species-level top-1 accuracy on the 69,335-sequence test set reproduces the reported 90.05% within a small tolerance
- DADA2's accuracy ranks above QIIME 2 (85.01%), SINTAX (73.13%), and Kraken 2 (28.05%) but below DeepTaxa (92.96%)
- All 69,335 held-out test sequences receive a classification (no sequence drops or subset evaluation)
- Per-rank accuracy is computed for all seven ranks (domain through species) on the same shared test set across all four benchmark tools
- Wall-clock classification time is recorded alongside accuracy for fair runtime comparison

## Limitations

- Reference-based classifiers can only detect species already represented in the reference catalog, leaving uncharacterized 'dark matter' unclassified
- K-mer and alignment-based benchmarks degrade on closely related species and short, error-prone sequences; SINTAX's recommended 0.8 bootstrap cutoff leaves many species-level assignments unclassified
- Results are specific to Greengenes2 2024.09 full-length 16S; performance across SILVA, RDP, or independent environmental/clinical datasets was not tested in this benchmark

## Evidence

- [methods] DADA2 version 1.30 (via Bioconductor in R 4.3.3) using the assignTaxonomy function with the Greengenes2 2024.09 training FASTA: "DADA2 version 1.30 (via Bioconductor in R 4.3.3) using the assignTaxonomy function with the Greengenes2 2024.09 training FASTA"
- [abstract] DeepTaxa exceeds DADA2 (90.05%) and QIIME 2 (85.01%) at the species rank on the same held-out test set: "DeepTaxa exceeds DADA2 (90.05%) and QIIME 2 (85.01%) at the species rank on the same held-out test set"
- [methods] we conducted a benchmarking study comparing its performance against widely used taxonomic classification tools, including DADA2 (Callahan et al. 2016), QIIME 2, Kraken2, and SINTAX: "we conducted a benchmarking study comparing its performance against widely used taxonomic classification tools, including DADA2 (Callahan et al. 2016), QIIME 2, Kraken2, and SINTAX"
- [results] under its recommended 0.8 bootstrap-confidence cutoff, many species-level assignments fall below the threshold and are returned as unclassified: "under its recommended 0.8 bootstrap-confidence cutoff, many species-level assignments fall below the threshold and are returned as unclassified, lowering its species accuracy to 73.13%"
- [readme] gg_2024_09_testing.fna.gz     (69,335 sequences, ~24 MB): [evidence span withheld — anchor status: external_doc]
