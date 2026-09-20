---
name: metagenomic-classifier-benchmarking
description: Use when comparing taxonomic classification accuracy (sensitivity/precision)
  of metagenomic classifiers on simulated read sets with known origins, e.g. Kraken
  vs BLAST-based or k-mer-based classifiers.
when_to_use_negative:
- Reads lack ground-truth taxonomic origins (e.g. real environmental samples), so
  sensitivity and precision cannot be computed
- The task is abundance estimation rather than per-read classification; abundance
  programs like MetaPhlAn classify only a small percentage of sequences and are not
  comparable under this protocol
- Reads come from taxa with no genus-level entry in the database or with fewer than
  two other taxa sharing the clade, since the exclusion rules void the measurement
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kraken
  role: Primary classifier under evaluation; assigns taxonomic labels via exact k-mer
    LCA queries against the standard Kraken database
  repo: https://github.com/DerrickWood/kraken
- name: Megablast
  role: Alignment-based comparison classifier; the fastest BLAST program used as the
    accuracy reference point
- name: PhymmBL
  role: Comparison classifier combining BLAST results with interpolated Markov model
    scores
- name: Naive Bayes Classifier (NBC)
  role: Comparison classifier applying a Bayesian rule to distributions of k-mers
    within a genome
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:5a0462f276ed00ebdfb78e10db677f759bc04c4e859e79f68b3fae9692a85254
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DerrickWood/kraken
  tools:
  - Kraken
  - Megablast
  - Naive Bayes Classifier (NBC)
  - PhymmBL
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# metagenomic-classifier-benchmarking

## Summary

Benchmark genus-level sensitivity and precision of metagenomic sequence classifiers (Kraken, Megablast, PhymmBL, PhymmBL65, NBC) on simulated metagenomes such as HiSeq, MiSeq and simBA-5, using a standardized read preprocessing and clade-aware scoring protocol.

## When to use

Use when comparing taxonomic classification accuracy (sensitivity/precision) of metagenomic classifiers on simulated read sets with known origins, e.g. Kraken vs BLAST-based or k-mer-based classifiers.

## When NOT to use

- Reads lack ground-truth taxonomic origins (e.g. real environmental samples), so sensitivity and precision cannot be computed
- The task is abundance estimation rather than per-read classification; abundance programs like MetaPhlAn classify only a small percentage of sequences and are not comparable under this protocol
- Reads come from taxa with no genus-level entry in the database or with fewer than two other taxa sharing the clade, since the exclusion rules void the measurement

## Inputs

- Simulated metagenome read sets (HiSeq, MiSeq, simBA-5 FASTQ) with known ground-truth genome origins
- Standard Kraken database built with Jellyfish k-mer LCA values from NCBI RefSeq complete microbial genomes
- NCBI taxonomy database for genus-level clade resolution

## Outputs

- Classifier-by-metagenome table of genus-level sensitivity and precision (e.g. table1_genus_sensitivity_precision.csv)
- Per-classifier taxonomic assignments for every read in each metagenome

## How to apply

Retrieve the simulated metagenomes (HiSeq, MiSeq, and simBA-5 simulated with Mason at a 5x-default error rate, i.e. ~2% average mismatch rate). Trim all reads to remove low-quality bases and adapter sequences, then concatenate paired mates with an 'NNNNN' spacer so each pair is classified as a single unit (Kraken ignores k-mers spanning ambiguous bases). Classify reads with Kraken's exact k-mer LCA queries against the standard Kraken database, and with Megablast, PhymmBL, PhymmBL65 and the Naive Bayes Classifier. Score genus-level sensitivity and precision per classifier per metagenome, excluding reads whose origin lacks a genus-level taxonomic entry or for which fewer than two other taxa at the excluded rank share the origin's clade at the measured rank. Assemble a classifier-by-metagenome metrics table and compare Kraken against Megablast, the fastest BLAST program.

## Related tools

- **Kraken** (Primary classifier under evaluation; assigns taxonomic labels via exact k-mer LCA queries against the standard Kraken database) — https://github.com/DerrickWood/kraken
- **Megablast** (Alignment-based comparison classifier; the fastest BLAST program used as the accuracy reference point)
- **PhymmBL** (Comparison classifier combining BLAST results with interpolated Markov model scores)
- **Naive Bayes Classifier (NBC)** (Comparison classifier applying a Bayesian rule to distributions of k-mers within a genome)

## Examples

```
kraken --db $KRAKEN_DB --output kraken_hiseq.out hiseq_trimmed_mates_NNNNN.fa
```

## Evaluation signals

- Kraken's genus-level sensitivity should fall within 2.5 percentage points of Megablast's across all three simulated metagenomes (HiSeq, MiSeq, simBA-5)
- Each read should be classified exactly once by each classifier, with paired mates joined by 'NNNNN' treated as a single classified unit
- Reads excluded by the clade rules (no genus-level entry, or fewer than two other taxa at the excluded rank sharing the measured-rank clade) must be dropped identically for all classifiers
- Output CSV must contain one sensitivity and one precision value per classifier-per-metagenome cell
- Known hard cases, e.g. Pelosinus fermentans in the HiSeq metagenome and Proteus vulgaris in the MiSeq metagenome, should show elevated genus-level misclassification

## Limitations

- Paired mates joined with 'NNNNN' are ignored by Kraken at k-mers spanning the ambiguous bases, slightly reducing usable k-mer evidence per pair
- Some true origins cannot be correctly identified at genus level, e.g. Pelosinus fermentans (HiSeq metagenome) and Proteus vulgaris (MiSeq metagenome)
- simBA-5 uses a 5x-multiplied Mason default error model (~2% average mismatch rate); results may not transfer to reads with different error profiles
- Reads lacking a taxonomic entry at the measured/excluded rank, or lacking sister taxa sharing the clade, must be excluded and reduce effective sample size

## Evidence

- [results] Across all three simulated metagenomes (HiSeq, MiSeq, simBA-5), Kraken's genus-level sensitivity was within 2.5 percentage points of Megablast's: [evidence span withheld — anchor status: not_found]
- [methods] Paired-read concatenation: [evidence span withheld — anchor status: not_found]
- [methods] simBA-5 error model: "we multiplied the default mismatch and indel rates by five, resulting in an average mismatch rate of 2%"
- [methods] Clade exclusion rule: [evidence span withheld — anchor status: not_found]
- [methods] Trimming step: "All sequences were trimmed to remove low quality bases and adapter sequences"
- [discussion] Ambiguous k-mer handling: [evidence span withheld — anchor status: not_found]
- [readme] README pointer: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
