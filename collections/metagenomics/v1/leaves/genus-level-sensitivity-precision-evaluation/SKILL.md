---
name: genus-level-sensitivity-precision-evaluation
description: Use when comparing metagenomic sequence classifiers on the same read
  sets and you need genus-level sensitivity/precision tables, or when quantifying
  accuracy loss from a reduced Kraken database such as MiniKraken 4-GB or Kraken-GB.
when_to_use_negative:
- The goal is abundance estimation rather than per-read classification — abundance
  programs (e.g. MetaPhlAn) classify only a small percentage of sequences and are
  not comparable on per-read sensitivity/precision
- Reads lack a genus-level taxonomic entry for their origin, or fewer than two other
  taxa at the excluded rank share the origin's clade — such reads must be excluded,
  so evaluation on such data is invalid
- The database variant under test was not built by the documented procedure (e.g.
  MiniKraken shrinking factor 19, Kraken-GB adapter and contig-end k-mer removal),
  since the expected accuracy deltas will not be reproducible
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: Kraken
  role: Exact k-mer LCA classifier being benchmarked; also the classifier used with
    the MiniKraken, Kraken-GB and -Q variants
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter used to build the standard Kraken database before
    LCA overwriting
- name: Megablast
  role: Fastest BLAST program used as the accuracy comparator
- name: PhymmBL
  role: Alignment + interpolated Markov model classifier included in the genus-level
    comparison
- name: Naive Bayes Classifier (NBC)
  role: k-mer Bayesian classifier included in the genus-level comparison
- name: Mason
  role: Illumina-model read simulator used to create the simBA-5 metagenome at 5x-default
    error rates
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:c0ed19c652a69b37e4c93dc3224a556ac4883e6f3d7b8f31871b65be69d67d54
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  - Mason
  - Megablast
  - Naive Bayes Classifier (NBC)
  - PhymmBL
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# genus-level-sensitivity-precision-evaluation

## Summary

Benchmark taxonomic classifiers by computing genus-level sensitivity and precision on metagenomic read sets (real HiSeq/MiSeq and simulated simBA-5/simMC reads), comparing Kraken against Megablast, PhymmBL, PhymmBL65 and NBC, and measuring database-reduction effects (MiniKraken, Kraken-GB, -Q mode).

## When to use

Use when comparing metagenomic sequence classifiers on the same read sets and you need genus-level sensitivity/precision tables, or when quantifying accuracy loss from a reduced Kraken database such as MiniKraken 4-GB or Kraken-GB.

## When NOT to use

- The goal is abundance estimation rather than per-read classification — abundance programs (e.g. MetaPhlAn) classify only a small percentage of sequences and are not comparable on per-read sensitivity/precision
- Reads lack a genus-level taxonomic entry for their origin, or fewer than two other taxa at the excluded rank share the origin's clade — such reads must be excluded, so evaluation on such data is invalid
- The database variant under test was not built by the documented procedure (e.g. MiniKraken shrinking factor 19, Kraken-GB adapter and contig-end k-mer removal), since the expected accuracy deltas will not be reproducible

## Inputs

- FASTQ/FASTA read sets from HiSeq and MiSeq metagenomes (SRA accessions SRS019120, SRS014468, SRS015055)
- Simulated metagenomes generated with Mason (SNP rate 0.1%, indel rate 0.1%; 5x-default for simBA-5)
- Kraken standard database built from complete RefSeq microbial genomes with NCBI taxonomy
- Genus-level taxonomic ground-truth (origin taxon at the measured rank) for each read

## Outputs

- table1_genus_sensitivity_precision.csv: classifier-by-metagenome genus-level sensitivity and precision
- accuracy_table.csv: sensitivity/precision for standard Kraken vs MiniKraken, -Q mode, and Kraken-GB

## How to apply

Build the standard Kraken database by counting k-mers from complete RefSeq microbial genomes with Jellyfish and overwriting the 4-byte count spaces with LCA taxonomic IDs from the NCBI taxonomy. Trim all reads to remove low-quality bases and adapter sequences, and concatenate paired mates with an 'NNNNN' spacer so each pair is classified as a single unit. Classify identical read sets with Kraken and with each comparator (Megablast, PhymmBL, PhymmBL65, NBC); for database-reduction studies, also build MiniKraken by removing the first 18 of every block of 19 records (shrinking factor 19) and Kraken-GB from GenBank draft genomes with adapter k-mers plus the first and last 20 k-mers of each draft contig removed, plus quick-operation (-Q) mode. Score genus-level sensitivity and precision per classifier-by-metagenome combination, excluding reads whose origin lacks a taxonomic entry at the measured or excluded rank, or whose excluded-rank clade is not shared by at least two other represented taxa. Validate by checking that computed deltas reproduce the reported pattern: ~11% MiniKraken sensitivity drop on real-sequence metagenomes, >25-point drop on simBA-5 (5x Mason error, ~2% mismatch rate), MiniKraken precision at or above standard Kraken, and Kraken sensitivity within 2.5 points of Megablast on HiSeq, MiSeq and simBA-5.

## Related tools

- **Kraken** (Exact k-mer LCA classifier being benchmarked; also the classifier used with the MiniKraken, Kraken-GB and -Q variants) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter used to build the standard Kraken database before LCA overwriting)
- **Megablast** (Fastest BLAST program used as the accuracy comparator)
- **PhymmBL** (Alignment + interpolated Markov model classifier included in the genus-level comparison)
- **Naive Bayes Classifier (NBC)** (k-mer Bayesian classifier included in the genus-level comparison)
- **Mason** (Illumina-model read simulator used to create the simBA-5 metagenome at 5x-default error rates)

## Evaluation signals

- Kraken's genus-level sensitivity is within 2.5 percentage points of Megablast's on HiSeq, MiSeq and simBA-5
- MiniKraken genus-level sensitivity drops ~11% relative to standard Kraken on real-sequence metagenomes (HiSeq/MiSeq)
- MiniKraken sensitivity drops by more than 25 percentage points on the high-error simBA-5 metagenome
- MiniKraken precision is at or above standard Kraken's precision on all three metagenomes
- Kraken-GB sensitivity exceeds standard Kraken's on the HiSeq and MiSeq metagenomes

## Limitations

- Kraken cannot correctly identify some genera present in the data, e.g. Pelosinus fermentans in the HiSeq metagenome and Proteus vulgaris in the MiSeq metagenome, which is often classified incorrectly at the genus level
- Reads whose origin lacks a taxonomic entry at either the measured or excluded rank must be dropped from the experiment, shrinking the evaluated set
- Kraken ignores k-mers with ambiguous nucleotides, so k-mers spanning 'N' spacers in concatenated mate pairs do not affect classification
- Jellyfish and Mason versions are not specified in the article, which may affect exact reproducibility

## Evidence

- [discussion] MiniKraken 4-GB database by removing the first 18 of every block of 19 records: "we removed the first 18 of every block of 19 records in the standard Kraken database"
- [discussion] Paired-mate concatenation protocol for per-pair classification: [evidence span withheld — anchor status: not_found]
- [discussion] simBA-5 error model: 5x-default mismatch/indel rates: "we multiplied the default mismatch and indel rates by five, resulting in an average mismatch rate of 2%"
- [discussion] Read-exclusion rule requiring two other taxa at the excluded rank: [evidence span withheld — anchor status: not_found]
- [discussion] Pre-classification trimming of all read sets: "All sequences were trimmed to remove low quality bases and adapter sequences"
- [readme] Kraken operation and installation are documented in the project README/manual: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
