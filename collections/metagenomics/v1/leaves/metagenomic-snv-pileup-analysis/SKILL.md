---
name: metagenomic-snv-pileup-analysis
description: Use when you have VAMB-binned MAGs plus original shotgun reads and need
  to quantify within-sample strain diversity — i.e. which MAG clusters contain multiple
  variants (mixed alleles) per sample across one or more environments.
when_to_use_negative:
- Input is a single isolate genome or reference-based strain profiling is sufficient
  — MAGinator's mixed-allele detection is designed for de novo MAG clusters recovered
  from shotgun metagenomes
- You only need species-level taxonomic composition — marker-gene profilers such as
  MetaPhlAn or mOTUs are the faster route and MAGinator's SNV phylogenies add unnecessary
  compute
- MAG clusters contain too few samples for signature-gene refinement (controlled via
  --min_samples), making per-cluster SNV statistics unreliable
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: MAGinator
  role: Runs the Snakemake pipeline that maps reads to the non-redundant gene catalogue,
    identifies de novo signature genes, and collects per-cluster/per-sample SNV pileups
    used to call mixed alleles
  repo: https://github.com/Russel88/MAGinator
- name: Snakemake
  role: Workflow engine MAGinator is implemented in, providing integrated work distribution
    for processing large-scale metagenomics data
  repo: https://github.com/Russel88/MAGinator
- name: Vamb
  role: Produces the clusters.tsv MAG binning input that defines the MAG clusters
    whose SNVs are profiled
  repo: https://github.com/RasmussenLab/vamb
- name: dRep
  role: Recommended to create the species-level clusters from binned genomes before
    setting up MAGinator input files
  repo: https://github.com/MrOlm/drep
- name: GTDB-tk database (release 220)
  role: Taxonomic database used to annotate bins and call ORFs during the MAGinator
    run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:0d7d35419893540e32234cfc1925e971de60920ec74552871acf32db291239a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - GTDB-tk database (release 220)
  - MAGinator
  - Snakemake
  - Vamb
  - dRep
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metagenomic-snv-pileup-analysis

## Summary

Detects strain mixtures within de novo MAG clusters by profiling single-nucleotide variants in signature and marker genes with MAGinator, then classifying MAG clusters as mixed-allele when samples harbour multiple alleles above the allele-frequency cutoff (e.g. 4154 of 20,765 MAGs in COPSAC2010; 168 of 195 clusters in the honey bee gut; ~68% in Tara Oceans).

## When to use

Use when you have VAMB-binned MAGs plus original shotgun reads and need to quantify within-sample strain diversity — i.e. which MAG clusters contain multiple variants (mixed alleles) per sample across one or more environments.

## When NOT to use

- Input is a single isolate genome or reference-based strain profiling is sufficient — MAGinator's mixed-allele detection is designed for de novo MAG clusters recovered from shotgun metagenomes
- You only need species-level taxonomic composition — marker-gene profilers such as MetaPhlAn or mOTUs are the faster route and MAGinator's SNV phylogenies add unnecessary compute
- MAG clusters contain too few samples for signature-gene refinement (controlled via --min_samples), making per-cluster SNV statistics unreliable

## Inputs

- VAMB clusters.tsv mapping contigs to MAG bins
- Contigs FASTA with unique sequence names
- reads.csv manifest (SampleName,PathToForwardReads,PathToReverseReads)
- GTDB-tk release 220 database (gtdbtk_r220_data.tar.gz)

## Outputs

- phylo/stats.tab per run (non-N fraction, signature/marker gene counts, read depth, bases below allele-frequency cutoff per MAG cluster)
- phylo/pileup SNV information per MAG cluster and sample
- mixed_allele_summary.tsv with counts and percentages of mixed-allele MAG clusters per environment

## How to apply

Assemble MAGinator inputs per environment (VAMB clusters.tsv, a contigs fasta with unique contig names, and a reads.csv manifest of SampleName, forward and reverse fastq paths) and run MAGinator with the GTDB-tk release 220 database so reads map to the non-redundant gene catalogue and SNV information is collected per MAG cluster and sample. Parse the phylo/stats.tab output, which reports the non-N fraction, number of signature and marker genes, read depth, and number of bases not reaching the allele-frequency cutoff for each MAG cluster. Using the phylo/pileup SNV information, classify a MAG cluster as mixed-allele when its signature or marker genes show multiple alleles above the allele-frequency cutoff in at least one sample. Count mixed-allele MAG clusters per environment, compute percentages, and validate against reported values (COPSAC2010 4154/20,765 MAGs; honey bee gut 168/195 clusters, 86% proportion of clusters with mixed alleles; Tara Oceans 68% vs 46% in COPSAC2010).

## Related tools

- **MAGinator** (Runs the Snakemake pipeline that maps reads to the non-redundant gene catalogue, identifies de novo signature genes, and collects per-cluster/per-sample SNV pileups used to call mixed alleles) — https://github.com/Russel88/MAGinator
- **Snakemake** (Workflow engine MAGinator is implemented in, providing integrated work distribution for processing large-scale metagenomics data) — https://github.com/Russel88/MAGinator
- **Vamb** (Produces the clusters.tsv MAG binning input that defines the MAG clusters whose SNVs are profiled) — https://github.com/RasmussenLab/vamb
- **dRep** (Recommended to create the species-level clusters from binned genomes before setting up MAGinator input files) — https://github.com/MrOlm/drep
- **GTDB-tk database (release 220)** (Taxonomic database used to annotate bins and call ORFs during the MAGinator run)

## Examples

```
maginator -v vamb_clusters.tsv -r reads.csv -c contigs.fasta -o my_output -g "/path/to/GTDB-Tk/database/release220/"
```

## Evaluation signals

- Recomputed COPSAC2010 count matches 4154 mixed-allele MAGs out of 20,765 MAGs (Supplementary Table 5)
- Honey bee gut run yields 168 of 195 MAG clusters with at least one mixed-allele sample (~86% proportion)
- Tara Oceans run yields ~68% of clusters with mixed alleles, correctly higher than COPSAC2010's 46%
- phylo/stats.tab contains non-empty non-N fraction, signature/marker gene counts and read depth for every MAG cluster in the analysis
- mixed_allele_summary.tsv row counts and per-environment percentages match the recomputed counts (internal consistency check)

## Limitations

- High completeness of MAGinator may introduce noise or misclassification, which influences beta diversity estimation
- A subset of signature genes defined in COPSAC2010 was missing in the CHILD cohort, presumably causing abundance underestimation for a subset of samples
- Users eliminating SNV/synteny noise via parameter changes do so at the expense of sensitivity

## Evidence

- [intro] MAGinator also enables the creation of Single Nucleotide Variant (SNV's) resolution phylogenetic trees from the signature genes.: "MAGinator also enables the creation of Single Nucleotide Variant (SNV’s) resolution phylogenetic trees from the signature genes."
- [discussion] While the honey-bee gut exhibits the highest proportion of MAG clusters with mixed alleles (86%): "While the honey-bee gut exhibits the highest proportion of MAG clusters with mixed alleles (86%)"
- [discussion] the Tara Oceans exhibit a higher proportion (68%) compared to that of COPSAC2010 (46%): "the Tara Oceans exhibit a higher proportion (68%) compared to that of COPSAC2010 (46%)"
- [methods] Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis: "Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis"
- [readme] MAGinator creates a phylogenetic tree for each species so you can associate your metadata with subspecies/strain level differences: "MAGinator creates a phylogenetic tree for each species so you can associate your metadata with subspecies/strain level differences"
