---
name: metagenomic-gene-cluster-functional-summarization
description: Use when you have MAGinator synteny_clusters.tab output plus a non-redundant
  gene catalogue and need to attach KEGG module assignments per synteny cluster and
  measure functional coherence.
when_to_use_negative:
- You have no MAGinator/VAMB output (no clusters.tsv or synteny_clusters.tab) — the
  skill requires the synteny and gene-cluster association tables as input
- The functional annotation of individual genes is already available in merged form
  and you only need per-gene KO terms — go straight to the merge rather than re-running
  emapper.py
- You need operon-aware pathway reconstruction — the paper notes accuracy is limited
  because MAGinator lacks operon awareness
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0176
tools:
- name: MAGinator
  role: produces the synteny_clusters.tab association table by grouping gene clusters
    found adjacent on contigs
  repo: github.com/Russel88/MAGinator
- name: eggNOG-mapper
  role: annotates the non-redundant gene catalogue in diamond mode to obtain KEGG
    orthologs (column 15 of emapper output)
  repo: https://github.com/eggnogdb/eggnog-mapper
- name: VAMB
  role: clusters.tsv file supplies the contig-to-MAG-cluster mapping merged with the
    functional annotations
  repo: https://github.com/RasmussenLab/vamb
- name: Snakemake
  role: workflow engine in which MAGinator (and hence the synteny clustering step)
    is implemented
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:3730e209568b30465dc55754baee0091638659c7415467634606031a09771520
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAGinator
  - Snakemake
  - VAMB
  - eggNOG-mapper
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0176
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# metagenomic-gene-cluster-functional-summarization

## Summary

Annotate MAGinator's synteny clusters with eggNOG-mapper KEGG orthologs and quantify intra-cluster functional agreement, as done for the COPSAC2010 cohort (35,798 qualifying clusters of 746,251 total).

## When to use

Use when you have MAGinator synteny_clusters.tab output plus a non-redundant gene catalogue and need to attach KEGG module assignments per synteny cluster and measure functional coherence.

## When NOT to use

- You have no MAGinator/VAMB output (no clusters.tsv or synteny_clusters.tab) — the skill requires the synteny and gene-cluster association tables as input
- The functional annotation of individual genes is already available in merged form and you only need per-gene KO terms — go straight to the merge rather than re-running emapper.py
- You need operon-aware pathway reconstruction — the paper notes accuracy is limited because MAGinator lacks operon awareness

## Inputs

- MAGinator synteny_clusters.tab association table mapping gene clusters to synteny clusters
- VAMB clusters.tsv file mapping contigs to MAG clusters
- all_genes_nonredundant.fasta — nucleotide sequences of gene cluster representatives
- eggNOG-mapper output (emapper annotations.tsv) with KEGG ortholog in column 15

## Outputs

- Per-synteny-cluster predominant KEGG module assignment
- Intra-cluster functional agreement fraction per synteny cluster
- Summary statistics: total synteny clusters, clusters with a single shared KEGG module, and fraction of modules exceeding 80% agreement

## How to apply

Run the MAGinator synteny-clustering step to group gene clusters by genomic adjacency, producing synteny_clusters.tab (COPSAC2010 yielded 746,251 synteny clusters). Annotate the non-redundant gene catalogue (all_genes_nonredundant.fasta, nucleotide sequences of gene-cluster representatives) with eggNOG-mapper in diamond mode. Parse the emapper output by cutting column 15 (KEGG ortholog), stripping the header and trailing summary lines, removing unannotated ('-') entries, stripping the 'ko:' prefix, and splitting comma-separated KO terms. Merge these annotations with the VAMB clusters.tsv mapping and the synteny cluster associations. For each qualifying synteny cluster (>=5 genes and >=2 annotated genes, giving 35,798 qualifying clusters in COPSAC2010), determine the predominant KEGG module and the fraction of genes sharing it. Summarize: total clusters, clusters where all genes share one module (28,341 in COPSAC2010), and the fraction of modules with >80% intra-cluster agreement (80.5%).

## Related tools

- **MAGinator** (produces the synteny_clusters.tab association table by grouping gene clusters found adjacent on contigs) — github.com/Russel88/MAGinator
- **eggNOG-mapper** (annotates the non-redundant gene catalogue in diamond mode to obtain KEGG orthologs (column 15 of emapper output)) — https://github.com/eggnogdb/eggnog-mapper
- **VAMB** (clusters.tsv file supplies the contig-to-MAG-cluster mapping merged with the functional annotations) — https://github.com/RasmussenLab/vamb
- **Snakemake** (workflow engine in which MAGinator (and hence the synteny clustering step) is implemented)

## Examples

```
mkdir test_out/functional_annotation; emapper.py -i test/genes/all_genes_rep_seq.fasta --output test_out/functional_annotation -m diamond --cpu 38
```

## Evaluation signals

- Total synteny cluster count matches the reported 746,251 for the COPSAC2010 run
- After applying the >=5 genes and >=2 annotated genes filter, the qualifying cluster count is 35,798
- Clusters whose genes all share a single KEGG module number 28,341
- 80.5% of modules show more than 80% intra-cluster agreement
- Parsed emapper output has no '-' (unannotated) entries and no residual 'ko:' prefixes or header/summary lines

## Limitations

- Accuracy of synteny clusters is limited by MAGinator's lack of operon awareness
- Users eliminating synteny noise via parameter changes do so at the expense of sensitivity
- Only synteny clusters meeting the >=5 genes / >=2 annotated genes threshold are scored, so small or sparsely annotated clusters are excluded from the summary

## Evidence

- [readme] MAGinator clusters gene clusters into synteny clusters - Syntenic genes are usually part of the same pathway or have similar functions: "MAGinator clusters gene clusters into synteny clusters - Syntenic genes are usually part of the same pathway or have similar functions"
- [readme] emapper diamond invocation on gene representatives: "emapper.py -i test/genes/all_genes_rep_seq.fasta --output test_out/functional_annotation -m diamond --cpu 38"
- [readme] KEGG ortholog column selection from emapper output: "In this case the KEGG ortholog column 15 was picked from the eggNOG-mapper output."
- [intro] synteny clusters identify shared pathways within subspecies: "By combining the information from both contigs and gene content we identify synteny clusters of genes within subspecies, yielding information on shared pathways for the genes"
- [discussion] operon awareness limitation: "Currently, accuracy is limited by MAGinator’s lack of operon awareness."
- [methods] EggNOG mapper recommendation in methods: "we recommend using EggNOG mapper (https://github.com/eggnogdb/eggnog-mapper)"
