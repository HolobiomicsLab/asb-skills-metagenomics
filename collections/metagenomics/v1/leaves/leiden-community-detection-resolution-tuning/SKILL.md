---
name: leiden-community-detection-resolution-tuning
description: Use when a weighted graph linking viral contigs (e.g., an integrative
  Hi-C interaction plus host proximity graph G_int) must be partitioned into draft
  bins and the clustering resolution is unknown.
when_to_use_negative:
- When no Hi-C/proximity graph exists and bins must come from composition or co-abundance
  features only (e.g., too few samples for reliable co-abundance profiles, where CoCoNet/vRhyme
  are also impaired).
- When the goal is classifying already-binned viral contigs taxonomically to order/family
  rather than binning (e.g., Demovir).
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: ViralCC
  role: Pipeline that constructs G_host and G_hic, integrates them into G_int, and
    runs the resolution-tuned Leiden clustering (viralcc.py pipeline).
  repo: https://github.com/dyxstat/ViralCC
- name: leidenalg
  role: Python library executing the Leiden clustering on G_int under Reichardt-Bornholdt
    modularity with swept resolution r.
- name: scikit-learn
  role: Computes the silhouette score used to select the resolution r maximizing cluster
    quality.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:d38ab35ca95b05dbe6f762c3de56a164ab1789b806b5060c8114291b55b90b25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - ViralCC
  - leidenalg
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# leiden-community-detection-resolution-tuning

## Summary

Cluster an integrated metagenomic Hi-C graph with Leiden community detection (leidenalg) under Reichardt-Bornholdt modularity, sweeping the resolution parameter r and selecting the value maximizing the silhouette score to reconstruct draft viral genomes.

## When to use

Use when a weighted graph linking viral contigs (e.g., an integrative Hi-C interaction plus host proximity graph G_int) must be partitioned into draft bins and the clustering resolution is unknown.

## When NOT to use

- When no Hi-C/proximity graph exists and bins must come from composition or co-abundance features only (e.g., too few samples for reliable co-abundance profiles, where CoCoNet/vRhyme are also impaired).
- When the goal is classifying already-binned viral contigs taxonomically to order/family rather than binning (e.g., Demovir).

## Inputs

- Integrative graph G_int combining the Hi-C interaction graph and the host proximity graph of viral contigs
- Hi-C alignments in name-sorted BAM (e.g., MAP_SORTED.bam) filtered with --min-mapq (default 30) and --min-match (default 30)
- Host proximity graph built with --min-k (default 4) from the viral contig list (viral_contigs.txt)

## Outputs

- cluster_viral_contig.txt (2 columns: viral contig name and group number, no header)
- Draft viral bin FASTA files (VIRAL_BIN0000.fa, VIRAL_BIN0001.fa, ...) in the VIRAL_BIN folder
- Auxiliary tables viral_contig_info.csv and prokaryotic_contig_info.csv (contig name, length, GC-content) and log file viralcc.log

## How to apply

Build the unified graph G_int by combining the Hi-C interaction graph (alignments filtered with --min-mapq 30 and --min-match 30) with the host proximity graph (built with --min-k, default 4). Run Leiden clustering via leidenalg on G_int under Reichardt-Bornholdt modularity, sweeping the resolution parameter r and computing the silhouette score (scikit-learn) for each r to select the value that maximizes clustering quality. Fix --random-seed at 42 for reproducibility. On the mock human gut dataset this procedure reconstructs 26 near-complete, 2 substantially complete, and 4 moderately complete vMAGs.

## Related tools

- **ViralCC** (Pipeline that constructs G_host and G_hic, integrates them into G_int, and runs the resolution-tuned Leiden clustering (viralcc.py pipeline).) — https://github.com/dyxstat/ViralCC
- **leidenalg** (Python library executing the Leiden clustering on G_int under Reichardt-Bornholdt modularity with swept resolution r.)
- **scikit-learn** (Computes the silhouette score used to select the resolution r maximizing cluster quality.)

## Examples

```
python ./viralcc.py pipeline --min-k 4 --min-mapq 30 --min-match 30 --random-seed 42 final.contigs.fa MAP_SORTED.bam viral_contigs.txt OUTPUT_directory
```

## Evaluation signals

- cluster_viral_contig.txt exists with exactly 2 columns of contig names and group numbers and no header.
- The VIRAL_BIN directory contains draft bin FASTA files named VIRAL_BIN0000.fa, VIRAL_BIN0001.fa, etc.
- The reproduced clustering output on the Test dataset matches the reference output in Test/out_test.
- The selected resolution r maximizes the silhouette score across the swept values.
- On the mock human gut dataset, the expected recovery is 26 near-complete, 2 substantially complete, and 4 moderately complete vMAGs.

## Limitations

- Leveraging Hi-C proximity ligation to discover co-infection of multiple phages within the same cell remains unexplored/future work, so clusters may not resolve co-infecting phages.
- Whether the method can resolve closely related viruses residing in the same bacterial host is unexplored.
- Reproducibility depends on fixing --random-seed (42); different seeds or resolution sweeps may alter bin boundaries.

## Evidence

- [intro] After integrating two graphs, ViralCC employs Leiden clustering to reconstruct draft viral genomes: "After integrating two graphs, ViralCC employs Leiden clustering to reconstruct draft viral genomes"
- [other] Run Leiden clustering with resolution sweep and silhouette selection: [evidence span withheld — anchor status: not_found]
- [other] Output format of clustering results: [evidence span withheld — anchor status: not_found]
- [other] Validation against reference output: [evidence span withheld — anchor status: not_found]
- [readme] README overview of graph integration and Leiden clustering: "The two graphs are then integrated together, followed by the Leiden graph clustering using the integrative graph to generate draft viral genomes."
