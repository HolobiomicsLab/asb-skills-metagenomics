---
name: taxonomic-metadata-mapping
description: Use when genome-to-taxon mapping is needed to rank catalogue elements
  (e.g. ISC insertion sequences) by how many genera and species their hits span across
  an indexed genome database.
when_to_use_negative:
- Genome-to-taxon assignments are already present in the hit table, so no metadata
  join is required
- The query is for single-genome placement rather than breadth-of-sharing aggregation
  across an indexed genome database
- The elements being ranked are not catalogue entries with genome hits (e.g. raw reads
  without a COBS index)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: COBS
  role: Provides the sample-hits table by querying the ISC catalogue against a compact
    bit-sliced signature index of 661 405 microbial genomes
- name: Palidis
  role: Source of the Insertion Sequence Catalogue (ISC) whose ISs are mapped to taxa
    and ranked by sharing breadth
  repo: https://github.com/blue-moon22/palidis
- name: ISfinder
  role: Reference database used to annotate known ISs and flag ISC entries absent
    from ISfinder (e.g. IS1249 = IS_length_1391-IPR001207_495_804)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:4510a6a7663341582972d74f2de26ecfd99480be3ff2f21174a986c9b4b7c662
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis
  tools:
  - COBS
  - ISfinder
  - Palidis
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# taxonomic-metadata-mapping

## Summary

Maps COBS sample-hit genome identifiers to genus and species labels via a study metadata table for the 661,405 indexed microbial genomes, then aggregates distinct-genus and distinct-species counts per insertion sequence to quantify taxonomic sharing breadth of the ISC catalogue.

## When to use

Use when genome-to-taxon mapping is needed to rank catalogue elements (e.g. ISC insertion sequences) by how many genera and species their hits span across an indexed genome database.

## When NOT to use

- Genome-to-taxon assignments are already present in the hit table, so no metadata join is required
- The query is for single-genome placement rather than breadth-of-sharing aggregation across an indexed genome database
- The elements being ranked are not catalogue entries with genome hits (e.g. raw reads without a COBS index)

## Inputs

- COBS sample-hits table (query hits against the COBS index of 661 405 microbial genomes)
- Study metadata table mapping genome identifiers to species and genus for the 661,405 indexed genomes
- ISfinder annotation status per insertion sequence

## Outputs

- Per-IS sharing table (is_sharing_summary.csv) with IS id, distinct-genera count, distinct-species count, and ISfinder status
- Ranked list of most widely shared insertion sequences

## How to apply

Load the COBS sample-hits table and the study metadata for the 661,405 indexed microbial genomes with pandas. Map each COBS hit genome to its species and genus via the metadata table, then aggregate distinct-species and distinct-genera counts per insertion sequence. Rank ISs by sharing breadth; in the Palidis paper this reproduced IS_length_1391-IPR001207_495_804 (= IS1249 in ISfinder) as the most widely shared IS, present in 21 genera and 46 species, matching Figures 3-4. Finally cross-reference each genus-level IS set against ISfinder annotation status and write a per-IS sharing table (IS id, genera count, species count, ISfinder status).

## Related tools

- **COBS** (Provides the sample-hits table by querying the ISC catalogue against a compact bit-sliced signature index of 661 405 microbial genomes)
- **Palidis** (Source of the Insertion Sequence Catalogue (ISC) whose ISs are mapped to taxa and ranked by sharing breadth) — https://github.com/blue-moon22/palidis
- **ISfinder** (Reference database used to annotate known ISs and flag ISC entries absent from ISfinder (e.g. IS1249 = IS_length_1391-IPR001207_495_804))

## Examples

```
import pandas as pd; hits = pd.read_csv('cobs_sample_hits.tsv', sep='\t'); meta = pd.read_csv('metadata_661405.tsv', sep='\t'); m = hits.merge(meta, on='genome'); sharing = m.groupby('IS_id').agg(genera=('genus','nunique'), species=('species','nunique')); sharing.sort_values(['genera','species'], ascending=False).to_csv('is_sharing_summary.csv')
```

## Evaluation signals

- IS_length_1391-IPR001207_495_804 (IS1249) is reproduced as the most widely shared IS with 21 genera and 46 species
- Aggregated counts use distinct species and distinct genera per IS, not raw hit counts
- Every COBS hit genome maps to a species and genus in the metadata table (no unmapped identifiers)
- Summary table contains exactly the columns: IS id, genera count, species count, ISfinder status
- Values match those reported in the paper's Figures 3-4

## Limitations

- ISC v1.0.0 was built only from metagenomes of the healthy human oral cavity and stool samples, so taxonomic sharing profiles are grounded in ISs discovered from those sources
- The ISC contains only ISs with ITRs of 25 or more nucleotides (as short as ten nucleotides are missed at default settings), so sharing statistics exclude ISs lacking ITRs
- Hits depend on the COBS index composition (661 405 microbial genomes); broader or different genome sets will change the genera/species counts

## Evidence

- [intro] querying them against ISfinder or a compact bit-sliced signature (COBS) index of 661 405 microbial genomes: "querying them against ISfinder or a compact bit-sliced signature (COBS) index [11] of 661 405 microbial genomes"
- [intro] Microbial genomes can be annotated for ISs by querying reference databases of known transposable elements, such as ISfinder: "Microbial genomes can be annotated for ISs by querying reference databases of known transposable elements, such as ISfinder [7]"
- [discussion] since Palidis was only applied to metagenomes sequenced from the healthy human oral cavity and stool samples: "since Palidis was only applied to metagenomes sequenced from the healthy human oral cavity and stool samples, it is recommended ISC v1.0.0 is used as a reference"
- [discussion] the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length: "the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis, although ITRs can be as short as ten nucleotides in length"
