---
name: cobs-compact-index-querying
description: Use when you need to predict the isolate origins of novel insertion sequences
  (or other query sequences) by screening them against a large set of microbial genomes,
  and when you must attach ENA BioSample metadata and taxonomy to the hits.
when_to_use_negative:
- The goal is to discover novel ISs from metagenomic reads rather than trace known
  query sequences against genomes (use Palidis/pal-MEM discovery instead)
- The query sequences should be annotated against a curated reference IS database
  (ISfinder) rather than screened across raw isolate genomes
- Genomes of interest are not represented in the 661,405-genome ENA index, so absence
  of a hit would not be informative
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: COBS
  role: Compact bit-sliced signature index queried with the ISC FASTA (threshold -t
    0.9) to identify genomes containing each insertion sequence; output hits are mapped
    to ENA BioSample accessions
  repo: https://github.com/blue-moon22/palidis_paper_analysis
- name: Palidis
  role: Upstream pipeline that generated the 879-unique-IS ISC query set from 264
    human metagenomes before COBS origin tracing
  repo: https://github.com/blue-moon22/palidis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:f0594198d1b0b3467e35c6997843cf6de7cacad9f893ba3739e8b7b5b520549d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis_paper_analysis
  tools:
  - COBS
  - Palidis
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cobs-compact-index-querying

## Summary

Query a FASTA catalogue of insertion sequences against a compact bit-sliced signature (COBS) index of 661,405 ENA microbial genomes to trace candidate origins of each IS, mapping positive genome hits back to ENA BioSample accessions and their taxonomic identities.

## When to use

Use when you need to predict the isolate origins of novel insertion sequences (or other query sequences) by screening them against a large set of microbial genomes, and when you must attach ENA BioSample metadata and taxonomy to the hits.

## When NOT to use

- The goal is to discover novel ISs from metagenomic reads rather than trace known query sequences against genomes (use Palidis/pal-MEM discovery instead)
- The query sequences should be annotated against a curated reference IS database (ISfinder) rather than screened across raw isolate genomes
- Genomes of interest are not represented in the 661,405-genome ENA index, so absence of a hit would not be informative

## Inputs

- FASTA file of query insertion sequences (ISC, 879 unique ISs)
- COBS compact index file 661k.cobs_compact covering 661,405 ENA bacterial genomes
- Taxonomic identity metadata for ENA BioSample accessions

## Outputs

- Per-IS isolate occurrence table mapping ISs to BioSample accessions (isolate_query_summary.tsv)
- Summary counts: number of positive BioSamples, taxonomically identified samples, and distinct genera/species
- List of genomes passing the 0.9 k-mer threshold per query IS

## How to apply

Download the query FASTA (e.g. the ISC of 879 unique insertion sequences from github.com/blue-moon22/ISC) and the public COBS compact index 661k.cobs_compact covering 661,405 ENA bacterial genomes. Run cobs query (version 0.1.2) with the FASTA as query file against the compact index, setting the threshold parameter to -t 0.9 so only genomes containing at least 90% of an IS's k-mers are reported. Parse the COBS output to map each positive genome hit to its ENA BioSample accession and count distinct ISs and distinct BioSamples (expected in the reference run: 155 ISs across 791 BioSamples). Attach taxonomic identity metadata for the positive BioSamples and tally samples with known taxonomy plus distinct genera and species (expected: 684 samples spanning 63 genera and 138 species, excluding species labelled 'sp.'). Write the per-IS isolate occurrence table and summary counts to isolate_query_summary.tsv and verify the reproduced counts match the reported values.

## Related tools

- **COBS** (Compact bit-sliced signature index queried with the ISC FASTA (threshold -t 0.9) to identify genomes containing each insertion sequence; output hits are mapped to ENA BioSample accessions) — https://github.com/blue-moon22/palidis_paper_analysis
- **Palidis** (Upstream pipeline that generated the 879-unique-IS ISC query set from 264 human metagenomes before COBS origin tracing) — https://github.com/blue-moon22/palidis

## Examples

```
cobs query -t 0.9 --query ISC.fasta 661k.cobs_compact > cobs_is_hits.tsv
```

## Evaluation signals

- Reference reproduction yields 155 distinct ISs across 791 distinct BioSamples after parsing COBS output
- Of the 791 positive BioSamples, 684 carry an associated taxonomic identity spanning 63 known genera and 138 known species (not labelled sp.)
- Every positive genome hit resolves to a valid ENA BioSample accession before taxonomy attachment
- Threshold of -t 0.9 is applied such that reported genomes contain at least 90% of each query IS's k-mers
- isolate_query_summary.tsv reproduces the reported per-IS occurrence table and summary counts (155 ISs, 791 BioSamples, 684 taxonomically identified, 63 genera, 138 species)

## Limitations

- COBS reporting depends on k-mer presence; the -t 0.9 threshold means genomes with less than 90% of an IS's k-mers are not reported, so divergent or partial copies may be missed
- ISC v1.0.0 was built only from healthy human oral cavity and stool metagenomes, so origin tracing is biased toward annotating isolates sourced from comparable environments
- Samples lacking taxonomic identity metadata cannot contribute to genus/species tallies (791 BioSamples but only 684 taxonomically identified)

## Evidence

- [intro] COBS index intro quote: "Palidis can also predict the origins of these ISs by querying them against ISfinder or a compact bit-sliced signature (COBS) index [11] of 661 405 microbial genomes"
- [other] task workflow COBS invocation: [evidence span withheld — anchor status: not_found]
- [other] task workflow expected counts: [evidence span withheld — anchor status: not_found]
- [abstract] ISC download location: "The Insertion Sequence Catalogue is available to download here: https://github.com/blue-moon22/ISC."
