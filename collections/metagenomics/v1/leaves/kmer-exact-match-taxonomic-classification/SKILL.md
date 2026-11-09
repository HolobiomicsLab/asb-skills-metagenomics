---
name: kmer-exact-match-taxonomic-classification
description: Use when classifying large metagenomic read sets (e.g., HiSeq/MiSeq simulated
  metagenomes, millions of 100 bp Illumina reads) with taxonomic labels, and BLAST-speed
  alignment is too slow but Megablast-level genus accuracy is required.
when_to_use_negative:
- The goal is abundance estimation of marker taxa only (e.g., MetaPhlAn-style profiling),
  not classification of every read
- Reads require inexact/sensitive alignment to distant homologs with no genome represented
  in the database — exact k-mer matching cannot detect them
- Draft/contaminated reference genomes are used without removing adapter k-mers and
  terminal contig k-mers, as Kraken-GB does
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: Kraken
  role: Performs the exact k-mer LCA queries and root-to-leaf path scoring to assign
    taxonomic labels to each read
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter used to build the Kraken database before count
    spaces are overwritten with LCA taxonomic IDs
- name: Megablast
  role: Fastest BLAST program used as the accuracy benchmark for genus-level sensitivity
    and precision comparison
- name: Mason
  role: Read simulator used with its Illumina model to generate the simBA-5 metagenome
    at 5x default mismatch/indel rates (average mismatch rate 2%)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:338f70cb6f4a4aa6636d7bd3cdadf5e8061212d0c4cede1b22cc31f50152f3c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  - Mason
  - Megablast
  - PhymmBL
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# kmer-exact-match-taxonomic-classification

## Summary

Classify metagenomic DNA sequences by exact k-mer lookup in a precomputed LCA database (Kraken), achieving genus-level sensitivity and precision within ~2.5 percentage points of Megablast while running over 900x faster. Uses the standard Kraken database built with Jellyfish k-mer counts overwritten with k-mer LCA taxonomic IDs.

## When to use

Use when classifying large metagenomic read sets (e.g., HiSeq/MiSeq simulated metagenomes, millions of 100 bp Illumina reads) with taxonomic labels, and BLAST-speed alignment is too slow but Megablast-level genus accuracy is required.

## When NOT to use

- The goal is abundance estimation of marker taxa only (e.g., MetaPhlAn-style profiling), not classification of every read
- Reads require inexact/sensitive alignment to distant homologs with no genome represented in the database — exact k-mer matching cannot detect them
- Draft/contaminated reference genomes are used without removing adapter k-mers and terminal contig k-mers, as Kraken-GB does

## Inputs

- FASTQ or FASTA metagenomic read files (e.g., HiSeq, MiSeq simulated metagenomes of 10,000 sequences each)
- Paired-end read pairs optionally concatenated with 'NNNNN' spacer
- Standard Kraken database (Jellyfish k-mer store with LCA taxonomic IDs, NCBI taxonomy)

## Outputs

- Per-read taxonomic assignments from Kraken
- Classifier-by-metagenome genus-level sensitivity and precision table (table1_genus_sensitivity_precision.csv)

## How to apply

Build a standard Kraken database by counting k-mers with the Jellyfish multithreaded k-mer counter, then overwrite the 4-byte count spaces with each k-mer's LCA taxonomic ID derived from the NCBI taxonomy, setting LCAs of stored and new taxa on conflicts. Queries collect all k-mers in a read (ignoring ambiguous nucleotides), map each to its LCA taxon, build a weighted classification tree, and score root-to-leaf paths to pick the label. Trim reads for low-quality bases and adapters first; if pairs are classified as one unit, concatenate mates with an 'NNNNN' spacer since Kraken ignores k-mers spanning ambiguous nucleotides. Classify with exact k-mer LCA queries against the standard database, then score genus-level sensitivity and precision, excluding reads whose origin lacks a genus entry or for which fewer than two other taxa at the excluded rank share the origin's clade. Benchmark against Megablast, PhymmBL, PhymmBL65 and NBC on HiSeq, MiSeq and simBA-5 (Mason, 5x default error rate, ~2% mismatch) metagenomes.

## Related tools

- **Kraken** (Performs the exact k-mer LCA queries and root-to-leaf path scoring to assign taxonomic labels to each read) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter used to build the Kraken database before count spaces are overwritten with LCA taxonomic IDs)
- **Megablast** (Fastest BLAST program used as the accuracy benchmark for genus-level sensitivity and precision comparison)
- **Mason** (Read simulator used with its Illumina model to generate the simBA-5 metagenome at 5x default mismatch/indel rates (average mismatch rate 2%))

## Examples

```
kraken --db $KRAKEN_DB --threads 8 reads.fa > reads.kraken
```

## Evaluation signals

- Kraken's genus-level sensitivity across HiSeq, MiSeq and simBA-5 is within 2.5 percentage points of Megablast's
- Output CSV contains one row per classifier-by-metagenome combination with genus-level sensitivity and precision metrics
- Excluded reads (origin lacks genus entry, or fewer than two other taxa at the excluded rank share the origin's measured-rank clade) are filtered from sensitivity/precision scoring
- Classification speed on 100 bp reads exceeds 4.1 million reads per minute in the fastest mode (909x faster than Megablast)
- Paired reads joined with 'NNNNN' are classified as a single unit, with k-mers spanning the Ns ignored during classification

## Limitations

- Kraken cannot classify organisms absent from its database as novel taxa; e.g., Pelosinus fermentans in the HiSeq metagenome cannot be correctly identified at the genus level, and Proteus vulgaris reads in the MiSeq metagenome are often misclassified at the genus level
- Draft GenBank microbial genomes may contain contaminating sequences, and GenBank draft contig terminal k-mers must be removed to avoid spurious assignments
- K-mers with ambiguous nucleotides are ignored, so the 'NNNNN' spacer between paired mates does not contribute classification evidence
- The MiniKraken 4-GB reduced database (removing 18 of every 19 records) trades memory for potential accuracy loss

## Evidence

- [abstract] Kraken is an ultrafast and highly accurate program for assigning taxonomic labels to metagenomic DNA sequences.: "Kraken is an ultrafast and highly accurate program for assigning taxonomic labels to metagenomic DNA sequences."
- [discussion] Jellyfish count spaces reused for LCA IDs: [evidence span withheld — anchor status: not_found]
- [discussion] Paired mate concatenation with NNNNN spacer: [evidence span withheld — anchor status: not_found]
- [discussion] Ambiguous nucleotide k-mers ignored: [evidence span withheld — anchor status: not_found]
- [discussion] simBA-5 error rate: "we multiplied the default mismatch and indel rates by five, resulting in an average mismatch rate of 2%"
- [readme] README title: "Kraken taxonomic sequence classification system"
