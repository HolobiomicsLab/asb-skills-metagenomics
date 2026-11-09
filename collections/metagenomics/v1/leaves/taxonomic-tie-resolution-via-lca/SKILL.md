---
name: taxonomic-tie-resolution-via-lca
description: Use when implementing or auditing k-mer-based taxonomic label assignment
  where root-to-leaf path scores can tie, and a deterministic, conservative label
  (the LCA of tied leaves) must be emitted.
when_to_use_negative:
- The query has no k-mers present in the database — Kraken leaves such sequences unclassified,
  and no tie resolution applies
- The classification method is alignment-based (e.g., BLAST/Megablast or MEGAN LCA
  over BLAST hits) rather than Kraken's RTL path scoring over a k-mer LCA database,
  since tie-breaking semantics differ
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: Kraken
  role: Implements the RTL path-scoring classification and the LCA-of-tied-leaves
    tie-breaking rule
  repo: https://github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter whose 4-byte count spaces are overwritten with
    k-mer LCA taxon IDs to form the Kraken database queried during scoring
- name: NCBI taxonomy database
  role: Provides the taxonomy tree in which the LCA of tied leaves is computed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:6042fb815e39915878c4e6827024127d9bc30817a5fa9cf6612801adf5ad4731
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DerrickWood/kraken
  tools:
  - Jellyfish
  - Kraken
  - NCBI taxonomy database
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Taxonomic tie resolution via LCA

## Summary

A decision rule from Kraken's classification algorithm: when multiple root-to-leaf (RTL) paths in the k-mer-weighted classification tree tie for the maximum score, the query sequence is labeled with the lowest common ancestor (LCA) of the tied leaves rather than an arbitrary winner.

## When to use

Use when implementing or auditing k-mer-based taxonomic label assignment where root-to-leaf path scores can tie, and a deterministic, conservative label (the LCA of tied leaves) must be emitted.

## When NOT to use

- The query has no k-mers present in the database — Kraken leaves such sequences unclassified, and no tie resolution applies
- The classification method is alignment-based (e.g., BLAST/Megablast or MEGAN LCA over BLAST hits) rather than Kraken's RTL path scoring over a k-mer LCA database, since tie-breaking semantics differ

## Inputs

- Query DNA sequence (e.g., 100 bp Illumina read or paired mates concatenated with 'NNNNN')
- Kraken k-mer database (Jellyfish count spaces storing k-mer LCA taxon IDs)
- NCBI taxonomy tree

## Outputs

- Sequence ID with an assigned taxon (LCA of tied maximum-scoring leaves) written to the classification output
- 'Unclassified' flag for sequences with no database k-mer hits

## How to apply

First build the weighted classification tree for a query: enumerate all k-mers (skipping k-mers with ambiguous nucleotides), look up each k-mer's LCA taxon via the minimizer index plus binary search, and add one unit of weight along the root-to-LCA path for each k-mer. Score every RTL path as the sum of all node weights along the path. If a unique path has the maximum score, assign its leaf's taxon; if two or more paths tie at the maximum, compute the LCA of the tied leaves in the NCBI taxonomy tree and assign that ancestor as the sequence label. This tie-breaking follows directly from Kraken's design, where each k-mer contributes equal unit weight, so ties signal genuinely ambiguous evidence that should be reported at a higher taxonomic rank. Sequences with no k-mer hits in the Jellyfish-derived database (4-byte count spaces overwritten with LCA taxon IDs) are left unclassified rather than tie-broken.

## Related tools

- **Kraken** (Implements the RTL path-scoring classification and the LCA-of-tied-leaves tie-breaking rule) — https://github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter whose 4-byte count spaces are overwritten with k-mer LCA taxon IDs to form the Kraken database queried during scoring)
- **NCBI taxonomy database** (Provides the taxonomy tree in which the LCA of tied leaves is computed)

## Evaluation signals

- Construct a synthetic query whose k-mer LCA hits produce exactly two equal-weight RTL paths; verify the emitted label equals the LCA of the two leaves, not either leaf alone
- Verify every k-mer contributes unit weight along its root-to-LCA path, so each path score equals the count of k-mers supporting that leaf's clade
- Check that queries containing ambiguous nucleotides ('N') have those spanning k-mers excluded before tree construction, matching Kraken's ambiguity handling
- Confirm sequences with zero database k-mer hits are written as unclassified and never receive a tie-resolved label

## Limitations

- Tie resolution can only move the label up the taxonomy tree — genuinely ambiguous reads will be labeled at a genus or higher rank, and Kraken may still misclassify at fine ranks (e.g., Pelosinus fermentans not identified at genus level in the HiSeq metagenome, Proteus vulgaris in the MiSeq metagenome)
- K-mers spanning ambiguous nucleotides are ignored and contribute no evidence, which can shift ties in reads containing 'N' characters (including the 'NNNNN' separator used when joining paired mates)

## Evidence

- [discussion] each root-to-leaf (RTL) path in the classification tree is scored by calculating the sum of all node weights along the path: "each root-to-leaf (RTL) path in the classification tree is scored by calculating the sum of all node weights along the path"
- [abstract] Kraken is an ultrafast and highly accurate program for assigning taxonomic labels to metagenomic DNA sequences.: "Kraken is an ultrafast and highly accurate program for assigning taxonomic labels to metagenomic DNA sequences."
- [discussion] Kraken ignores k-mers with ambiguous nucleotides, so the k-mers that span these 'N' characters do not affect classification: [evidence span withheld — anchor status: not_found]
- [discussion] the 4-byte spaces Jellyfish used to store the k-mer counts in the database file are instead used by Kraken to store the taxonomic ID numbers: "the 4-byte spaces Jellyfish used to store the k-mer counts in the database file are instead used by Kraken to store the taxonomic ID numbers"
