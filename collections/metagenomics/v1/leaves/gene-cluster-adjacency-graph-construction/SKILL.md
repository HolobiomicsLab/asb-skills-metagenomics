---
name: gene-cluster-adjacency-graph-construction
description: Use when you have VAMB-derived gene cluster assignments plus ORF sequences
  retaining per-contig gene ordering, and you want to group gene clusters into synteny
  clusters based on genomic adjacency to infer shared pathways.
when_to_use_negative:
- When operon-level precision is required — accuracy is limited by MAGinator's lack
  of operon awareness
- When gene calls lack per-contig ordering information, since adjacency cannot be
  computed
- When the goal is pathway reconstruction from annotation alone (e.g. KEGG via EggNOG
  mapper) rather than genomic co-localization
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MAGinator
  role: Implements the adjacency-graph construction and MCL partitioning of gene clusters
    into synteny clusters within its Snakemake workflow
  repo: https://github.com/Russel88/MAGinator
- name: Snakemake
  role: Workflow engine executing the rules that build the weighted adjacency graph
    and run MCL, installing dependencies automatically
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:8f40dd07c3c559a53105b19a4ad46de9fb290dd66b62016f03f3928456a8e5c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  - Snakemake
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gene-cluster-adjacency-graph-construction

## Summary

MAGinator's method for building synteny clusters: a weighted adjacency graph over gene clusters, with edge weights counting how often genes from each cluster pair are genomically adjacent on contigs, partitioned with Markov clustering (MCL) so that co-localized genes are grouped together.

## When to use

Use when you have VAMB-derived gene cluster assignments plus ORF sequences retaining per-contig gene ordering, and you want to group gene clusters into synteny clusters based on genomic adjacency to infer shared pathways.

## When NOT to use

- When operon-level precision is required — accuracy is limited by MAGinator's lack of operon awareness
- When gene calls lack per-contig ordering information, since adjacency cannot be computed
- When the goal is pathway reconstruction from annotation alone (e.g. KEGG via EggNOG mapper) rather than genomic co-localization

## Inputs

- all_genes_cluster.tsv (gene cluster assignments for all ORFs)
- all_genes.fna (ORF nucleotide sequences carrying per-contig gene ordering)
- Snakemake workflow rules from the MAGinator pipeline

## Outputs

- Intermediate adjacency-graph files under genes/synteny/
- tabs/synteny_clusters.tab (synteny cluster association for every gene cluster)

## How to apply

Load the gene cluster assignments from all_genes_cluster.tsv together with the ORF nucleotide sequences (all_genes.fna), which carry the per-contig gene ordering, using MAGinator's Snakemake workflow rules. Build a weighted adjacency graph whose nodes are the gene clusters and whose edge weights encode how often genes from each pair of clusters are found adjacent on contigs, storing intermediate files under genes/synteny/. Partition this weighted graph with MCL (Markov clustering) at the default inflation parameter to obtain synteny clusters of co-localized genes. The rationale is that syntenic genes are usually part of the same pathway or have related functions, so the resulting clusters improve functional annotation of MAG clusters. Finally write the per-gene-cluster synteny association to tabs/synteny_clusters.tab. Note accuracy is limited by MAGinator's lack of operon awareness; users can reduce noise at the expense of sensitivity via user-modifiable parameters.

## Related tools

- **MAGinator** (Implements the adjacency-graph construction and MCL partitioning of gene clusters into synteny clusters within its Snakemake workflow) — https://github.com/Russel88/MAGinator
- **Snakemake** (Workflow engine executing the rules that build the weighted adjacency graph and run MCL, installing dependencies automatically)

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out --max_mem 180
```

## Evaluation signals

- tabs/synteny_clusters.tab exists and covers every gene cluster from all_genes_cluster.tsv
- Resulting synteny clusters average approximately 3 genes per cluster, as reported for the method
- Genes assigned to the same synteny cluster are genomically adjacent on contigs (spot-check against all_genes.fna ordering)
- Intermediate files are present under genes/synteny/, confirming the adjacency graph step ran

## Limitations

- Accuracy is limited by MAGinator's lack of operon awareness
- Users can eliminate synteny noise at the expense of sensitivity by altering user-modifiable parameters
- Unannotated VAMB clusters remain in the analysis, so synteny clusters may include genes without taxonomic consensus

## Evidence

- [readme] Improve your functional annotation by grouping your genes in synteny clusters based on genomic adjacency: "Improve your functional annotation by grouping your genes in synteny clusters based on genomic adjacency"
- [readme] Syntenic genes are usually part of the same pathway or have similar functions: "MAGinator clusters gene clusters into synteny clusters - Syntenic genes are usually part of the same pathway or have similar functions"
- [intro] By combining the information from both contigs and gene content we identify synteny clusters of genes within subspecies: "By combining the information from both contigs and gene content we identify synteny clusters of genes within subspecies, yielding information on shared pathways for the genes"
- [discussion] Currently, accuracy is limited by MAGinator’s lack of operon awareness.: "Currently, accuracy is limited by MAGinator’s lack of operon awareness."
- [discussion] Users can also eliminate noise at the expense of sensitivity by altering a number of user-modifiable parameters.: "Users can also eliminate noise at the expense of sensitivity by altering a number of user-modifiable parameters."
