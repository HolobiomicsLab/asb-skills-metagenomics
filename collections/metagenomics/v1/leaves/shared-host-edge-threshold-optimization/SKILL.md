---
name: shared-host-edge-threshold-optimization
description: Use when constructing a host proximity graph over viral contigs from
  a Hi-C interaction graph and need a data-driven threshold on the number of shared
  host contigs required to link two viral contigs.
when_to_use_negative:
- When viral contig identity or a complete Hi-C interaction graph is unavailable and
  |E_hic| cannot be computed, the |E_host| ≤ |E_hic| constraint cannot be evaluated
- When binning viral contigs from shotgun-only data with co-abundance profiles (e.g.,
  CoCoNet, vRhyme), where no Hi-C virus-host proximity structure exists
- When annotating host contigs taxonomically as a prerequisite step (e.g., TAXAassign
  for HiCBin) rather than tuning a proximity-graph edge threshold
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3174
tools:
- name: ViralCC
  role: Pipeline that builds the Hi-C interaction graph and the host proximity graph
    with automatic k selection via the |E_host| ≤ |E_hic| rule, then applies Leiden
    clustering
  repo: https://github.com/dyxstat/ViralCC
- name: pysam
  role: Python library used to load the name-sorted Hi-C BAM and filter alignments
    by MAPQ ≥ 30 and ≥ 30 matches when building the interaction graph
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:73a89675ac3cb72c6caedc1ed583f79714b0de02dcf6b56832eb092365c23d12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - ViralCC
  - pysam
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# shared-host-edge-threshold-optimization

## Summary

Automatic selection of the shared-host threshold k used to build ViralCC's host proximity graph, in which two viral contigs are linked when they share at least k host contigs contacted in Hi-C, so that the proximity signal complements rather than overwhelms the Hi-C interaction signal.

## When to use

Use when constructing a host proximity graph over viral contigs from a Hi-C interaction graph and need a data-driven threshold on the number of shared host contigs required to link two viral contigs.

## When NOT to use

- When viral contig identity or a complete Hi-C interaction graph is unavailable and |E_hic| cannot be computed, the |E_host| ≤ |E_hic| constraint cannot be evaluated
- When binning viral contigs from shotgun-only data with co-abundance profiles (e.g., CoCoNet, vRhyme), where no Hi-C virus-host proximity structure exists
- When annotating host contigs taxonomically as a prerequisite step (e.g., TAXAassign for HiCBin) rather than tuning a proximity-graph edge threshold

## Inputs

- Name-sorted Hi-C alignment BAM (e.g., MAP_SORTED.bam) of paired-end Hi-C reads aligned to assembled contigs
- Assembly FASTA of contigs (e.g., final.contigs.fa) for applying the --min-len 1000 bp contig length filter
- List of putative viral contigs (e.g., viral_contigs.txt) identified from the assembly

## Outputs

- Host proximity graph G_host linking pairs of viral contigs that share at least k host contigs (k selected automatically, e.g., k = 30 on the mock human gut dataset with 2698 edges)
- Selected k threshold value and edge counts |E_hic| and |E_host| used in the selection

## How to apply

Load the name-sorted Hi-C BAM with pysam, retaining alignments with MAPQ ≥ 30 (--min-mapq default), at least 30 matches (--min-match default), for contigs of length ≥ 1000 bp (--min-len default), then build the Hi-C interaction graph and record its edge count |E_hic|. For each candidate threshold k ≥ 4 (the --min-k lower bound), construct a candidate host proximity graph in which two viral contigs are connected when they share at least k host contigs contacted in Hi-C. Select the k that maximizes the number of host-proximity edges subject to the constraint |E_host| ≤ |E_hic|; this constraint ensures the proximity-derived edges complement rather than overwhelm the direct Hi-C interaction signal. On the mock human gut dataset this rule selects k = 30, yielding 2698 edges in G_host. Validate the implemented rule end-to-end on the packaged simulated dataset before trusting it on new data.

## Related tools

- **ViralCC** (Pipeline that builds the Hi-C interaction graph and the host proximity graph with automatic k selection via the |E_host| ≤ |E_hic| rule, then applies Leiden clustering) — https://github.com/dyxstat/ViralCC
- **pysam** (Python library used to load the name-sorted Hi-C BAM and filter alignments by MAPQ ≥ 30 and ≥ 30 matches when building the interaction graph)

## Examples

```
python ./viralcc.py pipeline -v Test/final.contigs.fa Test/MAP_SORTED.bam Test/viral_contigs.txt Test/out_test
```

## Evaluation signals

- The selected k satisfies |E_host| ≤ |E_hic|, i.e., host-proximity edges never exceed Hi-C interaction graph edges
- The rule reproduces the reported mock human gut dataset result: 2698 edges in G_host at the selected k = 30
- Candidate thresholds are enumerated only for k ≥ 4, matching the --min-k lower bound
- The pipeline runs end-to-end on the packaged Test directory dataset (final.contigs.fa, MAP_SORTED.bam, viral_contigs.txt) and completes the graph construction step without error

## Limitations

- The fraction of spurious contacts in the host proximity graph cannot be estimated on real datasets because true labels of viral contigs are unknown
- Some proximity-ligation virus-bacteria associations may arise from proximity mechanisms other than infection, so proximity-graph-derived linkages should be validated (e.g., by CRISPR spacer analysis) before drawing host-infection conclusions
- Lower bound on k is fixed by --min-k (default 4), which may exclude very weakly connected candidates

## Evidence

- [results] In the host proximity graph, two viral contigs are linked when they share at least k (tuned to 30 on the mock human gut dataset) common host contigs by Hi-C interaction, resulting in 2698 edges in G_host.: [evidence span withheld — anchor status: not_found]
- [methods] --min-mapq: Minimum acceptable alignment quality (default 30): "--min-mapq: Minimum acceptable alignment quality (default 30)"
- [readme] ViralCC not only considers the Hi-C interaction graph, but also puts forward a novel host proximity graph of viral contigs as a complementary source of information: "puts forward a novel host proximity graph of viral contigs as a complementary source of information to the remarkably sparse Hi-C interaction map"
- [methods] Select the k that maximizes the number of host-proximity edges subject to |E_host| ≤ |E_hic|, so the proximity signal complements rather than overwhelms the Hi-C interaction signal.: "ViralCC leverages the virus-host proximity structure as a complementary information source for the Hi-C interactions."
