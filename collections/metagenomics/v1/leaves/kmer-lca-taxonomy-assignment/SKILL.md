---
name: kmer-lca-taxonomy-assignment
description: Use when metagenomic reads (e.g. Illumina FASTQ) need fast taxonomic
  classification via exact k-mer matches and LCA of the taxa sharing those k-mers.
when_to_use_negative:
- Abundance estimation across only known marker genomes is the goal (Kraken classifies
  every read rather than estimating abundances like MetaPhlAn)
- Sequences share no exact k-mer similarity to database genomes — with no k-mer hits
  Kraken leaves them unclassified rather than finding inexact alignments
- A k-mer containing ambiguous nucleotides (e.g. spanning 'N' junction characters)
  — Kraken ignores such k-mers, so pairs joined with 'NNNNN' contribute no cross-junction
  evidence
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kraken
  role: 'Performs the classification: k-mer LCA lookups, weighted classification-tree
    construction, and RTL path scoring'
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Counts k-mers to build the database whose count spaces Kraken overwrites with
    LCA taxon IDs
- name: NCBI taxonomy database
  role: Provides the taxonomic tree used for LCA computation and root-to-leaf path
    scoring
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:7c2d5eefd85af18cc7ff984e084583fed1c956dfba1406cb9059e8122b294ca9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  - NCBI taxonomy database
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# kmer-lca-taxonomy-assignment

## Summary

Assigns taxonomic labels to metagenomic DNA sequences by exact k-mer matching against a Kraken/Jellyfish database of k-mer LCA taxa, then applying the root-to-leaf path-scoring classification algorithm. Provides ultrafast, alignment-free classification at accuracy comparable to Megablast.

## When to use

Use when metagenomic reads (e.g. Illumina FASTQ) need fast taxonomic classification via exact k-mer matches and LCA of the taxa sharing those k-mers.

## When NOT to use

- Abundance estimation across only known marker genomes is the goal (Kraken classifies every read rather than estimating abundances like MetaPhlAn)
- Sequences share no exact k-mer similarity to database genomes — with no k-mer hits Kraken leaves them unclassified rather than finding inexact alignments
- A k-mer containing ambiguous nucleotides (e.g. spanning 'N' junction characters) — Kraken ignores such k-mers, so pairs joined with 'NNNNN' contribute no cross-junction evidence

## Inputs

- FASTQ/FASTA metagenomic read files (e.g. trimmed Illumina reads)
- Kraken k-mer database (Jellyfish count spaces overwritten with k-mer LCA taxon IDs)
- NCBI taxonomy tree files

## Outputs

- Kraken classification output: sequence ID with assigned NCBI taxon per read (unclassified if no k-mer hits)
- Kraken report with per-taxon read counts and fractions (e.g. hmp_saliva_kraken_report.tsv)

## How to apply

Build a Kraken database by counting k-mers with Jellyfish, then overwrite the 4-byte count spaces with each k-mer's LCA taxonomic ID over complete bacterial, viral, and human genomes; on conflicts, store the LCA of the stored and new taxon. K-mers are grouped by minimizer (with XOR-toggling of half the bits of each M-mer's canonical representation to even the minimizer distribution) and stored sorted by canonical representation; queries use index lookup plus binary search, reusing the previous query range when the minimizer is unchanged. For each query sequence, enumerate k-mers (skipping those with ambiguous nucleotides), look up LCA taxa, build a weighted classification tree from the taxonomy root adding one unit of weight per k-mer along root-to-LCA paths, and score every root-to-leaf (RTL) path by summing node weights. Assign the maximum-scoring path's leaf taxon to the sequence, resolving ties by taking the LCA of tied leaves; sequences with no k-mer hits are unclassified. For paired reads, concatenate mates with 'NNNNN' so each pair is classified as a single unit.

## Related tools

- **Kraken** (Performs the classification: k-mer LCA lookups, weighted classification-tree construction, and RTL path scoring) — github.com/DerrickWood/kraken
- **Jellyfish** (Counts k-mers to build the database whose count spaces Kraken overwrites with LCA taxon IDs)
- **NCBI taxonomy database** (Provides the taxonomic tree used for LCA computation and root-to-leaf path scoring)

## Examples

```
kraken --db $DB --output reads.kraken trimmed_reads.fastq; kraken-report --db $DB reads.kraken > hmp_saliva_kraken_report.tsv
```

## Evaluation signals

- Unclassified fraction should be reported and fall in an expected range — for HMP saliva samples (SRS019120, SRS014468, SRS015055) roughly 68.2% unclassified; treat values within a few percentage points as consistent
- Genus-level distribution of classified reads should match known sample composition — Streptococcus ~30%, Haemophilus ~17%, Prevotella ~13% for the combined HMP saliva samples
- Reads whose origin genome is in the database should be labeled at or above that genome's taxon (no misclassification below the true clade for full-genome matches)
- K-mers with ambiguous nucleotides must be skipped and not contribute weight to the classification tree
- Ties among maximum-scoring RTL paths must resolve to the LCA of the tied leaves, not an arbitrary leaf

## Limitations

- K-mers with ambiguous nucleotides are ignored, so k-mers spanning 'N' characters do not affect classification
- Classification depends entirely on exact k-mer matches; genomes absent from the database yield unclassified or higher-rank labels
- Some genomes cannot be correctly identified at the genus level (e.g. Pelosinus fermentans in the HiSeq metagenome; Proteus vulgaris often misclassified at genus level in the MiSeq metagenome)
- Draft microbial genomes in GenBank can contain contaminating sequences; Kraken-GB removes k-mers from known adapters and the first/last 20 k-mers of each draft contig

## Evidence

- [discussion] Kraken classifies a sequence by querying the database for each k-mer in the sequence and using the resulting set of LCA taxa: [evidence span withheld — anchor status: not_found]
- [discussion] RTL path scoring: "each root-to-leaf (RTL) path in the classification tree is scored by calculating the sum of all node weights along the path"
- [discussion] Jellyfish count spaces reused for LCA IDs: [evidence span withheld — anchor status: not_found]
- [discussion] Ambiguous nucleotide handling: [evidence span withheld — anchor status: not_found]
- [discussion] Paired-read concatenation: [evidence span withheld — anchor status: not_found]
- [readme] Kraken availability: "Kraken taxonomic sequence classification system ... Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
