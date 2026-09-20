---
name: multiple-sequence-alignment-of-viral-genes
description: Use when starting from PROKKA-annotated genome GFF files (e.g. 56 RefSeq
  coronavirus genomes) and homologous gene families must be aligned, tree-ordered,
  and collapsed into representative sequences for downstream LCA assignment.
when_to_use_negative:
- Genomes are unknown or unannotatable — 'HUBDesign relies on annotated genomes to
  efficiently cluster sequences. Probes cannot be designed for targets with unknown,
  or unannotatable genomes.'
- Input is raw, unannotated assemblies or reads rather than annotated GFF files; annotation
  (PROKKA is suggested) must precede clustering.
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: HUBDesign (bin/Clustering.pl)
  role: 'Runs the clustering stage: groups homologous gene sequences, aligns them,
    builds neighbour-joining trees, and writes RepSeq.fna'
  repo: https://github.com/zacherydickson/HUBDesign
- name: PROKKA
  role: Suggested annotation tool producing the GFF genome annotations that are the
    clustering input
- name: MAFFT
  role: Default-supported multiple sequence aligner (version 7.0+) for gene families
- name: MUSCLE
  role: Default-supported multiple sequence aligner (version 3.8.425+) for gene families
- name: CLUSTAL OMEGA
  role: Default-supported multiple sequence aligner (version 1.2.1+) for gene families
- name: BioPerl
  role: Perl library (1.7.10+) used to drive the multiple sequence aligners and build
    neighbour-joining gene trees
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:7b24f9dd6f8cb8979fd73ee8c5d24087f372b2d4e910e8875f4617f33a02d9c2
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
  - BioPerl
  - CLUSTAL OMEGA
  - HUBDesign
  - HUBDesign (bin/Clustering.pl)
  - MAFFT
  - MUSCLE
  - PROKKA
  - Perl
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# multiple-sequence-alignment-of-viral-genes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters homologous gene sequences from annotated viral genomes using supported multiple sequence aligners (MAFFT 7.0+, MUSCLE 3.8.425+, CLUSTAL OMEGA 1.2.1+) with BioPerl-built neighbour-joining gene trees, collapsing sequences at nodes with root-to-tip divergence of at most 15% into representative consensus sequences (RepSeq.fna).

## When to use

Use when starting from PROKKA-annotated genome GFF files (e.g. 56 RefSeq coronavirus genomes) and homologous gene families must be aligned, tree-ordered, and collapsed into representative sequences for downstream LCA assignment.

## When NOT to use

- Genomes are unknown or unannotatable — 'HUBDesign relies on annotated genomes to efficiently cluster sequences. Probes cannot be designed for targets with unknown, or unannotatable genomes.'
- Input is raw, unannotated assemblies or reads rather than annotated GFF files; annotation (PROKKA is suggested) must precede clustering.

## Inputs

- PROKKA-annotated genome GFF files (e.g. for 56 RefSeq coronavirus genomes listed in GenomeInfo.tab)
- Guide.tree file (Newick, e.g. generated from NCBI taxonomy lineages)
- GenomeInfo.tab (tab-delimited file of paths-to-annotated-genome and taxon-id)

## Outputs

- RepSeq.fna — FASTA of representative (consensus) sequences collapsed at <=15% root-to-tip divergence
- Cluster information files detailing the composition of gene clusters (consumed by bin/Assignment.pl)

## How to apply

Take PROKKA-annotated GFF files for the input genomes and run bin/Clustering.pl to group homologous gene sequences into clusters. Align each gene family with a supported aligner — MAFFT version 7.0+, MUSCLE version 3.8.425+, or CLUSTAL OMEGA version 1.2.1+ — invoked via BioPerl 1.7.10+, and build neighbour-joining gene trees. Collapse sequences at nodes whose root-to-tip divergence is at most 15% (the --r2t-divergence parameter) into representative consensus sequences, rationale being that hybridization to sequences up to 20% divergent is possible so within-cluster divergence must stay well below that ceiling. Write the representative sequences to RepSeq.fna, which feeds the Assignment step where each representative is placed at the LCA of the organisms possessing it. Correctness depends on the guide tree (e.g. derived from NCBI taxonomy lineages) constraining how nested relationships between input genomes are interpreted.

## Related tools

- **HUBDesign (bin/Clustering.pl)** (Runs the clustering stage: groups homologous gene sequences, aligns them, builds neighbour-joining trees, and writes RepSeq.fna) — https://github.com/zacherydickson/HUBDesign
- **PROKKA** (Suggested annotation tool producing the GFF genome annotations that are the clustering input)
- **MAFFT** (Default-supported multiple sequence aligner (version 7.0+) for gene families)
- **MUSCLE** (Default-supported multiple sequence aligner (version 3.8.425+) for gene families)
- **CLUSTAL OMEGA** (Default-supported multiple sequence aligner (version 1.2.1+) for gene families)
- **BioPerl** (Perl library (1.7.10+) used to drive the multiple sequence aligners and build neighbour-joining gene trees)

## Examples

```
bin/Clustering.pl [-options] GFF_Files ... > RepSeq.fna
```

## Evaluation signals

- RepSeq.fna is produced and every cluster's representative sequences satisfy the <=15% root-to-tip divergence threshold (--r2t-divergence)
- Cluster composition files are in a format preserved for the downstream Assignment step (bin/Assignment.pl ClusterInfo ... > Assignment.tsv runs without format errors)
- All input taxa (e.g. all 56 coronavirus genomes) are represented by at least one cluster/representative in the output
- The pipeline completes only with a supported aligner present (MAFFT 7.0+, MUSCLE 3.8.425+, or CLUSTAL OMEGA 1.2.1+) and Perl 5.10.0+ / BioPerl 1.7.10+ installed
- For the full test dataset, output can be compared to the reference outputs in the repository's test directory

## Limitations

- As more divergent baits are less effective, the divergence threshold trades breadth of coverage against specificity; genes partitioned by nominal taxonomy rather than observed sequence similarity caused flawed cluster assignments in earlier versions (e.g. the sepsis probe set, Streptococcus spp.)
- Requires annotated genomes; unannotatable targets cannot be clustered or probed
- Pipeline development was ongoing, and version discrepancies exist: the current HUBDesign constructs a new cluster-assignment tree based on observed clusters rather than using only a user-provided tree

## Evidence
