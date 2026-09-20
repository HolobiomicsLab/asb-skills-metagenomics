---
name: co-abundance-correlation-ranking
description: Use when estimating abundances of de novo MAG clusters from shotgun metagenomics
  and needing to pick cluster-specific signature genes from a non-redundant gene count
  matrix before read-based abundance computation.
when_to_use_negative:
- Input is a database of pre-derived strain marker genes rather than de novo signature
  genes identified from the given data (cross-cohort transfer of signature genes can
  underestimate abundance, as seen for COPSAC2010 genes in the CHILD cohort).
- The MAG cluster is present in fewer than --min_samples samples, since signature
  gene refinement is not performed in that case.
- Fewer than --min_mapped_signature_genes signature genes are detected in a sample
  — the MAG cluster should be filtered from the analysis instead.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3308
tools:
- name: MAGinator
  role: Implements co-abundance correlation ranking of genes within MAG clusters and
    the rank-based negative binomial refinement of de novo signature genes used for
    abundance estimation
  repo: https://github.com/Russel88/MAGinator
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:8beb8b5ec725d9e5fd028cd1dd65025a0a4d2e70a8d31b81c9d48c5d9a17606f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Co-abundance correlation ranking

## Summary

Ranks genes within each MAG cluster by co-abundance correlation across samples and selects an initial de novo signature gene set, then refines it with a rank-based negative binomial model of expected detected genes to enable accurate per-sample MAG abundance estimation.

## When to use

Use when estimating abundances of de novo MAG clusters from shotgun metagenomics and needing to pick cluster-specific signature genes from a non-redundant gene count matrix before read-based abundance computation.

## When NOT to use

- Input is a database of pre-derived strain marker genes rather than de novo signature genes identified from the given data (cross-cohort transfer of signature genes can underestimate abundance, as seen for COPSAC2010 genes in the CHILD cohort).
- The MAG cluster is present in fewer than --min_samples samples, since signature gene refinement is not performed in that case.
- Fewer than --min_mapped_signature_genes signature genes are detected in a sample — the MAG cluster should be filtered from the analysis instead.

## Inputs

- small_gene_count_matrix.tsv (read count matrix of genes that do not cluster across MAG clusters)
- metagenomicspecies.tab (MAG/MGS cluster assignments)
- VAMB clusters.tsv file
- reads.csv (SampleName,PathToForwardReads,PathToReverseReads)
- contigs fasta with unique names

## Outputs

- signature_genes_cluster.tsv (signature gene assignments per MAG cluster)
- abundance/abundance_phyloseq.RData (absolute abundance table)
- read-count_detected-genes.pdf (signature gene optimization diagnostics)

## How to apply

Load the small_gene_count_matrix.tsv read count matrix (genes that do not cluster across MAG clusters) together with MAG cluster assignments from metagenomicspecies.tab, and restrict it to non-redundant genes found in only one MAG cluster each to curate a MAG-cluster-specific gene set. Within each MAG cluster, rank genes by co-abundance correlation across samples and select the initial signature gene set sized by --num_signature_genes. Fit the rank-based negative binomial signature gene model — which models the expected number of detected signature genes given the number of reads mapping to them — and refine the selected genes, performing refinement only for MAG clusters present in at least --min_samples samples. After refinement, compute per-sample MAG cluster abundances on a reads-per-bp basis with the --abundance_calculation scheme ('sum', 'ot_trunc', or 'tt_trunc' with --tail-percentage), having redistributed ambiguous mapping reads based on the uniquely mapped reads profile. The rationale is that statistically refined signature genes are the 'ideal set suitable for abundance estimation' of phylogenetically separated MAG clusters.

## Related tools

- **MAGinator** (Implements co-abundance correlation ranking of genes within MAG clusters and the rank-based negative binomial refinement of de novo signature genes used for abundance estimation) — https://github.com/Russel88/MAGinator

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out --max_mem 180
```

## Evaluation signals

- signature_genes_cluster.tsv exists and lists signature genes for each MAG cluster
- A MAG cluster is only analyzed in samples where at least --min_mapped_signature_genes signature genes are detected
- Signature gene refinement is only applied to MAG clusters present in at least --min_samples samples
- The exported read-count_detected-genes.pdf diagnostics show the fit of the negative binomial model relating mapped reads to detected signature gene counts
- Fidelity of resulting abundances can be benchmarked on the CAMI strain-madness dataset, as done in the article

## Limitations

- Refinement is skipped for MAG clusters present in fewer than --min_samples samples
- A subset of signature genes defined in one cohort (COPSAC2010) was missing when applied to another cohort (CHILD), presumably causing abundance underestimation for a subset of samples
- MAGinator lacks operon awareness, which can limit downstream accuracy (synteny clustering), and users altering parameters to eliminate noise do so at the expense of sensitivity

## Evidence

- [intro] This is done by identifying a set of signature genes directly from the given data and refining them according to statistical modelling to pick the ideal set suitable for abundance estimation: "This is done by identifying a set of signature genes directly from the given data and refining them according to statistical modelling to pick the ideal set suitable for abundance estimation"
- [readme] MAGinator uses a statistical model to find the best genes for calculating accurate abundances: "MAGinator uses a statistical model to find the best genes for calculating accurate abundances"
- [methods] Use --min_samples to alter the number of samples with the MAG cluster present in order to perform signature gene refinement: "Use --min_samples to alter the number of samples with the MAG cluster present in order to perform signature gene refinement"
- [methods] Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis: "Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis"
