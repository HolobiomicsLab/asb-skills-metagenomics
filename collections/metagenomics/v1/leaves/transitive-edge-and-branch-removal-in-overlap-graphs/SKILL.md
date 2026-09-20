---
name: transitive-edge-and-branch-removal-in-overlap-graphs
description: Use when haplotigs produced by local assembly of each read cluster must
  be merged into master contigs via a master overlap graph, and the graph contains
  transitive edges or branch (bifurcation) nodes that block unique-path contig extension.
when_to_use_negative:
- Input is raw metagenomic short reads rather than stage-a haplotigs — reads must
  first pass StrainXpress 'Clustering Reads' and 'Local Assembly' stages before the
  master overlap graph exists
- Assembly of high-error-rate third-generation sequencing reads, whose error rates
  introduce unresolved issues in the overlap-graph paradigm
- When an existing per-cluster assembly already represents the final output and no
  cross-cluster contig merging is required
edam_operation: http://edamontology.org/operation_0526
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3308
tools:
- name: StrainXpress
  role: 'Implements the global assembly stage: builds the master overlap graph over
    stage-a haplotigs, removes transitive edges, resolves branches, and extends master
    contigs along unique paths'
  repo: https://github.com/HaploKit/StrainXpress
- name: minimap2
  role: Computes the pairwise overlaps between haplotigs that form the edges of the
    master overlap graph (retained only if overlap >100 bp at identity >=0.99)
  repo: https://github.com/lh3/minimap2
- name: HaploConduct
  role: Dependency of StrainXpress whose overlap-graph-based haplotype reconstruction
    (SAVAGE) inspired the graph construction thresholds used in global assembly
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:b849438e9da2f7e43292a32810fe1bf9bebf46a0cae262a1da49680972975333
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - HaploConduct
  - StrainXpress
  - minimap2
  edam_operation: http://edamontology.org/operation_0526
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# transitive-edge-and-branch-removal-in-overlap-graphs

## Summary

Simplifies a master overlap graph built over cluster-specific haplotigs during StrainXpress's Global Assembly stage by removing transitive edges and resolving bifurcation (branch) structures, so that master contigs can be extended iteratively along unique paths into final strain-aware contigs.

## When to use

Use when haplotigs produced by local assembly of each read cluster must be merged into master contigs via a master overlap graph, and the graph contains transitive edges or branch (bifurcation) nodes that block unique-path contig extension.

## When NOT to use

- Input is raw metagenomic short reads rather than stage-a haplotigs — reads must first pass StrainXpress 'Clustering Reads' and 'Local Assembly' stages before the master overlap graph exists
- Assembly of high-error-rate third-generation sequencing reads, whose error rates introduce unresolved issues in the overlap-graph paradigm
- When an existing per-cluster assembly already represents the final output and no cross-cluster contig merging is required

## Inputs

- Stage-a haplotigs (cluster-specific contig sequences) from StrainXpress local assembly of each read cluster
- Pairwise overlap records between haplotigs computed with minimap2 (overlap length and identity per edge)

## Outputs

- Simplified master overlap graph (transitive edges removed, branch structures resolved)
- final_contigs.fasta — master contigs written as FASTA into the stageb output folder

## How to apply

Collect the haplotigs output by StrainXpress stage-a local assembly as the node sequences of the master overlap graph. Compute pairwise overlaps between haplotigs with minimap2 and retain only edges with overlap length >100 bp at identity >=0.99, thresholds inspired by earlier overlap-graph work. Simplify the graph by removing transitive edges (edges implied by longer composite paths) and resolving branch (bifurcation) structures, since only unambiguous topology allows deterministic extension. Iteratively extend contigs along unique (unambiguous) paths, repeating until no contig can be extended further. Write the resulting master contigs as FASTA to the stageb folder as final_contigs.fasta. The rationale is that overlap graphs highlight co-occurring strain-specific mutation patterns, but only after transitive and branching edges are pruned can strain-resolved master contigs be traversed unambiguously.

## Related tools

- **StrainXpress** (Implements the global assembly stage: builds the master overlap graph over stage-a haplotigs, removes transitive edges, resolves branches, and extends master contigs along unique paths) — https://github.com/HaploKit/StrainXpress
- **minimap2** (Computes the pairwise overlaps between haplotigs that form the edges of the master overlap graph (retained only if overlap >100 bp at identity >=0.99)) — https://github.com/lh3/minimap2
- **HaploConduct** (Dependency of StrainXpress whose overlap-graph-based haplotype reconstruction (SAVAGE) inspired the graph construction thresholds used in global assembly) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python ../scripts/strainxpress.py -fq all_reads.fq
```

## Evaluation signals

- Every retained edge in the master overlap graph has overlap length >100 bp and identity >=0.99
- No transitive edges remain: any edge implied by a longer composite path through intermediate haplotig nodes has been removed
- No unresolved bifurcations remain: contig extension proceeds only along unique (unambiguous) paths and the iterative extension terminates when no contig can be extended further
- final_contigs.fasta exists in the stageb output folder and parses as a well-formed FASTA file of assembled contig sequences
- Master contigs represent merges of haplotigs from multiple read clusters, consistent with the overlap-layout-consensus (OLC) design of StrainXpress

## Limitations

- Branch (bifurcation) structures cannot be traversed — extension stops at ambiguities, so strain-specific contigs cannot always span complete genomes
- The short length of Illumina-type reads constrains contig length, preventing full-length strain-specific contigs even after graph simplification
- The high error rate of third-generation sequencing reads introduces a range of issues that still need to be resolved for this overlap-graph approach

## Evidence

- [results] Global Assembly stage and its position in the StrainXpress pipeline: "the parts that refer to the different stages of StrainXpress, ‘Clustering Reads’, ‘Local Assembly’ and ‘Global Assembly’"
- [discussion] Rationale for using overlap graphs to capture strain-specific sequence patterns: "patterns of co-occurring mutations—as the decisive characteristic sequential phenomenon of strains—can be highlighted optimally with overlap graphs"
- [readme] StrainXpress is an OLC-based de novo assembler whose global assembly builds master contigs from overlap graphs: "StrainXpress is a de novo assembly method which base on overlap-layout-consensus (OLC) paradigm and can fast and accurately assemble high complexity metagenome sequencing data at strain resolution."
- [readme] Precedent for overlap-graph node/edge semantics that inspired the master graph thresholds: [evidence span withheld — anchor status: external_doc]
