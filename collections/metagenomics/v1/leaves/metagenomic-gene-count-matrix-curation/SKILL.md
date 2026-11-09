---
name: metagenomic-gene-count-matrix-curation
description: 'Use when profiling de novo MAG clusters from shotgun metagenomics requires
  cluster-specific signature genes: you have a gene read-count matrix (small_gene_count_matrix.tsv)
  and MAG cluster assignments (metagenomicspecies.tab) and need abundance-grade gene
  sets.'
when_to_use_negative:
- Genes of interest cluster across multiple MAG clusters (the matrix must be restricted
  to genes found in only one MAG cluster, so a redundant/full count matrix needs upstream
  de-redundancy first)
- A MAG cluster is present in fewer than --min_samples samples — signature gene refinement
  is skipped and the initial set cannot be statistically refined
- Only reference-based marker gene profiling is needed (e.g. MetaPhlAn, MetaPhyler,
  mOTUs) and no de novo MAG clusters or strain-level phylogenies are required
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MAGinator
  role: Curates the MAG-cluster-specific gene matrix, selects and refines de novo
    signature genes via the rank-based negative binomial model, and computes per-sample
    abundances
  repo: https://github.com/Russel88/MAGinator
- name: Vamb
  role: Upstream binner producing the clusters.tsv MAG bins whose cluster assignments
    feed the signature gene curation
  repo: https://github.com/RasmussenLab/vamb
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:1f6e11f55bb009149b878355825f82da8df84975199d821612688199c36be45b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  - Vamb
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metagenomic-gene-count-matrix-curation

## Summary

Curation of a MAG-cluster-specific gene count matrix and statistical selection/refinement of de novo signature genes, as implemented in MAGinator: non-redundant genes are ranked by co-abundance and refined against a rank-based negative binomial model to yield an ideal set for MAG cluster abundance estimation.

## When to use

Use when profiling de novo MAG clusters from shotgun metagenomics requires cluster-specific signature genes: you have a gene read-count matrix (small_gene_count_matrix.tsv) and MAG cluster assignments (metagenomicspecies.tab) and need abundance-grade gene sets.

## When NOT to use

- Genes of interest cluster across multiple MAG clusters (the matrix must be restricted to genes found in only one MAG cluster, so a redundant/full count matrix needs upstream de-redundancy first)
- A MAG cluster is present in fewer than --min_samples samples — signature gene refinement is skipped and the initial set cannot be statistically refined
- Only reference-based marker gene profiling is needed (e.g. MetaPhlAn, MetaPhyler, mOTUs) and no de novo MAG clusters or strain-level phylogenies are required

## Inputs

- small_gene_count_matrix.tsv — read-count matrix of genes that do not cluster across MAG clusters
- metagenomicspecies.tab — MAG (metagenomic species) cluster assignments
- clusters.tsv from VAMB (upstream, via MAGinator -v)
- comma-separated reads.csv with SampleName,PathToForwardReads,PathToReverseReads (for read mapping upstream)

## Outputs

- signature_genes_cluster.tsv — signature gene assignments per MAG cluster
- abundance/abundance_phyloseq.RData — absolute abundance table (reads-per-bp basis)
- read-count_detected-genes.pdf — signature gene optimization diagnostics from the signature_genes directory

## How to apply

Load the small_gene_count_matrix.tsv read-count matrix, which contains only genes that do not cluster across MAG clusters (produced by the upstream read-mapping and gene-counting step), together with the MAG cluster assignments from metagenomicspecies.tab. Restrict the matrix to non-redundant genes found in only one MAG cluster each to curate a MAG-cluster-specific gene set. Within each MAG cluster, rank genes by co-abundance correlation across samples and select an initial signature gene set sized by --num_signature_genes. Fit the rank-based negative binomial signature gene model — which models the expected number of detected signature genes given the number of reads mapping to those genes — and refine the selected genes, performing refinement only for MAG clusters present in at least --min_samples samples. Filter MAG clusters from samples where fewer than --min_mapped_signature_genes signature genes are detected, then compute per-sample abundances on a reads-per-bp basis with --abundance_calculation ('sum', 'ot_trunc', or 'tt_trunc' with --tail-percentage) after redistributing ambiguous mapping reads using the uniquely mapped reads profile.

## Related tools

- **MAGinator** (Curates the MAG-cluster-specific gene matrix, selects and refines de novo signature genes via the rank-based negative binomial model, and computes per-sample abundances) — https://github.com/Russel88/MAGinator
- **Vamb** (Upstream binner producing the clusters.tsv MAG bins whose cluster assignments feed the signature gene curation) — https://github.com/RasmussenLab/vamb

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out --max_mem 180
```

## Evaluation signals

- signature_genes_cluster.tsv exists and lists signature genes for each MAG cluster
- A MAG cluster is analyzed only in samples where at least --min_mapped_signature_genes signature genes are detected
- Every gene retained in the curated matrix maps to exactly one MAG cluster (non-redundant across clusters)
- The initial signature gene set size matches --num_signature_genes before refinement
- read-count_detected-genes.pdf diagnostics show the negative binomial fit of detected-gene counts versus read counts for refined clusters

## Limitations

- MAGinator may introduce noise or misclassification that influences estimation of beta diversity (from high completeness)
- A subset of signature genes defined in one cohort (COPSAC2010) was missing in another (CHILD cohort), presumably causing abundance underestimation for a subset of samples
- Signature gene refinement is only performed for MAG clusters present in at least --min_samples samples, leaving others with unrefined sets

## Evidence

- [methods] negative binomial statistical model: [evidence span withheld — anchor status: not_found]
- [intro] signature gene identification rationale: "This is done by identifying a set of signature genes directly from the given data and refining them according to statistical modelling to pick the ideal set suitable for abundance estimation"
- [readme] statistical model claim: "MAGinator uses a statistical model to find the best genes for calculating accurate abundances"
- [methods] min_mapped_signature_genes filter: "Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis"
- [methods] min_samples refinement filter: "Use --min_samples to alter the number of samples with the MAG cluster present in order to perform signature gene refinement"
