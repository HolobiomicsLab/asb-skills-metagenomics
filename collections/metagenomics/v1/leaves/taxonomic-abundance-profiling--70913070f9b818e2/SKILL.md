---
name: taxonomic-abundance-profiling
description: Use when raw metagenomic FASTQ reads (e.g., Illumina data from NCBI SRA
  accessions) must be assigned taxonomic labels and summarized as a genus-level distribution
  plus unclassified fraction.
when_to_use_negative:
- The goal is strain- or species-level abundance estimation from marker genes (e.g.,
  MetaPhlAn-style profiling) rather than per-read taxonomic label assignment via exact
  k-mer matching
- Input is an assembled contig set or a precomputed feature/abundance table rather
  than raw short-read FASTQ data
- Reads require sensitive inexact alignment because they have no detectable exact
  k-mer similarity to genomes in the database (Kraken relies on exact-match k-mer
  database queries)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kraken
  role: Assigns taxonomic labels to trimmed metagenomic reads via exact k-mer matches
    and the LCA classification tree; generates the per-accession reports used for
    genus-level profiling
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter used to build the Kraken database; its 4-byte
    count spaces are overwritten with k-mer LCA taxonomic IDs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:3fd8702d3b1a0425df7dc6d60eb835c6b544b23b271951d1bf90e6922a6ebc41
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# taxonomic-abundance-profiling

## Summary

Classify metagenomic FASTQ reads with Kraken's exact k-mer/LCA approach and summarize genus-level abundance distributions among classified reads, as done for three Human Microbiome Project saliva samples (SRS019120, SRS014468, SRS015055).

## When to use

Use when raw metagenomic FASTQ reads (e.g., Illumina data from NCBI SRA accessions) must be assigned taxonomic labels and summarized as a genus-level distribution plus unclassified fraction.

## When NOT to use

- The goal is strain- or species-level abundance estimation from marker genes (e.g., MetaPhlAn-style profiling) rather than per-read taxonomic label assignment via exact k-mer matching
- Input is an assembled contig set or a precomputed feature/abundance table rather than raw short-read FASTQ data
- Reads require sensitive inexact alignment because they have no detectable exact k-mer similarity to genomes in the database (Kraken relies on exact-match k-mer database queries)

## Inputs

- FASTQ files for SRA accessions SRS019120, SRS014468, SRS015055 (paired-end Illumina reads)
- Standard Kraken database built from complete bacterial, viral, and human genomes with Jellyfish k-mer counts replaced by LCA taxonomic IDs
- NCBI taxonomy database

## Outputs

- Kraken report per accession with taxonomic classification counts
- Fraction of unclassified reads per sample/combined (expected ~68.2% for the HMP saliva set)
- Genus-level distribution among classified reads (hmp_saliva_kraken_report.tsv): Streptococcus 30%, Haemophilus 17%, Prevotella 13%

## How to apply

Download FASTQ files for the target accessions from the NCBI Sequence Read Archive and trim all sequences to remove low-quality bases and adapter sequences. Build the standard Kraken database by counting k-mers with the Jellyfish multithreaded k-mer counter over complete bacterial, viral, and human genomes, then overwrite Jellyfish's 4-byte count spaces with the k-mers' LCA taxonomic IDs. Run Kraken on the trimmed reads, concatenating paired mates with 'NNNNN' between them so each pair is classified as a single unit (Kraken ignores k-mers spanning ambiguous Ns, so the joiner does not affect classification). Generate a Kraken report per accession, compute the fraction of unclassified reads, and compute the genus-level distribution among classified reads, combining the samples into a single summary.

## Related tools

- **Kraken** (Assigns taxonomic labels to trimmed metagenomic reads via exact k-mer matches and the LCA classification tree; generates the per-accession reports used for genus-level profiling) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter used to build the Kraken database; its 4-byte count spaces are overwritten with k-mer LCA taxonomic IDs)

## Examples

```
kraken --db $KRAKEN_DB --threads 8 trimmed_reads.fa > kraken.out && kraken-report --db $KRAKEN_DB kraken.out > hmp_saliva_kraken_report.tsv
```

## Evaluation signals

- The combined summary file (hmp_saliva_kraken_report.tsv) is produced with per-accession reports and a genus-level distribution among classified reads
- Unclassified fraction is approximately 68.2% (values within a few percentage points treated as consistent)
- Dominant classified genera match Streptococcus (30%), Haemophilus (17%), and Prevotella (13%) within a few percentage points
- Paired reads were concatenated with 'NNNNN' and classified as single units; k-mers spanning the Ns are ignored by Kraken and do not affect classification
- Database construction completed with Jellyfish k-mer count spaces successfully overwritten by LCA taxonomic IDs

## Limitations

- A majority of HMP saliva reads (~68.2%) remain unclassified, so genus-level distributions cover only the classified subset
- Exact k-mer matching means reads without exact database k-mer hits may be misclassified or unclassified; e.g., Pelosinus fermentans and Proteus vulgaris were often incorrectly identified at the genus level in test metagenomes
- k-mer LCA values collapsed on conflicts can push labels up the taxonomy tree when k-mers are shared across taxa

## Evidence

- [discussion] All sequences were trimmed to remove low quality bases and adapter sequences: "All sequences were trimmed to remove low quality bases and adapter sequences"
- [discussion] Paired mates concatenated with NNNNN: [evidence span withheld — anchor status: not_found]
- [discussion] Kraken database stores LCA IDs in Jellyfish count spaces: [evidence span withheld — anchor status: not_found]
- [discussion] Three HMP saliva accessions retrieved from SRA: "We retrieved the sequences of three accessions (SRS019120, SRS014468 and SRS015055) from the NCBI Sequence Read Archive"
- [discussion] Kraken ignores k-mers with ambiguous nucleotides: [evidence span withheld — anchor status: not_found]
- [readme] Kraken README overview: "Kraken taxonomic sequence classification system"
