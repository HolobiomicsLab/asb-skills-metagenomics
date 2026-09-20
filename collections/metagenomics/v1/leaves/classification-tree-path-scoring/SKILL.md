---
name: classification-tree-path-scoring
description: Use when reconstructing or re-implementing Kraken's taxonomic label assignment
  for metagenomic DNA reads, or when scoring query sequences against a k-mer LCA database
  built with Jellyfish and the NCBI taxonomy tree.
when_to_use_negative:
- The task requires inexact alignment or quantifying abundance rather than per-read
  taxonomic classification — Kraken relies on exact-match k-mer database queries rather
  than inexact alignment of sequences.
- The query sequences are short k-mers containing ambiguous nucleotides spanning 'N'
  characters, which are ignored and cannot contribute classification evidence.
- The k-mer database does not contain the organism's k-mers — sequences with no k-mers
  in the database must be left unclassified, not force-labeled.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: Kraken
  role: 'Implements the classification algorithm: k-mer LCA lookups, weighted classification-tree
    construction, and RTL path scoring of query sequences'
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter whose 4-byte count spaces are repurposed to store
    k-mer LCA taxonomic IDs in the database
- name: NCBI taxonomy database
  role: Provides the taxonomy tree structure and taxon IDs used to build the weighted
    classification tree and resolve LCA values
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:f483d38ac446e55fa4a336b58b059c897fbe80b68a15580750cb4da6f8d1a821
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  - NCBI taxonomy database
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# classification-tree-path-scoring

## Summary

Reimplements Kraken's RTL path-scoring classification algorithm: each k-mer of a query sequence votes for its LCA taxon in a weighted classification tree, and the taxon of the maximum-scoring root-to-leaf path is assigned to the sequence. Ties are resolved by taking the LCA of tied leaves; sequences with no database k-mer hits are left unclassified.

## When to use

Use when reconstructing or re-implementing Kraken's taxonomic label assignment for metagenomic DNA reads, or when scoring query sequences against a k-mer LCA database built with Jellyfish and the NCBI taxonomy tree.

## When NOT to use

- The task requires inexact alignment or quantifying abundance rather than per-read taxonomic classification — Kraken relies on exact-match k-mer database queries rather than inexact alignment of sequences.
- The query sequences are short k-mers containing ambiguous nucleotides spanning 'N' characters, which are ignored and cannot contribute classification evidence.
- The k-mer database does not contain the organism's k-mers — sequences with no k-mers in the database must be left unclassified, not force-labeled.

## Inputs

- FASTA/FASTQ query sequences (metagenomic DNA reads)
- Kraken k-mer database: Jellyfish count spaces storing LCA taxon IDs for each k-mer
- NCBI taxonomy tree (nodes.dmp / names.dmp)
- minimizer index over the k-mer database

## Outputs

- Per-sequence taxonomic assignments (sequence ID with assigned NCBI taxon)
- Sequences with no k-mer hits labeled unclassified

## How to apply

Load the Kraken k-mer database (Jellyfish count spaces storing k-mer LCA taxon IDs) together with the NCBI taxonomy tree. For each query sequence, enumerate all k-mers and skip any k-mer containing ambiguous nucleotides, since Kraken ignores k-mers with ambiguous nucleotides. Look up each k-mer's LCA taxon via minimizer index lookup followed by binary search within the stored range, first searching the previous query range when the minimizer is unchanged. Build a weighted classification tree from the taxonomy root, adding one unit of weight along the root-to-LCA path for each k-mer hit, then score every root-to-leaf (RTL) path as the sum of all node weights along the path. Assign the leaf of the maximum-scoring path to the sequence, resolving ties by taking the LCA of the tied leaves, and mark sequences with no k-mer hits as unclassified.

## Related tools

- **Kraken** (Implements the classification algorithm: k-mer LCA lookups, weighted classification-tree construction, and RTL path scoring of query sequences) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter whose 4-byte count spaces are repurposed to store k-mer LCA taxonomic IDs in the database)
- **NCBI taxonomy database** (Provides the taxonomy tree structure and taxon IDs used to build the weighted classification tree and resolve LCA values)

## Evaluation signals

- Sequences whose k-mers are absent from the database must appear in the output labeled as unclassified.
- The assigned taxon must be the leaf of the maximum root-to-leaf path in the weighted tree, where each node weight equals the number of query k-mers whose LCA maps to that node.
- Tied maximum-scoring paths must be resolved by outputting the LCA of the tied leaves, not an arbitrary leaf.
- K-mers spanning ambiguous 'N' characters must not contribute weight to the classification tree.
- On simulated reads (e.g., Mason with SNP rate 0.1% and indel rate 0.1%), reads from genomes present in the database should classify at the correct taxon at rates consistent with the paper's reported accuracy.

## Limitations

- K-mers with ambiguous nucleotides are ignored, so k-mers spanning 'N' characters (including the 'NNNNN' joiner between paired mates) do not affect classification.
- Reads from organisms absent or underrepresented in the database may be misclassified at higher ranks (e.g., Pelosinus fermentans in the HiSeq metagenome could not be correctly identified at the genus level).
- K-mers from draft GenBank contigs can be contaminating; the Kraken-GB database removed k-mers from known adapter sequences and the first and last 20 k-mers of each draft contig to mitigate this.
- Classification quality depends on the large number of sequenced microbial genomes (over 8,500 at publication time); sparse databases yield shallow LCA assignments.

## Evidence

- [discussion] RTL path scoring sums node weights: "each root-to-leaf (RTL) path in the classification tree is scored by calculating the sum of all node weights along the path"
- [discussion] Jellyfish count spaces store LCA taxon IDs: [evidence span withheld — anchor status: not_found]
- [discussion] Ambiguous nucleotides are ignored: [evidence span withheld — anchor status: not_found]
- [discussion] Each k-mer is separate evidence: "This algorithm, illustrated in Figure 1, allows Kraken to consider each k-mer within a sequence as a separate piece of evidence"
- [intro] Exact k-mer queries enable speed: "This speed advantage derives in large part from the use of exact-match database queries of k-mers, rather than inexact alignment of sequences"
- [readme] README points to Kraken manual: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
