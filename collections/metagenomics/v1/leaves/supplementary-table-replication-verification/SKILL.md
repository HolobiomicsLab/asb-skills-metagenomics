---
name: supplementary-table-replication-verification
description: Use when a task asks to replicate or validate published per-species or
  per-genus probe coverage statistics from a probe design paper, e.g. checking that
  a sepsis probe FASTA covers all strains at genus level and 53.3% at species level.
when_to_use_negative:
- The input is raw sequencing reads or capture enrichment data rather than a designed
  probe FASTA — this skill validates probe composition, not capture performance
- The probe set was produced by a different pipeline version with different cluster
  partitioning (the sepsis set has a known cluster-partitioning flaw corrected only
  in newer HUBDesign versions), so exact count replication against other versions
  is not expected
- Genomes lack annotation or taxon mapping — per-species aggregation requires PROKKA-style
  annotated genomes with genome-to-taxon assignments
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: HUBDesign
  role: Source of the sepsis probe FASTA and probe-design parameters (probes may hybridize
    to sequences up to ~20% divergent, informing the identity threshold)
  repo: https://github.com/zacherydickson/HUBDesign
- name: BLASTn
  role: Aligns each probe sequence against BLASTn databases of the 1,926 annotated
    genomes to assign probes to species and genus
- name: PROKKA
  role: Suggested annotation tool producing the annotated genome GFF inputs used to
    build the genome set
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:b857aac3d21c05b997f2a1fc0238a142f431f363cbc270e511112e7e6418941e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/zacherydickson/HUBDesign
  repo_url: https://github.com/zacherydickson/HUBDesign
  tools:
  - BLASTn
  - HUBDesign
  - PROKKA
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# supplementary-table-replication-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A verification skill for reproducing the per-species probe-count composition of the HUBDesign sepsis probe set (1,926 bacterial genomes, 81 species, 35 genera) by BLASTn-aligning the probe FASTA back to the annotated genomes and comparing aggregate counts against the paper's Table S6 anchor values.

## When to use

Use when a task asks to replicate or validate published per-species or per-genus probe coverage statistics from a probe design paper, e.g. checking that a sepsis probe FASTA covers all strains at genus level and 53.3% at species level.

## When NOT to use

- The input is raw sequencing reads or capture enrichment data rather than a designed probe FASTA — this skill validates probe composition, not capture performance
- The probe set was produced by a different pipeline version with different cluster partitioning (the sepsis set has a known cluster-partitioning flaw corrected only in newer HUBDesign versions), so exact count replication against other versions is not expected
- Genomes lack annotation or taxon mapping — per-species aggregation requires PROKKA-style annotated genomes with genome-to-taxon assignments

## Inputs

- sepsis probe FASTA (probes/Sepsis directory of zacherydickson/HUBDesign)
- genome-to-taxon metadata for 1,926 annotated sepsis bacterial genomes (81 species, 35 genera)
- BLASTn database built from the annotated genome set

## Outputs

- sepsis_species_probe_counts.tsv (per-species probe-count table)
- per-genus and per-species coverage fractions (genus-level: 100% of strains; species-level: 53.3%)

## How to apply

Retrieve the sepsis probe FASTA (from the HUBDesign repository's probes/Sepsis directory) together with genome-to-taxon metadata for the 1,926 annotated sepsis bacterial genomes spanning 81 species and 35 genera. Build BLASTn (version 2.9.1+ compatible) databases from the annotated genome set and align every probe sequence against all genomes. Assign each probe's best alignment to its source genome's species and genus, applying a permissive identity threshold because HUBDesign probes are designed to hybridize to sequences up to 20% divergent. Aggregate alignments into per-species and per-genus probe counts and compute the fraction of strains with at least one probe at each taxonomic level, writing the table to sepsis_species_probe_counts.tsv. Validate by checking the table's extremes against the reported Table S6 values: C. botulinum 12, C. perfringens 44, C. tetani 71, R. prowazekii 53, B. burgdorferi 90, median 478; spiked strains range from 110 (S. sanguinis) to 564 (B. multivorans).

## Related tools

- **HUBDesign** (Source of the sepsis probe FASTA and probe-design parameters (probes may hybridize to sequences up to ~20% divergent, informing the identity threshold)) — https://github.com/zacherydickson/HUBDesign
- **BLASTn** (Aligns each probe sequence against BLASTn databases of the 1,926 annotated genomes to assign probes to species and genus)
- **PROKKA** (Suggested annotation tool producing the annotated genome GFF inputs used to build the genome set)

## Examples

```
blastn -db sepsis_genomes -query probes/Sepsis/sepsis_probes.fna -outfmt '6 qseqid sseqid pident' -perc_identity 80 > probe_hits.tsv
```

## Evaluation signals

- Every one of the 1,926 strains has at least one genus-level probe (100% genus coverage)
- Species-level coverage fraction equals 53.3% of strains
- Table extremes match Table S6: C. botulinum 12, C. perfringens 44, C. tetani 71, R. prowazekii 53, B. burgdorferi 90 probes, median 478
- Spiked strain counts fall within the reported range: 110 probes for S. sanguinis to 564 for B. multivorans
- Output file sepsis_species_probe_counts.tsv is a tab-delimited per-species table with one row per species (81 species expected)

## Limitations

- The published sepsis probe set was produced with an earlier HUBDesign version having a cluster-partitioning flaw (nominal taxonomy rather than observed sequence similarity), so counts replicate only against that version's output
- Probe-to-genome assignment requires a permissive identity threshold since probes are designed to hybridize to sequences up to 20% divergent; too strict a threshold will undercount coverage
- The probe set contains highly divergent probes (e.g. Streptococcus spp.) due to the same cluster partitioning issue, which can affect count interpretation

## Evidence

- [other] The sepsis probe set targets all 1,926 bacterial strains with at least genus-level probes, and 53.3% of strains are targeted at the species level.: [evidence span withheld — anchor status: not_found]
- [discussion] Version dependency of the sepsis set: [evidence span withheld — anchor status: not_found]
