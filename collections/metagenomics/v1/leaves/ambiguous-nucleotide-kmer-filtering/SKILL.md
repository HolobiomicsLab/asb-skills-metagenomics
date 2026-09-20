---
name: ambiguous-nucleotide-kmer-filtering
description: Use when enumerating k-mers from metagenomic reads that contain ambiguous
  bases ('N'), adapter remnants, or concatenated paired mates, to decide which k-mers
  contribute LCA taxa to the classification tree.
when_to_use_negative:
- The input is a raw k-mer count table or feature matrix rather than nucleotide sequences
  — there are no per-base characters to screen for ambiguity
- A downstream alignment-based classifier (e.g. BLAST/Megablast) is being used, where
  ambiguous bases can be handled by alignment scoring rather than k-mer exclusion
- Abundance estimation workflows (e.g. MetaPhlAn) that only classify a small percentage
  of sequences and do not enumerate all query k-mers
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kraken
  role: Implements the k-mer enumeration with ambiguous-nucleotide filtering, minimizer-index
    lookup of LCA taxa, and root-to-leaf path scoring
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter used to build the Kraken database whose count
    spaces store k-mer LCA taxon IDs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:4978f5d2fbde5a8cf124f2b80efaf7ac6ac702f3f2c17e7f2fc00cda78efc129
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# ambiguous-nucleotide-kmer-filtering

## Summary

Kraken's rule of skipping any k-mer containing ambiguous nucleotides (e.g. 'N') during k-mer enumeration and database lookup, so that low-quality or masked positions do not contribute spurious evidence to the classification tree.

## When to use

Use when enumerating k-mers from metagenomic reads that contain ambiguous bases ('N'), adapter remnants, or concatenated paired mates, to decide which k-mers contribute LCA taxa to the classification tree.

## When NOT to use

- The input is a raw k-mer count table or feature matrix rather than nucleotide sequences — there are no per-base characters to screen for ambiguity
- A downstream alignment-based classifier (e.g. BLAST/Megablast) is being used, where ambiguous bases can be handled by alignment scoring rather than k-mer exclusion
- Abundance estimation workflows (e.g. MetaPhlAn) that only classify a small percentage of sequences and do not enumerate all query k-mers

## Inputs

- Query metagenomic DNA sequences (FASTA/FASTQ reads, optionally paired mates concatenated with 'NNNNN')
- Kraken k-mer database (Jellyfish count spaces storing k-mer LCA taxon IDs) with minimizer index and NCBI taxonomy tree

## Outputs

- Set of valid k-mer LCA taxa per query sequence (ambiguous-nucleotide k-mers excluded)
- Per-sequence taxonomic label in the classification output, or an unclassified marking when no valid k-mer hits the database

## How to apply

While enumerating all k-mers of a query sequence for Kraken classification, check each k-mer for ambiguous nucleotide characters and skip any k-mer containing them rather than attempting a minimizer-index lookup. This matters especially when paired mates are concatenated with 'NNNNN', since k-mers spanning the separator would otherwise pollute the weighted classification tree built from k-mer LCA values. Skipped k-mers simply contribute no unit of weight along the root-to-leaf path of their (nonexistent) LCA node; classification proceeds with the remaining valid k-mers, and a sequence whose k-mers are all skipped or absent from the database is left unclassified.

## Related tools

- **Kraken** (Implements the k-mer enumeration with ambiguous-nucleotide filtering, minimizer-index lookup of LCA taxa, and root-to-leaf path scoring) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter used to build the Kraken database whose count spaces store k-mer LCA taxon IDs)

## Examples

```
kraken --db minikraken_4GB --output classifications.txt reads.fa  # k-mers spanning 'N' (e.g. 'NNNNN'-joined pairs) are ignored automatically
```

## Evaluation signals

- No k-mer containing 'N' (including those spanning the 'NNNNN' paired-mate separator) appears in the set of looked-up LCA taxa for a query
- Sequences whose k-mers all contain ambiguous nucleotides are marked unclassified in the classification output
- Number of skipped k-mers equals the count of windows overlapping ambiguous characters (read length − k + 1 positions containing 'N', bounded)
- Classification of reads without ambiguous bases is unaffected by enabling this filter (invariant: identical taxon labels with and without 'N'-containing reads)

## Limitations

- K-mers spanning ambiguous bases are entirely discarded, so heavily masked or low-quality reads contribute less evidence and may end up unclassified
- Filtering is binary per k-mer — no quality-score weighting is applied; a single 'N' disqualifies the whole k-mer

## Evidence

- [discussion] Kraken ignores k-mers with ambiguous nucleotides: [evidence span withheld — anchor status: not_found]
- [discussion] Paired mates joined with NNNNN: [evidence span withheld — anchor status: not_found]
- [discussion] K-mer lookup and classification tree: "each root-to-leaf (RTL) path in the classification tree is scored by calculating the sum of all node weights along the path"
- [discussion] Unclassified sequences: [evidence span withheld — anchor status: not_found]
