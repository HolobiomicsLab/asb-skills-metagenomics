---
name: taxonomic-probe-count-tabulation
description: Use when you have a published probe/bait FASTA plus genome-to-taxon metadata
  and need to compute what fraction of strains are covered at genus vs. species level.
when_to_use_negative:
- Your goal is to design new probes rather than audit an existing probe FASTA's taxonomic
  coverage — use the HUBDesign pipeline (Clustering/Assignment/Identification/Filtering/Selection)
  instead
- Input is raw sequencing read data needing human-read filtration, deduplication,
  or enrichment analysis rather than probe-set coverage quantification
- The genomes are unannotated or taxonomically unmappable — HUBDesign relies on annotated
  genomes and probes cannot be designed for unknown genomes, so coverage aggregation
  against a taxon map has no basis
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BLASTn
  role: Aligns each probe sequence against a database of the panel genomes to find
    which species/genera each probe hits
- name: HUBDesign
  role: Source of the probe set being audited; designs probes at multiple taxonomic
    levels using sequence homology
  repo: github.com/zacherydickson/HUBDesign
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:a5a8cf5f9dc73b8567eded4dcb746735a5d0576ce680eac576a867027e84ee68
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/zacherydickson/HUBDesign
  tools:
  - BLASTn
  - HUBDesign
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# taxonomic-probe-count-tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how thoroughly a designed probe set (e.g., the HUBDesign sepsis probe FASTA) covers a genome panel at each taxonomic level, by aligning every probe against all genomes with BLASTn and aggregating hits into per-species and per-genus probe counts. This characterizes the breadth-versus-specificity tradeoff HUBDesign intentionally builds in via homology-based multi-level probe design.

## When to use

Use when you have a published probe/bait FASTA plus genome-to-taxon metadata and need to compute what fraction of strains are covered at genus vs. species level.

## When NOT to use

- Your goal is to design new probes rather than audit an existing probe FASTA's taxonomic coverage — use the HUBDesign pipeline (Clustering/Assignment/Identification/Filtering/Selection) instead
- Input is raw sequencing read data needing human-read filtration, deduplication, or enrichment analysis rather than probe-set coverage quantification
- The genomes are unannotated or taxonomically unmappable — HUBDesign relies on annotated genomes and probes cannot be designed for unknown genomes, so coverage aggregation against a taxon map has no basis

## Inputs

- Probe set FASTA (e.g., HUBDesign sepsis probes from the probes/Sepsis directory)
- Annotated bacterial genome set (1,926 sepsis genomes; PROKKA-annotation is the pipeline's suggested format)
- Genome-to-taxon metadata mapping each genome to species and genus (81 species, 35 genera for the sepsis dataset)

## Outputs

- Per-species probe-count table (sepsis_species_probe_counts.tsv)
- Per-genus and per-species coverage fractions of the strain panel (all 1,926 strains at genus level; 53.3% at species level)

## How to apply

Retrieve the probe FASTA (e.g., from the GitHub probes/Sepsis directory) together with genome-to-taxon metadata for the annotated genome panel (for the sepsis set: 1,926 bacterial genomes, 81 species, 35 genera). Build BLASTn databases from the genome set and align every probe sequence against all genomes, using a permissive identity threshold because HUBDesign probes may hybridize to sequences up to ~20% divergent. Assign each probe's best alignment to the species and genus of the genome it hits, then aggregate into per-species and per-genus probe-count tables and compute the fraction of strains with at least one probe at each level. For the sepsis probe set, expect all 1,926 strains covered at genus level and 53.3% at species level; write per-species counts to a TSV such as sepsis_species_probe_counts.tsv.

## Related tools

- **BLASTn** (Aligns each probe sequence against a database of the panel genomes to find which species/genera each probe hits)
- **HUBDesign** (Source of the probe set being audited; designs probes at multiple taxonomic levels using sequence homology) — github.com/zacherydickson/HUBDesign

## Examples

```
makeblastdb -in sepsis_genomes.fna -dbtype nucl; blastn -query probes/Sepsis/probes.fna -db sepsis_genomes -out hits.tsv -outfmt 6 -perc_identity 80
```

## Evaluation signals

- Table extremes match the reported Table S6 replication values: C. botulinum 12, C. perfringens 44, C. tetani 71, R. prowazekii 53, B. burgdorferi 90, median 478 probes per species
- Spiked-strain probe counts fall in the reported range of 110 (S. sanguinis) to 564 (B. multivorans)
- Every one of the 1,926 strains has at least one genus-level probe (coverage fraction = 1.0 at genus level)
- Species-level coverage fraction equals 53.3% of the 1,926 strains
- Output TSV has one row per species (81 rows) with probe counts consistent with the sum of per-probe best-hit assignments

## Limitations

- The sepsis probe set was produced with an earlier HUBDesign version carrying a cluster-partitioning flaw: genera were considered independently even where targeted sequences were shared only at family level, so some probes are not truly specific to their nominal taxon (e.g., highly divergent Streptococcus spp. probes)
- Probe hits must be scored with a permissive identity threshold because probes may hybridize to sequences up to ~20% divergent, which blurs species-level assignment boundaries
- No blacklist/contaminant databases were provided to the pipeline, so probe counts can include probes capturing off-target or contaminant-like sequences

## Evidence
