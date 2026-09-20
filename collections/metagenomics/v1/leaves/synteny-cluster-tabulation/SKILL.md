---
name: synteny-cluster-tabulation
description: Use when you have MAGinator gene cluster assignments (all_genes_cluster.tsv)
  plus ORF nucleotide sequences carrying contig gene order (all_genes.fna) and need
  co-localized gene groups for pathway-level functional insight.
when_to_use_negative:
- You only need gene presence/absence or abundance per MAG — synteny clustering adds
  adjacency-based grouping, not per-gene quantification.
- Your goal is strain-level phylogeny or abundance estimation — those use MAGinator's
  signature gene and readmapping steps, not the synteny module.
- Operon-level precision is required — MAGinator explicitly lacks operon awareness,
  limiting synteny cluster accuracy.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MAGinator
  role: 'Implements the synteny clustering module: builds the adjacency graph from
    contig gene ordering and runs MCL to emit tabs/synteny_clusters.tab'
  repo: https://github.com/Russel88/MAGinator
- name: Snakemake
  role: Workflow engine executing MAGinator's rules that orchestrate the synteny clustering
    steps and intermediate file handling
  repo: https://github.com/Russel88/MAGinator
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:1ea8028ca6cb068aa7726e246ca8cf083e15265068bd5721f4180a4841065c9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  - Snakemake
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# synteny-cluster-tabulation

## Summary

Groups MAGinator gene clusters into synteny clusters using a weighted adjacency graph built from per-contig gene ordering, partitioned with MCL, so that genomically co-localized genes (usually same pathway or related function) are reported per gene cluster.

## When to use

Use when you have MAGinator gene cluster assignments (all_genes_cluster.tsv) plus ORF nucleotide sequences carrying contig gene order (all_genes.fna) and need co-localized gene groups for pathway-level functional insight.

## When NOT to use

- You only need gene presence/absence or abundance per MAG — synteny clustering adds adjacency-based grouping, not per-gene quantification.
- Your goal is strain-level phylogeny or abundance estimation — those use MAGinator's signature gene and readmapping steps, not the synteny module.
- Operon-level precision is required — MAGinator explicitly lacks operon awareness, limiting synteny cluster accuracy.

## Inputs

- all_genes_cluster.tsv (gene cluster assignments from MAGinator's ORF clustering)
- all_genes.fna (ORF nucleotide sequences encoding per-contig gene ordering)

## Outputs

- tabs/synteny_clusters.tab (synteny cluster association for every gene cluster)
- intermediate adjacency-graph files under genes/synteny/

## How to apply

Load the gene cluster assignments from all_genes_cluster.tsv together with the ORF nucleotide sequences in all_genes.fna, which carry the per-contig gene ordering, using MAGinator's Snakemake rules. Build a weighted adjacency graph whose nodes are gene clusters and whose edge weights encode how often genes from each pair of clusters occur adjacent on contigs, writing intermediates under genes/synteny/. Partition this graph with MCL (Markov clustering) at the default inflation parameter to obtain synteny clusters of co-localized genes. Write the synteny cluster association for every gene cluster to tabs/synteny_clusters.tab. The rationale is that syntenic genes are usually part of the same pathway or have similar functions, so the tab improves functional annotation of MAGs and clades.

## Related tools

- **MAGinator** (Implements the synteny clustering module: builds the adjacency graph from contig gene ordering and runs MCL to emit tabs/synteny_clusters.tab) — https://github.com/Russel88/MAGinator
- **Snakemake** (Workflow engine executing MAGinator's rules that orchestrate the synteny clustering steps and intermediate file handling) — https://github.com/Russel88/MAGinator

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out
```

## Evaluation signals

- tabs/synteny_clusters.tab exists and covers all gene clusters from all_genes_cluster.tsv.
- Resulting synteny clusters average approximately 3 genes per cluster, as reported for the method.
- Genes placed in the same synteny cluster are genomically adjacent on contigs (adjacency-derived edge weights > 0 between their clusters).
- Intermediate files under genes/synteny/ (adjacency graph) are produced by the Snakemake rules before MCL partitioning.
- Expected output matches the reference run published on Zenodo (doi:10.5281/zenodo.8279036) for the test samples.

## Limitations

- Accuracy is limited by MAGinator's lack of operon awareness.
- Users can eliminate synteny noise via user-modifiable parameters, but only at the expense of sensitivity.

## Evidence

- [readme] Improve your functional annotation by grouping your genes in synteny clusters based on genomic adjacency: "Improve your functional annotation by grouping your genes in synteny clusters based on genomic adjacency"
- [readme] MAGinator clusters gene clusters into synteny clusters - Syntenic genes are usually part of the same pathway or have similar functions: "MAGinator clusters gene clusters into synteny clusters - Syntenic genes are usually part of the same pathway or have similar functions"
- [intro] By combining the information from both contigs and gene content we identify synteny clusters of genes within subspecies, yielding information on shared pathways: "By combining the information from both contigs and gene content we identify synteny clusters of genes within subspecies, yielding information on shared pathways for the genes"
- [discussion] Currently, accuracy is limited by MAGinator’s lack of operon awareness.: "Currently, accuracy is limited by MAGinator’s lack of operon awareness."
- [discussion] Users can also eliminate noise at the expense of sensitivity by altering a number of user-modifiable parameters.: "Users can also eliminate noise at the expense of sensitivity by altering a number of user-modifiable parameters."
