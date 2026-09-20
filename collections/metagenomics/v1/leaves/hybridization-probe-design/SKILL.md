---
name: hybridization-probe-design
description: Use when you have annotated genomes (e.g., PROKKA GFFs) and NCBI taxon-ids
  for a diverse pathogen group and need a bounded, tiled bait set that captures known
  plus up-to-20%-divergent related sequences.
when_to_use_negative:
- Targets have unknown or unannotatable genomes — HUBDesign relies on annotated genomes
  and can only target close relatives
- The goal is detection of a single known locus via PCR/amplicon methods rather than
  hybridization-capture of diverse whole genomes
- The input is raw sequencing reads or an unannotated metagenome assembly rather than
  annotated genomes with taxon-ids
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3673
tools:
- name: HUBDesign
  role: Main pipeline orchestrating clustering, assignment, identification, filtering,
    and selection of probe sets
  repo: github.com/zacherydickson/HUBDesign
- name: SA_BOND
  role: 'Identification stage: finds oligos unique to each pseudo-genome; most memory-intensive
    step'
  repo: github.com/zacherydickson/HUBDesign
- name: PROKKA
  role: Suggested tool to annotate input genomes (kingdom set appropriately, e.g.,
    viruses) before probe design
- name: BLASTn
  role: 'Filtering stage: removes candidate probes capturing blacklist (e.g., human
    GRCh38) off-target sequences'
- name: MAFFT
  role: Multiple sequence alignment used to collapse divergent gene sequences into
    representative sequences
- name: dustmasker
  role: Low-complexity region masking during filtering of candidate probes
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:b6cc7e30152fc673ea899ca2f4e1af8db1b05b66f65f06c38d4d5717c356ebb1
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
  - BioPerl
  - CLUSTAL OMEGA
  - HUBDesign
  - Identification.pl
  - MAFFT
  - MUSCLE
  - PROKKA
  - Parallel::ForkManager
  - Perl
  - SA_BOND
  - dustmasker
  - sdust
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# hybridization-probe-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Design compact hybridization-capture probe sets (e.g., 75-mers) that simultaneously and specifically enrich diverse, divergent microbial targets over host/environmental background, using HUBDesign's hierarchical clustering, pseudo-genome construction, and balanced bait selection.

## When to use

Use when you have annotated genomes (e.g., PROKKA GFFs) and NCBI taxon-ids for a diverse pathogen group and need a bounded, tiled bait set that captures known plus up-to-20%-divergent related sequences.

## When NOT to use

- Targets have unknown or unannotatable genomes — HUBDesign relies on annotated genomes and can only target close relatives
- The goal is detection of a single known locus via PCR/amplicon methods rather than hybridization-capture of diverse whole genomes
- The input is raw sequencing reads or an unannotated metagenome assembly rather than annotated genomes with taxon-ids

## Inputs

- PROKKA-annotated genome files (GFF) for each target organism
- GenomeInfo.tab: tab-delimited file of paths-to-annotated-genome and taxon-id
- Guide.tree: newick guide tree built from NCBI taxonomy lineages of the input genomes
- BLAST blacklist databases (e.g., GRCh38 human genome or NCBI non-redundant nucleotide database)
- Parameters: --probe-count, --probe-length (75), --tiling-density, --r2t-divergence, --penetrance

## Outputs

- TiledBaits.fna: FASTA of final probe sequences (e.g., 13,500 x 75 nt probes)
- Tab-delimited file describing the source and position of probe sequences
- CandidateBaitRegions.tab and FilteredCandidates.tab: candidate probe regions before and after off-target filtering
- RepSeq.fna: representative cluster sequences
- Assignment.tsv: cluster-to-node assignment table
- Newick-formatted tree describing taxon relationships

## How to apply

Annotate input genomes with PROKKA and build a GenomeInfo.tab mapping annotated genome paths to NCBI taxon-ids, plus a guide tree from NCBI taxonomy lineages. Run the five HUBDesign phases: Clustering collapses genes at most 15% divergent (r2t-divergence 15%, penetrance 0.5–0.95) into representative sequences via MAFFT; Assignment places representatives at the lowest common ancestor of possessing organisms; Identification runs SA_BOND to find 75-mer candidates unique to each concatenated pseudo-genome; Filtering removes off-target candidates with BLASTn against blacklist databases (e.g., GRCh38 human) and dustmasker/sdust low-complexity masking; Selection picks a balanced tiled set capped by --probe-count at the chosen --tiling-density. The rationale is that sequence homology allows probes to capture both known and novel targets at multiple taxonomic levels while keeping counts low (e.g., 13,500 probes for 56 coronaviruses; 26,870 for 1,926 sepsis genomes).

## Related tools

- **HUBDesign** (Main pipeline orchestrating clustering, assignment, identification, filtering, and selection of probe sets) — github.com/zacherydickson/HUBDesign
- **SA_BOND** (Identification stage: finds oligos unique to each pseudo-genome; most memory-intensive step) — github.com/zacherydickson/HUBDesign
- **PROKKA** (Suggested tool to annotate input genomes (kingdom set appropriately, e.g., viruses) before probe design)
- **BLASTn** (Filtering stage: removes candidate probes capturing blacklist (e.g., human GRCh38) off-target sequences)
- **MAFFT** (Multiple sequence alignment used to collapse divergent gene sequences into representative sequences)
- **dustmasker** (Low-complexity region masking during filtering of candidate probes)

## Examples

```
bin/HUBDesign --guide-tree test/Guide.tree --tiling-density 1 --probe-count 13500 --probe-length 75 --output-dir test_res --verbose --keep test/GenomeInfo.tab
```

## Evaluation signals

- Output FASTA contains exactly the requested --probe-count probes at the specified --probe-length (e.g., 13,500 probes of 75 nt for coronavirus)
- Summary statistics reproduce Table 1 targets: coronavirus 25.0% nucleotide coverage, 4.72x depth, efficiency 1.87; sepsis 26,870 probes, 2.09% coverage, 3.64x depth, efficiency 29.3
- Every retained candidate probe shares no more than 15 consecutive identities and meets the 75% overall identity threshold versus oligos from other pseudo-genomes, confirmed in uniqueness_report.tsv
- FilteredCandidates.tab contains no probes hitting blacklist BLAST databases (e.g., GRCh38) beyond thresholds
- Probe count per input genome is balanced; unexpectedly-small probe sets indicate --tiling-density set too high to balance across organisms

## Limitations

- HUBDesign relies on annotated genomes; probes cannot be designed for targets with unknown or unannotatable genomes
- SA_BOND memory scales linearly with genome size and organism number, but time requirements grow much more quickly with diversity
- Enrichment level is target-sequence dependent and will vary for each probe set produced
- Earlier HUBDesign versions had a cluster-partitioning flaw (nominal taxonomy rather than observed sequence similarity) affecting the published sepsis probe set, including divergent Streptococcus spp. probes
- Validation used only artificial pooled samples, not patient or environmental samples; damaged/modified nucleic acids in real samples could reduce capture efficacy
- No contaminant blacklist databases are provided; avoiding reagent-contaminant capture is suggested as future capability

## Evidence
