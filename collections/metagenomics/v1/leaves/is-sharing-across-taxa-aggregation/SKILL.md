---
name: is-sharing-across-taxa-aggregation
description: Use when a COBS sample-hits table of ISC insertion sequences queried
  against 661,405 isolate genomes must be joined to genus/species metadata to identify
  the most widely shared IS and ISfinder coverage gaps.
when_to_use_negative:
- When the ISs themselves still need to be discovered from reads — run Palidis (pal-MEM
  ITR detection, CD-HIT-EST clustering) first; this skill only aggregates existing
  COBS hit tables
- When genomes lack genus/species assignments in the metadata table, since the aggregation
  depends on mapping every COBS hit genome to species and genus
- When the query index is not the 661,405-genome COBS isolate index, since validation
  values (21 genera / 46 species for IS1249) are specific to that dataset
tools:
- name: COBS
  role: Provides the compact bit-sliced signature index of 661,405 microbial genomes
    used to obtain sample-hits for each ISC insertion sequence
- name: Palidis
  role: Pipeline that generated the ISC catalogue of insertion sequences whose cross-taxa
    sharing is aggregated here
  repo: https://github.com/blue-moon22/palidis
- name: ISfinder
  role: Reference database used to assign known-IS names and determine annotation
    status (e.g. IS1249 for IS_length_1391-IPR001207_495_804)
- name: ISC
  role: Source catalogue (879 insertion sequences) whose members are queried against
    the COBS index and whose FASTA entries supply the IS ids
  repo: https://github.com/blue-moon22/ISC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:c1637bbc70deb83d76b1e023e84ba50f404375cbd89bfde531defe90c063c649
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis
  tools:
  - COBS
  - ISC
  - ISfinder
  - Palidis
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# is-sharing-across-taxa-aggregation

## Summary

Aggregates COBS sample-hits from the Insertion Sequence Catalogue (ISC) against 661,405 indexed microbial genomes to count, per IS, the number of distinct bacterial genera and species it occurs in, ranking insertion sequences by taxonomic breadth and flagging ISfinder annotation gaps.

## When to use

Use when a COBS sample-hits table of ISC insertion sequences queried against 661,405 isolate genomes must be joined to genus/species metadata to identify the most widely shared IS and ISfinder coverage gaps.

## When NOT to use

- When the ISs themselves still need to be discovered from reads — run Palidis (pal-MEM ITR detection, CD-HIT-EST clustering) first; this skill only aggregates existing COBS hit tables
- When genomes lack genus/species assignments in the metadata table, since the aggregation depends on mapping every COBS hit genome to species and genus
- When the query index is not the 661,405-genome COBS isolate index, since validation values (21 genera / 46 species for IS1249) are specific to that dataset

## Inputs

- COBS sample-hits table of ISC insertion sequences queried against the 661,405-genome isolate index
- Study metadata table mapping the 661,405 indexed genomes to species and genus labels
- ISfinder annotation status (present/absent) for each ISC insertion sequence

## Outputs

- Per-IS sharing table is_sharing_summary.csv with columns: IS id, genera count, species count, ISfinder status
- Ranked list of insertion sequences by taxonomic breadth (top hit: IS_length_1391-IPR001207_495_804 / IS1249, 21 genera, 46 species)

## How to apply

Load the COBS sample-hits table and the study metadata for the 661,405 indexed microbial genomes with pandas. Map each COBS hit genome to its species and genus via the metadata table, then aggregate distinct-species and distinct-genera counts (e.g. pandas nunique) per insertion sequence. Rank ISs by sharing breadth and confirm the top hit; in the source analysis IS_length_1391-IPR001207_495_804 (ISfinder name IS1249) ranks first with presence in 21 genera and 46 species. Cross-reference each genus-level IS set against ISfinder annotation status to verify that ISs shared across multiple Bacteroides, Corynebacterium, Curibacterium and Prevotella species are exclusively those absent from ISfinder. Write the per-IS sharing table (IS id, genera count, species count, ISfinder status) to is_sharing_summary.csv, whose expected values were validated against the paper's Figures 3-4.

## Related tools

- **COBS** (Provides the compact bit-sliced signature index of 661,405 microbial genomes used to obtain sample-hits for each ISC insertion sequence)
- **Palidis** (Pipeline that generated the ISC catalogue of insertion sequences whose cross-taxa sharing is aggregated here) — https://github.com/blue-moon22/palidis
- **ISfinder** (Reference database used to assign known-IS names and determine annotation status (e.g. IS1249 for IS_length_1391-IPR001207_495_804))
- **ISC** (Source catalogue (879 insertion sequences) whose members are queried against the COBS index and whose FASTA entries supply the IS ids) — https://github.com/blue-moon22/ISC

## Examples

```
import pandas as pd; hits = pd.read_csv('cobs_sample_hits.tsv', sep='\t').merge(meta, on='genome'); sharing = hits.groupby('IS').agg(genera=('genus','nunique'), species=('species','nunique')).sort_values(['genera','species'], ascending=False); sharing.to_csv('is_sharing_summary.csv')
```

## Evaluation signals

- IS_length_1391-IPR001207_495_804 is reproduced as the most widely shared IS with 21 distinct genera and 46 distinct species, matching the paper's Figures 3-4
- The summary CSV has exactly the schema: IS id, genera count, species count, ISfinder status
- Genera/species counts computed as distinct (nunique) values after genome-to-taxon mapping are consistent between the COBS hits table and metadata join
- ISs found across multiple Bacteroides, Corynebacterium, Curibacterium and Prevotella species are all absent from ISfinder
- The ranked top of the sharing table is stable across reruns of the same COBS hits and metadata inputs

## Limitations

- ISC v1.0.0 was built only from metagenomes of the healthy human oral cavity and stool samples, so shared-IS findings are anchored to ISs discoverable by Palidis from those sources
- The catalogue only contains common DDE-type ISs with ITRs; ISs lacking ITRs are not represented and thus cannot appear in the sharing analysis
- ISC contains ISs with ITRs of 25 or more nucleotides as generated by Palidis, although ITRs can be as short as ten nucleotides, so short-ITR ISs shared across taxa are missed

## Evidence

- [intro] COBS query against 661,405 genomes: "querying them against ISfinder or a compact bit-sliced signature (COBS) index [11] of 661 405 microbial genomes"
- [abstract] Catalogue queried against isolate genomes: "Querying this catalogue against a large database of isolate genomes reveals evidence of horizontal gene transfer events across bacterial classes."
- [abstract] ISC contents: "Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel and not previously characterized."
- [readme] pal-MEM role: [evidence span withheld — anchor status: external_doc]
