---
name: strain-aware-haplotype-contig-merging
description: Use when assembling strain-aware metagenome contigs from cluster-specific
  haplotigs produced by a local assembly stage (StrainXpress stage-a output), needing
  global assembly into final master contigs.
when_to_use_negative:
- Input is raw metagenomic short reads that have not been clustered and locally assembled
  — the Clustering Reads and Local Assembly stages must run first
- Input is long third-generation sequencing reads — the paper notes the high error
  rate of third-generation reads remains an unresolved issue for this overlap-graph
  pipeline
- A reference-based (reference-assisted) assembly is desired — StrainXpress is a de
  novo method and this stage operates on de novo haplotigs
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: StrainXpress
  role: 'Implements the Global Assembly stage: builds the master overlap graph over
    haplotigs, simplifies it, and extends master contigs along unique paths into final_contigs.fasta'
  repo: https://github.com/HaploKit/StrainXpress
- name: minimap2
  role: Computes the pairwise overlaps between cluster-specific haplotigs used as
    edges of the master overlap graph
  repo: https://github.com/lh3/minimap2
- name: HaploConduct
  role: Dependency providing the overlap-graph-based assembly foundation (SAVAGE-style
    methods) on which StrainXpress's global assembly builds
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:b21e2baf136376ebe293eb783904b52da6c7dd79f0d3793281893f16cd3cc74c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - HaploConduct
  - StrainXpress
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# strain-aware-haplotype-contig-merging

## Summary

Reconstruct master strain-level contigs by building a master overlap graph over cluster-specific haplotigs (from StrainXpress local assembly), computing overlaps with minimap2, and simplifying/extending along unique paths. This is StrainXpress's Global Assembly stage that merges haplotigs into final contigs.

## When to use

Use when assembling strain-aware metagenome contigs from cluster-specific haplotigs produced by a local assembly stage (StrainXpress stage-a output), needing global assembly into final master contigs.

## When NOT to use

- Input is raw metagenomic short reads that have not been clustered and locally assembled — the Clustering Reads and Local Assembly stages must run first
- Input is long third-generation sequencing reads — the paper notes the high error rate of third-generation reads remains an unresolved issue for this overlap-graph pipeline
- A reference-based (reference-assisted) assembly is desired — StrainXpress is a de novo method and this stage operates on de novo haplotigs

## Inputs

- Haplotigs (contig sequences) from StrainXpress local assembly of each read cluster (stage-a output)
- minimap2-installed Linux environment with python3, scipy, pandas (per README dependencies)

## Outputs

- final_contigs.fasta in the stageb folder — well-formed FASTA of assembled master (strain-level) contigs

## How to apply

Collect the haplotigs produced by local assembly of each read cluster (stage-a output) as the node sequences for the global assembly. Compute pairwise overlaps between haplotigs using minimap2 and build a master overlap graph, retaining only edges with overlap length >100 bp and identity >=0.99 (thresholds inspired by earlier overlap-graph work). Simplify the graph by removing transitive edges and resolving branch (bifurcation) structures, then iteratively extend contigs only along unique (unambiguous) paths, repeating until no contig can be extended further. Write the resulting master contigs as FASTA to the stageb folder as final_contigs.fasta. The overlap-graph approach is used because patterns of co-occurring mutations—the decisive characteristic of strains—are highlighted optimally with overlap graphs.

## Related tools

- **StrainXpress** (Implements the Global Assembly stage: builds the master overlap graph over haplotigs, simplifies it, and extends master contigs along unique paths into final_contigs.fasta) — https://github.com/HaploKit/StrainXpress
- **minimap2** (Computes the pairwise overlaps between cluster-specific haplotigs used as edges of the master overlap graph) — https://github.com/lh3/minimap2
- **HaploConduct** (Dependency providing the overlap-graph-based assembly foundation (SAVAGE-style methods) on which StrainXpress's global assembly builds) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python ../scripts/strainxpress.py -fq all_reads.fq
```

## Evaluation signals

- final_contigs.fasta exists in the stageb output folder
- final_contigs.fasta parses as a well-formed FASTA file of assembled contig sequences
- Master overlap graph edges all satisfy overlap length >100 bp and identity >=0.99
- Contig extension terminates: iterating extension along unique paths until no contig can be extended further converges rather than cycles
- Assembled contigs increase genome fraction relative to cluster-level haplotigs alone (StrainXpress achieved 93.45% genome fraction vs 74.74% for SPAdes on low complexity data)

## Limitations

- The short length of Illumina-type reads constrains contig length, preventing computation of strain-specific contigs that span entire genomes
- Extension proceeds only along unique (unambiguous) paths, so ambiguous/bifurcating regions are not merged and may remain fragmented
- High error rate of third-generation sequencing reads introduces issues that are not resolved by this approach

## Evidence

- [methods] master overlap graph thresholds: [evidence span withheld — anchor status: not_found]
- [methods] global assembly workflow: [evidence span withheld — anchor status: not_found]
- [other] output location: [evidence span withheld — anchor status: not_found]
- [discussion] overlap graph rationale: "patterns of co-occurring mutations—as the decisive characteristic sequential phenomenon of strains—can be highlighted optimally with overlap graphs"
- [results] stages of the tool: "the parts that refer to the different stages of StrainXpress, ‘Clustering Reads’, ‘Local Assembly’ and ‘Global Assembly’"
- [discussion] read length limitation: "the short length of Illumina type reads puts constraints on the length of the contigs a strain aware assembler can compute"
