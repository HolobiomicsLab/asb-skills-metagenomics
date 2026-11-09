---
name: signature-gene-selection-from-gene-count-matrices
description: Use when you have a read-mapping gene count matrix (small_gene_count_matrix.tsv)
  plus MAG cluster assignments (metagenomicspecies.tab) and need to identify de novo
  signature genes for per-sample MAG cluster abundance estimation.
when_to_use_negative:
- No gene count matrix exists yet — the upstream read-mapping and gene-counting step
  must first have produced small_gene_count_matrix.tsv.
- The goal is marker-gene-database taxonomic profiling (MetaPhlAn, MetaPhyler, mOTUs)
  rather than de novo signature gene identification for MAG clusters.
- Target MAG clusters are absent from fewer than --min_samples samples, so refinement
  cannot be performed and selected genes would be unvalidated.
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: MAGinator
  role: Ranks MAG-cluster-specific genes by co-abundance correlation, fits the rank-based
    negative binomial signature gene model to refine the gene set, and writes signature_genes_cluster.tsv
    plus the abundance table
  repo: https://github.com/Russel88/MAGinator
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:566c1ba29858158bd428a1eebd3a3a381253ebce8d99ccfcec881e478acbb478
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Signature gene selection from gene count matrices

## Summary

MAGinator's de novo selection and statistical refinement of MAG-cluster-specific signature genes from a read-mapping gene count matrix, using co-abundance ranking and a rank-based negative binomial model to pick an ideal gene set for abundance estimation.

## When to use

Use when you have a read-mapping gene count matrix (small_gene_count_matrix.tsv) plus MAG cluster assignments (metagenomicspecies.tab) and need to identify de novo signature genes for per-sample MAG cluster abundance estimation.

## When NOT to use

- No gene count matrix exists yet — the upstream read-mapping and gene-counting step must first have produced small_gene_count_matrix.tsv.
- The goal is marker-gene-database taxonomic profiling (MetaPhlAn, MetaPhyler, mOTUs) rather than de novo signature gene identification for MAG clusters.
- Target MAG clusters are absent from fewer than --min_samples samples, so refinement cannot be performed and selected genes would be unvalidated.

## Inputs

- small_gene_count_matrix.tsv — read count matrix of genes that do not cluster across MAG clusters (from upstream read-mapping and gene-counting)
- metagenomicspecies.tab — MAG cluster assignments (e.g. from VAMB clusters grouped into metagenomic species)
- User parameters: --num_signature_genes, --min_samples, --min_mapped_signature_genes, --abundance_calculation, --tail-percentage

## Outputs

- signature_genes_cluster.tsv — signature gene assignments per MAG cluster
- abundance/abundance_phyloseq.RData — per-sample absolute MAG cluster abundance table (reads-per-bp basis)
- signature_genes/read-count_detected-genes.pdf — signature gene optimization diagnostics

## How to apply

Load small_gene_count_matrix.tsv (containing only genes that do not cluster across MAG clusters) together with the MAG cluster assignments from metagenomicspecies.tab, and restrict to non-redundant genes found in only one MAG cluster each to build a MAG-cluster-specific gene set. Within each MAG cluster, rank genes by co-abundance correlation across samples and select an initial signature gene set sized by --num_signature_genes. Fit the rank-based negative binomial model of detected signature genes versus mapped reads, and refine the selected genes only for MAG clusters present in at least --min_samples samples. In downstream abundance estimation, filter MAG clusters from samples where fewer than --min_mapped_signature_genes signature genes are detected, redistribute ambiguous mapping reads based on the uniquely mapped reads profile, and compute per-sample abundances on a reads-per-bp basis with the --abundance_calculation scheme ('sum', 'ot_trunc', or 'tt_trunc' with --tail-percentage). The rationale is that statistical modelling picks the ideal gene set suited to accurate abundance estimation rather than arbitrary marker selection.

## Related tools

- **MAGinator** (Ranks MAG-cluster-specific genes by co-abundance correlation, fits the rank-based negative binomial signature gene model to refine the gene set, and writes signature_genes_cluster.tsv plus the abundance table) — https://github.com/Russel88/MAGinator

## Examples

```
maginator -v vamb_clusters.tsv -r reads.csv -c contigs.fasta -o my_output -g /path/to/GTDB-Tk/database/release220/
```

## Evaluation signals

- signature_genes_cluster.tsv exists and lists signature genes for each MAG cluster
- A MAG cluster is only analyzed in samples where at least --min_mapped_signature_genes signature genes are detected
- Signature gene refinement is performed only for MAG clusters present in at least --min_samples samples
- The selected gene set contains only non-redundant genes found in exactly one MAG cluster
- The diagnostic plot signature_genes/read-count_detected-genes.pdf is exported, showing expected vs detected signature gene counts under the negative binomial model

## Limitations

- A subset of signature genes defined in one cohort (COPSAC2010) was missing in the CHILD cohort, presumably causing abundance underestimation for a subset of samples — de novo signature genes may not transfer across cohorts.
- MAG clusters absent from fewer than --min_samples samples cannot undergo signature gene refinement.
- Detection noise: a high number of detected signature genes may introduce noise or misclassification influencing beta diversity estimation.

## Evidence

- [intro] Identify and refine de novo signature genes: "This is done by identifying a set of signature genes directly from the given data and refining them according to statistical modelling to pick the ideal set suitable for abundance estimation"
- [readme] Statistical model for best genes: "MAGinator uses a statistical model to find the best genes for calculating accurate abundances"
- [methods] Min mapped signature genes filter: "Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis"
- [methods] Min samples for refinement: "Use --min_samples to alter the number of samples with the MAG cluster present in order to perform signature gene refinement"
