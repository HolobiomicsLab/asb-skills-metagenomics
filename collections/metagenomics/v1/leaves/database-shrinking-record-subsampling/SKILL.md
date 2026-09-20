---
name: database-shrinking-record-subsampling
description: Use when a Kraken database (e.g., the ~70-GB standard RefSeq build) exceeds
  available RAM and a reduced-memory variant must be created by record subsampling,
  and the resulting sensitivity/precision deltas need to be measured.
when_to_use_negative:
- The database already fits in available RAM — subsampling only removes k-mers and
  strictly reduces sensitivity on real metagenomes.
- The goal is maximum precision per classified read on high-error data — MiniKraken
  is more precise, but if sensitivity on high-error simulated reads (simBA-5) matters,
  the full database is required.
- You need species- or strain-level resolution from a low-memory build — the article's
  validation is framed at the genus level only.
edam_operation: http://edamontology.org/operation_3472
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kraken
  role: Classification engine used to build, subsample, and query the standard and
    MiniKraken databases and to compute genus-level sensitivity/precision
  repo: https://github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter used to create the initial database whose count
    spaces are overwritten with LCA taxonomic IDs
- name: Mason
  role: Read simulator used to generate the simBA-5 and simMC validation metagenomes
    with its Illumina model
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:6685d3ac33d4ad4607031d19acd7440f1770e3a615717b11ad1337533d16e8c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DerrickWood/kraken
  tools:
  - Jellyfish
  - Kraken
  - Mason
  edam_operation: http://edamontology.org/operation_3472
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# database-shrinking-record-subsampling

## Summary

Reduce a large Kraken k-mer/LCA database to fit memory-constrained machines (MiniKraken) by deterministically removing records in fixed-size blocks, then quantify the accuracy trade-off as genus-level sensitivity loss versus precision gain compared to the full database.

## When to use

Use when a Kraken database (e.g., the ~70-GB standard RefSeq build) exceeds available RAM and a reduced-memory variant must be created by record subsampling, and the resulting sensitivity/precision deltas need to be measured.

## When NOT to use

- The database already fits in available RAM — subsampling only removes k-mers and strictly reduces sensitivity on real metagenomes.
- The goal is maximum precision per classified read on high-error data — MiniKraken is more precise, but if sensitivity on high-error simulated reads (simBA-5) matters, the full database is required.
- You need species- or strain-level resolution from a low-memory build — the article's validation is framed at the genus level only.

## Inputs

- Standard Kraken database built from complete RefSeq microbial genomes (Jellyfish k-mer counts overwritten with LCA taxonomic IDs)
- NCBI taxonomy files for LCA assignment
- Real-sequence metagenome reads (SRA accessions SRS019120, SRS014468, SRS015055; FASTQ)
- Simulated metagenome reads generated with Mason (SNP/indel rate 0.1%, multiplied by five for simBA-5)
- Memory budget for the shrunk database (target 4 GB; shrinking factor 19)

## Outputs

- MiniKraken 4-GB database of subsampled k-mer/LCA records
- Genus-level sensitivity and precision table (accuracy_table.csv) for standard Kraken vs MiniKraken vs -Q mode
- Sensitivity deltas per metagenome (real-sequence, simBA-5, simMC)

## How to apply

Build the standard Kraken database first: count k-mers from complete RefSeq microbial genomes with Jellyfish, then overwrite the 4-byte count spaces with LCA taxonomic IDs derived from the NCBI taxonomy. To shrink it, generate a MiniKraken 4-GB database by removing the first 18 of every block of 19 records from the standard database (shrinking factor 19), a deterministic subsample rather than a random one. Classify the same read sets (real-sequence HiSeq/MiSeq metagenomes from SRA accessions SRS019120, SRS014468, SRS015055, and Mason-simulated simBA-5/simMC sets) with both the standard and MiniKraken databases, and optionally with quick-operation (-Q) mode. Compute genus-level sensitivity and precision for each database/mode combination, excluding reads whose origin lacks a taxonomic entry at the measured or excluded rank, or whose excluded-rank clade is not shared by at least two other represented taxa. Compile deltas into an accuracy table: accept the shrinkage when MiniKraken shows roughly an 11% genus-level sensitivity drop on real-sequence metagenomes, a >25-point drop on the high-error simBA-5 metagenome, precision at or above the standard Kraken's, and no such penalty on the low-complexity simMC set.

## Related tools

- **Kraken** (Classification engine used to build, subsample, and query the standard and MiniKraken databases and to compute genus-level sensitivity/precision) — https://github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter used to create the initial database whose count spaces are overwritten with LCA taxonomic IDs)
- **Mason** (Read simulator used to generate the simBA-5 and simMC validation metagenomes with its Illumina model)

## Examples

```
kraken --db minikraken_4GB --threads 8 reads.fastq > classified.txt; kraken-report --db minikraken_4GB classified.txt > kraken_report.txt
```

## Evaluation signals

- Genus-level sensitivity on real-sequence metagenomes (HiSeq/MiSeq) drops approximately 11% relative to standard Kraken.
- Sensitivity drop on the high-error simBA-5 metagenome exceeds 25 percentage points.
- MiniKraken precision is at or above standard Kraken's precision on all three real-sequence metagenomes.
- Shrunk database size is approximately 4 GB, consistent with removing the first 18 of every 19 records (shrinking factor 19).
- Classification of simMC (medium-complexity, 31 genera) shows a much smaller accuracy penalty, confirming the loss tracks database/genome coverage.

## Limitations

- MiniKraken's genus-level sensitivity on real-sequence metagenomes drops ~11% versus full Kraken, and by more than 25 percentage points on the high-error simBA-5 metagenome.
- Shrinking is deterministic block removal (first 18 of every 19 records), so which k-mers survive is a fixed subsample; no alternative sampling schemes are evaluated.
- Accuracy was validated only at the genus level and only on three real-sequence metagenomes plus simulated sets; read pairs were concatenated with 'NNNNN' and treated as a single unit.
- The Kraken README provides no operational detail on building MiniKraken databases; subsampling specifics come from the article text only.

## Evidence

- [methods] we removed the first 18 of every block of 19 records in the standard Kraken database: "we removed the first 18 of every block of 19 records in the standard Kraken database"
- [methods] the 4-byte spaces Jellyfish used to store the k-mer counts in the database file are instead used by Kraken to store the taxonomic ID numbers: "the 4-byte spaces Jellyfish used to store the k-mer counts in the database file are instead used by Kraken to store the taxonomic ID numbers"
- [methods] If a read's origin lacked a taxonomic entry at either of the measured or excluded ranks, it was not used for that particular experiment: [evidence span withheld — anchor status: not_found]
- [results] We retrieved the sequences of three accessions (SRS019120, SRS014468 and SRS015055) from the NCBI Sequence Read Archive: "We retrieved the sequences of three accessions (SRS019120, SRS014468 and SRS015055) from the NCBI Sequence Read Archive"
- [readme] Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken.: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
