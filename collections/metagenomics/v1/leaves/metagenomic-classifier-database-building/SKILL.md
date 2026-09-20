---
name: metagenomic-classifier-database-building
description: Use when building a custom Kraken taxonomy database from a set of reference
  genomes (e.g. NCBI RefSeq complete microbial genomes) so every k-mer maps to an
  LCA taxonomic label.
when_to_use_negative:
- When the goal is abundance estimation rather than per-read classification — abundance
  programs like MetaPhlAn only classify a small percentage of sequences and serve
  a different purpose
- When a prebuilt Kraken database already covers the taxa of interest and rebuilding
  from a custom genome library is unnecessary
- When classification requires inexact alignment of divergent sequences — Kraken relies
  on exact k-mer matches, so highly novel taxa absent from the library may be misclassified
  (e.g. Pelosinus fermentans not identified at genus level)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kraken
  role: Classifier whose k-mer-to-LCA database this skill constructs; its build step
    writes LCA taxon IDs into the Jellyfish database structure
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter used to create the initial 31-mer count database
    over the genomic library before LCA IDs overwrite the count spaces
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:b3f2d38e9fe59357a1239178dda392416595a5cf62a9b91a9fa1e8b57d5086bb
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
yes_rate: 1.0
n_claims: 5
---

# metagenomic-classifier-database-building

## Summary

Constructs a k-mer-to-LCA taxonomy database for the Kraken metagenomic classifier by counting 31-mers with Jellyfish over a user-specified genome library (e.g. NCBI RefSeq complete microbial genomes) and storing each k-mer's lowest common ancestor taxon ID in place of its count. This exact-match database enables Kraken's ultrafast k-mer-based sequence classification.

## When to use

Use when building a custom Kraken taxonomy database from a set of reference genomes (e.g. NCBI RefSeq complete microbial genomes) so every k-mer maps to an LCA taxonomic label.

## When NOT to use

- When the goal is abundance estimation rather than per-read classification — abundance programs like MetaPhlAn only classify a small percentage of sequences and serve a different purpose
- When a prebuilt Kraken database already covers the taxa of interest and rebuilding from a custom genome library is unnecessary
- When classification requires inexact alignment of divergent sequences — Kraken relies on exact k-mer matches, so highly novel taxa absent from the library may be misclassified (e.g. Pelosinus fermentans not identified at genus level)

## Inputs

- Reference genomic library (e.g. NCBI RefSeq complete microbial genomes, FASTA)
- NCBI taxonomy database files (nodes.dmp, names.dmp)
- k-mer length parameter (default k = 31)

## Outputs

- Kraken database file mapping each 31-mer to its LCA NCBI taxon ID (Jellyfish format with counts replaced by taxonomic IDs)
- Minimizer index over the k-mer records for binary-search lookup

## How to apply

Download the complete microbial genomes from the NCBI RefSeq database as the reference library and retrieve taxon information (nodes.dmp/names.dmp) from the NCBI taxonomy database. Run the Jellyfish multithreaded k-mer counter over the genomic library with k = 31 (Kraken's default) to produce a Jellyfish count database. Then overwrite the 4-byte count spaces in the Jellyfish database file with the taxonomic ID of each k-mer's LCA: process each library sequence one at a time, storing the sequence's taxon ID for each of its 31-mers, and where a k-mer's LCA value was previously set, compute the LCA of the stored value and the current sequence's taxon and store that instead. Use the XOR operation to toggle half the bits of each M-mer's canonical representation before lexicographical comparison so the minimizer distribution is evened out for indexed lookup. K-mers containing ambiguous nucleotides are ignored. Validate by spot-checking that queried k-mers return valid NCBI taxon IDs and that k-mers shared across divergent taxa resolve to their common ancestor's LCA ID.

## Related tools

- **Kraken** (Classifier whose k-mer-to-LCA database this skill constructs; its build step writes LCA taxon IDs into the Jellyfish database structure) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter used to create the initial 31-mer count database over the genomic library before LCA IDs overwrite the count spaces)

## Evaluation signals

- Querying a finished database with k-mers from the library returns valid NCBI taxon IDs
- K-mers shared across divergent taxa resolve to their common ancestor's LCA ID, not to a descendant taxon
- Every non-ambiguous 31-mer in the reference library has a populated taxonomic ID in the 4-byte slot previously used by Jellyfish for counts
- Minimizer index lookups followed by binary search locate each k-mer record consistently (no k-mers missing from their minimizer block)

## Limitations

- K-mers containing ambiguous nucleotides (N) are ignored and cannot contribute to classification
- Taxa absent from the reference genome library cannot be classified correctly (e.g. Pelosinus fermentans was not identified at genus level in the HiSeq metagenome; Proteus vulgaris was often incorrectly classified in the MiSeq metagenome)
- Draft GenBank microbial genomes may contain contaminating sequences from many different sources, which can corrupt k-mer LCA values unless contaminant k-mers are removed

## Evidence

- [discussion] Database construction overwrites Jellyfish counts with LCA taxonomic IDs: [evidence span withheld — anchor status: not_found]
- [discussion] Conflict resolution computes LCA of stored and new taxon: [evidence span withheld — anchor status: not_found]
- [discussion] Reference library and taxonomy sources: [evidence span withheld — anchor status: not_found]
- [discussion] Minimizer XOR trick for even distribution: [evidence span withheld — anchor status: not_found]
- [discussion] Default k-mer length of 31: "Kraken ignores k-mers with ambiguous nucleotides"
- [abstract] Kraken is a k-mer-based classifier using exact alignment: "Using exact alignment of k-mers, Kraken achieves classification accuracy comparable to the fastest BLAST program."
