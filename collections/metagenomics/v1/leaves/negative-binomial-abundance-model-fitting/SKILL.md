---
name: negative-binomial-abundance-model-fitting
description: Use when estimating per-sample abundances of MAG clusters from shotgun
  metagenomics and you need to pick an optimal, statistically validated set of de
  novo signature genes from a gene count matrix.
when_to_use_negative:
- When MAGs already have database-derived strain marker genes — the article notes
  a similar cross-cohort transfer phenomenon would be expected, and de novo signature
  genes from one cohort can be missing in another, causing abundance underestimation
- When the input is raw reads/contigs rather than an upstream read-mapping gene count
  matrix — signature gene identification requires the read-mapping and gene-counting
  step to have been completed first
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: MAGinator
  role: Implements the negative binomial signature gene model, gene selection/refinement,
    filtering, and abundance estimation pipeline
  repo: https://github.com/Russel88/MAGinator
- name: Snakemake
  role: Workflow engine that distributes and executes the signature gene identification
    and abundance estimation steps
  repo: https://github.com/Russel88/MAGinator
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:e984abb4da173a94f94df14480c7c80717f341871c9941905fe3262fdf878b1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  - Snakemake
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# negative-binomial-abundance-model-fitting

## Summary

Fit a rank-based negative binomial model of expected detected signature genes versus mapped read counts to select and refine de novo signature genes for MAGinator MAG-cluster abundance estimation.

## When to use

Use when estimating per-sample abundances of MAG clusters from shotgun metagenomics and you need to pick an optimal, statistically validated set of de novo signature genes from a gene count matrix.

## When NOT to use

- When MAGs already have database-derived strain marker genes — the article notes a similar cross-cohort transfer phenomenon would be expected, and de novo signature genes from one cohort can be missing in another, causing abundance underestimation
- When the input is raw reads/contigs rather than an upstream read-mapping gene count matrix — signature gene identification requires the read-mapping and gene-counting step to have been completed first

## Inputs

- small_gene_count_matrix.tsv read count matrix (genes that do not cluster across MAG clusters)
- MAG cluster assignments from metagenomicspecies.tab
- parameters: --num_signature_genes, --min_samples, --min_mapped_signature_genes, --abundance_calculation ('sum', 'ot_trunc', or 'tt_trunc'), --tail-percentage

## Outputs

- signature_genes_cluster.tsv (signature gene assignments per MAG cluster)
- abundance/abundance_phyloseq.RData (absolute abundance table)
- signature_genes/read-count_detected-genes.pdf (signature gene optimization diagnostics)

## How to apply

Load the small_gene_count_matrix.tsv read count matrix (containing only genes that do not cluster across MAG clusters) together with MAG cluster assignments from metagenomicspecies.tab, and restrict it to non-redundant genes found in only one MAG cluster each. Within each MAG cluster, rank genes by co-abundance correlation across samples and select an initial signature gene set sized by --num_signature_genes. Then fit the rank-based negative binomial signature gene model — which models the expected number of detected signature genes in a sample from the number of reads mapping to those genes — and refine the selected gene set, performing refinement only for MAG clusters present in at least --min_samples samples. Drop MAG clusters from a sample when fewer than --min_mapped_signature_genes signature genes are detected, then compute per-sample abundances on a reads-per-bp basis using --abundance_calculation ('sum', 'ot_trunc', or 'tt_trunc' with --tail-percentage) after redistributing ambiguous mapping reads according to the uniquely mapped read profile.

## Related tools

- **MAGinator** (Implements the negative binomial signature gene model, gene selection/refinement, filtering, and abundance estimation pipeline) — https://github.com/Russel88/MAGinator
- **Snakemake** (Workflow engine that distributes and executes the signature gene identification and abundance estimation steps) — https://github.com/Russel88/MAGinator

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out --max_mem 180
```

## Evaluation signals

- signature_genes_cluster.tsv exists and lists signature genes for each MAG cluster
- A MAG cluster is only analyzed in samples where at least --min_mapped_signature_genes signature genes are detected
- Signature gene refinement is only performed for MAG clusters present in at least --min_samples samples
- abundance/abundance_phyloseq.RData loads and contains per-sample MAG cluster abundances on a reads-per-bp basis
- The exported read-count_detected-genes.pdf diagnostics show the negative binomial fit between read counts and detected signature gene numbers

## Limitations

- A subset of signature genes defined in COPSAC2010 was missing in the CHILD cohort, presumably causing abundance underestimation for a subset of the samples
- High completeness of MAGinator may introduce noise or misclassification affecting beta diversity estimation
- Ambiguous mapping reads must be redistributed based on the uniquely mapped reads profile, which relies on that profile being informative

## Evidence

- [methods] NB model basis: [evidence span withheld — anchor status: not_found]
- [intro] de novo selection rationale: "This is done by identifying a set of signature genes directly from the given data and refining them according to statistical modelling to pick the ideal set suitable for abundance estimation"
- [methods] min_samples refinement filter: "Use --min_samples to alter the number of samples with the MAG cluster present in order to perform signature gene refinement"
- [methods] min_mapped_signature_genes filter: "Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis"
- [readme] README statistical model statement: "MAGinator uses a statistical model to find the best genes for calculating accurate abundances"
