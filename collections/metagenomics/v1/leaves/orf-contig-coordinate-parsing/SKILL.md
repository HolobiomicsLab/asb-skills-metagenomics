---
name: orf-contig-coordinate-parsing
description: Use when you have VAMB gene clusters (all_genes_cluster.tsv) plus ORF
  FASTA sequences embedding per-contig gene order, and you need genomic adjacency
  information to build synteny clusters.
when_to_use_negative:
- You need true operon-level structure — MAGinator's synteny clustering lacks operon
  awareness, so operon inference is out of scope
- Input lacks per-contig gene ordering (e.g. gene clusters without ORF sequences on
  contigs) — adjacency cannot be derived
- You only need taxonomic/abundance profiling without functional annotation improvement
  via syntenic grouping
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MAGinator
  role: Parses ORF sequences and contig coordinates, builds the gene-cluster adjacency
    graph, and partitions it into synteny clusters written to tabs/synteny_clusters.tab
  repo: https://github.com/Russel88/MAGinator
- name: Snakemake
  role: Workflow engine whose rules implement the ORF/coordinate parsing and synteny
    graph construction steps
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:7d28d4cabdedc84b944b5daac00f3cc1482259d179c59a501b2541c936ef6edd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  - Snakemake
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# orf-contig-coordinate-parsing

## Summary

Parse MAGinator's all_genes.fna ORF nucleotide sequences — whose headers carry per-contig gene ordering — together with all_genes_cluster.tsv gene-cluster assignments, to establish which gene clusters occur adjacent on contigs as the basis for synteny clustering.

## When to use

Use when you have VAMB gene clusters (all_genes_cluster.tsv) plus ORF FASTA sequences embedding per-contig gene order, and you need genomic adjacency information to build synteny clusters.

## When NOT to use

- You need true operon-level structure — MAGinator's synteny clustering lacks operon awareness, so operon inference is out of scope
- Input lacks per-contig gene ordering (e.g. gene clusters without ORF sequences on contigs) — adjacency cannot be derived
- You only need taxonomic/abundance profiling without functional annotation improvement via syntenic grouping

## Inputs

- all_genes_cluster.tsv — gene cluster assignments for every ORF
- all_genes.fna — ORF nucleotide sequences with headers encoding per-contig gene ordering
- contigs.fasta — contig sequences with unique names (upstream MAGinator input defining contig identity)

## Outputs

- Weighted adjacency graph of gene clusters with edge weights encoding adjacency frequency (intermediates under genes/synteny/)
- tabs/synteny_clusters.tab — synteny cluster association for every gene cluster

## How to apply

Load the gene cluster assignments from all_genes_cluster.tsv alongside the ORF nucleotide sequences in all_genes.fna, whose per-contig ordering encodes which genes sit next to each other on each contig. From the ORF headers, derive the contig origin and the positional order of each gene on that contig. Build a weighted adjacency graph whose nodes are the gene clusters and whose edge weights count how often genes from each pair of clusters are found adjacent on contigs, storing intermediates under genes/synteny/. The rationale is that genomically adjacent (syntenic) genes are usually part of the same pathway or share related functions, so adjacency-derived grouping improves functional annotation. Note this is not operon-aware — accuracy is bounded by MAGinator's lack of operon awareness, and users can trade sensitivity for lower noise via user-modifiable parameters.

## Related tools

- **MAGinator** (Parses ORF sequences and contig coordinates, builds the gene-cluster adjacency graph, and partitions it into synteny clusters written to tabs/synteny_clusters.tab) — https://github.com/Russel88/MAGinator
- **Snakemake** (Workflow engine whose rules implement the ORF/coordinate parsing and synteny graph construction steps)

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out --max_mem 180
```

## Evaluation signals

- tabs/synteny_clusters.tab exists and covers the gene clusters from all_genes_cluster.tsv
- Resulting synteny clusters average approximately 3 genes per cluster as reported for the method
- Every gene cluster in all_genes_cluster.tsv receives exactly one synteny-cluster assignment
- Synteny-cluster membership reflects genomic adjacency: genes grouped together are close to each other on contigs, consistent with all_genes.fna ordering

## Limitations

- Accuracy is limited by MAGinator's lack of operon awareness
- Users can eliminate synteny noise at the expense of sensitivity by altering user-modifiable parameters

## Evidence

- [readme] MAGinator clusters gene clusters into synteny clusters - Syntenic genes are usually part of the same pathway or have a similar functions: "MAGinator clusters gene clusters into synteny clusters - Syntenic genes are usually part of the same pathway or have a similar functions"
- [readme] Improve your functional annotation by grouping your genes in synteny clusters based on genomic adjacency: "Improve your functional annotation by grouping your genes in synteny clusters based on genomic adjacency"
- [discussion] Currently, accuracy is limited by MAGinator’s lack of operon awareness.: "Currently, accuracy is limited by MAGinator’s lack of operon awareness."
