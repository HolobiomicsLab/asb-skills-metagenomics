---
name: lowest-common-ancestor-assignment
description: Use when building or interpreting an exact k-mer-to-taxon database for
  metagenomic classification, e.g. deciding which taxonomic ID a shared 31-mer should
  carry when it appears in multiple genomes.
when_to_use_negative:
- When reads require inexact/alignment-based sensitivity because they have only weak
  similarity to known genomes — Kraken uses exact k-mer matches only
- 'When k-mers span ambiguous nucleotides (''N''): Kraken ignores these k-mers, so
  classification of such reads relies only on the remaining k-mers'
- When the target genome is absent from the reference library and its k-mers are shared
  only with distant relatives — labels will resolve only to a high-level ancestor
  (e.g. Pelosinus fermentans not correctly identified at genus level)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0632
tools:
- name: Kraken
  role: Stores k-mer-to-LCA records and classifies sequences by scoring root-to-leaf
    paths in a weighted classification tree of k-mer LCA assignments; default k =
    31
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter that builds the count database whose 4-byte count
    spaces Kraken overwrites with LCA taxonomic IDs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:15ce15b8de9127edfc370db7af74ac730e258a51ced487e3d6dba9d704b9d757
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lowest-common-ancestor-assignment

## Summary

Assigns each k-mer in a reference database the lowest common ancestor (LCA) taxon of all organisms whose genomes contain that k-mer, forming the core of Kraken's k-mer-to-taxonomy lookup used for metagenomic sequence classification.

## When to use

Use when building or interpreting an exact k-mer-to-taxon database for metagenomic classification, e.g. deciding which taxonomic ID a shared 31-mer should carry when it appears in multiple genomes.

## When NOT to use

- When reads require inexact/alignment-based sensitivity because they have only weak similarity to known genomes — Kraken uses exact k-mer matches only
- When k-mers span ambiguous nucleotides ('N'): Kraken ignores these k-mers, so classification of such reads relies only on the remaining k-mers
- When the target genome is absent from the reference library and its k-mers are shared only with distant relatives — labels will resolve only to a high-level ancestor (e.g. Pelosinus fermentans not correctly identified at genus level)

## Inputs

- Complete microbial genomes from NCBI RefSeq (FASTA reference library)
- NCBI taxonomy database files (nodes.dmp, names.dmp)
- Metagenomic DNA reads (e.g. FASTA/FASTQ, 100 bp Illumina reads) for classification
- Taxon ID per library sequence

## Outputs

- Kraken database mapping each 31-mer to the taxonomic ID of its LCA (Jellyfish file with count spaces overwritten by LCA taxon IDs)
- Taxonomic label per input sequence, derived from a weighted classification tree of k-mer LCA assignments

## How to apply

Build the database from complete microbial genomes in NCBI RefSeq plus NCBI taxonomy files (nodes.dmp/names.dmp). Run the Jellyfish multithreaded k-mer counter over the genomic library with the default k of 31, then overwrite the 4-byte count spaces Jellyfish used with the taxonomic ID of each k-mer's LCA. Process library sequences one at a time: store each sequence's taxon ID for its 31-mers, and on conflict compute the LCA of the stored value and the current sequence's taxon before storing. At classification time, collect a read's k-mers, map each to its LCA taxon, build a weighted classification tree, and score root-to-leaf (RTL) paths by summing node weights. The rationale is that exact-match k-mer database queries give the speed needed for metagenomic data, unlike inexact alignment. For MiniKraken, shrink the database by removing the first 18 of every block of 19 records.

## Related tools

- **Kraken** (Stores k-mer-to-LCA records and classifies sequences by scoring root-to-leaf paths in a weighted classification tree of k-mer LCA assignments; default k = 31) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter that builds the count database whose 4-byte count spaces Kraken overwrites with LCA taxonomic IDs)

## Evaluation signals

- Querying a finished database returns a valid NCBI taxon ID for spot-checked k-mers
- K-mers shared across divergent taxa resolve to their common ancestor's LCA ID, not a leaf taxon
- The database file's 4-byte fields contain taxonomic IDs where Jellyfish count values previously were, confirming the overwrite step succeeded
- Classification accuracy at genus level is comparable to Megablast (sensitivity ~similar), while classifying 100 bp reads at over 4.1 million reads per minute
- Ambiguous-nucleotide k-mers (spanning 'N') are excluded and do not affect the classification tree

## Limitations

- K-mers with ambiguous nucleotides are ignored, so 'N'-spanning k-mers provide no classification evidence
- Organisms absent or under-represented in the reference library can be misclassified at genus level (e.g. Pelosinus fermentans in the HiSeq metagenome; Proteus vulgaris in the MiSeq metagenome)
- Draft genomes in the Kraken-GB database can carry contamination; k-mers from known adapter sequences and the first/last 20 k-mers of each draft contig must be removed
- MiniKraken's 4-GB database subsamples records (removing the first 18 of every block of 19), trading sensitivity for memory

## Evidence

- [abstract] Kraken is an ultrafast and highly accurate program for assigning taxonomic labels to metagenomic DNA sequences.: "Kraken is an ultrafast and highly accurate program for assigning taxonomic labels to metagenomic DNA sequences."
- [intro] This speed advantage derives in large part from the use of exact-match database queries of k-mers, rather than inexact alignment of sequences: "This speed advantage derives in large part from the use of exact-match database queries of k-mers, rather than inexact alignment of sequences"
- [discussion] the 4-byte spaces Jellyfish used to store the k-mer counts in the database file are instead used by Kraken to store the taxonomic ID numbers of the k-mers' LCA values: [evidence span withheld — anchor status: not_found]
- [discussion] if a k-mer from a sequence has had its LCA value previously set, then the LCA of the stored value and the current sequence's taxon is calculated and that LCA is stored for the k-mer: [evidence span withheld — anchor status: not_found]
- [discussion] Kraken ignores k-mers with ambiguous nucleotides, so the k-mers that span these 'N' characters do not affect classification: [evidence span withheld — anchor status: not_found]
- [readme] for information on installing and operating Kraken.: "for information on installing and operating Kraken."
