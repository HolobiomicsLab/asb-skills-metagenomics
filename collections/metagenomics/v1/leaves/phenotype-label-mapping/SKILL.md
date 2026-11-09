---
name: phenotype-label-mapping
description: 'Use when evaluating genotype-to-phenotype classifiers: you have genome
  FASTAs plus externally curated phenotype labels (e.g., Bergey''s Manual, GIDEON)
  and need to align predicted trait labels with the reference labels to compute accuracy.'
when_to_use_negative:
- Input genomes are already gene-predicted and you only need Pfam annotation onward
  — use 'traitar phenotype ... from_genes' instead of the full from_nucleotides workflow
- Genomes are highly incomplete (e.g., SAGs) and you need high-specificity labels
  — a missing prediction may just reflect absent Pfam families, not a true phenotype
  absence
- You need phenotypes beyond Traitar's 67 traits (e.g., the plant biomass predictor,
  which is not included in the Traitar release)
tools:
- name: Traitar
  role: Runs the phenotype prediction pipeline (Prodigal ORF calling, HMMer Pfam annotation,
    phypat and phypat+PGL classification) whose labels are mapped to reference labels
  repo: https://github.com/hzi-bifo/traitar
- name: prodigal
  role: Gene prediction called internally by the 'traitar phenotype from_nucleotides'
    workflow
- name: hmmer/hmmsearch
  role: Annotates predicted coding sequences against Pfam 27.0 models to build protein
    family profiles used for classification
- name: GNU parallel
  role: Enables parallel execution of the Traitar phenotype workflow via the -c parameter
- name: pip
  role: Recommended installation route for traitar ('pip install traitar-<version>.tar.gz
    --user')
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:8ebf0c22892b75b3666738407253db8f013652c131ae14d8462befda82f2b67d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - GNU parallel
  - Python
  - Traitar
  - hmmer/hmmsearch
  - pip
  - prodigal
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# phenotype-label-mapping

## Summary

Map Traitar's 67 phenotype predictions (from phypat and phypat+PGL classifiers) onto curated phenotype labels extracted from literature sources such as Bergey's Manual of Systematic Bacteriology or GIDEON, so classifier accuracy can be scored against an independent gold standard.

## When to use

Use when evaluating genotype-to-phenotype classifiers: you have genome FASTAs plus externally curated phenotype labels (e.g., Bergey's Manual, GIDEON) and need to align predicted trait labels with the reference labels to compute accuracy.

## When NOT to use

- Input genomes are already gene-predicted and you only need Pfam annotation onward — use 'traitar phenotype ... from_genes' instead of the full from_nucleotides workflow
- Genomes are highly incomplete (e.g., SAGs) and you need high-specificity labels — a missing prediction may just reflect absent Pfam families, not a true phenotype absence
- You need phenotypes beyond Traitar's 67 traits (e.g., the plant biomass predictor, which is not included in the Traitar release)

## Inputs

- Bacterial genome FASTA files (e.g., 296 Bergey's Manual species, draft genomes acceptable)
- Traitar sample file with sample_file_name/sample_name(/category) columns
- Curated phenotype label table extracted from Bergey's Manual of Systematic Bacteriology or GIDEON (presence/absence per trait)
- Pfam 27.0 (Pfam-A.hmm) HMM models downloaded or registered via 'traitar pfam --local'

## Outputs

- Per-genome phenotype predictions for 67 traits in <out_dir>/phenotype_prediction (predictions_majority-vote_combined.txt and flat variant)
- Per-classifier prediction tables under phypat/ and phypat+PGL/
- Phenotype heatmaps (heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_combined.png)
- Overall and per-phenotype accuracy values comparing predictions to curated reference labels

## How to apply

First install Traitar via pip with its dependencies (prodigal, hmmer hmmsearch, GNU parallel) and obtain the Pfam 27.0 (Pfam-A.hmm) models with 'traitar pfam'. Run 'traitar phenotype <in_dir> <sample_file> from_nucleotides <out_dir>', which calls Prodigal for ORF prediction, HMMer for Pfam annotation, and then assigns the 67 phenotypes per genome with both the phyletic-pattern (phypat) and phylogeny-aware (phypat+PGL) classifiers. Read the tab-separated prediction files (e.g., predictions_majority-vote_combined.txt, or per-classifier files under phypat/ and phypat+PGL/), where a negative prediction is encoded 0, phypat-only 1, phypat+PGL-only 2, and both 3. Curate reference phenotype labels for the 296 Bergey's Manual species (or GIDEON taxa), then compare predicted versus curated labels per trait and compute overall accuracy per classifier. Accept the mapping as correct if phypat reaches ~72.9% and phypat+PGL ~72.1% overall accuracy on the 296 Bergey's Manual species, mirroring the published evaluation.

## Related tools

- **Traitar** (Runs the phenotype prediction pipeline (Prodigal ORF calling, HMMer Pfam annotation, phypat and phypat+PGL classification) whose labels are mapped to reference labels) — https://github.com/hzi-bifo/traitar
- **prodigal** (Gene prediction called internally by the 'traitar phenotype from_nucleotides' workflow)
- **hmmer/hmmsearch** (Annotates predicted coding sequences against Pfam 27.0 models to build protein family profiles used for classification)
- **GNU parallel** (Enables parallel execution of the Traitar phenotype workflow via the -c parameter)
- **pip** (Recommended installation route for traitar ('pip install traitar-<version>.tar.gz --user'))

## Examples

```
traitar phenotype <in_dir> samples.txt from_nucleotides traitar_out -c 2
```

## Evaluation signals

- Overall accuracy of the phypat classifier on the 296 Bergey's Manual species is ~72.9%
- Overall accuracy of the phypat+PGL classifier is ~72.1%, matching phypat on per-phenotype and overall performance
- Output directory contains gene_prediction/, pfam_annotation/, and phenotype_prediction/ subfolders with prediction text files and three combined heatmaps
- Prediction table encodes negatives as 0, phypat-only as 1, phypat+PGL-only as 2, and agreement by both as 3
- Exactly 67 phenotype traits are assigned per genome

## Limitations

- On Bergey's Manual bacteria Traitar was less reliable than on GIDEON data (72.9% phypat / 72.1% phypat+PGL vs. higher cross-validation accuracy)
- For incomplete genomes, absence of a phenotype prediction may simply reflect missing protein families in the input genome; phypat assigns more phenotypes to incomplete genomes but with more false positives
- Genotype-phenotype inference uses only presence/absence of protein families; other omics data are not integrated
- In Docker, the -c parallel option is not guaranteed to work and output files are owned by root
- Traitar requires external Pfam 27.0 models, which are not distributed with the package

## Evidence

- [intro] Bergey's Manual evaluation scope: [evidence span withheld — anchor status: not_found]
- [results] Published accuracy on Bergey's set: [evidence span withheld — anchor status: not_found]
- [readme] Prediction file encoding: [evidence span withheld — anchor status: external_doc]
- [readme] Standard phenotype workflow: "will trigger the standard workflow of Traitar, which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs"
- [discussion] Trait model composition: "assigning 67 different phenotypes to bacteria on the basis of the protein family contents of their genomes."
