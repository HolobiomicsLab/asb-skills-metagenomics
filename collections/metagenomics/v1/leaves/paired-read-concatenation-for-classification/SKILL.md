---
name: paired-read-concatenation-for-classification
description: Use when classifying paired-end reads from simulated or real metagenomes
  (e.g., HiSeq, MiSeq or simBA-5 read sets) with a k-mer LCA classifier that accepts
  single sequences, and you want both mates of a pair to vote together for one taxon.
when_to_use_negative:
- When the classifier accepts paired-end input natively or when read mates must be
  scored independently (e.g., per-mate sensitivity analyses)
- When k-mer content spanning the mate junction matters — the 'NNNNN' spacer creates
  ambiguous k-mers that Kraken ignores, so any signal across the insert is lost
- For long single reads or contigs that are not paired-end data
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Kraken
  role: Taxonomic classifier that consumes the 'NNNNN'-concatenated pairs as single
    units via exact k-mer LCA queries against the standard Kraken database
  repo: github.com/DerrickWood/kraken
- name: Mason
  role: Read simulator used to generate the simBA-5 metagenome (5x-default mismatch/indel
    rates, ~2% average mismatch) whose paired reads are concatenated before classification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:069ae28d2c95b90b957ae8a3afb8e1d14fc5c913e8ee07fe58cfe78b500c49bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Kraken
  - Mason
  - Megablast
  - PhymmBL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# paired-read-concatenation-for-classification

## Summary

Concatenates paired-end metagenomic read mates with an 'NNNNN' spacer so that a taxonomic classifier such as Kraken processes each mate pair as a single sequence unit rather than two independent reads, preserving paired evidence during LCA-based k-mer classification.

## When to use

Use when classifying paired-end reads from simulated or real metagenomes (e.g., HiSeq, MiSeq or simBA-5 read sets) with a k-mer LCA classifier that accepts single sequences, and you want both mates of a pair to vote together for one taxon.

## When NOT to use

- When the classifier accepts paired-end input natively or when read mates must be scored independently (e.g., per-mate sensitivity analyses)
- When k-mer content spanning the mate junction matters — the 'NNNNN' spacer creates ambiguous k-mers that Kraken ignores, so any signal across the insert is lost
- For long single reads or contigs that are not paired-end data

## Inputs

- Trimmed paired-end FASTQ read files (mate 1 and mate 2) from a simulated or real metagenome
- Pre-built Kraken standard database (k-mer LCA table derived from RefSeq complete microbial genomes and the NCBI taxonomy)

## Outputs

- One concatenated FASTA/FASTQ record per read pair, with mates joined by the 'NNNNN' spacer
- Per-pair taxonomic assignments from Kraken (classified/unclassified labels with taxonomy IDs)
- Per-metagenome genus-level sensitivity and precision metrics for the classifier comparison table

## How to apply

After trimming low-quality bases and adapter sequences, join each read pair by concatenating the mates with the five-nucleotide spacer 'NNNNN' between them, producing one FASTA/FASTQ record per pair. Kraken ignores k-mers containing ambiguous nucleotides, so k-mers spanning the 'NNNNN' junction are automatically discarded and do not influence the k-mer-to-LCA database lookups; the surviving k-mers from both mates jointly populate the classification tree for a single root-to-leaf path score. Classify the concatenated records with Kraken against the standard Kraken database (exact k-mer LCA queries) in the same run as, and identically to, any single-end control reads. This was the treatment applied to the HiSeq, MiSeq and simBA-5 metagenomes (the latter simulated with Mason at a 5x-default error rate, ~2% average mismatch) before scoring genus-level sensitivity and precision against Megablast, PhymmBL, PhymmBL65 and NBC.

## Related tools

- **Kraken** (Taxonomic classifier that consumes the 'NNNNN'-concatenated pairs as single units via exact k-mer LCA queries against the standard Kraken database) — github.com/DerrickWood/kraken
- **Mason** (Read simulator used to generate the simBA-5 metagenome (5x-default mismatch/indel rates, ~2% average mismatch) whose paired reads are concatenated before classification)

## Examples

```
kraken --db $DBNAME concatenated_pairs.fa > pairs.kraken.out
```

## Evaluation signals

- Every output record contains exactly one 'NNNNN' spacer separating the two mates, with no adapter or low-quality bases remaining upstream of concatenation
- Kraken reports one classification (or unclassified label) per concatenated pair, not two — record counts in input and output match
- Unambiguous k-mers from both mates contribute to a single root-to-leaf path score; spot-check that a pair's label is consistent with joint (not split) k-mer evidence
- Genus-level sensitivity of Kraken on the concatenated read sets falls within ~2.5 percentage points of Megablast's on the HiSeq, MiSeq and simBA-5 metagenomes
- Reads whose origin lacks a genus-level entry, or with fewer than two other taxa sharing the origin's clade at the measured rank, are excluded from the sensitivity/precision scoring

## Limitations

- Kraken ignores k-mers with ambiguous nucleotides, so the k-mers spanning the 'NNNNN' spacer contribute no classification evidence
- Kraken could not correctly identify some organisms at the genus level (e.g., Pelosinus fermentans in the HiSeq metagenome, Proteus vulgaris in the MiSeq metagenome), which concatenation does not fix
- Database composition matters: draft GenBank genomes may contain contaminating sequences, and genus-level precision depends on taxonomic entries in the database

## Evidence

- [discussion] Pair mates joined with N spacer: [evidence span withheld — anchor status: not_found]
- [discussion] Ambiguous k-mers ignored by Kraken: [evidence span withheld — anchor status: not_found]
- [discussion] Pre-classification trimming: "All sequences were trimmed to remove low quality bases and adapter sequences"
- [discussion] simBA-5 simulation parameters: "we multiplied the default mismatch and indel rates by five, resulting in an average mismatch rate of 2%"
- [readme] README pointer to Kraken usage: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
