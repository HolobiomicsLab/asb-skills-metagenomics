---
name: protein-redundancy-clustering
description: Use when constructing a reference protein database for functional annotation
  (e.g. DIAMOND/BLAST searches) from large multi-source FASTA sets where redundant
  near-identical sequences would inflate search space and ambiguity.
when_to_use_negative:
- Input is already a non-redundant or curated reference (e.g. UniProt/Swiss-Prot reviewed
  subset alone) where clustering would destroy deliberate redundancy-free curation
- Building a taxonomic marker-gene database, where the article instead uses a stricter
  99% identity CD-HIT threshold on single-copy marker genes
- Sequences lack functional annotations and the goal is an annotated functional database
  — unannotated proteins must be discarded first (~1.5 million were in FunOMIC), leaving
  little to cluster
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0621
tools:
- name: CD-HIT
  role: Performs the 95% identity sequence clustering that removes redundancy from
    the annotated fungal protein set to produce FunOMIC-P
- name: FunOMIC
  role: Pipeline context that assembles the combined protein set, applies the annotation
    filter, and emits FunOMIC-P with EC number and KEGG pathway links
  repo: https://github.com/ManichanhLab/FunOMIC
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:01d5765786c3d031b7c348a90be96aeeba781476b95ca535f4bcd17d1a62fd17
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  repo_url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - CD-HIT
  - FunOMIC
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# protein-redundancy-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a non-redundant protein reference database by first discarding proteins lacking explicit functional annotation, then clustering the annotated set with CD-HIT at 95% identity. This is how the FunOMIC-P functional database was produced from genome coding genes plus the fungal UniProt subset.

## When to use

Use when constructing a reference protein database for functional annotation (e.g. DIAMOND/BLAST searches) from large multi-source FASTA sets where redundant near-identical sequences would inflate search space and ambiguity.

## When NOT to use

- Input is already a non-redundant or curated reference (e.g. UniProt/Swiss-Prot reviewed subset alone) where clustering would destroy deliberate redundancy-free curation
- Building a taxonomic marker-gene database, where the article instead uses a stricter 99% identity CD-HIT threshold on single-copy marker genes
- Sequences lack functional annotations and the goal is an annotated functional database — unannotated proteins must be discarded first (~1.5 million were in FunOMIC), leaving little to cluster

## Inputs

- Combined multi-source protein FASTA (genome coding genes from NCBI/JGI plus UniProt reviewed fungal proteins; ~4.9 million sequences)
- Per-protein functional annotations (EC numbers, KEGG pathways) used to filter and link entries

## Outputs

- Non-redundant protein FASTA (FunOMIC-P: 3,413,239 sequences)
- Cluster summary table reporting reduction from the ~4.9 million input genes after 95% identity clustering
- Protein accession-to-function mapping (EC numbers and KEGG pathways)

## How to apply

Combine amino-acid sequences from all contributing sources (in FunOMIC: coding genes from quality-filtered fungal genomes from NCBI/JGI plus 35,360 reviewed UniProt fungal proteins, ~4.9 million genes total). First apply an annotation filter: discard all protein sequences lacking an explicit functional annotation (~1.5 million proteins removed), so the final database links protein accessions to EC numbers and KEGG pathways. Then run CD-HIT with a 95% identity threshold to collapse redundancy, emitting the non-redundant FASTA. Note the deliberate contrast with the taxonomic database (FunOMIC-T), which used a stricter 99% identity CD-HIT clustering on marker genes; the looser 95% threshold is appropriate for the larger, function-oriented protein set.

## Related tools

- **CD-HIT** (Performs the 95% identity sequence clustering that removes redundancy from the annotated fungal protein set to produce FunOMIC-P)
- **FunOMIC** (Pipeline context that assembles the combined protein set, applies the annotation filter, and emits FunOMIC-P with EC number and KEGG pathway links) — https://github.com/ManichanhLab/FunOMIC

## Examples

```
cd-hit -i combined_annotated_proteins.fasta -o FunOMIC-P.fasta -c 0.95
```

## Evaluation signals

- Output FASTA contains exactly 3,413,239 non-redundant fungal protein sequences
- Cluster summary table reports the expected reduction from ~4.9 million input genes (about 1.5 million removed by the annotation filter, further reduction by 95% clustering)
- Every retained protein carries an explicit functional annotation and links to EC numbers and KEGG pathways
- No pair of retained sequences exceeds the 95% identity clustering threshold (spot-check with CD-HIT or an all-against-all alignment on a sample)
- Protein accessions resolve to their source database (JGI, NCBI, or UniProt)

## Limitations

- Proteins without explicit functional annotation must be discarded (~1.5 million in FunOMIC), so the database under-represents hypothetical/uncharacterized proteins
- High inter-kingdom conservation of protein-coding genes means bacterial contamination was not totally preventable in downstream functional annotation, even with strict mapping thresholds
- Clustering at 95% identity may collapse proteins with distinct functions; the alternative 99% threshold used for FunOMIC-T trades redundancy for finer resolution

## Evidence
