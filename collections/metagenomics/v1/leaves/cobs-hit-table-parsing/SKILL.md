---
name: cobs-hit-table-parsing
description: Use when you have COBS query results (per-IS genome hit lists) against
  the 661,405 microbial isolate genomes and need to quantify how widely each IS is
  shared across genera/species or identify ISfinder coverage gaps.
when_to_use_negative:
- Input is raw metagenomic reads or IS discovery output — use Palidis/palidis4 first
  to generate the ISC catalogue before querying COBS
- The ISs lack ITRs (Palidis/ISC only contains DDE-type ISs with ITRs of 25+ nucleotides),
  so COBS queries of such elements are out of scope
- You need de novo IS identification rather than prevalence/breadth statistics for
  already-identified ISs
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0798
tools:
- name: COBS
  role: Compact bit-sliced signature index of 661,405 microbial genomes queried with
    ISC ISs to produce the sample-hits table parsed here
- name: ISfinder
  role: Reference database of known transposable elements used to annotate ISs and
    identify coverage gaps (ISs shared across many species but absent from ISfinder)
- name: palidis4
  role: Upstream pipeline that generated the ISC catalogue of ISs whose COBS hits
    are parsed in this skill
  repo: https://github.com/blue-moon22/palidis4
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:af19376f1b793f0e1b2030f367d038aca633ba019c95bfa174eae74ef79a7464
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis4
  tools:
  - COBS
  - ISfinder
  - Palidis
  - palidis4
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0798
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# cobs-hit-table-parsing

## Summary

Parse COBS sample-hits tables from the 661,405-genome index to compute taxonomic sharing breadth (distinct genera and species) for each ISC insertion sequence, using study metadata to map genome hits to taxonomy.

## When to use

Use when you have COBS query results (per-IS genome hit lists) against the 661,405 microbial isolate genomes and need to quantify how widely each IS is shared across genera/species or identify ISfinder coverage gaps.

## When NOT to use

- Input is raw metagenomic reads or IS discovery output — use Palidis/palidis4 first to generate the ISC catalogue before querying COBS
- The ISs lack ITRs (Palidis/ISC only contains DDE-type ISs with ITRs of 25+ nucleotides), so COBS queries of such elements are out of scope
- You need de novo IS identification rather than prevalence/breadth statistics for already-identified ISs

## Inputs

- COBS sample-hits table (query IS vs. hit genomes) for the 661,405-genome COBS index
- Study metadata table mapping genome accessions to species and genus for the 661,405 indexed isolates
- ISC catalogue IS identifiers (e.g. IS_length_1391-IPR001207_495_804)
- ISfinder annotations for cross-referencing known vs. novel ISs

## Outputs

- Per-IS sharing table (is_sharing_summary.csv) with columns: IS id, distinct-genera count, distinct-species count, ISfinder annotation status
- Ranked list of ISs by taxonomic sharing breadth
- Set of widely shared ISs absent from ISfinder (coverage gaps)

## How to apply

Load the COBS sample-hits table for the 661,405 indexed microbial genomes together with the study metadata using pandas. Map each COBS hit genome to its species and genus via the metadata table, then aggregate distinct-species and distinct-genera counts per insertion sequence. Rank ISs by sharing breadth; in the source work IS_length_1391-IPR001207_495_804 (= IS1249 in ISfinder) ranked first, present in 21 genera and 46 species. Cross-reference each genus-level IS set against ISfinder annotation status to verify that ISs shared across multiple Bacteroides, Corynebacterium, Curtobacterium and Prevotella species are exclusively those absent from ISfinder, then write the per-IS table (IS id, genera count, species count, ISfinder status) to CSV.

## Related tools

- **COBS** (Compact bit-sliced signature index of 661,405 microbial genomes queried with ISC ISs to produce the sample-hits table parsed here)
- **ISfinder** (Reference database of known transposable elements used to annotate ISs and identify coverage gaps (ISs shared across many species but absent from ISfinder))
- **palidis4** (Upstream pipeline that generated the ISC catalogue of ISs whose COBS hits are parsed in this skill) — https://github.com/blue-moon22/palidis4

## Examples

```
import pandas as pd; hits = pd.read_csv('cobs_sample_hits.tsv', sep='\t'); meta = pd.read_csv('study_metadata.tsv', sep='\t'); m = hits.merge(meta, on='genome'); sharing = m.groupby('is_id')['species'].nunique().sort_values(ascending=False)
```

## Evaluation signals

- IS_length_1391-IPR001207_495_804 is reproduced as the most widely shared IS with presence in 21 genera and 46 species, matching Figures 3-4 of the paper
- Every hit genome in the sample-hits table maps to a genus/species in the metadata (no unmapped accessions silently dropped)
- Genera and species counts per IS are computed over DISTINCT taxa, not raw hit counts
- ISs found in multiple Bacteroides, Corynebacterium, Curtobacterium and Prevotella species are exclusively those absent from ISfinder
- Output is_sharing_summary.csv contains one row per queried IS with non-negative integer genera/species counts and an ISfinder status value

## Limitations

- ISC v1.0.0 was built only from healthy human oral cavity and stool metagenomes, limiting use as an annotation reference for isolates from other sources
- ISC contains only ISs with ITRs of 25 or more nucleotides; ITRs as short as ten nucleotides are missed at default Palidis settings
- The catalogue only contains common DDE-type ISs with ITRs; ISs lacking ITRs cannot be discovered or queried

## Evidence

- [intro] querying them against ISfinder or a compact bit-sliced signature (COBS) index [11] of 661 405 microbial genomes: "querying them against ISfinder or a compact bit-sliced signature (COBS) index [11] of 661 405 microbial genomes"
- [intro] Microbial genomes can be annotated for ISs by querying reference databases of known transposable elements, such as ISfinder: "Microbial genomes can be annotated for ISs by querying reference databases of known transposable elements, such as ISfinder [7]"
- [abstract] Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences: "Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel and not previously characterized."
- [abstract] The Insertion Sequence Catalogue is available to download here: "The Insertion Sequence Catalogue is available to download here: https://github.com/blue-moon22/ISC."
- [discussion] since Palidis was only applied to metagenomes sequenced from the healthy human oral cavity and stool samples: "since Palidis was only applied to metagenomes sequenced from the healthy human oral cavity and stool samples, it is recommended ISC v1.0.0 is used as a reference for annotating isolates sourced from"
