---
name: species-abundance-variance-explained-analysis
description: Use when you need to check whether MAGinator-estimated B. infantis/B.
  longum subspecies abundances from signature genes faithfully track independently
  profiled species-level abundance (e.g., MetaPhlAn B. longum) across cohorts.
when_to_use_negative:
- Input is already a species-level profile table and no de novo MAG or signature-gene
  abundances are involved — plain regression against reference profiles suffices without
  MAGinator mapping.
- The MAG cluster of interest lacks a sufficient set of refined signature genes, or
  samples fall below the --min_mapped_signature_genes detection threshold, making
  the fit/regression unstable.
- Cross-cohort transfer is the goal but the target cohort's gene catalogue is missing
  a subset of the donor-derived signature genes, which the paper notes causes abundance
  underestimation for a subset of samples.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MAGinator
  role: Maps reads to the signature-gene catalogue and produces per-sample subspecies
    abundance estimates via signature-read redistribution
  repo: https://github.com/Russel88/MAGinator
- name: MetaPhlAn
  role: Provides the independent species-level B. longum abundance reference against
    which summed subspecies abundances are regressed
- name: StrainPhlAn
  role: Assigns samples to the two mutually exclusive subspecies clusters used to
    cross-tabulate against MAGinator abundance estimates
- name: Snakemake
  role: Workflow engine implementing MAGinator's steps (read mapping, signature-gene
    fitting, phylogenetic analysis)
  repo: https://github.com/Russel88/MAGinator
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:52092ad0e2e6537c8d7bfaf4e4ab00ed4a31a10a3dfaef51ecfb2b81d57ec9e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  - MetaPhlAn
  - Snakemake
  - StrainPhlAn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# species-abundance-variance-explained-analysis

## Summary

Validates de novo signature-gene-based subspecies abundance estimates (MAGinator) by (a) measuring per-cohort fit MSE of the signature-gene model and (b) regressing the summed subspecies abundance trajectories against MetaPhlAn species-level abundances to quantify explained variance (R-squared).

## When to use

Use when you need to check whether MAGinator-estimated B. infantis/B. longum subspecies abundances from signature genes faithfully track independently profiled species-level abundance (e.g., MetaPhlAn B. longum) across cohorts.

## When NOT to use

- Input is already a species-level profile table and no de novo MAG or signature-gene abundances are involved — plain regression against reference profiles suffices without MAGinator mapping.
- The MAG cluster of interest lacks a sufficient set of refined signature genes, or samples fall below the --min_mapped_signature_genes detection threshold, making the fit/regression unstable.
- Cross-cohort transfer is the goal but the target cohort's gene catalogue is missing a subset of the donor-derived signature genes, which the paper notes causes abundance underestimation for a subset of samples.

## Inputs

- Shotgun metagenomics reads (FASTQ) from the COPSAC2010 and CHILD infant cohorts
- Non-redundant gene catalogue containing COPSAC2010-derived B. infantis and B. longum signature genes
- VAMB clusters.tsv file and contigs FASTA for MAGinator input
- MetaPhlAn species-level abundance table (B. longum)
- StrainPhlAn subspecies cluster assignments

## Outputs

- Per-sample B. infantis and B. longum subspecies abundance trajectories (MAGinator signature-gene model)
- Per-cohort signature-gene fit MSE values (COPSAC2010: 103.95; CHILD: 878.09)
- R-squared of summed subspecies abundances regressed on MetaPhlAn B. longum abundance (expected 87%)
- bifido_subspecies_validation_results.tsv with MSE, R-squared, and per-sample cluster/abundance assignments

## How to apply

Map shotgun reads to the non-redundant gene catalogue containing the COPSAC2010-derived B. infantis and B. longum signature genes with MAGinator's read-mapping step, which redistributes ambiguous reads based on uniquely mapping reads; require detection with --min_mapped_signature_genes. Fit the signature gene model per cohort to obtain per-sample subspecies abundances and compute the fit MSE (expected ~103.95 for COPSAC2010, ~878.09 for CHILD). Sum the two subspecies abundance trajectories per sample and regress them against the MetaPhlAn B. longum species abundance to quantify explained variance (expected R-squared ~87%). Independently, assign samples to the two mutually exclusive StrainPhlAn subspecies clusters and cross-tabulate cluster membership against MAGinator subspecies abundances to confirm the stratification. Write all metrics (per-cohort MSE, R-squared, per-sample cluster/abundance assignments) to a results table such as bifido_subspecies_validation_results.tsv.

## Related tools

- **MAGinator** (Maps reads to the signature-gene catalogue and produces per-sample subspecies abundance estimates via signature-read redistribution) — https://github.com/Russel88/MAGinator
- **MetaPhlAn** (Provides the independent species-level B. longum abundance reference against which summed subspecies abundances are regressed)
- **StrainPhlAn** (Assigns samples to the two mutually exclusive subspecies clusters used to cross-tabulate against MAGinator abundance estimates)
- **Snakemake** (Workflow engine implementing MAGinator's steps (read mapping, signature-gene fitting, phylogenetic analysis)) — https://github.com/Russel88/MAGinator

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out --max_mem 180
```

## Evaluation signals

- Recomputed signature-gene fit MSE equals 103.95 for COPSAC2010 and 878.09 for CHILD.
- R-squared of summed subspecies trajectories vs MetaPhlAn B. longum species abundance equals approximately 87%.
- StrainPhlAn clusters separate samples by MAGinator subspecies abundance consistent with the paper's Fig. 4 stratification.
- Per-sample output table (bifido_subspecies_validation_results.tsv) contains non-empty, mutually exclusive cluster assignments and abundance values for every included sample.
- Every sample included in the fit meets the --min_mapped_signature_genes detection criterion.

## Limitations

- The signature genes fit the COPSAC2010 cohort better (MSE 103.95) than the CHILD cohort (MSE 878.09), so cross-cohort transfer degrades fit.
- A subset of signature genes defined in COPSAC2010 was missing in the CHILD cohort, which presumably resulted in an underestimation of the abundance for a subset of the samples.
- High completeness of MAGinator may introduce noise or misclassification, which influences the estimation of beta diversity.

## Evidence

- [abstract] MAGinator provides de novo identification of subspecies-level microbes and accurate abundance estimates of metagenome-assembled genomes (MAGs).: "MAGinator provides de novo identification of subspecies-level microbes and accurate abundance estimates of metagenome-assembled genomes (MAGs)."
- [intro] This is done by identifying a set of signature genes directly from the given data and refining them according to statistical modelling to pick the ideal set suitable for abundance estimation: "This is done by identifying a set of signature genes directly from the given data and refining them according to statistical modelling to pick the ideal set suitable for abundance estimation"
- [methods] Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis: "Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis"
- [discussion] These results were reproduced in the CHILD cohort using the signature genes identified in COPSAC2010 for the two subspecies.: "These results were reproduced in the CHILD cohort using the signature genes identified in COPSAC2010 for the two subspecies."
- [readme] MAGinator uses a statistical model to find the best genes for calculating accurate abundances: "MAGinator uses a statistical model to find the best genes for calculating accurate abundances"
