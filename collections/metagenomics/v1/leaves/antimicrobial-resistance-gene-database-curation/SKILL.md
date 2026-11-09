---
name: antimicrobial-resistance-gene-database-curation
description: Use when merging several existing antimicrobial resistance (or biocide/metal
  resistance) gene catalogs into one non-redundant curated database, and you need
  reproducible filtering, deduplication, and centroid counts.
when_to_use_negative:
- You already have a curated, non-redundant ARG database and only need to profile
  reads against it — use ARGprofiler's KMA-based abundance profiling instead of re-curating.
- You need to process long-read sequencing data — ARGprofiler is designed to only
  work with short-read sequencing data.
- You need a single-source database specific to one scope — PanRes combines sources
  with different scopes, and which subsets are appropriate depends on the individual
  research question.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: GeneAssimilator
  role: Collects, merges, and clusters multi-FASTA gene catalogs into a combined,
    homology-reduced output database via an iterative recruit-cluster-refine procedure
    using Usearch
  repo: https://github.com/genomicepidemiology/gene_assimilator
- name: Usearch
  role: Identifies unique sequences with fastx_uniques (v11.0.667) and produces centroids
    with cluster_fast at 90% identity and 90% coverage
- name: PanRes
  role: The resulting combined ARG database produced by this curation workflow and
    shipped with ARGprofiler
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: ARGprofiler
  role: Snakemake pipeline that includes the PanRes database for downstream ARG abundance
    and flanking-region analysis
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:32d619ed1508c31026303f00dc529a7299d4ce55234575b1959b31184a0ec160
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# antimicrobial-resistance-gene-database-curation

## Summary

Build a combined ARG reference database (like PanRes) by downloading multiple source gene collections (ResFinder, ResFinderFG v2.0, CARD v3.2.5, MegaRes v3.0.0, AMRFinderPlus, ARGANNOT, MetalResistance), applying source-specific inclusion filters, deduplicating with Usearch fastx_uniques, and homology-reducing with Usearch cluster_fast at 90% identity and 90% coverage.

## When to use

Use when merging several existing antimicrobial resistance (or biocide/metal resistance) gene catalogs into one non-redundant curated database, and you need reproducible filtering, deduplication, and centroid counts.

## When NOT to use

- You already have a curated, non-redundant ARG database and only need to profile reads against it — use ARGprofiler's KMA-based abundance profiling instead of re-curating.
- You need to process long-read sequencing data — ARGprofiler is designed to only work with short-read sequencing data.
- You need a single-source database specific to one scope — PanRes combines sources with different scopes, and which subsets are appropriate depends on the individual research question.

## Inputs

- Seed gene collections in multi-FASTA format: ResFinder (2023-01-20), ResFinderFG v2.0, CARD v3.2.5, MegaRes v3.0.0, AMRFinderPlus v3.11/2022-12-19.1, ARGANNOT (V6_July2019)
- MetalResistance database from Zenodo doi 10.5281/zenodo.8108201
- Source-specific metadata needed for filtering (CARD model type, MegaRes tags, AMRFinderPlus element subtype/type fields)

## Outputs

- Merged set of collected ARG genes (30,400 genes from the source databases)
- FASTA of unique sequences (14,078 unique ARGs in PanRes)
- Centroid FASTA from 90% identity / 90% coverage clustering (5,280 centroids)
- Curation summary table, e.g. panres_curation_summary.tsv with unique-gene count, unique-sequence count, centroid count, and centroid length statistics (median 762 bp, range 93–5,972 bp)

## How to apply

Download the seed gene collections at pinned versions (e.g. ResFinder as of 20 January 2023, ResFinderFG v2.0, CARD v3.2.5, MegaRes v3.0.0, AMRFinderPlus v3.11/2022-12-19.1, ARGANNOT V6_July2019, plus MetalResistance from Zenodo doi 10.5281/zenodo.8108201). Apply per-source inclusion filters: keep only CARD genes under the protein homolog model, exclude MegaRes sequences tagged 'RequiresSNPConfirmation', keep only AMRFinderPlus 'AMR' subtype entries and only the 'BIOCIDE' and 'METAL' subtypes of the 'STRESS' type. Merge all retrieved multi-FASTA sequences and identify unique sequences with Usearch 11.0.667 fastx_uniques via GeneAssimilator's iterative recruit-cluster-refine procedure. Homology-reduce the unique ARGs with Usearch cluster_fast at 90% identity and 90% coverage to obtain centroid sequences, then compute counts and centroid length statistics (min, max, median) and write the summary (e.g. panres_curation_summary.tsv).

## Related tools

- **GeneAssimilator** (Collects, merges, and clusters multi-FASTA gene catalogs into a combined, homology-reduced output database via an iterative recruit-cluster-refine procedure using Usearch) — https://github.com/genomicepidemiology/gene_assimilator
- **Usearch** (Identifies unique sequences with fastx_uniques (v11.0.667) and produces centroids with cluster_fast at 90% identity and 90% coverage)
- **PanRes** (The resulting combined ARG database produced by this curation workflow and shipped with ARGprofiler) — https://github.com/genomicepidemiology/ARGprofiler
- **ARGprofiler** (Snakemake pipeline that includes the PanRes database for downstream ARG abundance and flanking-region analysis) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
usearch -cluster_fast unique_arg.fa -id 0.90 -cover 0.90 -centroids centroids.fa; usearch -fastx_uniques merged_arg.fa -fastaout unique_arg.fa
```

## Evaluation signals

- Unique-sequence count falls within a small tolerance of the reported 14,078 unique ARGs
- Centroid count at 90% identity / 90% coverage falls within a small tolerance of the reported 5,280 centroids
- Median centroid length matches the reported 762 bp, with lengths spanning the reported range 93–5,972 bp
- panres_curation_summary.tsv exists and contains the unique-gene count, unique-sequence count, centroid count, and length statistics fields
- All source-specific filters were applied: CARD entries restricted to the protein homolog model, MegaRes 'RequiresSNPConfirmation' sequences excluded, AMRFinderPlus limited to 'AMR' plus 'BIOCIDE'/'METAL' 'STRESS' subtypes

## Limitations

- PanRes combines databases with different sources and scopes, so it is up to the individual research question which subsets are appropriate for consideration
- Results depend on pinned database versions; downloading different versions of ResFinder, CARD, MegaRes, AMRFinderPlus, or ARGANNOT will change the counts
- Reproduced counts depend on the Usearch version (11.0.667) and the exact clustering thresholds (90% identity, 90% coverage), so deviations in parameters shift the centroid count

## Evidence

- [abstract] providing the PanRes database of 14 078 unique ARGs that combines several existing collections into one: "providing the PanRes database of 14 078 unique ARGs that combines several existing collections into one"
- [intro] ARGprofiler includes the PanRes database, a combined collection of current ARG databases: "ARGprofiler includes the PanRes database, a combined collection of current ARG databases"
- [discussion] it combines different sources and scopes, it will be up to the individual research questions, which subsets are appropriate for consideration: "it combines different sources and scopes, it will be up to the individual research questions, which subsets are appropriate for consideration"
- [readme] You need to have usearch in your path: [evidence span withheld — anchor status: external_doc]
