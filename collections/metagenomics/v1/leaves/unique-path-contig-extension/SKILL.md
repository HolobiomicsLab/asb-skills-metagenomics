---
name: unique-path-contig-extension
description: Use when assembling strain-resolved master contigs from cluster-specific
  haplotigs produced by local assembly, e.g. merging per-cluster contigs into final_contigs.fasta
  in a de novo strain-aware metagenome assembly pipeline.
when_to_use_negative:
- Input is raw short reads rather than cluster-specific haplotigs — the clustering
  and local assembly stages must run first
- Overlaps between contigs fall below the >100 bp / >=0.99 identity thresholds, leaving
  ambiguous or absent edges — no unique-path extension is possible
- A strain-aware assembler is not needed, e.g. isolates or a single reference genome,
  where standard assemblers suffice
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: StrainXpress
  role: 'Implements the global assembly: builds the master overlap graph from stage-a
    haplotigs and extends contigs along unique paths to produce final_contigs.fasta'
  repo: https://github.com/HaploKit/StrainXpress
- name: minimap2
  role: Computes pairwise overlaps between haplotigs used as edges of the master overlap
    graph
  repo: https://github.com/lh3/minimap2
- name: HaploConduct
  role: Dependency of StrainXpress; its overlap-graph-based assembly heritage (SAVAGE)
    inspired the overlap-graph approach
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:d187f7834fab6c1629c6500372587ed7f766bb61f3012c9499e85884165d1e63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - HaploConduct
  - StrainXpress
  - minimap2
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# unique-path-contig-extension

## Summary

The Global Assembly stage of StrainXpress, which combines cluster-specific haplotigs (stage-a output) into master contigs via a master overlap graph and iteratively extends contigs only along unique (unambiguous) paths.

## When to use

Use when assembling strain-resolved master contigs from cluster-specific haplotigs produced by local assembly, e.g. merging per-cluster contigs into final_contigs.fasta in a de novo strain-aware metagenome assembly pipeline.

## When NOT to use

- Input is raw short reads rather than cluster-specific haplotigs — the clustering and local assembly stages must run first
- Overlaps between contigs fall below the >100 bp / >=0.99 identity thresholds, leaving ambiguous or absent edges — no unique-path extension is possible
- A strain-aware assembler is not needed, e.g. isolates or a single reference genome, where standard assemblers suffice

## Inputs

- Haplotigs (cluster-specific contigs) from stage-a local assembly of each read cluster
- minimap2 pairwise overlap alignments between haplotigs

## Outputs

- final_contigs.fasta in the stageb folder: master contigs assembled from unambiguous overlap-graph paths

## How to apply

Collect the haplotigs produced by local assembly of each read cluster (stage-a output) as the node sequences of a master overlap graph. Compute pairwise overlaps between haplotigs with minimap2 and retain only edges with overlap length >100 bp and identity >=0.99, thresholds inspired by earlier overlap-graph work. Simplify the graph by removing transitive edges and resolving branch (bifurcation) structures. Iteratively extend contigs only along unique (unambiguous) paths, repeating until no contig can be extended further, then write the resulting master contigs as FASTA to the stageb folder as final_contigs.fasta.

## Related tools

- **StrainXpress** (Implements the global assembly: builds the master overlap graph from stage-a haplotigs and extends contigs along unique paths to produce final_contigs.fasta) — https://github.com/HaploKit/StrainXpress
- **minimap2** (Computes pairwise overlaps between haplotigs used as edges of the master overlap graph) — https://github.com/lh3/minimap2
- **HaploConduct** (Dependency of StrainXpress; its overlap-graph-based assembly heritage (SAVAGE) inspired the overlap-graph approach) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python ../scripts/strainxpress.py -fq all_reads.fq
```

## Evaluation signals

- final_contigs.fasta exists in the stageb output folder and is a well-formed FASTA file of assembled contig sequences
- All retained overlap edges satisfy overlap length >100 bp and identity >=0.99
- Graph simplification removed transitive edges and resolved branch (bifurcation) structures before extension
- Extension terminated: iterating again yields no contig that can be extended further along a unique path
- Master contigs cover sequence from multiple cluster-specific haplotigs, indicating successful merging across clusters

## Limitations

- The short length of Illumina-type reads constrains contig length, preventing full-length strain-specific contigs that span genomes
- Only unique (unambiguous) paths are extended; ambiguous regions with branching overlaps terminate extension
- Thresholds (>100 bp, >=0.99 identity) were tuned for short-read strain-aware assembly; the high error rate of third-generation sequencing reads remains unresolved

## Evidence

- [results] overlap thresholds: [evidence span withheld — anchor status: not_found]
- [results] global assembly stage: "the different stages of StrainXpress, ‘Clustering Reads’, ‘Local Assembly’ and ‘Global Assembly’"
- [readme] OLC paradigm: "StrainXpress is a de novo assembly method which base on overlap-layout-consensus (OLC) paradigm and can fast and accurately assemble high complexity metagenome"
- [discussion] illumina read limitation: "the short length of Illumina type reads puts constraints on the length of the contigs a strain aware assembler can compute."
