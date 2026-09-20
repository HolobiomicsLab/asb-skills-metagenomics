---
name: mag-binning-and-classification
description: Use when shotgun metagenomic paired-end reads need to be assembled, binned
  into MAGs, and taxonomically classified, or when testing whether MAG classification
  accuracy (sensitivity, precision, FDR, balanced accuracy, F1) is robust to assembler
  choice.
when_to_use_negative:
- Input is raw unassembled short reads that are not quality-controlled or trimmed
  — the workflow assumes FastQC/Sickle (-q 32 -l 50) preprocessed reads for assembly
- The taxonomic classifier's reference must be prokaryotic (RefSeq bacteria/archaea);
  eukaryotic or viral MAGs are out of scope
- Machine has less than the ~22Gb storage for the kMetaShot HDF5 reference or insufficient
  RAM for reference loading (kMetaShot requires at least 32Gb memory to load the reference)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: FastQC
  role: Quality check of paired-end mock community reads before trimming
- name: Sickle
  role: Read trimming with -q 32 -l 50
- name: metaSPAdes
  role: Meta-assembly of trimmed reads (v3.15.2, k 35,57,79,99) to produce contigs
    for one MAG set
- name: MegaHIT
  role: Alternative meta-assembly (v1.2.9, k 35,57,79,99) to test assembler independence
- name: MetaBAT2
  role: Binning contigs into MAGs per assembler with --maxP 99 --minS 98 -m 1500
- name: kMetaShot
  role: Alignment-free k-mer/minimizer taxonomic classification of MAGs against the
    HDF5 reference with ass2ref=0.2
  repo: https://github.com/gdefazio/kMetaShot
- name: Mash
  role: ANI comparison (>=97%) of unlabelled MAGs against RefSeq genomes for strain-level
    refinement
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:b578be99c365e4b413ac70cd93d4047b916a528cb572b9eb9304aab2c8b92cd6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - FastQC
  - Mash
  - MegaHIT
  - MetaBAT2
  - Sickle
  - kMetaShot
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# mag-binning-and-classification

## Summary

A workflow that reconstructs MAGs from metagenomic reads (FastQC/Sickle QC, metaSPAdes/MegaHIT assembly, MetaBAT2 binning) and classifies them taxonomically up to strain level with the alignment-free kMetaShot k-mer/minimizer classifier, assessing whether classification reliability depends on the choice of meta-assembler.

## When to use

Use when shotgun metagenomic paired-end reads need to be assembled, binned into MAGs, and taxonomically classified, or when testing whether MAG classification accuracy (sensitivity, precision, FDR, balanced accuracy, F1) is robust to assembler choice.

## When NOT to use

- Input is raw unassembled short reads that are not quality-controlled or trimmed — the workflow assumes FastQC/Sickle (-q 32 -l 50) preprocessed reads for assembly
- The taxonomic classifier's reference must be prokaryotic (RefSeq bacteria/archaea); eukaryotic or viral MAGs are out of scope
- Machine has less than the ~22Gb storage for the kMetaShot HDF5 reference or insufficient RAM for reference loading (kMetaShot requires at least 32Gb memory to load the reference)

## Inputs

- Paired-end shotgun metagenomic FASTQ reads (e.g. SRA SRR606249 mock community reads)
- Contigs from metaSPAdes v3.15.2 and MegaHIT v1.2.9 assemblies (k 35,57,79,99)
- kMetaShot HDF5 reference (kMetaShot_bacteria_archaea_2025-05-22.h5 or kMetaShot_reference.h5)
- RefSeq bacterial/archaeal genomes for ANI-based strain refinement (Mash)

## Outputs

- MAG sets binned per assembler (MetaBAT2 output FASTA bins)
- kMetaShot taxonomic classifications per MAG at strain/species/genus levels with ass2ref=0.2 filtering
- assembler_comparison_metrics.csv with sensitivity, precision, FDR, balanced accuracy, and F1-score for both MAG sets, before and after strain refinement

## How to apply

Download paired-end reads (e.g. SRA accession SRR606249 from the Shakya mock community), QC with FastQC, and trim with Sickle using -q 32 -l 50. Meta-assemble trimmed reads independently with metaSPAdes v3.15.2 and MegaHIT v1.2.9, both with k-mer list 35, 57, 79, 99, then bin contigs from each assembler separately with MetaBAT2 v2:2.15 using --maxP 99 --minS 98 -m 1500 to obtain two MAG sets. Classify both MAG sets with kMetaShot (installed via conda from bioconda) against the HDF5 prokaryotic reference using the prevalence-based classifier with the ass2ref filter set to 0.2, which drops classifications where the ratio of observed to expected taxon minimizers is unreliable. For strain-level assignments lacking reference strain annotation, estimate ANI against RefSeq genomes with Mash (>=97%) and assign unlabelled genomes to the highest-ANI strain. Compute sensitivity, precision, FDR, balanced accuracy, and F1-score for each assembler's classifications before and after strain refinement, recording them side by side to confirm kMetaShot performance is meta-assembler independent.

## Related tools

- **FastQC** (Quality check of paired-end mock community reads before trimming)
- **Sickle** (Read trimming with -q 32 -l 50)
- **metaSPAdes** (Meta-assembly of trimmed reads (v3.15.2, k 35,57,79,99) to produce contigs for one MAG set)
- **MegaHIT** (Alternative meta-assembly (v1.2.9, k 35,57,79,99) to test assembler independence)
- **MetaBAT2** (Binning contigs into MAGs per assembler with --maxP 99 --minS 98 -m 1500)
- **kMetaShot** (Alignment-free k-mer/minimizer taxonomic classification of MAGs against the HDF5 reference with ass2ref=0.2) — https://github.com/gdefazio/kMetaShot
- **Mash** (ANI comparison (>=97%) of unlabelled MAGs against RefSeq genomes for strain-level refinement)

## Examples

```
kMetaShot_classifier_NV.py -b bins/ -r kMetaShot_reference/kMetaShot_bacteria_archaea.h5 -p 10 -o output_dir -a 0.2
```

## Evaluation signals

- The metrics table reports all five metrics (sensitivity, precision, FDR, balanced accuracy, F1-score) for both the metaSPAdes- and MegaHIT-derived MAG sets
- Classification performance is comparable across the two assemblers, with kMetaShot correctly assigning the highest number of MAGs at species and genus levels with the lowest number of false positives in both cases
- kMetaShot classifications are only retained when the ass2ref ratio passes the 0.2 threshold, ensuring unreliable classifications (low observed/expected minimizer ratio) are dropped
- Strain-level assignments for genomes lacking reference strain annotation are supported by Mash ANI >= 97% against the assigned RefSeq strain
- The kMetaShot output directory is populated for all bins and the kMetaShot_test.py -r reference check passes before classification

## Limitations

- kMetaShot requires a large HDF5 reference (~22Gb storage; at least 32Gb memory to load) and high parallelism increases RAM usage
- Only 11 strains and 2 species of the 939 HMP genomes were not classified at all, and strain-level sensitivity (~82.0%) is lower than species (~89.2%) and genus (~92.9%) levels
- The benchmark's conclusion of assembler independence was established on the real mock community (Shakya et al., SRR606249) dataset and may not transfer to all environments

## Evidence

- [methods] After a quality check with FastQC [58] and trimming with Sickle [59] (-q 32 -l 50): "After a quality check with FastQC [58] and trimming with Sickle [59] (-q 32 -l 50)"
- [methods] meta-assembly with metaSPAdes and MegaHIT: "PE reads were meta-assembled with metaSpades (v3.15.2) [44] and MegaHIT (v1.2.9) [45] (for both -k 35, 57, 79, 99)"
- [methods] MAG binning with MetaBAT2: "the provided contigs were binned separately with MetaBAT2 (v2:2.15) [35] (--maxP 99 --minS 98 -m 1,500) to obtain MAGs"
- [results] ass2ref filter at 0.2: "we implemented ass2ref (the ratio among the observed and expected taxa minimizer), as a measure of the reliability of the kMetaShot classifications."
- [results] assembler-independent finding: [evidence span withheld — anchor status: not_found]
- [readme] reference storage requirement: "kMetaShot reference represents prokaryotic RefSeq genomes and requires about 22Gb of storage."
