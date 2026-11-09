---
name: neighbour-joining-tree-construction
description: Use when annotated (e.g., PROKKA GFF) gene sequences from multiple taxa
  must be grouped into homologous clusters with phylogenetic support before assigning
  representatives to taxonomic nodes for probe or pseudo-genome design.
when_to_use_negative:
- Target genomes are unknown or unannotatable — HUBDesign relies on annotated genomes
  to efficiently cluster sequences, so probes cannot be designed for such targets
- Sequences within a candidate cluster exceed the r2t-divergence threshold and should
  not be collapsed (over-collapsing would produce representatives too divergent for
  effective capture)
- Only a single genome or a handful of closely related genomes is involved — cluster/LCA-based
  pseudo-genome construction adds complexity without benefit
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: HUBDesign
  role: Pipeline under which bin/Clustering.pl groups homologous gene sequences and
    collapses <=15%-divergent sequences into representatives
  repo: https://github.com/zacherydickson/HUBDesign
- name: MAFFT
  role: Default-supported multiple sequence aligner (7.0+) used on gene families before
    neighbour-joining tree construction
- name: MUSCLE
  role: Alternative multiple sequence aligner (3.8.425+) supported for gene-family
    alignment
- name: CLUSTAL OMEGA
  role: Alternative multiple sequence aligner (1.2.1+) supported for gene-family alignment
- name: BioPerl
  role: Perl library (1.7.10+) used to drive alignment and tree handling in the clustering
    stage
- name: PROKKA
  role: Suggested annotation tool producing the GFF gene annotations that are input
    to clustering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:fd6a8ce48ff5ba41cab20d3e7ac0b155f3fbe082e89d4528a99a892d2de755f8
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
  - MAFFT
  - MUSCLE
  - PROKKA
  - Perl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# neighbour-joining-tree-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Within the HUBDesign clustering stage, homologous gene sequences from PROKKA-annotated genomes are aligned with a supported multiple sequence aligner (MAFFT 7.0+, MUSCLE 3.8.425+, or CLUSTAL OMEGA 1.2.1+) via BioPerl, neighbour-joining gene trees are built, and sequences whose root-to-tip divergence is at most 15% are collapsed into representative consensus sequences (RepSeq.fna) that later feed LCA-based pseudo-genome construction.

## When to use

Use when annotated (e.g., PROKKA GFF) gene sequences from multiple taxa must be grouped into homologous clusters with phylogenetic support before assigning representatives to taxonomic nodes for probe or pseudo-genome design.

## When NOT to use

- Target genomes are unknown or unannotatable — HUBDesign relies on annotated genomes to efficiently cluster sequences, so probes cannot be designed for such targets
- Sequences within a candidate cluster exceed the r2t-divergence threshold and should not be collapsed (over-collapsing would produce representatives too divergent for effective capture)
- Only a single genome or a handful of closely related genomes is involved — cluster/LCA-based pseudo-genome construction adds complexity without benefit

## Inputs

- PROKKA-annotated GFF files for input genomes (e.g., 56 RefSeq coronavirus genomes named by taxon-id)
- GenomeInfo.tab mapping genome paths to taxon-ids
- Optional Guide.tree (newick) derived from NCBI taxonomy to guide nested relationships

## Outputs

- RepSeq.fna — FASTA of representative (consensus) cluster sequences
- Cluster information file describing the composition of gene clusters (feeding Assignment.pl)
- Downstream Assignment.tsv mapping representatives to LCA nodes

## How to apply

Extract gene families from the annotated genomes (PROKKA GFF files, e.g., the 56 RefSeq coronavirus genomes listed in GenomeInfo.tab) and run HUBDesign's bin/Clustering.pl. Align each gene family using a supported multiple sequence aligner — MAFFT 7.0+, MUSCLE 3.8.425+, or CLUSTAL OMEGA 1.2.1+ — driven through BioPerl 1.7.10+ under Perl 5.10.0+. Build neighbour-joining gene trees from the alignments and collapse sequences at nodes whose root-to-tip divergence is at most 15% into representative consensus sequences; the --r2t-divergence parameter controls this maximum divergence and can be tuned to balance breadth of coverage against bait specificity. Write representatives to RepSeq.fna, then pass cluster information to bin/Assignment.pl for lowest-common-ancestor assignment (guided by --guide-tree, e.g., the NCBI-taxonomy-derived Guide.tree) and pseudo-genome construction with --penetrance 0.5.

## Related tools

- **HUBDesign** (Pipeline under which bin/Clustering.pl groups homologous gene sequences and collapses <=15%-divergent sequences into representatives) — https://github.com/zacherydickson/HUBDesign
- **MAFFT** (Default-supported multiple sequence aligner (7.0+) used on gene families before neighbour-joining tree construction)
- **MUSCLE** (Alternative multiple sequence aligner (3.8.425+) supported for gene-family alignment)
- **CLUSTAL OMEGA** (Alternative multiple sequence aligner (1.2.1+) supported for gene-family alignment)
- **BioPerl** (Perl library (1.7.10+) used to drive alignment and tree handling in the clustering stage)
- **PROKKA** (Suggested annotation tool producing the GFF gene annotations that are input to clustering)

## Examples

```
bin/Clustering.pl --r2t-divergence 0.15 genome_1.gff genome_2.gff > RepSeq.fna
```

## Evaluation signals

- RepSeq.fna is produced with representative sequences covering all 56 input coronavirus taxa (each taxon represented by at least one pseudo-genome downstream)
- Collapsed sequences within each cluster show root-to-tip divergence of at most 15% (respecting --r2t-divergence)
- Each gene family was aligned with a supported aligner version (MAFFT 7.0+, MUSCLE 3.8.425+, or CLUSTAL OMEGA 1.2.1+)
- Cluster composition file is parseable by bin/Assignment.pl without format drift between pipeline phases
- Guide.tree-derived taxa match the taxon-ids in GenomeInfo.tab when --guide-tree is used

## Limitations

- Requires annotated genomes as input; probes cannot be designed for unknown or unannotatable genomes, only close relatives
- The r2t-divergence threshold must be chosen carefully: more divergent baits are less effective, so breadth of coverage trades off against specificity
- Newer HUBDesign versions construct a new cluster-assignment tree based on observed clusters (optionally guided by a user tree), a version discrepancy relative to the validated published version; earlier versions had a cluster-partitioning flaw using nominal taxonomy rather than observed sequence similarity
- Memory and time requirements of the downstream SA_BOND identification step grow quickly with genome size and number of organisms

## Evidence
