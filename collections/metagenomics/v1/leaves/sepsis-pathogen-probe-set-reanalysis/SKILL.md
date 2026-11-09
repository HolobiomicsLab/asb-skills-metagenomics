---
name: sepsis-pathogen-probe-set-reanalysis
description: Use when auditing probe coverage of an existing HUBDesign probe set (e.g.,
  probes/Sepsis FASTA) against annotated bacterial genomes to quantify per-species
  and per-genus targeting fractions.
when_to_use_negative:
- The goal is de novo probe design rather than auditing an existing probe set — use
  the HUBDesign pipeline (Clustering, Assignment, Identification, Filtering, Selection)
  instead
- Input genomes are unannotated or unannotatable; HUBDesign relies on annotated genomes
  and probes cannot be designed for unknown genomes
- The probe set targets non-bacterial diversity (e.g., the 56 RefSeq coronavirus genomes),
  where the sepsis genome/taxon metadata does not apply
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: HUBDesign
  role: Source of the sepsis probe set being reanalyzed and of the annotated-genome
    workflow it was designed with
  repo: https://github.com/zacherydickson/HUBDesign
- name: BLASTn
  role: Aligns each probe sequence against BLASTn databases built from the annotated
    genome set to assign probes to species/genus
- name: PROKKA
  role: Suggested tool for producing the annotated genomes used as input to the probe
    design and reanalysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:ec1693d6f1b485ee6ce411402baa4cb8df25d5774eacedbea719e2dea08eb536
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
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# sepsis-pathogen-probe-set-reanalysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reanalyzes the HUBDesign sepsis probe set by aligning every probe sequence against BLASTn databases built from the 1,926 annotated sepsis bacterial genomes (81 species, 35 genera) to compute per-species and per-genus probe coverage, confirming that all strains have genus-level probes and 53.3% have species-level probes.

## When to use

Use when auditing probe coverage of an existing HUBDesign probe set (e.g., probes/Sepsis FASTA) against annotated bacterial genomes to quantify per-species and per-genus targeting fractions.

## When NOT to use

- The goal is de novo probe design rather than auditing an existing probe set — use the HUBDesign pipeline (Clustering, Assignment, Identification, Filtering, Selection) instead
- Input genomes are unannotated or unannotatable; HUBDesign relies on annotated genomes and probes cannot be designed for unknown genomes
- The probe set targets non-bacterial diversity (e.g., the 56 RefSeq coronavirus genomes), where the sepsis genome/taxon metadata does not apply

## Inputs

- Sepsis probe FASTA from the HUBDesign repo probes/Sepsis directory
- Genome-to-taxon metadata for 1,926 annotated sepsis bacterial genomes (81 species, 35 genera)
- Annotated genome sequences (PROKKA-annotated) for building BLASTn databases

## Outputs

- sepsis_species_probe_counts.tsv — per-species probe-count table
- Per-genus probe counts and fraction of strains covered at genus level (target: 100%)
- Fraction of strains covered at species level (target: 53.3%)

## How to apply

Retrieve the sepsis probe FASTA from the probes/Sepsis directory of the HUBDesign repository along with genome-to-taxon metadata for the 1,926 annotated sepsis bacterial genomes. Build BLASTn databases (BLASTn 2.9.1+ compatible) from the annotated genome set and align every probe sequence against all genomes. Assign each probe's best alignment to its source genome's species and genus, using a permissive identity threshold because probes may hybridize to sequences up to ~20% divergent (HUBDesign's stated homology range). Aggregate alignments into per-species and per-genus probe counts, then compute the fraction of strains with at least one probe at each taxonomic level. Write the per-species probe-count table to sepsis_species_probe_counts.tsv.

## Related tools

- **HUBDesign** (Source of the sepsis probe set being reanalyzed and of the annotated-genome workflow it was designed with) — https://github.com/zacherydickson/HUBDesign
- **BLASTn** (Aligns each probe sequence against BLASTn databases built from the annotated genome set to assign probes to species/genus)
- **PROKKA** (Suggested tool for producing the annotated genomes used as input to the probe design and reanalysis)

## Examples

```
makeblastdb -in sepsis_genomes.fna -dbtype nucl && blastn -query probes/Sepsis/probes.fna -db sepsis_genomes -outfmt "6 qseqid sseqid pident" -out sepsis_probe_hits.tsv
```

## Evaluation signals

- Per-species table extremes match the published Table S6 replication values: C. botulinum 12, C. perfringens 44, C. tetani 71, R. prowazekii 53, B. burgdorferi 90, median 478
- Spiked strains fall in the reported range: 110 probes for S. sanguinis to 564 probes for B. multivorans
- Fraction of strains with at least genus-level coverage equals 1.0 (all 1,926 strains covered)
- Fraction of strains with species-level coverage equals 0.533 (53.3%)
- Output TSV contains a probe count for each of the 81 species in the genome-to-taxon metadata

## Limitations

- The sepsis probe set was produced with an earlier HUBDesign version and has a cluster-partitioning flaw (nominal taxonomy rather than observed sequence similarity), so some probes are not truly specific to their nominal genus
- Highly divergent probes (e.g., for Streptococcus spp.) were included due to the same cluster-partitioning issue and may inflate apparent cross-hybridization
- A permissive identity threshold must be used since probes may hybridize to sequences up to 20% divergent; too strict a threshold will undercount true coverage

## Evidence

- [results] The sepsis dataset covers 1,926 bacterial strains at genus level; 53.3% at species level: [evidence span withheld — anchor status: not_found]
