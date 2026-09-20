---
name: taxonomic-identifier-mapping
description: Use when building a k-mer-to-LCA taxonomy database from NCBI RefSeq genomes
  so each k-mer in a metagenomic classifier resolves to a specific NCBI taxon ID.
when_to_use_negative:
- You need abundance estimation of a metagenome rather than per-sequence taxonomic
  labels — abundance programs classify only a small percentage of sequences and this
  skill builds a classification database instead
- Your k-mers contain ambiguous nucleotides — Kraken ignores k-mers with ambiguous
  nucleotides, so 'N'-spanning k-mers will not be mapped
- The genomes of interest are draft GenBank assemblies with contaminating sequences
  unless you filter out k-mers from known adapters and contig ends first
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kraken
  role: Builds and queries the k-mer-to-LCA database; overwrites Jellyfish count spaces
    with LCA taxonomic IDs and computes LCAs on conflicts
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter producing the count database whose 4-byte count
    fields are repurposed to hold LCA taxonomic IDs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:df34dff2ab00c0dbc436b1ada637a85953ebd747d7ba130bec58d19f5deef337
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
yes_rate: 0.9166666666666666
n_claims: 12
---

# taxonomic-identifier-mapping

## Summary

Construction of a Kraken database in which every 31-mer from a reference genomic library is stored together with the lowest common ancestor (LCA) taxon of all genomes containing that k-mer, enabling exact-match lookup of the most specific taxonomic node during classification.

## When to use

Use when building a k-mer-to-LCA taxonomy database from NCBI RefSeq genomes so each k-mer in a metagenomic classifier resolves to a specific NCBI taxon ID.

## When NOT to use

- You need abundance estimation of a metagenome rather than per-sequence taxonomic labels — abundance programs classify only a small percentage of sequences and this skill builds a classification database instead
- Your k-mers contain ambiguous nucleotides — Kraken ignores k-mers with ambiguous nucleotides, so 'N'-spanning k-mers will not be mapped
- The genomes of interest are draft GenBank assemblies with contaminating sequences unless you filter out k-mers from known adapters and contig ends first

## Inputs

- Complete microbial genomes from the NCBI RefSeq database (reference genomic library, FASTA)
- NCBI taxonomy database files (nodes.dmp, names.dmp)
- Taxon ID assignments for each library sequence

## Outputs

- Kraken database file storing each 31-mer with its LCA taxonomic ID in the Jellyfish count slot
- Indexed k-mer database grouped by minimizer for binary-search queries

## How to apply

Download complete microbial genomes from NCBI RefSeq and taxon information (nodes.dmp/names.dmp) from the NCBI taxonomy database. Run the Jellyfish multithreaded k-mer counter over the genomic library to count all distinct 31-mers (Kraken's default k is 31), producing a Jellyfish count database. Then overwrite the 4-byte count spaces Jellyfish used with the taxonomic ID numbers of each k-mer's LCA value. Process library sequences one at a time: for each of a sequence's 31-mers store the sequence's taxon ID, and if a k-mer's LCA value was previously set, compute the LCA of the stored value and the current sequence's taxon and store that LCA instead. Validate by querying k-mers and confirming each returns a valid NCBI taxon ID, and that k-mers shared across divergent taxa resolve to their common ancestor's LCA ID.

## Related tools

- **Kraken** (Builds and queries the k-mer-to-LCA database; overwrites Jellyfish count spaces with LCA taxonomic IDs and computes LCAs on conflicts) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter producing the count database whose 4-byte count fields are repurposed to hold LCA taxonomic IDs)

## Examples

```
kraken-build --db DB --library-library refseq.fna --taxonomy nodes.dmp --kmer-len 31 --jellyfish-hash-size 100M --build
```

## Evaluation signals

- Querying arbitrary k-mers from the finished database returns valid NCBI taxon IDs
- K-mers shared across divergent taxa resolve to their common ancestor's LCA ID rather than any single descendant taxon
- Conflicting k-mer assignments during build are resolved by storing the LCA of the stored value and the new sequence's taxon, not by overwriting
- The database file size matches Jellyfish's layout with every 4-byte count field replaced by a taxonomic ID (no residual counts)
- A spot-check k-mer from a genome present once in the library resolves to that genome's taxon ID

## Limitations

- K-mers containing ambiguous nucleotides ('N') are ignored by Kraken and thus never mapped to a taxon ID
- Draft GenBank genomes may contain contaminating sequences that corrupt LCA values unless k-mers from adapters and the first/last 20 k-mers of each contig are removed (as done for Kraken-GB)
- Memory constraints may require a MiniKraken database (e.g., 4 GB, built by removing the first 18 of every block of 19 records), sacrificing specificity
- Accuracy depends on the breadth of the reference library; genomes absent from RefSeq limit correct assignment (e.g., Pelosinus fermentans could not be identified at genus level in the HiSeq metagenome)

## Evidence

- [discussion] Kraken's database stores records of a k-mer together with the LCA of all organisms whose genomes contain that k-mer; it is built from a user-specified library of genomes: [evidence span withheld — anchor status: not_found]
- [discussion] Overwriting Jellyfish count fields with LCA IDs: "the 4-byte spaces Jellyfish used to store the k-mer counts in the database file are instead used by Kraken to store the taxonomic ID"
- [discussion] LCA conflict resolution rule: [evidence span withheld — anchor status: not_found]
- [discussion] Reference and taxonomy data sources: [evidence span withheld — anchor status: not_found]
- [readme] README pointing to documentation: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
