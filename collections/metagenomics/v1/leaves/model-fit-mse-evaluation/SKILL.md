---
name: model-fit-mse-evaluation
description: Use when a signature-gene or marker-gene abundance model built in one
  cohort is applied to another cohort and you must quantify fit quality per cohort
  (e.g., reported MSE 103.95 for COPSAC2010 vs 878.09 for CHILD) to assess transferability.
when_to_use_negative:
- Input is already a validated species-level taxonomic profile table with no subspecies
  signature-gene model to evaluate — use direct profile comparison instead
- Samples lack the minimum number of detected signature genes (--min_mapped_signature_genes
  threshold not met), so the fit is undefined and abundance estimates would be unreliable
- Evaluating phylogenetic cluster separation (SNV trees) rather than numeric abundance
  fit — StrainPhlAn cluster cross-tabulation, not MSE, is the appropriate check
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MAGinator
  role: Profiles shotgun reads against the gene catalogue, redistributes ambiguous
    reads, and fits the signature gene model to produce per-sample subspecies abundances
    used in the MSE computation
  repo: https://github.com/Russel88/MAGinator
- name: MetaPhlAn
  role: Provides reference B. longum species abundances that the summed subspecies
    trajectories are regressed against to compute explained variance
- name: StrainPhlAn
  role: Assigns samples to mutually exclusive subspecies clusters used to cross-validate
    the MAGinator-estimated abundances
- name: Snakemake
  role: Workflow engine that runs the MAGinator pipeline steps (read mapping, signature
    gene fitting) reproducibly
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:ef33fb21a798f7e0d49ef2b0b577db32296a27400011f7aabe3008a0e876b7e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  - MetaPhlAn
  - Snakemake
  - StrainPhlAn
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# model-fit-mse-evaluation

## Summary

Evaluates how well a de novo signature-gene abundance model (e.g., MAGinator's B. infantis/B. longum signature genes) fits each cohort by recomputing the per-cohort mean squared error (MSE) between signature-gene-estimated and reference subspecies abundances, using MSE differences to judge cross-cohort transferability.

## When to use

Use when a signature-gene or marker-gene abundance model built in one cohort is applied to another cohort and you must quantify fit quality per cohort (e.g., reported MSE 103.95 for COPSAC2010 vs 878.09 for CHILD) to assess transferability.

## When NOT to use

- Input is already a validated species-level taxonomic profile table with no subspecies signature-gene model to evaluate — use direct profile comparison instead
- Samples lack the minimum number of detected signature genes (--min_mapped_signature_genes threshold not met), so the fit is undefined and abundance estimates would be unreliable
- Evaluating phylogenetic cluster separation (SNV trees) rather than numeric abundance fit — StrainPhlAn cluster cross-tabulation, not MSE, is the appropriate check

## Inputs

- Per-sample signature gene counts/abundances from MAGinator read-mapping against the non-redundant gene catalogue (with COPSAC2010-derived B. infantis and B. longum signature genes)
- Reference subspecies-level abundance estimates (e.g., from StrainPhlAn clusters or MetaPhlAn B. longum species abundances)
- Sample-by-sample metadata table linking samples to cohorts (COPSAC2010, CHILD)

## Outputs

- Per-cohort signature-gene model fit MSE values (expected: 103.95 for COPSAC2010, 878.09 for CHILD)
- R-squared of summed subspecies abundances regressed against MetaPhlAn B. longum species abundance (expected 87%)
- Per-sample subspecies abundance and cluster-assignment table (e.g., bifido_subspecies_validation_results.tsv)

## How to apply

Fit the signature gene model per cohort with MAGinator's signature-reads profiling (ambiguous reads redistributed based on uniquely mapping reads), requiring detection of the signature genes via --min_mapped_signature_genes so samples lacking sufficient signature genes are excluded from the fit. Per sample, obtain per-subspecies abundances for B. infantis and B. longum from the fit, then compute the mean squared error between model-estimated and reference abundances for each cohort separately. Compare the per-cohort MSE values: in the MAGinator paper the COPSAC2010-derived signature genes fit COPSAC2010 (MSE 103.95) better than the CHILD cohort (MSE 878.09), which flags cross-cohort degradation. As complementary validation, sum the two subspecies abundance trajectories per sample and regress against MetaPhlAn B. longum species abundance to report explained variance (87% in the paper). Write per-cohort MSE, R-squared, and per-sample assignments to a results table (e.g., bifido_subspecies_validation_results.tsv).

## Related tools

- **MAGinator** (Profiles shotgun reads against the gene catalogue, redistributes ambiguous reads, and fits the signature gene model to produce per-sample subspecies abundances used in the MSE computation) — https://github.com/Russel88/MAGinator
- **MetaPhlAn** (Provides reference B. longum species abundances that the summed subspecies trajectories are regressed against to compute explained variance)
- **StrainPhlAn** (Assigns samples to mutually exclusive subspecies clusters used to cross-validate the MAGinator-estimated abundances)
- **Snakemake** (Workflow engine that runs the MAGinator pipeline steps (read mapping, signature gene fitting) reproducibly)

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out --min_mapped_signature_genes 5
```

## Evaluation signals

- Recomputed signature-gene fit MSE equals 103.95 for COPSAC2010 and 878.09 for CHILD
- MetaPhlAn B. longum regression yields ~87% variance explained (R-squared 0.87)
- StrainPhlAn subspecies clusters separate samples by MAGinator-estimated subspecies abundance as in Fig. 4
- Every sample included in the fit passes the --min_mapped_signature_genes detection threshold
- Results table bifido_subspecies_validation_results.tsv contains per-cohort MSE, R-squared, and per-sample cluster/abundance assignments

## Limitations

- A subset of COPSAC2010 signature genes was missing in the CHILD cohort, presumably causing abundance underestimation for a subset of samples — one cause of the higher CHILD MSE (878.09 vs 103.95)
- MAGinator's high completeness may introduce noise or misclassification that influences abundance and beta-diversity estimates
- MAGinator lacks operon awareness, limiting synteny cluster accuracy (relevant when interpreting the underlying gene model)

## Evidence

- [results] The signature genes fit the COPSAC2010 cohort better (MSE 103.95) than the CHILD cohort (MSE 878.09), indicating the signature genes are better suited for profiling the specific subspecies in the COPSAC cohort.: [evidence span withheld — anchor status: not_found]
- [methods] Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis: "Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis"
- [abstract] MAGinator provides de novo identification of subspecies-level microbes and accurate abundance estimates of metagenome-assembled genomes (MAGs).: "MAGinator provides de novo identification of subspecies-level microbes and accurate abundance estimates of metagenome-assembled genomes (MAGs)."
- [discussion] With StrainPhlAn, we are able to identify 2 mutually exclusive clusters, each representing a subspecies.: "With StrainPhlAn, we are able to identify 2 mutually exclusive clusters, each representing a subspecies."
- [discussion] These results were reproduced in the CHILD cohort using the signature genes identified in COPSAC2010 for the two subspecies.: "These results were reproduced in the CHILD cohort using the signature genes identified in COPSAC2010 for the two subspecies."
