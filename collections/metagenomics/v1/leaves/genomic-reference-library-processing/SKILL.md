---
name: genomic-reference-library-processing
description: Use when building a Kraken-compatible taxonomy database from NCBI RefSeq
  complete microbial genomes so that exact k-mer lookups resolve to specific taxon
  IDs for metagenomic classification.
when_to_use_negative:
- The goal is abundance estimation of metagenomes (programs like MetaPhlAn classify
  only a small percentage of sequences and do not build k-mer-to-LCA databases)
- The classification strategy requires inexact alignment of sequences rather than
  exact-match database queries of k-mers
- Draft GenBank microbial genomes contain contaminating sequences and cannot be used
  directly without removing adapter-derived k-mers and terminal k-mers of draft contigs
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kraken
  role: Builds the k-mer-to-LCA database by overwriting Jellyfish count spaces with
    LCA taxonomic IDs and uses the database for exact k-mer lookup
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter used to create the initial 31-mer count database
    over the genomic reference library
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:bd7339c15e06dbfb5f1319ceb816fec8e3609be9db23e919b7f253c9dcb6d37d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# genomic-reference-library-processing

## Summary

Constructs a Kraken k-mer-to-LCA database from a user-specified library of complete genomes: count all 31-mers with Jellyfish, then overwrite the 4-byte count spaces with the NCBI taxonomic ID of each k-mer's lowest common ancestor (LCA).

## When to use

Use when building a Kraken-compatible taxonomy database from NCBI RefSeq complete microbial genomes so that exact k-mer lookups resolve to specific taxon IDs for metagenomic classification.

## When NOT to use

- The goal is abundance estimation of metagenomes (programs like MetaPhlAn classify only a small percentage of sequences and do not build k-mer-to-LCA databases)
- The classification strategy requires inexact alignment of sequences rather than exact-match database queries of k-mers
- Draft GenBank microbial genomes contain contaminating sequences and cannot be used directly without removing adapter-derived k-mers and terminal k-mers of draft contigs

## Inputs

- Complete microbial genomes from the NCBI RefSeq database (reference genomic library)
- NCBI taxonomy database files (nodes.dmp/names.dmp) providing taxon information

## Outputs

- Kraken database file in which each 31-mer record stores the NCBI taxonomic ID of the k-mer's LCA (originally Jellyfish count storage space repurposed)

## How to apply

Download complete microbial genomes from NCBI RefSeq as the reference library and retrieve NCBI taxonomy files (nodes.dmp/names.dmp) to support LCA computation. Run the Jellyfish multithreaded k-mer counter over the library with the default k of 31 to produce a count database. Process each library sequence one at a time, storing the sequence's taxon ID for each 31-mer; when a k-mer's LCA value was previously set, compute the LCA of the stored value and the current sequence's taxon and store that LCA instead. Finally, overwrite the 4-byte spaces Jellyfish used for k-mer counts with the taxonomic ID numbers of the k-mers' LCA values, yielding the lookup database used by Kraken's exact-match classification.

## Related tools

- **Kraken** (Builds the k-mer-to-LCA database by overwriting Jellyfish count spaces with LCA taxonomic IDs and uses the database for exact k-mer lookup) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter used to create the initial 31-mer count database over the genomic reference library)

## Evaluation signals

- Query k-mers from the finished database and confirm each returns a valid NCBI taxon ID
- Confirm k-mers shared across divergent taxa resolve to their common ancestor's LCA ID rather than a specific taxon
- Verify that the 4-byte count fields originally written by Jellyfish now contain taxonomic ID numbers of the k-mers' LCA values
- Check that classification accuracy on benchmark reads (e.g., simBA-5) is comparable to the fastest BLAST program while exact k-mer lookups return instantly

## Limitations

- K-mers containing ambiguous nucleotides are ignored, so k-mers spanning 'N' characters do not affect classification
- GenBank draft microbial genomes may contain contaminating sequences from many different sources, requiring removal of adapter-derived k-mers and the first and last 20 k-mers of each draft contig
- The default database with k = 31 requires substantial memory; a MiniKraken 4-GB database can be made by removing the first 18 of every block of 19 records, at reduced sensitivity

## Evidence

- [discussion] the 4-byte spaces Jellyfish used to store the k-mer counts in the database file are instead used by Kraken to store the taxonomic ID numbers of the k-mers' LCA values: [evidence span withheld — anchor status: not_found]
- [discussion] if a k-mer from a sequence has had its LCA value previously set, then the LCA of the stored value and the current sequence's taxon is calculated and that LCA is stored for the k-mer: [evidence span withheld — anchor status: not_found]
- [discussion] based on completed microbial genomes in the National Center for Biotechnology Information's (NCBI) RefSeq database: [evidence span withheld — anchor status: not_found]
- [discussion] Taxon information is obtained from the NCBI taxonomy database: "Taxon information is obtained from the NCBI taxonomy database"
- [discussion] we use the Jellyfish multithreaded k-mer counter [19] to create a database: "we use the Jellyfish multithreaded k-mer counter [19] to create a database"
- [readme] Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken.: "Please see the [Kraken webpage] or the [Kraken manual]
for information on installing and operating Kraken."
