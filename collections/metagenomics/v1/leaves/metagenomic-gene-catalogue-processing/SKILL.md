---
name: metagenomic-gene-catalogue-processing
description: Use when a metagenomic analysis needs gene clusters grouped by genomic
  adjacency (synteny) to associate functional content and shared pathways with MAG
  clusters or subspecies clades.
when_to_use_negative:
- When the goal is species-level or strain-level abundance estimation rather than
  functional/pathway interpretation of gene neighborhoods
- When accuracy depends on operon-aware clustering — MAGinator currently lacks operon
  awareness, which limits synteny cluster accuracy
- When only reads or assembled contigs are available without VAMB clusters.tsv, contigs.fasta,
  and read files — the synteny module operates downstream of MAGinator's gene clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MAGinator
  role: 'Implements the synteny clustering module: builds the weighted adjacency graph
    of gene clusters and partitions it with MCL, writing synteny_clusters.tab'
  repo: https://github.com/Russel88/MAGinator
- name: Snakemake
  role: Workflow engine whose rules orchestrate loading gene cluster assignments and
    ORF sequences, running the adjacency graph construction and MCL partitioning
- name: VAMB
  role: Produces the upstream clusters.tsv of contig bins whose genes feed into the
    synteny clustering
  repo: https://github.com/RasmussenLab/vamb
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:02e46a3c657d95976bf19861ef4bb134bc10b8e33a193c438d5ffe5e7c9e0e5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  - Snakemake
  - VAMB
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# metagenomic-gene-catalogue-processing

## Summary

MAGinator's synteny clustering module groups gene clusters into synteny clusters by building a weighted adjacency graph of co-localized genes on contigs and partitioning it with MCL, so that syntenic genes — usually part of the same pathway or with related functions — end up in the same cluster.

## When to use

Use when a metagenomic analysis needs gene clusters grouped by genomic adjacency (synteny) to associate functional content and shared pathways with MAG clusters or subspecies clades.

## When NOT to use

- When the goal is species-level or strain-level abundance estimation rather than functional/pathway interpretation of gene neighborhoods
- When accuracy depends on operon-aware clustering — MAGinator currently lacks operon awareness, which limits synteny cluster accuracy
- When only reads or assembled contigs are available without VAMB clusters.tsv, contigs.fasta, and read files — the synteny module operates downstream of MAGinator's gene clustering

## Inputs

- all_genes_cluster.tsv (gene cluster assignments for all ORFs)
- all_genes.fna (ORF nucleotide sequences carrying per-contig gene ordering)
- MAGinator Snakemake workflow (rules driving the synteny module)

## Outputs

- tabs/synteny_clusters.tab (synteny cluster association for every gene cluster)
- Intermediate adjacency-graph files under genes/synteny/

## How to apply

Load the gene cluster assignments from all_genes_cluster.tsv together with the ORF nucleotide sequences (all_genes.fna), which carry the per-contig gene ordering, using MAGinator's Snakemake workflow rules. Build a weighted adjacency graph whose nodes are the gene clusters and whose edge weights encode how often genes from each pair of clusters are found adjacent on contigs, storing intermediate files under genes/synteny/. Partition this weighted adjacency graph with MCL (Markov clustering) at the default inflation parameter to obtain synteny clusters of co-localized genes. Finally, write the synteny cluster association for every gene cluster to tabs/synteny_clusters.tab, where gene clusters in the same synteny cluster are genomically adjacent. The rationale is that combining contig-derived adjacency information with gene content identifies synteny clusters within subspecies, yielding information on shared pathways for the genes.

## Related tools

- **MAGinator** (Implements the synteny clustering module: builds the weighted adjacency graph of gene clusters and partitions it with MCL, writing synteny_clusters.tab) — https://github.com/Russel88/MAGinator
- **Snakemake** (Workflow engine whose rules orchestrate loading gene cluster assignments and ORF sequences, running the adjacency graph construction and MCL partitioning)
- **VAMB** (Produces the upstream clusters.tsv of contig bins whose genes feed into the synteny clustering) — https://github.com/RasmussenLab/vamb

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out --max_mem 180
```

## Evaluation signals

- tabs/synteny_clusters.tab exists and covers the gene clusters from all_genes_cluster.tsv
- Resulting synteny clusters average approximately 3 genes per cluster, as reported for the method
- Gene clusters assigned to the same synteny cluster are genomically adjacent on contigs (per the per-contig ordering in all_genes.fna)
- Intermediate files are produced under genes/synteny/ indicating the adjacency graph was built before MCL partitioning
- Syntenic genes show coherent functional annotation (e.g. same KEGG pathway from EggNOG mapper annotation), consistent with syntenic genes usually being part of the same pathway

## Limitations

- Accuracy is limited by MAGinator's lack of operon awareness
- Users can eliminate synteny noise at the expense of sensitivity by altering user-modifiable parameters
- Synteny clusters can only group genes present in the VAMB gene clusters; unannotated or filtered bins may leave genes out of the adjacency graph

## Evidence

- [readme] MAGinator clusters gene clusters into synteny clusters - Syntenic genes are usually part of the same pathway or have similar functions: "MAGinator clusters gene clusters into synteny clusters - Syntenic genes are usually part of the same pathway or have similar functions"
- [readme] Improve your functional annotation by grouping your genes in synteny clusters based on genomic adjacency: "Improve your functional annotation by grouping your genes in synteny clusters based on genomic adjacency"
- [intro] By combining the information from both contigs and gene content we identify synteny clusters of genes within subspecies, yielding information on shared pathways for the genes: "By combining the information from both contigs and gene content we identify synteny clusters of genes within subspecies, yielding information on shared pathways for the genes"
- [discussion] Currently, accuracy is limited by MAGinator's lack of operon awareness.: [evidence span withheld — anchor status: not_found]
- [discussion] It is implemented in Snakemake to take advantage of the integrated work distribution capabilities necessary for processing large-scale metagenomics data.: "It is implemented in Snakemake to take advantage of the integrated work distribution capabilities necessary for processing large-scale metagenomics data."
