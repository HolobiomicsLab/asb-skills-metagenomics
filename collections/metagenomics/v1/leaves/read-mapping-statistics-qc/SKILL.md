---
name: read-mapping-statistics-qc
description: Use when shotgun reads have been mapped back to de novo MAG clusters
  (e.g. VAMB bins via MAGinator) and you must QC mapping depth/allele-frequency statistics
  to flag MAG clusters with mixed strains per sample or environment.
when_to_use_negative:
- Input is already a computed abundance or mixed-allele summary table — no re-mapping
  or SNV pileup is needed.
- Data are not shotgun metagenomic reads mappable back to assembled MAGs (e.g. 16S
  amplicon data), since SNV-based mixed-allele classification requires read-level
  alleles on signature/marker genes.
- Read depth is so low that most bases fall below the allele frequency cutoff, making
  mixed-allele calls unreliable from stats.tab/pileup alone.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MAGinator
  role: Maps reads to the MAG-derived non-redundant gene catalogue, produces phylo/stats.tab
    and phylo/pileup SNV output used for mixed-allele QC and classification
  repo: https://github.com/Russel88/MAGinator
- name: Snakemake
  role: Workflow engine that runs and distributes the MAGinator mapping/statistics
    pipeline
- name: VAMB
  role: Produces the clusters.tsv MAG bin assignments that define the MAG clusters
    being QC'd
  repo: https://github.com/RasmussenLab/vamb
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:d01e2acd9b0f5c7ca9a979a4ca4d6f6b3e71dc38aa28b35cefd6b78ea94b6584
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  - Snakemake
  - VAMB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# read-mapping-statistics-qc

## Summary

QC and interpretation of MAGinator read-mapping statistics — the phylo/stats.tab (non-N fraction, signature/marker gene counts, read depth, bases not reaching the allele frequency cutoff) and phylo/pileup SNV information — to classify which MAG clusters harbour mixed alleles (multiple strain variants) in a sample.

## When to use

Use when shotgun reads have been mapped back to de novo MAG clusters (e.g. VAMB bins via MAGinator) and you must QC mapping depth/allele-frequency statistics to flag MAG clusters with mixed strains per sample or environment.

## When NOT to use

- Input is already a computed abundance or mixed-allele summary table — no re-mapping or SNV pileup is needed.
- Data are not shotgun metagenomic reads mappable back to assembled MAGs (e.g. 16S amplicon data), since SNV-based mixed-allele classification requires read-level alleles on signature/marker genes.
- Read depth is so low that most bases fall below the allele frequency cutoff, making mixed-allele calls unreliable from stats.tab/pileup alone.

## Inputs

- VAMB clusters.tsv (MAG bin assignments)
- contigs fasta with unique sequence names
- reads.csv manifest (SampleName,PathToForwardReads,PathToReverseReads)
- GTDB-tk release 220 database
- MAGinator phylo/stats.tab output
- MAGinator phylo/pileup SNV information

## Outputs

- mixed_allele_summary.tsv (per-environment mixed-allele MAG counts and percentages)
- per-cluster QC metrics (non-N fraction, signature/marker gene counts, read depth, bases below allele frequency cutoff)
- per-sample mixed-allele flags for MAG clusters

## How to apply

Assemble MAGinator inputs (VAMB clusters.tsv, contigs fasta with unique names, reads.csv manifest) and run MAGinator with the GTDB-tk release 220 database so reads are mapped to the non-redundant gene catalogue and SNV information is collected per MAG cluster and sample. Parse phylo/stats.tab, which reports the non-N fraction, number of signature and marker genes, read depth, and the number of bases not reaching the allele frequency cutoff for each MAG cluster. Using the phylo/pileup SNV information, classify a MAG cluster as mixed-allele when its signature or marker genes show multiple alleles above the allele frequency cutoff in at least one sample. Count mixed-allele MAG clusters per environment, compute percentages, write mixed_allele_summary.tsv, and validate against the reported per-environment targets (COPSAC2010 4154/20,765 MAGs; honey bee gut 168/195 clusters; Tara Oceans 37%).

## Related tools

- **MAGinator** (Maps reads to the MAG-derived non-redundant gene catalogue, produces phylo/stats.tab and phylo/pileup SNV output used for mixed-allele QC and classification) — https://github.com/Russel88/MAGinator
- **Snakemake** (Workflow engine that runs and distributes the MAGinator mapping/statistics pipeline)
- **VAMB** (Produces the clusters.tsv MAG bin assignments that define the MAG clusters being QC'd) — https://github.com/RasmussenLab/vamb

## Examples

```
maginator -v vamb_clusters.tsv -r reads.csv -c contigs.fasta -o my_output -g "/path/to/GTDB-Tk/database/release220/"
```

## Evaluation signals

- Recomputed mixed-allele counts match reported per-environment targets: 4154 of 20,765 MAGs in COPSAC2010, 168 of 195 clusters in the honey bee gut, and ~37% across Tara Oceans clusters.
- phylo/stats.tab is parsed without missing columns and every MAG cluster has values for non-N fraction, signature/marker gene counts, read depth, and bases not reaching the allele frequency cutoff.
- A MAG cluster is flagged mixed-allele only when signature or marker genes show multiple alleles above the allele frequency cutoff in at least one sample — spot-check a few clusters against the pileup.
- Percentages written to mixed_allele_summary.tsv are consistent with the article's reported proportions (honey bee gut ~86%, Tara Oceans 68% vs COPSAC2010 46% of clusters).
- Counts are reproduced independently across the three environments using the same allele-frequency cutoff, confirming pipeline parameter consistency.

## Limitations

- High completeness of MAGinator may introduce a certain level of noise or misclassification, which influences downstream estimation (e.g. beta diversity).
- Mixed-allele classification depends on the allele frequency cutoff and read depth; low-depth samples or MAG clusters with few covered signature/marker gene bases may be misclassified.
- Signature-gene transfer across cohorts can cause underestimation (e.g. a subset of COPSAC2010 signature genes was missing in the CHILD cohort), so cross-dataset QC comparisons may be biased.

## Evidence

- [intro] MAGinator also enables the creation of Single Nucleotide Variant (SNV’s) resolution phylogenetic trees from the signature genes: "MAGinator also enables the creation of Single Nucleotide Variant (SNV’s) resolution phylogenetic trees from the signature genes."
- [discussion] honey-bee gut exhibits the highest proportion of MAG clusters with mixed alleles: "While the honey-bee gut exhibits the highest proportion of MAG clusters with mixed alleles (86%)"
- [discussion] the Tara Oceans exhibit a higher proportion (68%) compared to that of COPSAC2010 (46%): "the Tara Oceans exhibit a higher proportion (68%) compared to that of COPSAC2010 (46%)"
- [readme] SNV-level resolution phylogenetic trees based on signature genes: "* SNV-level resolution phylogenetic trees based on signature genes"
