---
name: usearch-sequence-deduplication-and-clustering
description: Use when merging multiple gene reference collections (e.g. ResFinder,
  CARD, MegaRes, AMRFinderPlus, ARGANNOT) into one catalog and you need unique sequences
  plus homology-reduced centroids.
when_to_use_negative:
- You need read-level profiling of metagenomic samples against existing references
  — use ARGprofiler/KMA mapping instead of rebuilding the catalog.
- Your sequences require exact full-length identity handling or long-read assembly
  context — ARGprofiler only supports short-read data and PanRes is a static combined
  collection.
- You need strain-level resolution below 90% identity, since 90/90 clustering collapses
  sequences into centroids.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: Usearch
  role: Performs fastx_uniques deduplication and cluster_fast clustering at 90% identity
    / 90% coverage to build centroids
- name: GeneAssimilator
  role: Orchestrates the iterative recruit-cluster-refine procedure that merges input
    FASTA collections and invokes Usearch
  repo: https://github.com/genomicepidemiology/gene_assimilator
- name: PanRes
  role: The combined ARG database produced by this deduplication-and-clustering procedure
    (14,078 unique ARGs)
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:8423259a3e45400b11311f8fb0498dc7125cb781650f60604d0175fed14c5806
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomicepidemiology/gene_assimilator
  tools:
  - ARGprofiler
  - GeneAssimilator
  - GeneAssimiliator
  - PanRes
  - Usearch
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# usearch-sequence-deduplication-and-clustering

## Summary

Deduplicate and cluster combined gene collections with Usearch 11.0.667 (fastx_uniques for deduplication, cluster_fast at 90% identity / 90% coverage for centroid extraction) as used to build the PanRes ARG database from ~30,400 genes down to 14,078 unique sequences and 5,280 centroids.

## When to use

Use when merging multiple gene reference collections (e.g. ResFinder, CARD, MegaRes, AMRFinderPlus, ARGANNOT) into one catalog and you need unique sequences plus homology-reduced centroids.

## When NOT to use

- You need read-level profiling of metagenomic samples against existing references — use ARGprofiler/KMA mapping instead of rebuilding the catalog.
- Your sequences require exact full-length identity handling or long-read assembly context — ARGprofiler only supports short-read data and PanRes is a static combined collection.
- You need strain-level resolution below 90% identity, since 90/90 clustering collapses sequences into centroids.

## Inputs

- Multi-FASTA files of gene sequences from source ARG databases (ResFinder, ResFinderFG v2.0, CARD v3.2.5, MegaRes v3.0.0, AMRFinderPlus v3.11/2022-12-19.1, ARGANNOT V6_July2019)
- MetalResistance gene collection (Zenodo doi 10.5281/zenodo.8108201)
- Usearch 11.0.667 binary available in PATH

## Outputs

- FASTA of unique sequences (14,078 unique ARGs in PanRes)
- FASTA of 90%-identity/90%-coverage centroid sequences (5,280 centroids in PanRes)
- Summary table of counts and centroid length statistics (panres_curation_summary.tsv with min, max, median length: 93–5,972 bp, median 762 bp)

## How to apply

Gather the source multi-FASTA gene collections (ResFinder 20 Jan 2023, ResFinderFG v2.0, CARD v3.2.5, MegaRes v3.0.0, AMRFinderPlus v3.11/2022-12-19.1, ARGANNOT V6_July2019), applying database-specific inclusion filters first (CARD protein homolog model only, exclude MegaRes 'RequiresSNPConfirmation', AMRFinderPlus 'AMR' plus 'BIOCIDE'/'METAL' STRESS subtypes). Merge the sequences and run Usearch 11.0.667 fastx_uniques to identify unique sequences, using GeneAssimilator's iterative recruit-cluster-refine procedure. Then cluster the unique ARGs with Usearch cluster_fast at 90% identity and 90% coverage to obtain centroid sequences. The rationale is that source databases overlap heavily, so deduplication and 90/90 clustering produce a non-redundant combined catalog (PanRes: 14,078 unique sequences, 5,280 centroids, median centroid length 762 bp, range 93–5,972 bp) suitable for read mapping in ARGprofiler.

## Related tools

- **Usearch** (Performs fastx_uniques deduplication and cluster_fast clustering at 90% identity / 90% coverage to build centroids)
- **GeneAssimilator** (Orchestrates the iterative recruit-cluster-refine procedure that merges input FASTA collections and invokes Usearch) — https://github.com/genomicepidemiology/gene_assimilator
- **PanRes** (The combined ARG database produced by this deduplication-and-clustering procedure (14,078 unique ARGs)) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
usearch11.0.667 -fastx_uniques merged_genes.fa -fastaout unique_args.fa; usearch11.0.667 -cluster_fast unique_args.fa -id 0.90 -cover 0.90 -centroids panres_centroids.fa
```

## Evaluation signals

- Unique-sequence count falls within a small tolerance of the reported 14,078 unique sequences.
- Centroid count falls within a small tolerance of the reported 5,280 centroids at 90% identity / 90% coverage.
- Median centroid length is close to the reported 762 bp, with lengths in the 93–5,972 bp range.
- The summary file (panres_curation_summary.tsv) exists and contains fields for unique input genes, unique sequences, centroid count, and centroid length statistics.
- No centroid should be identical to another after clustering (homology reduction succeeded).

## Limitations

- Usearch must be in PATH and is not open-source (license download required per the GeneAssimilator README dependencies: usearch plus R packages optparse, tidyverse, seqinr).
- PanRes combines sources with different scopes, so the appropriateness of using subsets is left to individual research questions.
- Only the first three letters of each input FASTA filename are used as an output prefix, which can cause collisions if source filenames share prefixes.

## Evidence

- [abstract] providing the PanRes database of 14 078 unique ARGs that combines several existing collections into one: "providing the PanRes database of 14 078 unique ARGs that combines several existing collections into one"
- [discussion] it combines different sources and scopes, it will be up to the individual research questions, which subsets are appropriate: "it combines different sources and scopes, it will be up to the individual research questions, which subsets are appropriate for consideration"
- [readme] You need to have usearch in your path (https://www.drive5.com/usearch/download.html).: "You need to have usearch in your path (https://www.drive5.com/usearch/download.html).
We also need a couple of R packages: optparse, tidyverse, seqinr."
- [readme] The three first letter in each input fasta file will be used as a prefix for some of the output.: [evidence span withheld — anchor status: external_doc]
