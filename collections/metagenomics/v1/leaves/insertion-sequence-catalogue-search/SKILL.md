---
name: insertion-sequence-catalogue-search
description: Use when you have FASTA sequences of insertion sequences (e.g. from ISC)
  and need to find which isolate genomes contain them and infer their taxonomic origins
  via compact index search.
when_to_use_negative:
- Your query sequences are raw reads or assemblies rather than IS catalogue entries
  — Palidis itself (not COBS querying) is the discovery tool for novel ISs from metagenomic
  reads
- The ISs of interest lack inverted terminal repeats, since ISC v1.0.0 only contains
  common DDE-type ISs with ITRs
- You need to annotate ISs in human metagenomes from sources outside the healthy oral
  cavity and stool, where ISC v1.0.0 is not recommended as a reference
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0798
tools:
- name: COBS
  role: Compact bit-sliced signature index used to query the 879 ISC insertion sequences
    against 661,405 microbial genomes (cobs query, version 0.1.2, threshold -t 0.9)
- name: Palidis
  role: Upstream pipeline that generated the ISC catalogue of insertion sequences
    being searched
  repo: https://github.com/blue-moon22/palidis
- name: Insertion Sequence Catalogue (ISC)
  role: FASTA reference of 879 unique ISs queried against the COBS index
  repo: https://github.com/blue-moon22/ISC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:a80ca4b7d3581d57290dba0eb99af8433e57b10b7cb5eb114e1270dce7bf0f48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis
  tools:
  - COBS
  - Insertion Sequence Catalogue (ISC)
  - Palidis
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0798
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# insertion-sequence-catalogue-search

## Summary

Traces the origins of insertion sequences (ISs) from the Insertion Sequence Catalogue (ISC) by querying the 879-IS FASTA against a COBS compact bit-sliced index of 661,405 ENA microbial isolate genomes, then attaching BioSample taxonomy to attribute ISs to known genera and species.

## When to use

Use when you have FASTA sequences of insertion sequences (e.g. from ISC) and need to find which isolate genomes contain them and infer their taxonomic origins via compact index search.

## When NOT to use

- Your query sequences are raw reads or assemblies rather than IS catalogue entries — Palidis itself (not COBS querying) is the discovery tool for novel ISs from metagenomic reads
- The ISs of interest lack inverted terminal repeats, since ISC v1.0.0 only contains common DDE-type ISs with ITRs
- You need to annotate ISs in human metagenomes from sources outside the healthy oral cavity and stool, where ISC v1.0.0 is not recommended as a reference

## Inputs

- ISC FASTA file of 879 unique insertion sequences (github.com/blue-moon22/ISC)
- COBS compact index 661k.cobs_compact covering 661,405 ENA bacterial isolate genomes
- Taxonomic identity metadata for ENA BioSample accessions

## Outputs

- List of genome/BioSample hits per insertion sequence at the 0.9 k-mer threshold
- Per-IS isolate occurrence table (isolate_query_summary.tsv)
- Summary counts: ISs found, BioSamples hit, taxonomically identified samples, distinct genera and species

## How to apply

Download the ISC FASTA of 879 unique insertion sequences from github.com/blue-moon22/ISC together with the pre-built public COBS index 661k.cobs_compact covering 661,405 ENA bacterial genomes. Run cobs query (version 0.1.2) with the ISC FASTA as the query file and set the threshold parameter to -t 0.9 so that only genomes containing at least 90% of an IS's k-mers are reported as hits — this stringent threshold reduces spurious partial matches. Parse the COBS output to map each positive genome hit to its ENA BioSample accession, then count distinct ISs and distinct BioSamples (the paper expects 155 ISs across 791 BioSamples). Attach taxonomic identity metadata for the positive BioSamples and tally samples with known taxonomy plus distinct genera and species (excluding species labelled 'sp.'); the reported reproduction gives 684 identified samples spanning 63 genera and 138 species. Write the per-IS isolate occurrence table and summary counts to isolate_query_summary.tsv and check the reproduced numbers against the published values.

## Related tools

- **COBS** (Compact bit-sliced signature index used to query the 879 ISC insertion sequences against 661,405 microbial genomes (cobs query, version 0.1.2, threshold -t 0.9))
- **Palidis** (Upstream pipeline that generated the ISC catalogue of insertion sequences being searched) — https://github.com/blue-moon22/palidis
- **Insertion Sequence Catalogue (ISC)** (FASTA reference of 879 unique ISs queried against the COBS index) — https://github.com/blue-moon22/ISC

## Examples

```
cobs query -t 0.9 --index 661k.cobs_compact --query ISC.fasta > cobs_hits.txt
```

## Evaluation signals

- COBS query at -t 0.9 reproduces 155 distinct ISC ISs hitting 791 distinct ENA BioSamples
- Taxonomy attachment yields 684 BioSamples with an associated taxonomic identity out of 791 hits
- The 684 identified samples span exactly 63 known genera and 138 known species (excluding species labelled 'sp.')
- Output file isolate_query_summary.tsv contains the per-IS isolate occurrence table plus the five summary counts matching published values
- COBS version 0.1.2 is used and the threshold parameter -t 0.9 is confirmed in the command line

## Limitations

- ISC v1.0.0 only contains ISs with ITRs of 25 or more nucleotides; ITRs as short as ten nucleotides are missed at default settings, so catalogue queries miss ISs with short ITRs
- ISC v1.0.0 was built only from healthy human oral cavity and stool metagenomes, limiting its use as an annotation reference outside those sources
- Querying at a 0.9 k-mer threshold reports only near-complete IS presence; divergent or partially deleted IS copies in isolates may be missed

## Evidence

- [intro] Palidis can also predict the origins of these ISs by querying them against ISfinder or a compact bit-sliced signature (COBS) index [11] of 661 405 microbial genomes: "Palidis can also predict the origins of these ISs by querying them against ISfinder or a compact bit-sliced signature (COBS) index [11] of 661 405 microbial genomes"
- [readme] COBS index covering 661k genomes and ISC download: [evidence span withheld — anchor status: external_doc]
- [abstract] Querying catalogue against isolates reveals HGT: "Querying this catalogue against a large database of isolate genomes reveals evidence of horizontal gene transfer events across bacterial classes."
- [discussion] ISC limitation: only ITR-containing DDE ISs: [evidence span withheld — anchor status: not_found]
