---
name: bacterial-phenotype-prediction
description: Use when you have tens or hundreds of bacterial genome sequences (nucleotide
  or protein FASTAs) and need trait calls for any of Traitar's 67 phenotypes, or when
  benchmarking genotype-to-phenotype classifiers against curated phenotype labels
  such as GIDEON or Bergey's Manual entries.
when_to_use_negative:
- The trait of interest is outside Traitar's 67 supported phenotypes (e.g. plant biomass
  degradation — that predictor is not included in the Traitar release).
- You need genotype-phenotype inference that integrates other omics data or expression
  levels — Traitar uses only presence/absence of protein families.
- You require reliable phenotype absence calls on incomplete genomes — an absence
  prediction may simply reflect missing protein families in the draft input genome.
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Traitar
  role: 'End-to-end microbial trait analyzer: orchestrates gene prediction, Pfam annotation,
    and phypat / phypat+PGL phenotype classification of 67 traits'
  repo: https://github.com/hzi-bifo/traitar
- name: Prodigal
  role: Predicts open reading frames from input nucleotide FASTAs in the standard
    from_nucleotides workflow
- name: HMMer (hmmsearch)
  role: Annotates predicted coding sequences against the Pfam 27.0 HMM models to build
    protein family profiles
- name: GNU parallel
  role: Parallelizes the pipeline across samples when the -c process-count option
    is set
- name: pip
  role: Recommended installation method for traitar from the Python packaging index
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:2240284db88a18b1aba3e23d6e65851131b0285ad8983689a92d5c3d7e8b4ab6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - GNU parallel
  - HMMer (hmmsearch)
  - Prodigal
  - Python
  - Python 2.7
  - Traitar
  - hmmer/hmmsearch
  - hmmsearch
  - matplotlib
  - pip
  - prodigal
  - scipy
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# bacterial-phenotype-prediction

## Summary

Predict 67 diverse bacterial phenotypes directly from genome sequences using Traitar, which trains phenotype classifiers on Pfam protein family profiles (phypat) and optionally on ancestral protein family gains/losses inferred by maximum likelihood on the species phylogeny (phypat+PGL), reducing wet-lab phenotyping work.

## When to use

Use when you have tens or hundreds of bacterial genome sequences (nucleotide or protein FASTAs) and need trait calls for any of Traitar's 67 phenotypes, or when benchmarking genotype-to-phenotype classifiers against curated phenotype labels such as GIDEON or Bergey's Manual entries.

## When NOT to use

- The trait of interest is outside Traitar's 67 supported phenotypes (e.g. plant biomass degradation — that predictor is not included in the Traitar release).
- You need genotype-phenotype inference that integrates other omics data or expression levels — Traitar uses only presence/absence of protein families.
- You require reliable phenotype absence calls on incomplete genomes — an absence prediction may simply reflect missing protein families in the draft input genome.

## Inputs

- Bacterial nucleotide (or protein) FASTA genome files for the samples to phenotype
- Traitar sample file (two mandatory columns: sample_file_name and sample_name, optional third column for sample category/gene_gff)
- Pfam 27.0 HMM models (Pfam-A.hmm), downloaded via `traitar pfam` or supplied locally
- Optional: externally gene-predicted sequences plus gene GFF files (with -g/--gene_gff_type, e.g. refseq) for the from_genes mode

## Outputs

- Gene prediction results in <out_dir>/gene_prediction
- Pfam protein family presence/absence profiles in <out_dir>/pfam_annotation
- Phenotype predictions for the 67 traits in <out_dir>/phenotype_prediction, including predictions_majority-vote_combined.txt (0/1/2/3 encoding) and predictions_flat_majority-votes_combined.txt
- Heatmaps: heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_comb.png with hierarchical clustering dendrograms
- Phenotype-relevant protein family tracks as GFF files in phypat/feat_gffs and phypat+PGL/feat_gffs

## How to apply

Install traitar from the Python packaging index with pip (not by cloning), satisfying dependencies: scipy, matplotlib, pandas, Prodigal, HMMer's hmmsearch and GNU parallel; obtain the Pfam 27.0 HMM models (Pfam-A.hmm) via `traitar pfam <download_folder>` or register a local copy with `traitar pfam --local`. Run `traitar phenotype <in_dir> <sample_file> from_nucleotides <out_dir>` so Prodigal predicts open reading frames, hmmsearch annotates coding sequences with Pfam families, and both classifiers (phypat on phyletic patterns alone; phypat+PGL additionally encoding ancestral protein family gains/losses to account for horizontal gene transfer) assign the 67 phenotypes per genome; use `from_genes` if gene prediction was already done externally and set the process count with `-c` (e.g. `-c 2`). Collect results from `<out_dir>/gene_prediction`, `<out_dir>/pfam_annotation` and `<out_dir>/phenotype_prediction`, where `predictions_majority-vote_combined.txt` encodes negative predictions as 0, phypat-only predictions as 1, phypat+PGL-only as 2, and predictions supported by both as 3. Rationale: phypat+PGL improves accuracy over phyletic-pattern-only prediction for individual phenotypes and overall, but phypat is more suitable for exploratory analysis since it assigns more phenotypes to incomplete genomes at the price of more false positives.

## Related tools

- **Traitar** (End-to-end microbial trait analyzer: orchestrates gene prediction, Pfam annotation, and phypat / phypat+PGL phenotype classification of 67 traits) — https://github.com/hzi-bifo/traitar
- **Prodigal** (Predicts open reading frames from input nucleotide FASTAs in the standard from_nucleotides workflow)
- **HMMer (hmmsearch)** (Annotates predicted coding sequences against the Pfam 27.0 HMM models to build protein family profiles)
- **GNU parallel** (Parallelizes the pipeline across samples when the -c process-count option is set)
- **pip** (Recommended installation method for traitar from the Python packaging index)

## Examples

```
traitar phenotype <in_dir> <sample_file> from_nucleotides <out_dir> -c 2
```

## Evaluation signals

- The phenotype prediction step assigns exactly the 67 supported phenotypes per genome, and per-classifier results appear in the phypat and phypat+PGL subfolders.
- predictions_majority-vote_combined.txt uses the documented encoding: 0 = negative, 1 = phypat-only, 2 = phypat+PGL-only, 3 = supported by both.
- Benchmark reproduction: on GIDEON I 10-fold nested cross-validation, macroaccuracy of ~82.6% (phypat) and ~85.5% (phypat+PGL) and accuracy of ~88.1%/89.8%; on the 42-species GIDEON II test set, macroaccuracy ~85.3%/86.7% and accuracy ~87.5%/87.9%; on 296 Bergey's Manual species, accuracy ~72.9% (phypat) and ~72.1% (phypat+PGL).
- Heatmap outputs (heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_comb.png) exist with hierarchical clustering dendrograms for both phenotypes and samples.
- Running the packaged example (`traitar phenotype <traitar_dir>/data/sample_data ... from_genes <out_dir> -c 2`) phenotypes Listeria grayi DSM_20601 and Listeria ivanovii WSLC3009 within ~5 minutes as sanity check.

## Limitations

- Genotype-phenotype inference takes into account only presence/absence of protein families; other omics data are not integrated and their integration is proposed only as future work.
- For incomplete genomes, an absent phenotype prediction may be due to absence of the relevant protein families from the input genome, not true trait absence.
- The phypat classifier assigns more phenotypes to incomplete genomes but with more false-positive predictions; it suits exploratory analysis.
- The plant biomass degradation predictor is not included in the released Traitar software.
- In the Docker container there is a problem with parallel usage, so the -c option is not guaranteed to work, and output files are owned by root.

## Evidence

- [abstract] Traitar predicts 67 phenotypes from genomes: "software that predicts traits of bacteria on the basis of their genomes"
- [intro] Automated prediction of 67 phenotypes from a genome sequence: "an easy-to-use, fully automated software framework for the accurate prediction of currently 67 phenotypes directly from a genome sequence"
- [methods] Install via pip: "We strongly encourage you to install traitar with pip rather than by manually cloning the repository."
- [methods] Pfam 27.0 models required: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [discussion] phypat+PGL improves accuracy: "improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall."
- [readme] Standard workflow: Prodigal + HMMer + phenotype models: "which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs in the <in_dir> for all samples in <sample_file> with Pfam families using HMMer"
- [readme] Prediction encoding in combined table: [evidence span withheld — anchor status: external_doc]
