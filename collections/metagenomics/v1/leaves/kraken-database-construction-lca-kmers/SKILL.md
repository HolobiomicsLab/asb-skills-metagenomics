---
name: kraken-database-construction-lca-kmers
description: Use when building a Kraken/MiniKraken k-mer database from reference genomes
  for taxonomic classification of metagenomic reads, or when comparing full versus
  shrunk databases in accuracy benchmarks.
when_to_use_negative:
- Input is raw sequencing reads needing classification — use the built database with
  the Kraken classifier, not this construction workflow
- Abundance estimation is the goal and only a small percentage of reads need labels
  — abundance programs like MetaPhlAn are faster and more appropriate
- Reference genomes are unavailable or the study targets taxa absent from RefSeq/GenBank,
  since accuracy depends on the sequenced genomes in the database
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Kraken
  role: Taxonomic sequence classification system whose database format (LCA-labeled
    k-mers sorted by minimizer) this skill constructs; also provides the database-building
    utilities
  repo: https://github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter used to produce the k-mer database file whose
    4-byte count spaces are overwritten with LCA taxonomic IDs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:61aeb7eb1af24675c5ef92539d8581e71870c0865b09a6b9a05e65770349a8ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DerrickWood/kraken
  tools:
  - Jellyfish
  - Kraken
  - Mason
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# kraken-database-construction-lca-kmers

## Summary

Constructs a Kraken taxonomic classification database by counting k-mers from complete RefSeq microbial genomes with Jellyfish and overwriting the 4-byte count spaces with LCA taxonomic IDs derived from the NCBI taxonomy. This LCA-labeled k-mer store is the foundation for Kraken's exact-match, minimizer-indexed classification and its reduced-memory MiniKraken variant.

## When to use

Use when building a Kraken/MiniKraken k-mer database from reference genomes for taxonomic classification of metagenomic reads, or when comparing full versus shrunk databases in accuracy benchmarks.

## When NOT to use

- Input is raw sequencing reads needing classification — use the built database with the Kraken classifier, not this construction workflow
- Abundance estimation is the goal and only a small percentage of reads need labels — abundance programs like MetaPhlAn are faster and more appropriate
- Reference genomes are unavailable or the study targets taxa absent from RefSeq/GenBank, since accuracy depends on the sequenced genomes in the database

## Inputs

- Complete NCBI RefSeq microbial genomes (bacterial and archaeal) in FASTA
- NCBI taxonomy database files (taxon IDs and hierarchy)
- GenBank draft microbial genomes (for Kraken-GB variant)
- Known adapter sequences (for Kraken-GB k-mer removal)

## Outputs

- Standard Kraken database (~70 GB) of k-mers labeled with LCA taxonomic IDs
- MiniKraken 4-GB database with shrinking factor 19 (first 18 of every 19 records removed)
- Kraken-GB database built from GenBank draft genomes with adapter and contig-end k-mers removed

## How to apply

Count k-mers from complete NCBI RefSeq microbial genomes using the Jellyfish multithreaded k-mer counter, then overwrite Jellyfish's 4-byte count spaces in the database file with the LCA taxonomic ID of each k-mer, obtained from the NCBI taxonomy database; when a k-mer was seen before, store the LCA of the stored taxon and the new sequence's taxon. Process library sequences one at a time so conflicting k-mers converge to their lowest common ancestor, making each k-mer a piece of taxonomic evidence. For the Kraken-GB variant, build from GenBank draft microbial genomes and remove k-mers from known adapter sequences as well as the first and last 20 k-mers of each draft contig to eliminate contaminating sequence bias. To create the 4-GB MiniKraken database, remove the first 18 of every block of 19 records from the standard database (shrinking factor 19), which trades sensitivity for memory while often improving precision. Group k-mers by minimizer, store them sorted by canonical representation, and toggle half the bits of each M-mer's canonical form with XOR to even out the minimizer distribution before indexing.

## Related tools

- **Kraken** (Taxonomic sequence classification system whose database format (LCA-labeled k-mers sorted by minimizer) this skill constructs; also provides the database-building utilities) — https://github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter used to produce the k-mer database file whose 4-byte count spaces are overwritten with LCA taxonomic IDs)

## Examples

```
kraken-build --db DB --library --jellyfish-hash-size 70G && kraken-build --db DB --taxids-for-kmers && kraken-build --db DB --shrink 19
```

## Evaluation signals

- Database file size matches the target: ~70 GB for the standard RefSeq build, 4 GB after applying the shrinking factor of 19
- Each k-mer record's taxon ID field holds a valid NCBI taxonomy LCA value (ambiguous-nucleotide k-mers absent, since Kraken ignores k-mers with ambiguous nucleotides)
- MiniKraken benchmark reproduces the paper's pattern: ~11% genus-level sensitivity drop on real-sequence metagenomes (SRS019120, SRS014468, SRS015055), >25-point drop on simBA-5, and precision at or above standard Kraken
- Kraken-GB build shows HiSeq/MiSeq sensitivity above the standard database, confirming contaminant and contig-end k-mer removal worked
- Random sampling of stored k-mers should resolve to LCA taxa consistent with the source genomes when re-derived from the RefSeq libraries

## Limitations

- Accuracy depends on database content: taxa absent from RefSeq (e.g., Pelosinus fermentans in the HiSeq metagenome, Proteus vulgaris in the MiSeq metagenome) cannot be correctly identified at genus level
- Memory footprint of the full database (~70 GB) may exceed available RAM, requiring the MiniKraken shrink, which sacrifices sensitivity (drops >25 percentage points on the high-error simBA-5 metagenome)
- Draft GenBank genomes can contain contaminating sequences from many different sources, motivating k-mer removal of adapters and the first/last 20 k-mers of each contig but leaving residual contamination risk
- Jellyfish version and Mason simulator version are not specified in the paper, complicating exact reproduction

## Evidence

- [discussion] the 4-byte spaces Jellyfish used to store the k-mer counts in the database file are instead used by Kraken to store the taxonomic ID numbers of the k-mers' LCA values: [evidence span withheld — anchor status: not_found]
- [discussion] if a k-mer from a sequence has had its LCA value previously set, then the LCA of the stored value and the current sequence's taxon is calculated and that LCA is stored for the k-mer: [evidence span withheld — anchor status: not_found]
- [discussion] we removed the first 18 of every block of 19 records in the standard Kraken database: "we removed the first 18 of every block of 19 records in the standard Kraken database"
- [discussion] we removed from the database those k-mers from known adapter sequences, as well as the first and last 20 k-mers from each of the draft contigs: "we removed from the database those k-mers from known adapter sequences, as well as the first and last 20 k-mers from each of the draft contigs"
- [discussion] based on completed microbial genomes in the National Center for Biotechnology Information's (NCBI) RefSeq database: [evidence span withheld — anchor status: not_found]
- [readme] Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken.: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
