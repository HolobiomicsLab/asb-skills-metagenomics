---
name: kmer-counting-database-construction
description: 'Use when building a Kraken classification database from a genome library:
  you need each distinct k-mer (default k=31) mapped to the LCA taxon of all genomes
  containing it, using Jellyfish counts plus NCBI taxonomy.'
when_to_use_negative:
- When the organisms of interest are not represented in the genome library — k-mers
  absent from the database cannot be classified, e.g. Pelosinus fermentans could not
  be correctly identified at the genus level
- When reads contain many ambiguous nucleotides, since Kraken ignores k-mers spanning
  'N' characters and they do not affect classification
- When a memory-constrained MiniKraken 4-GB database (built by dropping 18 of every
  19 records) would sacrifice too much specificity for the target taxa
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: Kraken
  role: 'Builds the k-mer-to-LCA database: processes each library sequence, sets/updates
    k-mer LCA taxon IDs in the Jellyfish count database, and later queries the database
    via minimizer index plus binary search'
  repo: https://github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter run over the genomic library to create the count
    database whose 4-byte count spaces Kraken overwrites with LCA taxon IDs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:cc7e9c27da1234b569412e371841487ad7f755d69b23877811f584f10a55db92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DerrickWood/kraken
  tools:
  - Jellyfish
  - Kraken
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# kmer-counting-database-construction

## Summary

Constructs a Kraken taxonomic database by counting 31-mers from a user-specified library of genomes (e.g., complete NCBI RefSeq microbial genomes) with Jellyfish, then overwriting each 4-byte k-mer count with the NCBI taxon ID of the k-mer's lowest common ancestor (LCA). This exact k-mer-to-LCA map is what enables Kraken's ultrafast metagenomic classification.

## When to use

Use when building a Kraken classification database from a genome library: you need each distinct k-mer (default k=31) mapped to the LCA taxon of all genomes containing it, using Jellyfish counts plus NCBI taxonomy.

## When NOT to use

- When the organisms of interest are not represented in the genome library — k-mers absent from the database cannot be classified, e.g. Pelosinus fermentans could not be correctly identified at the genus level
- When reads contain many ambiguous nucleotides, since Kraken ignores k-mers spanning 'N' characters and they do not affect classification
- When a memory-constrained MiniKraken 4-GB database (built by dropping 18 of every 19 records) would sacrifice too much specificity for the target taxa

## Inputs

- Complete microbial genomes from the NCBI RefSeq database (reference genomic library)
- NCBI taxonomy database files (nodes.dmp/names.dmp) with taxon IDs
- Choice of k-mer length (Kraken default k=31)

## Outputs

- Kraken database file mapping each k-mer to its LCA taxon ID (4-byte taxon IDs replacing Jellyfish k-mer counts)
- Optionally a MiniKraken reduced 4-GB database obtained by removing the first 18 of every block of 19 records in the standard Kraken database

## How to apply

Download complete microbial genomes from the NCBI RefSeq database as the reference library and retrieve taxon information (nodes.dmp/names.dmp) from the NCBI taxonomy database to support LCA computation. Run the Jellyfish multithreaded k-mer counter over the genomic library to count all distinct 31-mers (Kraken's default k), producing a Jellyfish count database. Then overwrite the 4-byte spaces Jellyfish used to store k-mer counts with the taxonomic ID numbers of the k-mers' LCA values. Process each library sequence one at a time: for each of its 31-mers, store the sequence's taxon ID; if a k-mer's LCA value was previously set, compute the LCA of the stored value and the current sequence's taxon and store that LCA instead. Rationale: shared k-mers across divergent taxa resolve to their common ancestor, so a query k-mer returns the most specific taxonomic node consistent with all genomes containing it.

## Related tools

- **Kraken** (Builds the k-mer-to-LCA database: processes each library sequence, sets/updates k-mer LCA taxon IDs in the Jellyfish count database, and later queries the database via minimizer index plus binary search) — https://github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter run over the genomic library to create the count database whose 4-byte count spaces Kraken overwrites with LCA taxon IDs)

## Evaluation signals

- Spot-check the finished database by querying k-mers and confirming each returns a valid NCBI taxon ID
- Confirm that k-mers shared across divergent taxa resolve to their common ancestor's LCA ID rather than any single taxon
- Verify the database file layout: 4-byte values at each k-mer record now hold LCA taxon IDs in place of Jellyfish counts
- Check k-mer length matches the intended default of 31 for Kraken

## Limitations

- Draft microbial genomes in GenBank can contain contaminating sequences from many different sources (Kraken-GB removes k-mers from known adapter sequences and the first and last 20 k-mers of each draft contig to mitigate this)
- Kraken ignores k-mers with ambiguous nucleotides, so k-mers spanning 'N' characters do not affect classification
- Speed evaluation requires pre-loading database files into memory (three reads into the OS cache), so the database must fit in RAM for full-speed operation
- Species absent or under-represented in the RefSeq library may be misclassified at shallower ranks (e.g., genus-level errors for Pelosinus fermentans and Proteus vulgaris)

## Evidence

- [discussion] Overwrite count spaces with LCA IDs: [evidence span withheld — anchor status: not_found]
- [discussion] LCA conflict resolution per sequence: [evidence span withheld — anchor status: not_found]
- [intro] Speed rationale for exact k-mer lookup: "This speed advantage derives in large part from the use of exact-match database queries of k-mers, rather than inexact alignment of sequences"
- [discussion] RefSeq library and NCBI taxonomy as inputs: [evidence span withheld — anchor status: not_found]
- [readme] README pointer to manual for operating Kraken: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
