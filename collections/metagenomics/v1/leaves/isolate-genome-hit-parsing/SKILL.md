---
name: isolate-genome-hit-parsing
description: Use when you have COBS query results against the 661k ENA compact index
  and need to resolve genome hits to BioSamples and taxonomic identities for IS origin
  tracing.
when_to_use_negative:
- The ISs under study lack inverted terminal repeats — the ISC only contains DDE-type
  ISs with ITRs of 25+ nucleotides, so hits cannot be expected
- You need IS origin prediction against curated reference IS databases rather than
  isolate genomes — query ISfinder instead of the COBS 661k index
- The genomes are metagenomic assemblies rather than isolates with ENA BioSample accessions
  and taxonomy metadata
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0798
tools:
- name: COBS
  role: Queries the ISC FASTA against a compact bit-sliced signature index of 661,405
    microbial genomes with a 0.9 threshold to find isolate genome hits
- name: Palidis
  role: Generated the 879-IS Insertion Sequence Catalogue that is queried against
    isolate genomes
  repo: https://github.com/blue-moon22/palidis
- name: ISC (Insertion Sequence Catalogue)
  role: The FASTA catalogue of 879 unique insertion sequences used as the query against
    the COBS index
  repo: https://github.com/blue-moon22/ISC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:1c3950ad1d85e1fa0be0cdcbb672a640e7c91483161f6fdee423d6c1c57ceda8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis
  tools:
  - COBS
  - ISC (Insertion Sequence Catalogue)
  - Palidis
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0798
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# isolate-genome-hit-parsing

## Summary

Parses COBS query output to map positive isolate-genome hits to ENA BioSample accessions, attaches taxonomic metadata, and tallies distinct ISs, BioSamples, genera, and species — the step that traces ISC insertion sequences back to their isolate origins.

## When to use

Use when you have COBS query results against the 661k ENA compact index and need to resolve genome hits to BioSamples and taxonomic identities for IS origin tracing.

## When NOT to use

- The ISs under study lack inverted terminal repeats — the ISC only contains DDE-type ISs with ITRs of 25+ nucleotides, so hits cannot be expected
- You need IS origin prediction against curated reference IS databases rather than isolate genomes — query ISfinder instead of the COBS 661k index
- The genomes are metagenomic assemblies rather than isolates with ENA BioSample accessions and taxonomy metadata

## Inputs

- ISC FASTA file of 879 unique insertion sequences (github.com/blue-moon22/ISC)
- Public COBS compact index 661k.cobs_compact covering 661,405 ENA bacterial genomes
- Taxonomic identity metadata for ENA BioSample accessions

## Outputs

- isolate_query_summary.tsv with per-IS isolate occurrence table and summary counts (155 ISs, 791 BioSamples, 684 taxonomically identified, 63 genera, 138 species)

## How to apply

Run cobs query (version 0.1.2) with the ISC FASTA of 879 unique insertion sequences as the query file against the 661k.cobs_compact index covering 661,405 ENA bacterial genomes, setting the threshold parameter to -t 0.9 so only genomes with at least 90% of the IS's k-mers present are reported. Parse the COBS output to map each positive genome hit to its ENA BioSample accession and count distinct ISs and distinct BioSamples (expected: 155 ISs across 791 BioSamples). Attach taxonomic identity metadata for the positive BioSamples and tally samples with known taxonomy plus distinct genera and species, excluding species labelled 'sp.' (expected: 684 samples from 63 genera and 138 species). Write the per-IS isolate occurrence table and summary counts to isolate_query_summary.tsv and verify reproduced counts match the reported values.

## Related tools

- **COBS** (Queries the ISC FASTA against a compact bit-sliced signature index of 661,405 microbial genomes with a 0.9 threshold to find isolate genome hits)
- **Palidis** (Generated the 879-IS Insertion Sequence Catalogue that is queried against isolate genomes) — https://github.com/blue-moon22/palidis
- **ISC (Insertion Sequence Catalogue)** (The FASTA catalogue of 879 unique insertion sequences used as the query against the COBS index) — https://github.com/blue-moon22/ISC

## Examples

```
cobs query -t 0.9 -i 661k.cobs_compact --query ISC.fasta > cobs_isolate_hits.txt
```

## Evaluation signals

- COBS query with -t 0.9 returns 155 distinct ISs hitting 791 distinct BioSamples
- Of the 791 BioSamples, 684 have an associated taxonomic identity
- The 684 taxonomically identified samples span exactly 63 known genera and 138 known species (not labelled sp.)
- isolate_query_summary.tsv row counts and per-IS occurrence columns reconcile with the summary totals

## Limitations

- The 90% k-mer presence threshold (-t 0.9) excludes genomes with partial IS presence, so more divergent or truncated copies may be missed
- Only samples with an associated taxonomic identity contribute to the genera/species tallies; 107 of the 791 BioSamples lacked taxonomy and are excluded
- ISC v1.0.0 was built only from healthy human oral cavity and stool metagenomes, limiting interpretation of isolate-origin inferences for other sources

## Evidence

- [intro] COBS query against 661k index with threshold: "querying them against ISfinder or a compact bit-sliced signature (COBS) index [11] of 661 405 microbial genomes"
- [discussion] ISC catalogue of 879 ISs: "we have released the first version of ISC, a catalogue containing 879 ISs"
- [abstract] Horizontal transfer signal from isolate queries: "Querying this catalogue against a large database of isolate genomes reveals evidence of horizontal gene transfer events across bacterial classes."
- [readme] ISC download location: [evidence span withheld — anchor status: external_doc]
- [discussion] ISC restricted to ITR-containing ISs: [evidence span withheld — anchor status: not_found]
