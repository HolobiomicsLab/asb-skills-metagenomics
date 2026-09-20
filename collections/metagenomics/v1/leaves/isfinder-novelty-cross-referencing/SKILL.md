---
name: isfinder-novelty-cross-referencing
description: Use when a set of predicted insertion sequence IDs (e.g. ISC entries
  like IS_length_1391-IPR001207_495_804) needs known/novel status resolved against
  ISfinder before ranking or reporting.
when_to_use_negative:
- The goal is de novo IS discovery from reads — use Palidis instead; this skill only
  annotates existing IS sequences against ISfinder
- The IS set comes from sources outside the healthy human oral cavity and stool samples
  that ISC v1.0.0 was built from, limiting the annotation reference's applicability
- Sequences are not insertion sequences with inverted terminal repeats — ISC only
  contains common DDE-type ISs with ITRs of 25 or more nucleotides
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3299
tools:
- name: ISfinder
  role: Reference database of known transposable elements used to annotate ISs and
    determine known vs novel status
- name: COBS
  role: Compact bit-sliced signature index of 661,405 microbial genomes queried to
    map ISs to genomes for breadth-of-sharing analysis alongside ISfinder status
- name: Palidis
  role: Upstream pipeline that generates the IS catalogue (ISC) whose entries are
    cross-referenced against ISfinder
  repo: https://github.com/blue-moon22/palidis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:5f3667ccbc58e3ec87a1cd66a27f6aaf2dc5fab50de8c9c62e6fef38e3b82fba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis
  tools:
  - COBS
  - ISfinder
  - Palidis
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# isfinder-novelty-cross-referencing

## Summary

Cross-references catalogue insertion sequences (e.g. the 879 ISs in ISC) against the ISfinder reference database to label each IS as known (with an ISfinder name such as IS1249) or novel, and to propagate that annotation into downstream sharing/coverage analyses.

## When to use

Use when a set of predicted insertion sequence IDs (e.g. ISC entries like IS_length_1391-IPR001207_495_804) needs known/novel status resolved against ISfinder before ranking or reporting.

## When NOT to use

- The goal is de novo IS discovery from reads — use Palidis instead; this skill only annotates existing IS sequences against ISfinder
- The IS set comes from sources outside the healthy human oral cavity and stool samples that ISC v1.0.0 was built from, limiting the annotation reference's applicability
- Sequences are not insertion sequences with inverted terminal repeats — ISC only contains common DDE-type ISs with ITRs of 25 or more nucleotides

## Inputs

- FASTA of catalogue insertion sequences (e.g. ISC v1.0.0 with 879 ISs, IDs like IS_length_1391-IPR001207_495_804)
- ISfinder reference database of known transposable elements
- COBS sample-hits table from querying ISs against the index of 661,405 microbial genomes
- Study metadata mapping genome IDs to species and genus

## Outputs

- Per-IS annotation table (IS id, ISfinder name or absence, genera count, species count) e.g. is_sharing_summary.csv
- Set of novel ISs absent from ISfinder (519 of 879 ISC v1.0.0 entries were novel)

## How to apply

Load the IS catalogue entries (e.g. the 879 ISs in ISC v1.0.0, named like IS_length_1391-IPR001207_495_804) and query each sequence against ISfinder, as done in the Palidis pipeline where microbial genomes are annotated by querying reference databases such as ISfinder. Attach the ISfinder match (e.g. IS1249 for IS_length_1391-IPR001207_495_804) or mark the IS as absent from ISfinder (novel/uncharacterized). Combine this per-IS annotation status with COBS sample-hits tables so that, for example, genus-level IS sharing in the 661,405-genome COBS index can be split by ISfinder status — the Palidis paper used this to show that ISs shared across multiple Bacteroides, Corynebacterium and Prevotella species are exclusively those absent from ISfinder. Emit a per-IS table (IS id, ISfinder name or 'absent', plus downstream metrics) and verify against published values (IS1249 shared across 21 genera and 46 species).

## Related tools

- **ISfinder** (Reference database of known transposable elements used to annotate ISs and determine known vs novel status)
- **COBS** (Compact bit-sliced signature index of 661,405 microbial genomes queried to map ISs to genomes for breadth-of-sharing analysis alongside ISfinder status)
- **Palidis** (Upstream pipeline that generates the IS catalogue (ISC) whose entries are cross-referenced against ISfinder) — https://github.com/blue-moon22/palidis

## Examples

```
python make_is_sharing_summary.py --cobs-hits sample_hits.tsv --metadata metadata.tsv --isfinder isfinder_hits.tsv --out is_sharing_summary.csv
```

## Evaluation signals

- IS_length_1391-IPR001207_495_804 resolves to ISfinder name IS1249 with 21 genera and 46 species, matching Figures 3-4 of the paper
- The per-IS summary contains non-null ISfinder status for every catalogue entry (519 of 879 expected to be absent/novel)
- Genus-level IS sets shared across multiple Bacteroides, Corynebacterium, Curibacterium and Prevotella species consist exclusively of ISs absent from ISfinder
- Distinct-species and distinct-genera counts per IS are reproducible when re-aggregating the COBS hits joined to metadata

## Limitations

- ISC v1.0.0 was built only from healthy human oral cavity and stool metagenomes, so it is recommended as a reference only for isolates from those sources
- The catalogue contains ISs with ITRs of 25 or more nucleotides; ITRs as short as ten nucleotides are missed at Palidis default settings
- Palidis/ISC only capture common DDE-type ISs with ITRs; ISs lacking ITRs cannot be discovered or annotated with this resource

## Evidence

- [intro] Microbial genomes can be annotated for ISs by querying reference databases of known transposable elements, such as ISfinder: "Microbial genomes can be annotated for ISs by querying reference databases of known transposable elements, such as ISfinder [7]"
- [intro] COBS index scope: "querying them against ISfinder or a compact bit-sliced signature (COBS) index [11] of 661 405 microbial genomes"
- [abstract] Novel IS counts: "Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel and not previously characterized."
- [discussion] ISC source restriction: "since Palidis was only applied to metagenomes sequenced from the healthy human oral cavity and stool samples, it is recommended ISC v1.0.0 is used as a reference for annotating isolates sourced from"
- [discussion] ITR length limitation: "the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis, although ITRs can be as short as ten nucleotides in length"
