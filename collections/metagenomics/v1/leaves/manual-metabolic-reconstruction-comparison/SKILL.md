---
name: manual-metabolic-reconstruction-comparison
description: Use when draft metagenome-assembled genomes (e.g. estimated 91% and 60%
  complete) need automated phenotype predictions that are validated against an existing
  expert metabolic reconstruction.
when_to_use_negative:
- The input genomes are complete isolates already covered by GIDEON/Bergey's phenotype
  labels, so cross-validated Traitar accuracy suffices without a bespoke reconstruction
  comparison
- No prior manual metabolic reconstruction or phenotype table exists for the genomes,
  leaving no ground truth to compare against
- The goal is to integrate transcriptomic or other omics evidence — Traitar inference
  uses only protein family presence/absence
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Traitar
  role: Predicts present/absent calls for 67 phenotypes from the Pfam protein family
    profiles of the population-genome assemblies for comparison with the manual reconstruction
  repo: https://github.com/hzi-bifo/traitar
- name: pip
  role: Installs traitar from the Python packaging index ('pip install traitar-<version>.tar.gz
    --user')
- name: Prodigal
  role: Open reading frame prediction executed by Traitar's from_nucleotides workflow
    on the unClos_1 and unFirm_1 assemblies
- name: HMMER (hmmsearch)
  role: Scans predicted coding sequences against Pfam 27.0 HMM models to build the
    protein family profiles used for phenotyping
- name: GNU parallel
  role: Enables parallel Traitar execution via the -c parameter (e.g. -c 2)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:de2c0ae77c38a68535a82b914f20f198c93fae5d9ab1c842962d9286d721df7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - GNU parallel
  - HMMER (hmmsearch)
  - Prodigal
  - Python
  - Traitar
  - matplotlib
  - pandas
  - pip
  - scipy
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# manual-metabolic-reconstruction-comparison

## Summary

Reproduces the phenotyping of the biogas reactor population genomes unClos_1 and unFirm_1 with Traitar and benchmarks the predicted present/absent calls for 67 phenotypes against the manual metabolic reconstruction of Frank et al. (Table 4), quantifying recall and agreement on absent phenotypes.

## When to use

Use when draft metagenome-assembled genomes (e.g. estimated 91% and 60% complete) need automated phenotype predictions that are validated against an existing expert metabolic reconstruction.

## When NOT to use

- The input genomes are complete isolates already covered by GIDEON/Bergey's phenotype labels, so cross-validated Traitar accuracy suffices without a bespoke reconstruction comparison
- No prior manual metabolic reconstruction or phenotype table exists for the genomes, leaving no ground truth to compare against
- The goal is to integrate transcriptomic or other omics evidence — Traitar inference uses only protein family presence/absence

## Inputs

- Genome assemblies (nucleotide FASTA) of the biogas reactor population genomes unClos_1 (estimated 91% complete) and unFirm_1 (estimated 60% complete)
- Traitar sample file (two-column TSV: sample_file_name, sample_name, optional category)
- Pfam 27.0 Pfam-A.hmm HMM models
- Manual metabolic reconstruction of Frank et al. (Table 4) with reconstruction-inferred phenotype calls

## Outputs

- Per-genome present/absent phenotype predictions for 67 traits (predictions_majority-vote_combined.txt and heatmap PNGs under <out_dir>/phenotype_prediction)
- Pfam annotation and gene prediction outputs (<out_dir>/pfam_annotation, <out_dir>/gene_prediction)
- Comparison table traitar_phenotype_comparison.csv with per-phenotype concordance and summary metrics (recall 6/7 = 87.5% on present phenotypes; 9/11 = 81.8% agreement on absent phenotypes)

## How to apply

Install traitar from the Python packaging index with pip after installing python-scipy, python-matplotlib, and python-pandas, and fetch the Pfam 27.0 (Pfam-A.hmm) HMM models with 'traitar pfam <download folder>' (or register a local Pfam folder with 'traitar pfam --local'). Verify that prodigal, hmmsearch, and GNU parallel are on the command line, since Traitar calls ORFs with Prodigal, scans Pfam families with HMMer, and parallelizes via GNU parallel. Run 'traitar phenotype <in_dir> <sample_file> from_nucleotides <out_dir>' on the unClos_1 and unFirm_1 assemblies to obtain present/absent calls for the 67 phenotypes from their Pfam protein family profiles. Compare each genome's phenotype calls against the Frank et al. manual metabolic reconstruction (Table 4), tallying concordant present and absent phenotypes. Compute recall on present phenotypes (target 6/7 = 87.5%) and agreement on absent phenotypes (target 9/11 = 81.8%), and write the per-phenotype predictions and comparison summary to traitar_phenotype_comparison.csv. For incomplete genomes, interpret absent calls cautiously since missing phenotypes may reflect missing protein families rather than true absence.

## Related tools

- **Traitar** (Predicts present/absent calls for 67 phenotypes from the Pfam protein family profiles of the population-genome assemblies for comparison with the manual reconstruction) — https://github.com/hzi-bifo/traitar
- **pip** (Installs traitar from the Python packaging index ('pip install traitar-<version>.tar.gz --user'))
- **Prodigal** (Open reading frame prediction executed by Traitar's from_nucleotides workflow on the unClos_1 and unFirm_1 assemblies)
- **HMMER (hmmsearch)** (Scans predicted coding sequences against Pfam 27.0 HMM models to build the protein family profiles used for phenotyping)
- **GNU parallel** (Enables parallel Traitar execution via the -c parameter (e.g. -c 2))

## Examples

```
traitar phenotype <in_dir> samples.txt from_nucleotides <out_dir> -c 2
```

## Evaluation signals

- Recall on reconstruction-inferred present phenotypes equals 6/7 = 87.5% across unClos_1 and unFirm_1
- Agreement on reconstruction-inferred absent phenotypes equals 9/11 = 81.8%
- Output directory contains gene_prediction, pfam_annotation, and phenotype_prediction subfolders with heatmap_phypat.png, heatmap_phypat_ggl.png, and heatmap_comb.png
- traitar_phenotype_comparison.csv contains one row per phenotype with concordant present and absent tallies summing to 7 present and 11 absent reconstruction-inferred phenotypes
- Pfam annotation step completed successfully (non-empty <out_dir>/pfam_annotation), confirming Pfam 27.0 models were registered via 'traitar pfam'

## Limitations

- For incomplete genomes such as unFirm_1 (~60% complete), absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genome, inflating apparent disagreement
- The phypat classifier assigned more phenotypes to incomplete genomes at the price of more false-positive predictions, so classifier choice affects the comparison
- Traitar's genotype-phenotype inference takes into account only presence and absence of protein families; other omics data are not integrated
- The plant biomass predictor prototype is not included in the released Traitar software
- Docker container output is owned by root and the -c parallel option is not guaranteed to work in Docker

## Evidence

- [intro] Characterize two novel Clostridiales species of a biogas reactor community with Traitar based on genomes recovered from metagenomics, verifying and complementing a manual metabolic reconstruction.: "We characterized two novel Clostridiales species of a biogas reactor community with Traitar on the basis of their genomes recovered with metagenomics. This verified and complemented a manual"
- [methods] Traitar requires Pfam 27.0 HMM models which must be downloaded separately.: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [methods] Traitar is installed via pip from the python packaging index.: "We strongly encourage you to install traitar with pip rather than by manually cloning the repository."
- [readme] Traitar's standard workflow predicts ORFs with Prodigal and annotates with Pfam using HMMer before phenotype prediction.: "will trigger the standard workflow of Traitar, which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs in the <in_dir> for all samples in"
- [discussion] For incomplete genomes, absence of a phenotype prediction may reflect missing protein families.: "One obvious limitation being for incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes."
- [readme] Sample file format for the traitar phenotype command.: "The sample file has one column for the sample file names and one for the names as specified by the user."
