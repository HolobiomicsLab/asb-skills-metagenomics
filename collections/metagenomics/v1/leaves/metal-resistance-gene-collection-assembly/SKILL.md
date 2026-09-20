---
name: metal-resistance-gene-collection-assembly
description: Use when building a merged, homology-reduced reference of metal-resistance
  (or stress-type) genes from versioned source databases, or when reproducing the
  PanRes curation counts of 14,078 unique sequences and 5,280 centroids.
when_to_use_negative:
- You need a single-database reference rather than a merged multi-source catalog —
  the PanRes combination merges sources with different scopes, and the article notes
  appropriateness of subsets is left to individual research questions
- Your goal is profiling ARG abundance in metagenomic reads rather than constructing
  the reference gene catalog itself (that is ARGprofiler/KMA's downstream role)
- Source database versions differ from the pinned versions (e.g. a newer CARD or MegaRes
  release), which would make reproduction of the 14,078/5,280 counts unreliable
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0622
tools:
- name: GeneAssimilator
  role: Orchestrates the merge of multi-fasta gene collections via the iterative recruit-cluster-refine
    procedure and outputs combined and homology-reduced catalogs with annotation files
  repo: https://github.com/genomicepidemiology/gene_assimilator
- name: Usearch
  role: Provides fastx_uniques (v11.0.667) to identify unique sequences and cluster_fast
    at 90% identity/90% coverage to produce centroid sequences
- name: ARGprofiler
  role: Consumes the resulting PanRes database (which embeds the metal-resistance
    gene centroids) for read-mapping abundance profiling
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: PanRes
  role: The combined ARG database collection (14,078 unique ARGs) that this assembly
    procedure builds and that includes METAL/BIOCIDE stress-subtype entries
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:059254a49d6736858514b783e6ea6de1f129d8ad3c229e15bd9acb66bc0c9b3c
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
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# metal-resistance-gene-collection-assembly

## Summary

Assembles a combined metal-resistance gene catalog by collecting seed gene sequences from the MetalResistance database (Zenodo doi 10.5281/zenodo.8108201) alongside multiple ARG databases, deduplicating them with Usearch, and homology-reducing them at 90% identity/90% coverage as done for the PanRes collection in ARGprofiler.

## When to use

Use when building a merged, homology-reduced reference of metal-resistance (or stress-type) genes from versioned source databases, or when reproducing the PanRes curation counts of 14,078 unique sequences and 5,280 centroids.

## When NOT to use

- You need a single-database reference rather than a merged multi-source catalog — the PanRes combination merges sources with different scopes, and the article notes appropriateness of subsets is left to individual research questions
- Your goal is profiling ARG abundance in metagenomic reads rather than constructing the reference gene catalog itself (that is ARGprofiler/KMA's downstream role)
- Source database versions differ from the pinned versions (e.g. a newer CARD or MegaRes release), which would make reproduction of the 14,078/5,280 counts unreliable

## Inputs

- MetalResistance database sequences (Zenodo doi 10.5281/zenodo.8108201)
- Seed multi-fasta gene collections: ResFinder (2023-01-20), ResFinderFG v2.0, CARD v3.2.5, MegaRes v3.0.0, AMRFinderPlus v3.11/2022-12-19.1, ARGANNOT (V6_July2019)
- Database metadata/annotation fields used for filtering (e.g. CARD model type, MegaRes RequiresSNPConfirmation tags, AMRFinderPlus type/subtype labels)

## Outputs

- Combined catalog of unique gene sequences (expected ~14,078 unique ARGs including METAL/BIOCIDE stress genes)
- Homology-reduced centroid sequences clustered at 90% identity / 90% coverage (expected ~5,280 centroids)
- Curation summary table (e.g. panres_curation_summary.tsv) with unique-input count, unique-sequence count, centroid count, and centroid length min/max/median

## How to apply

Download the seed gene collections at the paper's pinned versions: ResFinder (20 January 2023), ResFinderFG v2.0, CARD v3.2.5, MegaRes v3.0.0, AMRFinderPlus v3.11/2022-12-19.1, ARGANNOT (V6_July2019), and the MetalResistance database at Zenodo doi 10.5281/zenodo.8108201. Apply the inclusion filters — keep only CARD protein-homolog-model genes, drop MegaRes entries tagged 'RequiresSNPConfirmation', keep only AMRFinderPlus 'AMR' subtype entries plus 'BIOCIDE' and 'METAL' subtypes of the 'STRESS' type (the METAL subset is where this skill's metal-resistance genes come from). Merge all sequences into multi-fasta input for GeneAssimilator, which drives Usearch 11.0.667's fastx_uniques in an iterative recruit-cluster-refine procedure to identify 14,078 unique sequences. Then cluster the unique genes with Usearch cluster_fast at 90% identity and 90% coverage to obtain 5,280 centroid sequences; record centroid length statistics (median 762 bp, range 93–5,972 bp) in a summary table such as panres_curation_summary.tsv.

## Related tools

- **GeneAssimilator** (Orchestrates the merge of multi-fasta gene collections via the iterative recruit-cluster-refine procedure and outputs combined and homology-reduced catalogs with annotation files) — https://github.com/genomicepidemiology/gene_assimilator
- **Usearch** (Provides fastx_uniques (v11.0.667) to identify unique sequences and cluster_fast at 90% identity/90% coverage to produce centroid sequences)
- **ARGprofiler** (Consumes the resulting PanRes database (which embeds the metal-resistance gene centroids) for read-mapping abundance profiling) — https://github.com/genomicepidemiology/ARGprofiler
- **PanRes** (The combined ARG database collection (14,078 unique ARGs) that this assembly procedure builds and that includes METAL/BIOCIDE stress-subtype entries) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
python gene_assimilator.py --input_dir seed_fastas/ --output_dir panres_out/ && usearch -cluster_fast panres_out/unique.fasta -id 0.90 -cover 0.90 -centroids centroids.fasta
```

## Evaluation signals

- Unique-sequence count after fastx_uniques falls within a small tolerance of the reported 14,078
- Centroid count after cluster_fast at 90% identity/90% coverage falls within a small tolerance of the reported 5,280
- Median centroid length is close to the reported 762 bp, with lengths spanning the reported 93–5,972 bp range
- panres_curation_summary.tsv exists and contains the expected fields (unique input genes, unique sequences, centroids, length statistics)
- Filtered inputs correctly exclude non-protein-homolog CARD genes, 'RequiresSNPConfirmation' MegaRes sequences, and non-AMR/non-STRESS AMRFinderPlus entries

## Limitations

- Counts depend on exact database versions; any version drift in ResFinder, CARD, MegaRes, AMRFinderPlus, or ARGANNOT will change the unique/centroid counts
- Usearch must be present in the PATH and is a licensed binary; GeneAssimilator also requires R packages optparse, tidyverse, and seqinr
- PanRes combines sources with different scopes, so the metal-resistance subset may not be exhaustive or uniformly curated across databases

## Evidence

- [abstract] providing the PanRes database of 14 078 unique ARGs that combines several existing collections into one: "providing the PanRes database of 14 078 unique ARGs that combines several existing collections into one"
- [discussion] it combines different sources and scopes, it will be up to the individual research questions, which subsets are appropriate: "it combines different sources and scopes, it will be up to the individual research questions, which subsets are appropriate for consideration"
- [readme] The three first letter in each input fasta file will be used as a prefix for some of the output.: [evidence span withheld — anchor status: external_doc]
- [readme] You need to have usearch in your path: [evidence span withheld — anchor status: external_doc]
