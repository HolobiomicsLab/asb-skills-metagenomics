---
name: horizontal-gene-transfer-evidence-analysis
description: Use when you have a catalogue of putative ISs (e.g. the 879-sequence
  ISC FASTA) and need to identify their isolate origins, taxonomic sharing breadth,
  and whether cross-genera or cross-class sharing implies horizontal transfer.
when_to_use_negative:
- Your IS candidates lack inverted terminal repeats (ITRs) — Palidis/ISC only discovers
  common DDE-type ISs with ITRs, so origin tracing via this catalogue will miss ITR-less
  elements
- You are annotating isolates from sources other than the healthy human oral cavity
  or stool — ISC v1.0.0 was built only from those metagenomes and is recommended as
  reference only for isolates sourced from them
- Your ISs fall outside the 500–3000 bp length range, e.g. elements lengthened by
  nested insertion events, which Palidis misses
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: COBS
  role: Compact bit-sliced signature index queried with the ISC FASTA to find IS occurrences
    across 661,405 microbial genomes at a 0.9 threshold
- name: Palidis
  role: Pipeline that generated the ISC catalogue of ISs (via ITR detection with maximal
    exact matching) that is subsequently traced to isolate origins
  repo: https://github.com/blue-moon22/palidis
- name: Insertion Sequence Catalogue (ISC)
  role: The query FASTA of 879 unique insertion sequences searched against the COBS
    index and ISfinder
  repo: https://github.com/blue-moon22/ISC
- name: ISfinder
  role: Reference database used to annotate known ISs and identify ISfinder coverage
    gaps among widely shared ISs
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:064a3c71fdbb8b3c2c0cb97f21ece09245bb24a23f3a649d4552e6cfa41e860f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis
  tools:
  - COBS
  - ISfinder
  - Insertion Sequence Catalogue (ISC)
  - Palidis
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# horizontal-gene-transfer-evidence-analysis

## Summary

Traces the origins of insertion sequences discovered from metagenomes by querying the Insertion Sequence Catalogue (ISC) against ISfinder and a COBS compact bit-sliced index of 661,405 ENA microbial genomes, using taxonomic breadth of hits as evidence for horizontal gene transfer across bacterial classes.

## When to use

Use when you have a catalogue of putative ISs (e.g. the 879-sequence ISC FASTA) and need to identify their isolate origins, taxonomic sharing breadth, and whether cross-genera or cross-class sharing implies horizontal transfer.

## When NOT to use

- Your IS candidates lack inverted terminal repeats (ITRs) — Palidis/ISC only discovers common DDE-type ISs with ITRs, so origin tracing via this catalogue will miss ITR-less elements
- You are annotating isolates from sources other than the healthy human oral cavity or stool — ISC v1.0.0 was built only from those metagenomes and is recommended as reference only for isolates sourced from them
- Your ISs fall outside the 500–3000 bp length range, e.g. elements lengthened by nested insertion events, which Palidis misses

## Inputs

- ISC FASTA of 879 unique insertion sequences (github.com/blue-moon22/ISC)
- COBS compact index 661k.cobs_compact of 661,405 ENA microbial genomes
- Study metadata table mapping genome/BioSample accessions to species and genus
- ISfinder reference annotations

## Outputs

- Per-IS isolate occurrence table (isolate_query_summary.tsv) with IS id, BioSample hits, and taxonomic identity
- Per-IS sharing table (is_sharing_summary.csv) with IS id, distinct genera count, distinct species count, and ISfinder annotation status
- Summary counts of ISs, BioSamples, taxonomically identified samples, and genera/species breadth

## How to apply

Download the ISC FASTA of 879 unique ISs from github.com/blue-moon22/ISC and the public COBS compact index 661k.cobs_compact covering 661,405 ENA bacterial genomes. Run cobs query (v0.1.2) with the ISC FASTA against 661k.cobs_compact using threshold -t 0.9, so only genomes containing at least 90% of an IS's k-mers are reported; expect roughly 155 ISs across 791 BioSample hits. Map positive genomes to ENA BioSample accessions and attach taxonomic metadata to tally distinct genera and species (expected: 684 taxonomically identified samples spanning 63 genera and 138 species). Aggregate distinct-species and distinct-genera counts per IS to measure sharing breadth — the most widely shared IS, IS_length_1391-IPR001207_495_804 (IS1249 in ISfinder), spans 21 genera and 46 species — and cross-reference per-genus IS sets against ISfinder annotation status to flag coverage gaps. Presence of the same IS across many genera or bacterial classes is interpreted as evidence of horizontal gene transfer events.

## Related tools

- **COBS** (Compact bit-sliced signature index queried with the ISC FASTA to find IS occurrences across 661,405 microbial genomes at a 0.9 threshold)
- **Palidis** (Pipeline that generated the ISC catalogue of ISs (via ITR detection with maximal exact matching) that is subsequently traced to isolate origins) — https://github.com/blue-moon22/palidis
- **Insertion Sequence Catalogue (ISC)** (The query FASTA of 879 unique insertion sequences searched against the COBS index and ISfinder) — https://github.com/blue-moon22/ISC
- **ISfinder** (Reference database used to annotate known ISs and identify ISfinder coverage gaps among widely shared ISs)

## Examples

```
cobs query -i 661k.cobs_compact -q ISC.fasta -t 0.9 > is_isolate_hits.tsv
```

## Evaluation signals

- COBS query with -t 0.9 on the 879-IS FASTA against 661k.cobs_compact reproduces 155 ISs hitting 791 BioSamples
- Taxonomic attachment yields 684 samples with known identity spanning 63 genera and 138 known species (excluding 'sp.' labels)
- IS_length_1391-IPR001207_495_804 (ISfinder name IS1249) ranks first in sharing breadth with 21 genera and 46 species, matching the paper's Figures 3-4
- ISs shared across multiple Bacteroides, Corynebacterium, Curtobacterium and Prevotella species are those absent from ISfinder, confirming coverage-gap consistency
- Output tables conform to the expected schemas of isolate_query_summary.tsv and is_sharing_summary.csv (IS id, genera count, species count, ISfinder status)

## Limitations

- Palidis/ISC only contains common DDE-type ISs with ITRs; it cannot discover ISs lacking ITRs
- The catalogue contains only ISs with ITRs of 25 or more nucleotides, although ITRs can be as short as ten nucleotides; lowering the minimum ITR length and k-mer size is more computationally intensive
- ISs flanked by ITRs beyond the 500–3000 bp proximity range (e.g. lengthened by nested insertion events) are missed
- ISC v1.0.0 was built only from healthy human oral cavity and stool metagenomes, limiting its use as an annotation reference to isolates from those sources
- No changelog is available for the resource

## Evidence

- [abstract] Querying this catalogue against a large database of isolate genomes reveals evidence of horizontal gene transfer events across bacterial classes.: "Querying this catalogue against a large database of isolate genomes reveals evidence of horizontal gene transfer events across bacterial classes."
- [intro] Palidis can also predict the origins of these ISs by querying them against ISfinder or a compact bit-sliced signature (COBS) index: "Palidis can also predict the origins of these ISs by querying them against ISfinder or a compact bit-sliced signature (COBS) index [11] of 661 405 microbial genomes"
- [discussion] Palidis is currently only equipped with discovering ISs with ITRs.: "Palidis is currently only equipped with discovering ISs with ITRs."
- [readme] pal-MEM finds inverted terminal repeats (ITRs) in large genomes, including metagenomes.: [evidence span withheld — anchor status: external_doc]
