---
name: mixed-allele-frequency-statistics
description: Use when you have run (or plan to run) MAGinator on shotgun metagenomics
  data with VAMB MAG clusters and need to quantify within-cluster strain mixtures
  — i.e. which samples/MAG clusters carry more than one variant of a MAG.
when_to_use_negative:
- The goal is species-level taxonomic profiling or abundance estimation only, not
  strain-level within-cluster variant diversity
- Samples fail the --min_mapped_signature_genes detection threshold, so their SNV
  statistics cannot be included in the phylogenetic/mixed-allele analysis
- No MAG clusters exist (e.g. input is only a marker-gene profile from MetaPhlAn/mOTUs)
  — there are no signature genes to collect SNV information from
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MAGinator
  role: Maps reads to the non-redundant gene catalogue, collects SNV information per
    MAG cluster and sample, and emits phylo/stats.tab, phylo/stats_genes.tab and phylo/pileup
    used for mixed-allele classification
  repo: https://github.com/Russel88/MAGinator
- name: Snakemake
  role: Workflow engine in which MAGinator is implemented, distributing the pileup
    and statistics steps over samples
  repo: https://github.com/Russel88/MAGinator
- name: Vamb
  role: Produces the clusters.tsv MAG cluster input files that define the MAG clusters
    whose allele mixtures are assessed
  repo: https://github.com/RasmussenLab/vamb
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:a5823be7bee91bd3844eaa8e3b6ca7c5206e0778d170c7c597371859dbfe052e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - FastTree
  - IQ-TREE
  - MAFFT
  - MAGinator
  - Samtools
  - Snakemake
  - Vamb
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mixed-allele-frequency-statistics

## Summary

Identify MAG clusters harbouring multiple intra-cluster variants ('mixed alleles') by parsing MAGinator's per-sample SNV statistics (phylo/stats.tab, phylo/pileup) and classifying clusters whose signature or marker genes show multiple alleles above the allele frequency cutoff in at least one sample.

## When to use

Use when you have run (or plan to run) MAGinator on shotgun metagenomics data with VAMB MAG clusters and need to quantify within-cluster strain mixtures — i.e. which samples/MAG clusters carry more than one variant of a MAG.

## When NOT to use

- The goal is species-level taxonomic profiling or abundance estimation only, not strain-level within-cluster variant diversity
- Samples fail the --min_mapped_signature_genes detection threshold, so their SNV statistics cannot be included in the phylogenetic/mixed-allele analysis
- No MAG clusters exist (e.g. input is only a marker-gene profile from MetaPhlAn/mOTUs) — there are no signature genes to collect SNV information from

## Inputs

- VAMB clusters.tsv (MAG cluster assignments)
- Contigs FASTA with unique sequence names
- reads.csv manifest (SampleName,PathToForwardReads,PathToReverseReads)
- Per-sample BAM files under mapped_reads/bams/
- GTDB-tk release 220 database
- phylo/stats.tab and phylo/pileup SNV output from MAGinator

## Outputs

- Per-MAG-cluster mixed-allele classification (mixed vs. non-mixed per sample)
- Aggregated per-environment summary table (e.g. mixed_allele_summary.tsv) with counts and percentages
- phylo/stats.tab: non-N fraction, signature/marker gene counts, read depth, bases not reaching allele frequency cutoff
- phylo/stats_genes.tab: per-gene breakdown of SNV statistics

## How to apply

Run MAGinator on the VAMB clusters.tsv, contigs fasta (unique names) and reads.csv manifest with the GTDB-tk release 220 database so reads are mapped to the non-redundant gene catalogue and SNVs are collected per MAG cluster and sample via Samtools mpileup on mapped_reads/bams/. Parse phylo/stats.tab, which reports per MAG cluster the non-N fraction, number of signature and marker genes, read depth, and number of bases not reaching the allele frequency cutoff (per-gene breakdown in phylo/stats_genes.tab). Restrict the analysis to samples passing the --min_mapped_signature_genes detection threshold. Using the phylo/pileup SNV information, classify a MAG cluster as mixed-allele when its signature or marker genes show multiple alleles above the allele frequency cutoff in at least one sample; count mixed-allele MAG clusters per environment and write the aggregated table (e.g. mixed_allele_summary.tsv) with counts and percentages.

## Related tools

- **MAGinator** (Maps reads to the non-redundant gene catalogue, collects SNV information per MAG cluster and sample, and emits phylo/stats.tab, phylo/stats_genes.tab and phylo/pileup used for mixed-allele classification) — https://github.com/Russel88/MAGinator
- **Snakemake** (Workflow engine in which MAGinator is implemented, distributing the pileup and statistics steps over samples) — https://github.com/Russel88/MAGinator
- **Vamb** (Produces the clusters.tsv MAG cluster input files that define the MAG clusters whose allele mixtures are assessed) — https://github.com/RasmussenLab/vamb

## Examples

```
maginator -v vamb_clusters.tsv -r reads.csv -c contigs.fasta -o my_output -g "/path/to/GTDB-Tk/database/release220/"
```

## Evaluation signals

- phylo/stats.tab exists and contains, for each MAG cluster, the non-N fraction, number of signature and marker genes, read depth, and number of bases not reaching the allele frequency cutoff
- A MAG cluster is only called mixed-allele when multiple alleles above the allele frequency cutoff appear in at least one sample in phylo/pileup
- Recomputed COPSAC2010 counts match 4154 of 20,765 MAGs with mixed alleles (Supplementary Table 5)
- Honey bee gut result: 168 of 195 clusters with at least one mixed-allele sample (~70% average; article reports 86% of MAG clusters with mixed alleles)
- Tara Oceans cluster-level mixed-allele proportion reproduces the reported value (37% per recomputation; article reports 68% of clusters vs. 46% for COPSAC2010)

## Limitations

- MAGinator's high completeness may introduce noise or misclassification that influences beta diversity and strain-level interpretation
- Signature genes defined de novo in one cohort (e.g. COPSAC2010) may be missing in another (e.g. CHILD cohort), presumably causing abundance underestimation for a subset of samples; a similar phenomenon is expected for database-derived strain marker genes
- Bases below the allele frequency cutoff are excluded, so low-depth samples may fail to reveal mixed alleles even when strains co-exist

## Evidence

- [intro] MAGinator also enables the creation of Single Nucleotide Variant (SNV’s) resolution phylogenetic trees from the signature genes.: "MAGinator also enables the creation of Single Nucleotide Variant (SNV’s) resolution phylogenetic trees from the signature genes."
- [discussion] mixed-allele proportions per environment: "While the honey-bee gut exhibits the highest proportion of MAG clusters with mixed alleles (86%)"
- [methods] sample inclusion threshold for phylogenetic analysis: "Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis"
- [readme] MAGinator output basis for SNV statistics: "MAGinator creates a phylogenetic tree for each species so you can associate your metadata with subspecies/strain level differences"
