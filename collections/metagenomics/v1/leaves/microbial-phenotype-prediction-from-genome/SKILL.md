---
name: microbial-phenotype-prediction-from-genome
description: Use when you have bacterial nucleotide FASTA assemblies or predicted
  genes (e.g. population genomes such as unClos_1/unFirm_1) and need phenotype calls
  for 67 traits to verify or complement a metabolic reconstruction.
when_to_use_negative:
- The task requires integrating transcriptomic, proteomic, or other omics data — Traitar
  only takes presence/absence of protein families into account
- You need trait prediction for features outside the 67 supported phenotypes (e.g.
  the plant biomass predictor prototype is not included in the release)
- You require confident phenotype absence calls from highly incomplete genomes, since
  absence may simply reflect missing protein families in the input
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: Traitar
  role: 'Core tool: predicts 67 phenotypes from Pfam protein family profiles of bacterial
    genomes (phypat and phypat+PGL classifiers)'
  repo: https://github.com/hzi-bifo/traitar
- name: Prodigal
  role: Open reading frame prediction on input nucleotide assemblies in the from_nucleotides
    workflow
- name: HMMER (hmmsearch)
  role: Annotates predicted coding sequences with Pfam families prior to phenotype
    classification
- name: pip
  role: Installs traitar from the Python packaging index; manual repository cloning
    is discouraged
- name: GNU parallel
  role: Enables parallel execution of the Traitar workflow via the -c parameter
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:f984cf572783c158f98dacd29b4f060d11f279c02c21bca1d96578a834b17def
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# microbial-phenotype-prediction-from-genome

## Summary

Predict up to 67 bacterial phenotypes (present/absent) directly from genome sequences using Traitar, which trains phenotype classifiers on Pfam protein family profiles. Useful for phenotyping draft/incomplete genomes (e.g. metagenome-assembled genomes) and cross-checking manual metabolic reconstructions.

## When to use

Use when you have bacterial nucleotide FASTA assemblies or predicted genes (e.g. population genomes such as unClos_1/unFirm_1) and need phenotype calls for 67 traits to verify or complement a metabolic reconstruction.

## When NOT to use

- The task requires integrating transcriptomic, proteomic, or other omics data — Traitar only takes presence/absence of protein families into account
- You need trait prediction for features outside the 67 supported phenotypes (e.g. the plant biomass predictor prototype is not included in the release)
- You require confident phenotype absence calls from highly incomplete genomes, since absence may simply reflect missing protein families in the input

## Inputs

- Bacterial genome assemblies as nucleotide FASTA files (e.g. MAGs unClos_1 and unFirm_1) OR protein/nucleotide gene FASTAs with from_genes
- Sample file (TSV) with sample_file_name and sample_name columns, optional category column
- Pfam 27.0 HMM models (Pfam-A.hmm), downloaded or registered locally
- Reference phenotype set for comparison, e.g. the Frank et al. manual metabolic reconstruction (Table 4)

## Outputs

- Phenotype present/absent prediction tables (predictions_majority-vote_combined.txt; 0/1/2/3 encoded) in <out_dir>/phenotype_prediction
- Heatmaps per classifier (heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_comb.png)
- Gene prediction results (<out_dir>/gene_prediction) and Pfam annotation (<out_dir>/pfam_annotation) with from_nucleotides
- Phenotype-relevant protein family GFF feature tracks (feat_gffs) and a per-phenotype comparison summary (e.g. traitar_phenotype_comparison.csv)

## How to apply

Install traitar from the Python packaging index with pip (after python-scipy, python-matplotlib, python-pandas; optionally in a python-virtualenv), and ensure Prodigal, HMMER hmmsearch, and GNU parallel are on the command line. Fetch the required Pfam 27.0 HMM models with 'traitar pfam <download folder>' (or register a local Pfam-A.hmm folder via 'traitar pfam --local'). Run the phenotype workflow ('traitar phenotype <in_dir> <sample_file> from_nucleotides <out_dir>' for assemblies, or from_genes when gene prediction is already done) to obtain present/absent calls for the 67 phenotypes from Pfam protein family profiles, optionally parallelized with '-c 2'. The sample file needs sample_file_name/sample_name (optionally category) columns. Interpret the encoded prediction table (0 = absent, 1 = phypat-only, 2 = phypat+PGL only, 3 = both classifiers) and compare calls against a reference, e.g. tally recall on reconstruction-inferred present phenotypes and agreement on absent phenotypes. For incomplete genomes, favor the phypat classifier for exploratory phenotyping while expecting more false positives.

## Related tools

- **Traitar** (Core tool: predicts 67 phenotypes from Pfam protein family profiles of bacterial genomes (phypat and phypat+PGL classifiers)) — https://github.com/hzi-bifo/traitar
- **Prodigal** (Open reading frame prediction on input nucleotide assemblies in the from_nucleotides workflow)
- **HMMER (hmmsearch)** (Annotates predicted coding sequences with Pfam families prior to phenotype classification)
- **pip** (Installs traitar from the Python packaging index; manual repository cloning is discouraged)
- **GNU parallel** (Enables parallel execution of the Traitar workflow via the -c parameter)

## Examples

```
traitar phenotype <in_dir> samples.txt from_nucleotides <out_dir> -c 2
```

## Evaluation signals

- Against the Frank et al. manual reconstruction, recall on present phenotypes should reach 6/7 (87.5%) and agreement on absent phenotypes 9/11 (81.8%) for unClos_1/unFirm_1
- The prediction table encodes 0 (absent), 1 (phypat only), 2 (phypat+PGL only), 3 (both) — values outside {0,1,2,3} indicate a parsing error
- Output directories gene_prediction, pfam_annotation, and phenotype_prediction are populated for a from_nucleotides run
- Cross-validation on GIDEON phenotype data should show the phypat classifier at high accuracy; a large drop in held-out accuracy suggests broken Pfam/HMMER setup
- Run on the packaged sample data (Listeria grayi DSM_20601, Listeria ivanovii WSLC3009) completes within ~5 minutes and reproduces the heatmap outputs

## Limitations

- For incomplete genomes, absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes (e.g. a 60%-complete bin), so absent calls are unreliable
- The phypat classifier assigns more phenotypes to incomplete genomes but at the price of more false-positive predictions
- Genotype-phenotype inference uses only protein family presence/absence; other omics data are not integrated (proposed future work)
- The plant biomass predictor prototype is not included in the released Traitar software
- Pfam 27.0 HMM models are not distributed with the package and must be downloaded separately; Docker runs have a parallel-usage problem (-c not guaranteed) and root-owned output

## Evidence

- [abstract] Traitar, software that predicts traits of bacteria on the basis of their genomes: "we have developed Traitar, software that predicts traits of bacteria on the basis of their genomes"
- [intro] accurate prediction of 67 phenotypes: "an easy-to-use, fully automated software framework for the accurate prediction of currently 67 phenotypes directly from a genome sequence"
- [methods] install traitar with pip: "We strongly encourage you to install traitar with pip rather than by manually cloning the repository."
- [methods] traitar requires Pfam 27.0 HMM models: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [readme] README: standard workflow from_nucleotides: "which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs in the <in_dir> for all samples in <sample_file> with Pfam families using HMMer and"
- [readme] README: prediction encoding 0-3: [evidence span withheld — anchor status: external_doc]
- [discussion] phypat better for exploratory analysis of incomplete genomes: "the phypat classifier is more suitable for exploratory analysis, as it assigned more phenotypes to incomplete genomes at the price of more false-positive predictions."
- [discussion] incomplete genomes absence caveat: "One obvious limitation being for incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes."
