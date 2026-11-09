---
name: classification-metric-benchmarking-across-assemblers
description: Decision support for claims that a MAG taxonomy classifier is robust
  to upstream assembly choices; triggered when the same read set can be assembled
  with two or more meta-assemblers (e.g. metaSPAdes v3.15.2 vs MegaHIT v1.2.9) and
  classifier metrics must be compared.
when_to_use_negative:
- The goal is to compare two different classifiers rather than two assemblers — use
  the classifier benchmarking design (kMetaShot vs GTDBtk vs CAMITAX) instead
- Only a single assembly exists and no second assembler can be applied, making an
  assembler-pairwise comparison impossible
- Input data are simulated reads from a synthetic community where no real mock reference
  community (e.g. Shakya et al.) with known ground truth is available for ANI/strain
  refinement
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0102
tools:
- name: kMetaShot
  role: Classifies the metaSPAdes- and MegaHIT-derived MAG sets against the HDF5 reference
    with the prevalence-based classifier and ass2ref=0.2 so metric differences can
    be attributed to the assembler
  repo: https://github.com/gdefazio/kMetaShot
- name: metaSPAdes
  role: One of the two meta-assemblers (v3.15.2) used to build contigs from the same
    trimmed reads with k-mer list 35, 57, 79, 99
- name: MegaHIT
  role: The second meta-assembler (v1.2.9) providing the comparison arm with the same
    k-mer list
- name: MetaBAT2
  role: Bins contigs from each assembler independently (v2:2.15, --maxP 99 --minS
    98 -m 1500) so each assembly yields its own MAG set
- name: Mash
  role: Estimates ANI of unlabelled MAGs against RefSeq genomes (>=97%) to assign
    unlabelled genomes to the highest-ANI strain for strain refinement
- name: FastQC
  role: Quality check of raw mock community reads before trimming
- name: Sickle
  role: Trims reads with -q 32 -l 50 prior to assembly
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:351eec06c43387395c762754b0e02ea018c4a7769141349df23fc3a34b3f1c40
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# classification-metric-benchmarking-across-assemblers

## Summary

A benchmarking design that tests whether MAG taxonomic classification performance (e.g. by kMetaShot) depends on the upstream meta-assembler by running two assemblers (metaSPAdes vs MegaHIT) on the same QC'd reads and comparing classifier metrics side by side.

## When to use

Decision support for claims that a MAG taxonomy classifier is robust to upstream assembly choices; triggered when the same read set can be assembled with two or more meta-assemblers (e.g. metaSPAdes v3.15.2 vs MegaHIT v1.2.9) and classifier metrics must be compared.

## When NOT to use

- The goal is to compare two different classifiers rather than two assemblers — use the classifier benchmarking design (kMetaShot vs GTDBtk vs CAMITAX) instead
- Only a single assembly exists and no second assembler can be applied, making an assembler-pairwise comparison impossible
- Input data are simulated reads from a synthetic community where no real mock reference community (e.g. Shakya et al.) with known ground truth is available for ANI/strain refinement

## Inputs

- Paired-end shotgun metagenomic reads from a real mock community (Shakya et al., SRA accession SRR606249) in FASTQ format
- Contigs from each meta-assembly (metaSPAdes and MegaHIT, -k 35,57,79,99) in FASTA format
- kMetaShot HDF5 prokaryotic reference (e.g. kMetaShot_bacteria_archaea_2025-05-22.h5)
- RefSeq genomes and NCBI taxonomy for strain-level ANI refinement

## Outputs

- Two binned MAG sets (one per assembler) produced by MetaBAT2
- kMetaShot taxonomic classifications for each MAG set (ass2ref=0.2), before and after Mash-based strain refinement
- assembler_comparison_metrics.csv reporting sensitivity, precision, FDR, balanced accuracy, and F1-score for both assemblers at species and genus levels

## How to apply

QC the paired-end mock community reads (SRR606249) with FastQC and trim with Sickle (-q 32 -l 50), then meta-assemble them independently with metaSPAdes v3.15.2 and MegaHIT v1.2.9 using the same k-mer list (35, 57, 79, 99) so the only varying factor is the assembler. Bin contigs from each assembly separately with MetaBAT2 v2:2.15 (--maxP 99 --minS 98 -m 1500) to yield two parallel MAG sets. Classify both MAG sets with kMetaShot against the HDF5 prokaryotic reference using the prevalence-based classifier with the ass2ref filter set to 0.2, and refine strain-level calls lacking reference annotation by ANI comparison with Mash (>=97% vs RefSeq genomes). Finally compute sensitivity, precision, FDR, balanced accuracy, and F1-score for each assembler before and after strain refinement, and record them side by side in assembler_comparison_metrics.csv; comparable metrics across assemblers demonstrate assembler-independent classification reliability.

## Related tools

- **kMetaShot** (Classifies the metaSPAdes- and MegaHIT-derived MAG sets against the HDF5 reference with the prevalence-based classifier and ass2ref=0.2 so metric differences can be attributed to the assembler) — https://github.com/gdefazio/kMetaShot
- **metaSPAdes** (One of the two meta-assemblers (v3.15.2) used to build contigs from the same trimmed reads with k-mer list 35, 57, 79, 99)
- **MegaHIT** (The second meta-assembler (v1.2.9) providing the comparison arm with the same k-mer list)
- **MetaBAT2** (Bins contigs from each assembler independently (v2:2.15, --maxP 99 --minS 98 -m 1500) so each assembly yields its own MAG set)
- **Mash** (Estimates ANI of unlabelled MAGs against RefSeq genomes (>=97%) to assign unlabelled genomes to the highest-ANI strain for strain refinement)
- **FastQC** (Quality check of raw mock community reads before trimming)
- **Sickle** (Trims reads with -q 32 -l 50 prior to assembly)

## Examples

```
kMetaShot_classifier_NV.py -b bins/metaspades/ -r kMetaShot_bacteria_archaea.h5 -p 10 -o out_metaspades -a 0.2
```

## Evaluation signals

- assembler_comparison_metrics.csv reports all five metrics (sensitivity, precision, FDR, balanced accuracy, F1-score) for both metaSPAdes- and MegaHIT-derived MAGs
- kMetaShot assigns the highest number of MAGs with the lowest number of false positives at species and genus levels on BOTH assembler-derived MAG sets (assembler independence)
- Both assemblies used the identical k-mer list (35, 57, 79, 99) and the identical binning parameters (--maxP 99 --minS 98 -m 1500), so the only variable is the assembler
- Strain-level assignments without reference strain annotation are resolved by ANI >=97% via Mash against RefSeq genomes
- The same ass2ref=0.2 filter is applied to both classification runs, so filtering does not confound the comparison

## Limitations

- Conclusions are tied to the Shakya et al. real mock community; a different community may respond differently to assembler choice
- Strain-level calls require ANI refinement (>=97% via Mash) when no reference strain annotation exists, adding a dependency on RefSeq genome coverage
- kMetaShot reference loading requires at least 32Gb of RAM, which constrains parallelism for both classification runs
- The comparison is only as fair as the binning parity — using different MetaBAT2 parameters per assembler would invalidate the metric comparison

## Evidence

- [results] kMetaShot's classification performances were meta-assembler independent: [evidence span withheld — anchor status: not_found]
- [methods] meta-assembly with metaSPAdes and MegaHIT: "PE reads were meta-assembled with metaSpades (v3.15.2) [44] and MegaHIT (v1.2.9) [45] (for both -k 35, 57, 79, 99)"
- [methods] MAG binning with MetaBAT2: "the provided contigs were binned separately with MetaBAT2 (v2:2.15) [35] (--maxP 99 --minS 98 -m 1,500) to obtain MAGs"
- [results] ass2ref filter set to 0.2: "For kMetaShot, an ass2ref of 0.2 was set for all datasets."
- [methods] ANI estimation with Mash and DBSCAN clustering at 97%: "squared matrices collecting pairwise ANI values between genomes are used to perform clustering, using DBSCAN"
- [readme] ass2ref parameter description from README: "ass2ref is a ratio between the number of MAG minimizers and the reference minimizers related to the assigned strain"
