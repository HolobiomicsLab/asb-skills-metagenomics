---
name: sequence-length-distribution-statistics
description: Use when a merged gene catalog (e.g. ARG sequences from ResFinder, CARD,
  MegaRes, AMRFinderPlus, ARGANNOT) has been deduplicated and clustered, and you must
  report or validate the resulting sequence-count and centroid-length statistics.
when_to_use_negative:
- Input is raw sequencing reads rather than a curated multi-fasta gene catalog — length
  statistics of reads do not validate database curation.
- The clustering thresholds differ from the PanRes parameters (90% identity / 90%
  coverage); counts and length distributions will not be comparable to the reported
  14,078 / 5,280 / 762 bp values.
- The goal is profiling ARG abundance in metagenomic samples (that is ARGprofiler's
  role) rather than characterizing the reference database itself.
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3071
tools:
- name: Usearch
  role: fastx_uniques identifies unique sequences in the merged catalog; cluster_fast
    at 90% identity / 90% coverage produces the centroid set whose length distribution
    is summarized
  repo: https://www.drive5.com/usearch/download.html
- name: GeneAssimilator
  role: Runs the iterative recruit–cluster–refine procedure that wraps Usearch to
    merge input gene collections and emit the combined/homology-reduced catalog being
    summarized
  repo: https://github.com/genomicepidemiology/gene_assimilator
- name: PanRes
  role: The combined ARG database (14,078 unique ARGs) whose centroid counts and length
    statistics this skill computes and validates
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:72bab3a99dc70545c4a3ed631989916944f2055cdcc748d84261152f8729ffdf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ARGprofiler
  - GeneAssimilator
  - GeneAssimiliator
  - PanRes
  - Usearch
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# sequence-length-distribution-statistics

## Summary

Compute summary length statistics (count, min, max, median) over a curated nucleotide sequence catalog — here the PanRes ARG centroid set — to characterize a merged, deduplicated, homology-reduced database and validate it against published values.

## When to use

Use when a merged gene catalog (e.g. ARG sequences from ResFinder, CARD, MegaRes, AMRFinderPlus, ARGANNOT) has been deduplicated and clustered, and you must report or validate the resulting sequence-count and centroid-length statistics.

## When NOT to use

- Input is raw sequencing reads rather than a curated multi-fasta gene catalog — length statistics of reads do not validate database curation.
- The clustering thresholds differ from the PanRes parameters (90% identity / 90% coverage); counts and length distributions will not be comparable to the reported 14,078 / 5,280 / 762 bp values.
- The goal is profiling ARG abundance in metagenomic samples (that is ARGprofiler's role) rather than characterizing the reference database itself.

## Inputs

- Merged multi-fasta of ARG genes collected from multiple source databases (e.g. ResFinder, ResFinderFG, CARD, MegaRes, AMRFinderPlus, ARGANNOT)
- Deduplicated unique-sequence fasta produced by Usearch fastx_uniques
- Centroid fasta from Usearch cluster_fast at 90% identity / 90% coverage
- Published reference values for unique-sequence count, centroid count, and median/length range (14,078; 5,280; 762 bp; 93–5,972 bp)

## Outputs

- Summary TSV (panres_curation_summary.tsv) with fields: number of unique input genes, unique sequences, centroids, and centroid length min, max, median
- Centroid length distribution statistics (min, max, median in bp)

## How to apply

After merging multi-fasta collections from the source ARG databases (ResFinder downloaded 20 January 2023, ResFinderFG v2.0, CARD v3.2.5, MegaRes v3.0.0, AMRFinderPlus v3.11/2022-12-19.1, ARGANNOT V6_July2019), first apply inclusion filters (e.g. keep only CARD protein-homolog-model genes, drop MegaRes 'RequiresSNPConfirmation' entries), then identify unique sequences with Usearch 11.0.667 fastx_uniques via GeneAssimilator. Cluster the unique ARGs with Usearch cluster_fast at 90% identity and 90% coverage to obtain centroids. Finally compute the number of unique input genes, unique sequences, centroids, and centroid length statistics (min, max, median), and compare against reported values (14,078 unique sequences; 5,280 centroids; median 762 bp; range 93–5,972 bp). The rationale is that count and length-distribution invariants are sensitive to upstream filter and clustering-threshold choices, so they serve as a cheap reproduction check of database curation.

## Related tools

- **Usearch** (fastx_uniques identifies unique sequences in the merged catalog; cluster_fast at 90% identity / 90% coverage produces the centroid set whose length distribution is summarized) — https://www.drive5.com/usearch/download.html
- **GeneAssimilator** (Runs the iterative recruit–cluster–refine procedure that wraps Usearch to merge input gene collections and emit the combined/homology-reduced catalog being summarized) — https://github.com/genomicepidemiology/gene_assimilator
- **PanRes** (The combined ARG database (14,078 unique ARGs) whose centroid counts and length statistics this skill computes and validates) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
usearch -cluster_fast unique_args.fa -id 0.90 -cover 0.90 -centroids panres_centroids.fa -uc cluster_map.uc
```

## Evaluation signals

- Unique-sequence count falls within a small tolerance of the reported 14,078 unique ARGs in PanRes.
- Centroid count falls within a small tolerance of the reported 5,280 centroids at 90% identity / 90% coverage clustering.
- Median centroid length is close to the reported 762 bp, and the length range brackets the reported 93–5,972 bp.
- panres_curation_summary.tsv exists and contains the expected fields (unique input genes, unique sequences, centroids, length min/max/median).
- Unique-sequence count is monotonically consistent with pipeline stages: unique input genes ≥ unique sequences ≥ centroid count.

## Limitations

- PanRes combines sources with different scopes, so the appropriateness of any subset is left to the individual research question.
- Counts depend on the exact database versions (e.g. CARD v3.2.5, MegaRes v3.0.0) and inclusion filters; newer releases will shift the statistics.
- Results depend on Usearch version (11.0.667) and its heuristic cluster_fast algorithm, so alternative clusterers may give slightly different centroids.

## Evidence

- [abstract] PanRes database contains 14,078 unique ARGs combined from several existing collections: "providing the PanRes database of 14 078 unique ARGs that combines several existing collections into one"
- [discussion] PanRes combines sources with different scopes; subset appropriateness is up to the research question: "it combines different sources and scopes, it will be up to the individual research questions, which subsets are appropriate for consideration"
- [readme] GeneAssimilator merges and clusters gene catalogs using Usearch dependencies: [evidence span withheld — anchor status: external_doc]
