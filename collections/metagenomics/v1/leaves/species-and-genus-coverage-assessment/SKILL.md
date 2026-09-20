---
name: species-and-genus-coverage-assessment
description: Use when a published HUBDesign probe FASTA (e.g., probes/Sepsis) must
  be evaluated for taxonomic breadth — which strains of a multi-genome design dataset
  carry species-level versus genus-level probes.
when_to_use_negative:
- Input is raw read data or an enrichment experiment rather than a finished probe
  FASTA plus annotated genome set — this skill assesses probe design coverage, not
  capture performance.
- 'The probe set targets genomes lacking annotations: HUBDesign relies on annotated
  genomes, so species/genus assignment via source-genome taxon IDs is impossible for
  unknown or unannotatable genomes.'
- You need to re-run probe design (Clustering/Assignment/Identification/Filtering/Selection)
  rather than characterize an existing probe set's taxonomic coverage.
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: BLASTn
  role: Aligns each probe sequence against BLASTn databases built from the 1,926 annotated
    sepsis genomes to find each probe's best alignment (version 2.9.1+ compatible)
  repo: github.com/zacherydickson/HUBDesign
- name: HUBDesign
  role: Source of the probe FASTA being assessed; its multi-taxonomic-level probe
    design (sequence homology, hybridization to ~20% divergent sequences) motivates
    the permissive identity threshold
  repo: github.com/zacherydickson/HUBDesign
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:154d4390cdf242d0e97248d76651d9e1fae2616e885895c184cf19963b93f828
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
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# species-and-genus-coverage-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies taxonomic coverage of an HUBDesign probe set by aligning every probe sequence against the full annotated genome set with BLASTn and aggregating hits into per-species and per-genus probe counts, to determine what fraction of strains are targeted at species level versus genus level (e.g., 53.3% species-level vs. 100% genus-level coverage for the 1,926-strain sepsis dataset).

## When to use

Use when a published HUBDesign probe FASTA (e.g., probes/Sepsis) must be evaluated for taxonomic breadth — which strains of a multi-genome design dataset carry species-level versus genus-level probes.

## When NOT to use

- Input is raw read data or an enrichment experiment rather than a finished probe FASTA plus annotated genome set — this skill assesses probe design coverage, not capture performance.
- The probe set targets genomes lacking annotations: HUBDesign relies on annotated genomes, so species/genus assignment via source-genome taxon IDs is impossible for unknown or unannotatable genomes.
- You need to re-run probe design (Clustering/Assignment/Identification/Filtering/Selection) rather than characterize an existing probe set's taxonomic coverage.

## Inputs

- Probe set in FASTA format (e.g., HUBDesign sepsis probes from the GitHub probes/Sepsis directory)
- Genome-to-taxon metadata mapping the annotated genome set (1,926 sepsis bacterial genomes; 81 species, 35 genera) to species and genus
- BLASTn databases built from the annotated genome set

## Outputs

- Per-species probe-count table (sepsis_species_probe_counts.tsv, tab-delimited)
- Per-genus probe counts
- Fraction of strains with at least one probe at genus level (reported: 100% of 1,926 strains)
- Fraction of strains with at least one probe at species level (reported: 53.3%)

## How to apply

Retrieve the probe FASTA from the repository probes/Sepsis directory together with genome-to-taxon metadata for the annotated sepsis bacterial genome set (1,926 strains, 81 species, 35 genera). Build BLASTn databases (BLASTn 2.9.1+ compatible) from the annotated genomes and align every probe sequence against all genomes. Assign each probe's best alignment to the species and genus of its source genome, applying a permissive identity threshold because HUBDesign probes may hybridize to sequences up to ~20% divergent. Aggregate alignments into per-species and per-genus probe counts and compute the fraction of strains with at least one probe at each taxonomic level, writing the per-species table to sepsis_species_probe_counts.tsv.

## Related tools

- **BLASTn** (Aligns each probe sequence against BLASTn databases built from the 1,926 annotated sepsis genomes to find each probe's best alignment (version 2.9.1+ compatible)) — github.com/zacherydickson/HUBDesign
- **HUBDesign** (Source of the probe FASTA being assessed; its multi-taxonomic-level probe design (sequence homology, hybridization to ~20% divergent sequences) motivates the permissive identity threshold) — github.com/zacherydickson/HUBDesign

## Examples

```
blastn -query probes/Sepsis/sepsis_probes.fna -db sepsis_1926_genomes -task blastn -outfmt '6 qseqid sseqid pident length' -out sepsis_probe_hits.tsv
```

## Evaluation signals

- Every one of the 1,926 bacterial strains must show at least one genus-level probe (100% genus-level coverage).
- Species-level coverage must equal 53.3% of strains, reproducing the published replication value.
- Per-species extremes in the output table must match Table S6 replication values: C. botulinum 12, C. perfringens 44, C. tetani 71, R. prowazekii 53, B. burgdorferi 90, median 478.
- Spiked strains must fall within the reported range: minimum 110 probes for S. sanguinis to maximum 564 for B. multivorans.
- Output file sepsis_species_probe_counts.tsv must be a well-formed tab-delimited table spanning all 81 species of the sepsis dataset.

## Limitations

- The sepsis probe set was produced with an earlier HUBDesign version carrying a cluster-partitioning flaw (nominal taxonomy rather than observed sequence similarity), so some probes are not truly specific to their nominal genus — per-species counts may include shared family-level sequences.
- Probes may hybridize to sequences up to ~20% divergent, so a permissive identity threshold is required; a strict threshold would understate coverage.
- HUBDesign requires annotated genomes; coverage cannot be assessed for unknown or unannotatable genomes, only for close relatives.

## Evidence
