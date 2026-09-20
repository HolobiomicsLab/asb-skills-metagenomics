---
name: rank-based-inverse-normal-transformation
description: Use when gene-level (functional) abundance values from metagenomic shotgun
  profiling are skewed/non-normal and must be transformed before case-control MWAS
  association testing, as done in OMARU's profiling stage.
when_to_use_negative:
- Input is already transformed or normally distributed abundance data where the rank
  transform would destroy meaningful quantitative differences between samples
- Target is phylogenetic (clade-level) abundance rather than gene-level functional
  abundance — use OMARU's phylogenetic QC path instead
- Raw gene counts are needed, e.g. for absolute-abundance or length-normalized reporting
  rather than association testing
edam_topics:
- http://edamontology.org/topic_3941
tools:
- name: OMARU
  role: End-to-end workflow that implements the gene abundance quantification and
    applies the rank-based inverse normal transformation within its functional profiling
    stage
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine executing the OMARU_QCed1.sm Snakefile that performs assembly,
    annotation, quantification, and the abundance transformation
  repo: https://github.com/toshi-kishikawa/OMARU
- name: MEGAHIT
  role: De novo assembles QCed shotgun reads per sample to produce contigs from which
    gene abundances are derived
- name: Prokka
  role: Predicts open reading frames on assembled contigs prior to gene identity assignment
    and abundance quantification
- name: DIAMOND
  role: Aligns predicted ORFs against UniRef90 to assign gene identities feeding the
    abundance matrix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:2ff30ff2d38230ae0cdcb1c5bf1ce058af80623a6fdf5e0517e907122d1f37b2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/toshi-kishikawa/OMARU
  repo_url: https://github.com/toshi-kishikawa/OMARU
  tools:
  - Conda
  - DIAMOND
  - MEGAHIT
  - OMARU
  - Prokka
  - Snakemake
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Rank-based inverse normal transformation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A normalization transform applied to OMARU's gene abundance matrix, in which quantified gene abundances are converted to ranks and mapped onto a normal distribution via inverse normal transformation, stabilizing non-normal abundance distributions before gene-level case-control association testing.

## When to use

Use when gene-level (functional) abundance values from metagenomic shotgun profiling are skewed/non-normal and must be transformed before case-control MWAS association testing, as done in OMARU's profiling stage.

## When NOT to use

- Input is already transformed or normally distributed abundance data where the rank transform would destroy meaningful quantitative differences between samples
- Target is phylogenetic (clade-level) abundance rather than gene-level functional abundance — use OMARU's phylogenetic QC path instead
- Raw gene counts are needed, e.g. for absolute-abundance or length-normalized reporting rather than association testing

## Inputs

- Per-sample gene abundance matrix derived from ORF alignment depth against UniRef90 (normalized by gene length and each sample's total read count)
- Gene-level QC-passing gene set (e.g. 185,663 genes for SCZ; 104,487 for T2D)

## Outputs

- Inverse-normally transformed gene abundance matrix used for gene-level case-control association testing
- Profiling summary tables and figures written to <OMARU_project_dir>/result/<Phenotype>_summary

## How to apply

Run OMARU's profiling stage (Snakefiles/OMARU_QCed1.sm) on QCed FASTQ files, de novo assembling reads per sample with MEGAHIT, predicting ORFs with Prokka, and assigning gene identities by DIAMOND alignment against UniRef90. Quantify gene abundance from ORF alignment depth, normalizing by gene length and each sample's total read count, then apply a rank-based inverse normal transformation to the abundance values — i.e. convert each gene's abundance vector to per-gene ranks and map the ranks to quantiles of the standard normal distribution. Apply the transformation after gene-level QC (which in OMARU's SCZ/T2D applications retained 185,663 and 104,487 genes respectively) so only genes passing QC enter the transformed matrix. The rationale is that raw alignment-depth-derived abundances are highly skewed; the rank-based transform makes their distributions tractable for the downstream gene association test with covariate adjustment (e.g. sex, age, BMI, top 30 PCs) and multiple-testing correction.

## Related tools

- **OMARU** (End-to-end workflow that implements the gene abundance quantification and applies the rank-based inverse normal transformation within its functional profiling stage) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine executing the OMARU_QCed1.sm Snakefile that performs assembly, annotation, quantification, and the abundance transformation) — https://github.com/toshi-kishikawa/OMARU
- **MEGAHIT** (De novo assembles QCed shotgun reads per sample to produce contigs from which gene abundances are derived)
- **Prokka** (Predicts open reading frames on assembled contigs prior to gene identity assignment and abundance quantification)
- **DIAMOND** (Aligns predicted ORFs against UniRef90 to assign gene identities feeding the abundance matrix)

## Examples

```
snakemake -s Snakefiles/OMARU_QCed1.sm
```

## Evaluation signals

- Verify the transformed gene abundance matrix exists and the gene count matches the post-QC expectations (185,663 genes for SCZ and 104,487 for T2D)
- Check that transformed values are on the normal-distribution scale (no negative/zero raw abundance artifacts; ranks mapped to normal quantiles) rather than raw alignment depth
- Confirm profiling summary tables and figures were written to <OMARU_project_dir>/result/<Phenotype>_summary
- Confirm the transformation was applied to samples that passed profiling QC and are listed in QCed2_sample_list.txt
- Downstream gene association P-values should show inflation suppression once covariates are adjusted as in the phylogenetic analyses

## Limitations

- Covariate integration into the case-control model is not implemented in OMARU; hidden confounding factors are handled separately rather than by model integration
- Metagenome analysis remains highly dependent on reference databases (e.g. UniRef90), and database development is a stated future challenge
- No specific versions of tools or reference databases are stated in the paper

## Evidence

- [methods] Quantify gene abundance from ORF alignment depth, normalizing by gene length and each sample's total read count, then apply a rank-based inverse normal transformation to the abundance values: [evidence span withheld — anchor status: not_found]
