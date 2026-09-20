---
name: gene-database-inclusion-filtering
description: Use when combining multiple reference gene collections (e.g. ResFinder,
  CARD, MegaRes, AMRFinderPlus, ARGANNOT, ResFinderFG) into one non-redundant database
  and you need reproducible source-specific inclusion filters plus identity/coverage-based
  clustering.
when_to_use_negative:
- Input is a single homogeneous database or an already non-redundant gene set — the
  merge/deduplicate/centroid pipeline adds no value
- You need a curated subset chosen by biological judgment rather than type/subtype
  filters; the paper notes that because PanRes combines different sources and scopes,
  'it will be up to the individual research questions, which subsets are appropriate
  for consideration'
- The workflow targets long reads or non-ARG gene families without their own metadata
  fields for the CARD/MegaRes/AMRFinderPlus filters
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: GeneAssimilator
  role: Iterative recruit–cluster–refine procedure that ingests multiple multi-FASTA
    gene collections, runs Usearch fastx_uniques, and outputs the merged catalog and
    homology-reduced centroids
  repo: https://github.com/genomicepidemiology/gene_assimilator
- name: Usearch
  role: fastx_uniques (v11.0.667) identifies unique sequences among merged genes;
    cluster_fast performs the 90%-identity/90%-coverage centroid clustering
- name: PanRes
  role: The combined ARG database produced by this filtering/clustering procedure
    and shipped inside ARGprofiler
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:c456b8fea0c36dfd523402f4895a6979c0ce3fee07a5f5a6f24f22bd0c038d98
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# gene-database-inclusion-filtering

## Summary

Merge heterogeneous antimicrobial resistance gene (ARG) databases into a single curated collection by applying source-specific inclusion filters, collapsing redundant sequences with Usearch fastx_uniques, and homology-reducing the result into centroids clustered at 90% identity and 90% coverage — the procedure used to build the PanRes database (30,400 genes → 14,078 unique sequences → 5,280 centroids).

## When to use

Use when combining multiple reference gene collections (e.g. ResFinder, CARD, MegaRes, AMRFinderPlus, ARGANNOT, ResFinderFG) into one non-redundant database and you need reproducible source-specific inclusion filters plus identity/coverage-based clustering.

## When NOT to use

- Input is a single homogeneous database or an already non-redundant gene set — the merge/deduplicate/centroid pipeline adds no value
- You need a curated subset chosen by biological judgment rather than type/subtype filters; the paper notes that because PanRes combines different sources and scopes, 'it will be up to the individual research questions, which subsets are appropriate for consideration'
- The workflow targets long reads or non-ARG gene families without their own metadata fields for the CARD/MegaRes/AMRFinderPlus filters

## Inputs

- Multi-FASTA seed gene files from ResFinder, ResFinderFG v2.0, CARD v3.2.5, MegaRes v3.0.0, AMRFinderPlus v3.11/2022-12-19.1, ARGANNOT (V6_July2019)
- MetalResistance database from Zenodo doi 10.5281/zenodo.8108201
- Per-database metadata needed for filtering (e.g. CARD model type, MegaRes 'RequiresSNPConfirmation' tags, AMRFinderPlus type/subtype fields)

## Outputs

- Unique-sequence FASTA (14,078 unique ARGs from 30,400 input genes in the reference run)
- Centroid FASTA from 90%-identity/90%-coverage clustering (5,280 centroids; median length 762 bp, range 93–5,972 bp)
- Summary table (e.g. panres_curation_summary.tsv) with counts of input genes, unique sequences, centroids, and centroid length min/max/median

## How to apply

Download each seed gene collection at pinned versions — ResFinder (20 January 2023), ResFinderFG v2.0, CARD v3.2.5, MegaRes v3.0.0, AMRFinderPlus v3.11/2022-12-19.1, ARGANNOT (V6_July2019), plus the MetalResistance database (Zenodo doi 10.5281/zenodo.8108201). Apply the paper's per-source inclusion filters: keep only CARD genes based on the protein homolog model, exclude MegaRes sequences tagged 'RequiresSNPConfirmation', keep only AMRFinderPlus 'AMR' subtype entries plus only 'BIOCIDE' and 'METAL' subtypes of the 'STRESS' type. Merge the retrieved multi-FASTA files and feed them through GeneAssimilator's iterative recruit–cluster–refine procedure, which uses Usearch 11.0.667 fastx_uniques to identify unique sequences. Homology-reduce the unique ARGs with Usearch cluster_fast at 90% identity and 90% coverage to produce centroid sequences. Record the number of input genes, unique sequences, centroids, and centroid length statistics to compare against the reported PanRes values.

## Related tools

- **GeneAssimilator** (Iterative recruit–cluster–refine procedure that ingests multiple multi-FASTA gene collections, runs Usearch fastx_uniques, and outputs the merged catalog and homology-reduced centroids) — https://github.com/genomicepidemiology/gene_assimilator
- **Usearch** (fastx_uniques (v11.0.667) identifies unique sequences among merged genes; cluster_fast performs the 90%-identity/90%-coverage centroid clustering)
- **PanRes** (The combined ARG database produced by this filtering/clustering procedure and shipped inside ARGprofiler) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
usearch -cluster_fast unique_seqs.fasta -id 0.90 -cov 0.90 -centroids centroids.fasta
```

## Evaluation signals

- Unique-sequence count falls within a small tolerance of the reported 14,078 unique sequences from 30,400 input genes
- Centroid count after cluster_fast at -id 0.90 -cov 0.90 is close to the reported 5,280 centroids
- Median centroid length is near the reported 762 bp, with lengths spanning the reported 93–5,972 bp range
- Every excluded CARD entry lacked a protein homolog model, no MegaRes 'RequiresSNPConfirmation' sequence survived filtering, and only AMRFinderPlus 'AMR' plus 'BIOCIDE'/'METAL' 'STRESS' subtypes remain
- The summary output file (e.g. panres_curation_summary.tsv) exists and contains the unique-sequence count, centroid count, and length statistic fields

## Limitations

- ARGprofiler/PanRes combines sources with different scopes, so choosing the appropriate subset for a given question is left to the user
- Results are sensitive to the pinned database versions (e.g. CARD v3.2.5, AMRFinderPlus 2022-12-19.1); newer downloads will change counts
- The cluster_fast 90%-identity/90%-coverage thresholds are fixed choices; other resistance-gene applications may need different clustering stringency

## Evidence

- [abstract] PanRes database contains 14,078 unique ARGs combined from several existing collections: "providing the PanRes database of 14 078 unique ARGs that combines several existing collections into one"
- [discussion] Subset appropriateness is left to the user because PanRes mixes sources and scopes: "it combines different sources and scopes, it will be up to the individual research questions, which subsets are appropriate for consideration"
- [readme] GeneAssimilator README describes the merge-and-cluster motivation this skill implements: [evidence span withheld — anchor status: external_doc]
- [readme] GeneAssimilator requires Usearch in PATH and R packages for the clustering workflow: [evidence span withheld — anchor status: external_doc]
