---
name: svm-based-phenotype-classification
description: Use when bacterial nucleotide FASTA genomes (or already-predicted proteins)
  need trait/phenotype assignment — e.g. the 67 Traitar trait labels such as motility,
  nitrate reduction, or L-arabinose fermentation.
when_to_use_negative:
- Genes/ORFs have already been predicted externally — use the from_genes mode (or
  skip ORF calling) rather than from_nucleotides
- Input is already a Pfam annotation/profile table — phenotype prediction alone suffices;
  re-running Prodigal/HMMER would be redundant
- Incomplete genomes where absence of a phenotype may simply reflect missing protein
  families from the draft input — interpret absent calls cautiously
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3673
tools:
- name: Traitar
  role: 'End-to-end framework: orchestrates ORF calling, Pfam annotation, and SVM
    phenotype prediction from the 67 trait models'
  repo: https://github.com/hzi-bifo/traitar
- name: Prodigal
  role: Predicts protein-coding open reading frames from input nucleotide FASTAs (from_nucleotides
    mode)
- name: HMMER
  role: Scans predicted proteins against Pfam 27.0 HMM models to build protein family
    profiles feeding the SVM classifiers
- name: Pfam
  role: Pfam 27.0 (Pfam-A.hmm) protein family models required as the annotation feature
    space
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:964d925b9f4d5304054e5ffb716654cd4c4c373265cbdf4017ed1d1f1bf6488e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - HMMER
  - Pfam
  - Prodigal
  - Traitar
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# svm-based-phenotype-classification

## Summary

Predicts 67 bacterial phenotypes from genome-derived Pfam protein-family profiles using Traitar's pre-trained L1-regularized linear SVM models, as described in Weimann et al. 2016 (mSystems).

## When to use

Use when bacterial nucleotide FASTA genomes (or already-predicted proteins) need trait/phenotype assignment — e.g. the 67 Traitar trait labels such as motility, nitrate reduction, or L-arabinose fermentation.

## When NOT to use

- Genes/ORFs have already been predicted externally — use the from_genes mode (or skip ORF calling) rather than from_nucleotides
- Input is already a Pfam annotation/profile table — phenotype prediction alone suffices; re-running Prodigal/HMMER would be redundant
- Incomplete genomes where absence of a phenotype may simply reflect missing protein families from the draft input — interpret absent calls cautiously

## Inputs

- Nucleotide FASTA file(s) of bacterial genome(s) (from_nucleotides mode)
- Alternatively, externally gene-predicted coding sequences (from_genes mode)
- Sample file (TSV: sample_file_name, sample_name, optional category; mandatory header row)

## Outputs

- phenotype prediction tables, e.g. phenotype_predictions.tsv / predictions_majority-vote_combined.txt covering the 67 traitar phenotype labels (0/1/2/3 encoding)
- Flat prediction table predictions_flat_majority-votes_combined.txt (one prediction per row)
- Heatmaps: heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_comb.png
- Intermediate outputs: gene_prediction/ (Prodigal) and pfam_annotation/ (HMMER)
- Phenotype-relevant protein family GFF feature tracks in phypat/feat_gffs and phypat+PGL/feat_gffs

## How to apply

Run the Traitar phenotype workflow, e.g. 'traitar phenotype <in_dir> <sample_file> from_nucleotides <out_dir>', which first predicts open reading frames with Prodigal and annotates the coding sequences for all samples with Pfam families (Pfam 27.0 HMM models) using HMMER, applying traitar thresholds of bit-score >= 25 and E value <= 1e-2 to call protein family profiles. From the filtered hits build the binary presence/absence Pfam profile vector per sample, then apply the pre-trained linear L1-regularized SVM voting committee to yield present/absent/imprecise assignments for the 67 phenotype labels. Choose the classifier deliberately: 'phypat' (phyletic patterns only) is more suitable for exploratory analysis, assigning more phenotypes at the price of more false positives, while 'phypat+PGL' incorporates ancestral protein family gains/losses to account for horizontal gene transfer and improves accuracy. Results land in <out_dir>/phenotype_prediction as tab-separated tables (e.g. predictions_majority-vote_combined.txt, encoding 0=negative, 1=phypat-only, 2=phypat+PGL-only, 3=both) and heatmaps.

## Related tools

- **Traitar** (End-to-end framework: orchestrates ORF calling, Pfam annotation, and SVM phenotype prediction from the 67 trait models) — https://github.com/hzi-bifo/traitar
- **Prodigal** (Predicts protein-coding open reading frames from input nucleotide FASTAs (from_nucleotides mode))
- **HMMER** (Scans predicted proteins against Pfam 27.0 HMM models to build protein family profiles feeding the SVM classifiers)
- **Pfam** (Pfam 27.0 (Pfam-A.hmm) protein family models required as the annotation feature space)

## Examples

```
traitar phenotype <in_dir> samples.txt from_nucleotides <out_dir> -c 2
```

## Evaluation signals

- phenotype_predictions.tsv exists and contains one row per input sample with columns covering the 67 traitar phenotype labels
- Encoded prediction values are within {0,1,2,3} where 0=negative, 1=phypat-only, 2=phypat+PGL-only, 3=supported by both algorithms
- Intermediates present: <out_dir>/gene_prediction (Prodigal ORFs) and <out_dir>/pfam_annotation (HMMER hits)
- Heatmaps (heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_comb.png) render with hierarchical clustering dendrograms for phenotypes and samples
- Number of samples in prediction tables equals the number of rows in the sample file

## Limitations

- For incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes
- The phypat classifier assigns more phenotypes to incomplete genomes but at the price of more false positives
- Genotype-phenotype inference uses only presence/absence of protein families; integration of other omics data is proposed as future work only
- Pfam 27.0 HMM models are not distributed with the package and must be downloaded separately (traitar pfam)
- Plant biomass predictor prototype is not included in the released Traitar software

## Evidence

- [readme] README basic usage: "will trigger the standard workflow of Traitar, which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs"
- [readme] README results encoding: [evidence span withheld — anchor status: external_doc]
- [methods] methods Pfam requirement: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [discussion] discussion phypat suitability: "the phypat classifier is more suitable for exploratory analysis, as it assigned more phenotypes to incomplete genomes at the price of more false-positive predictions."
- [intro] intro phenotype scope: "an easy-to-use, fully automated software framework for the accurate prediction of currently 67 phenotypes directly from a genome sequence"
