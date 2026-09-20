---
name: mcl-weighted-graph-clustering
description: Use when you have gene cluster assignments plus per-contig ORF ordering
  (all_genes_cluster.tsv + all_genes.fna) and want to group co-localized genes into
  synteny clusters for pathway-level functional insight.
when_to_use_negative:
- You need operon-aware synteny detection — MAGinator currently lacks operon awareness,
  limiting synteny cluster accuracy
- You only have gene sequence similarity (no contig-level positional information),
  since adjacency on contigs is the sole grouping signal
- Your input is raw reads or unclustered ORFs — gene clusters must already exist (e.g.
  from MAGinator's VAMB-based gene clustering step)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MAGinator
  role: 'Implements the synteny module: builds the weighted adjacency graph over gene
    clusters from contig adjacency counts and partitions it with MCL, emitting synteny_clusters.tab'
  repo: github.com/Russel88/MAGinator
- name: Snakemake
  role: Workflow engine that executes MAGinator's rules, including the graph construction
    and MCL clustering steps under genes/synteny/
  repo: https://github.com/Russel88/MAGinator
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:9a7c566adb703f724fbf8ab38c606b51fb6b28539487e3f551e080ce3b2e55f9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAGinator
  - Snakemake
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mcl-weighted-graph-clustering

## Summary

Clusters gene clusters into synteny clusters by building a weighted adjacency graph over gene clusters — edge weights encode how often genes from each pair of clusters occur adjacent on contigs — and partitioning it with Markov clustering (MCL) at the default inflation, as implemented in MAGinator's Snakemake workflow.

## When to use

Use when you have gene cluster assignments plus per-contig ORF ordering (all_genes_cluster.tsv + all_genes.fna) and want to group co-localized genes into synteny clusters for pathway-level functional insight.

## When NOT to use

- You need operon-aware synteny detection — MAGinator currently lacks operon awareness, limiting synteny cluster accuracy
- You only have gene sequence similarity (no contig-level positional information), since adjacency on contigs is the sole grouping signal
- Your input is raw reads or unclustered ORFs — gene clusters must already exist (e.g. from MAGinator's VAMB-based gene clustering step)

## Inputs

- all_genes_cluster.tsv — gene cluster assignments for all ORFs
- all_genes.fna — ORF nucleotide sequences encoding per-contig gene ordering

## Outputs

- tabs/synteny_clusters.tab — synteny cluster association for every gene cluster
- genes/synteny/ — intermediate adjacency-graph files

## How to apply

Load the gene cluster assignments from all_genes_cluster.tsv together with the ORF nucleotide sequences in all_genes.fna, which carry the per-contig gene ordering needed to infer adjacency. Build a weighted adjacency graph whose nodes are the gene clusters and whose edge weights count how often genes from each pair of clusters are found adjacent on contigs, storing intermediates under genes/synteny/. Partition this graph with MCL at the default inflation parameter to obtain synteny clusters of genomically co-localized genes; the method reports approximately 3 genes per synteny cluster on average. Write the synteny-cluster association for every gene cluster to tabs/synteny_clusters.tab. Rationale: syntenic genes are usually part of the same pathway or have related functions, so this grouping improves functional annotation of MAG clusters; note accuracy is currently limited by MAGinator's lack of operon awareness.

## Related tools

- **MAGinator** (Implements the synteny module: builds the weighted adjacency graph over gene clusters from contig adjacency counts and partitions it with MCL, emitting synteny_clusters.tab) — github.com/Russel88/MAGinator
- **Snakemake** (Workflow engine that executes MAGinator's rules, including the graph construction and MCL clustering steps under genes/synteny/) — https://github.com/Russel88/MAGinator

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out --max_mem 180
```

## Evaluation signals

- tabs/synteny_clusters.tab exists and covers all gene clusters from all_genes_cluster.tsv
- Resulting synteny clusters average approximately 3 genes per cluster, as reported for the method
- Genes placed in the same synteny cluster are genomically adjacent on contigs (spot-check against all_genes.fna ordering)
- Intermediate adjacency-graph files are present under genes/synteny/, confirming the graph construction step ran
- Differentially tuning MAGinator's user-modifiable parameters trades noise reduction against sensitivity — verify the chosen settings do not silently drop small synteny clusters

## Limitations

- Accuracy is limited by MAGinator's lack of operon awareness
- Users can eliminate synteny noise via parameter changes only at the expense of sensitivity

## Evidence

- [readme] Improve your functional annotation by grouping your genes in synteny clusters based on genomic adjacency: "Improve your functional annotation by grouping your genes in synteny clusters based on genomic adjacency"
- [readme] MAGinator clusters gene clusters into synteny clusters - Syntenic genes are usually part of the same pathway or have similar functions: "MAGinator clusters gene clusters into synteny clusters - Syntenic genes are usually part of the same pathway or have similar functions"
- [intro] By combining the information from both contigs and gene content we identify synteny clusters of genes within subspecies, yielding information on shared pathways for the genes: "By combining the information from both contigs and gene content we identify synteny clusters of genes within subspecies, yielding information on shared pathways for the genes"
- [discussion] Currently, accuracy is limited by MAGinator's lack of operon awareness.: "Currently, accuracy is limited by MAGinator’s lack of operon awareness."
- [discussion] Users can also eliminate noise at the expense of sensitivity by altering a number of user-modifiable parameters.: "Users can also eliminate noise at the expense of sensitivity by altering a number of user-modifiable parameters."
