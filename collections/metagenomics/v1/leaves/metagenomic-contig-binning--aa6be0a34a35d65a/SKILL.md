---
name: metagenomic-contig-binning
description: Use when starting from paired-end metagenomic shotgun reads (e.g. FASTQ)
  and non-redundant MAG sets are needed as input for downstream MAG taxonomy classification,
  especially for benchmarking classifiers.
when_to_use_negative:
- Input is already binned MAG FASTA files — skip assembly/binning and go straight
  to taxonomy classification (e.g. kMetaShot classifier)
- Reads are from an isolate genome or single-organism culture rather than a mixed
  community where metagenomic binning applies
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaBAT2
  role: Bins contigs from each meta-assembler separately (--maxP 99 --minS 98 -m 1500)
    to obtain the MAGs fed to the taxonomy classifiers
- name: FastQC
  role: Quality check of paired-end reads prior to trimming and assembly
- name: Sickle
  role: Trims reads with -q 32 -l 50 before meta-assembly
- name: metaSPAdes
  role: Meta-assembles trimmed reads (v3.15.2, k 35, 57, 79, 99) to produce contigs
    for binning
- name: MegaHIT
  role: Alternative meta-assembler (v1.2.9, k 35, 57, 79, 99) producing a second contig
    set for separate binning
- name: kMetaShot
  role: 'Downstream consumer: classifies the binned MAGs (ass2ref filter 0.2) up to
    strain level'
  repo: github.com/gdefazio/kMetaShot
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:70802aea15625c43d48f32ae129bce3c650f2755100533271c61b7bd8ed7ed39
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMITAX
  - FastQC
  - GTDBtk
  - Mash
  - MegaHIT
  - MetaBAT2
  - Sickle
  - kMetaShot
  - metaSPAdes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# metagenomic-contig-binning

## Summary

Bin metagenomic contigs into MAGs with MetaBAT2 after quality-checking and meta-assembling reads, to provide the bin-level inputs required by MAG taxonomy classifiers such as kMetaShot, GTDBtk, and CAMITAX.

## When to use

Use when starting from paired-end metagenomic shotgun reads (e.g. FASTQ) and non-redundant MAG sets are needed as input for downstream MAG taxonomy classification, especially for benchmarking classifiers.

## When NOT to use

- Input is already binned MAG FASTA files — skip assembly/binning and go straight to taxonomy classification (e.g. kMetaShot classifier)
- Reads are from an isolate genome or single-organism culture rather than a mixed community where metagenomic binning applies

## Inputs

- Paired-end metagenomic shotgun reads in FASTQ (e.g. SRA accession SRR606249, Illumina HiSeq 2000, 2 x 100 bp)
- Contigs from metaSPAdes or MegaHIT meta-assembly (both with k 35, 57, 79, 99)

## Outputs

- MAGs (binned contigs) as FASTA files per assembler (MetaBAT2 bins with --maxP 99 --minS 98 -m 1500)
- Separate MAG sets from metaSPAdes and MegaHIT suitable for direct input to kMetaShot_classifier_NV.py, GTDBtk, and CAMITAX

## How to apply

First run quality control on the paired-end reads with FastQC and trim with Sickle (-q 32 -l 50), then meta-assemble the trimmed reads with metaSPAdes (v3.15.2) or MegaHIT (v1.2.9) using k-mers 35, 57, 79, 99 — in the kMetaShot benchmark, each assembler was run separately. Bin the resulting contigs separately per assembler with MetaBAT2 (v2:2.15) using --maxP 99 --minS 98 -m 1500 to obtain MAGs. This binning step directly precedes classification with kMetaShot (ass2ref 0.2), GTDBtk v1.0.2 (labels converted from GTDB to NCBI taxonomy via ncbi_vs_gtdb_r89 tables), and CAMITAX v0.7.0. Rationale: kMetaShot works directly on the binning output, so consistent, per-assembler binning (metaSPAdes and MegaHIT sets were both evaluated) enables fair precision/sensitivity comparison of classifiers at species and genus levels against the known community composition.

## Related tools

- **MetaBAT2** (Bins contigs from each meta-assembler separately (--maxP 99 --minS 98 -m 1500) to obtain the MAGs fed to the taxonomy classifiers)
- **FastQC** (Quality check of paired-end reads prior to trimming and assembly)
- **Sickle** (Trims reads with -q 32 -l 50 before meta-assembly)
- **metaSPAdes** (Meta-assembles trimmed reads (v3.15.2, k 35, 57, 79, 99) to produce contigs for binning)
- **MegaHIT** (Alternative meta-assembler (v1.2.9, k 35, 57, 79, 99) producing a second contig set for separate binning)
- **kMetaShot** (Downstream consumer: classifies the binned MAGs (ass2ref filter 0.2) up to strain level) — github.com/gdefazio/kMetaShot

## Examples

```
metabat2 -i assembly.fasta -o MAGs/bin -m 1500 --maxP 99 --minS 98
```

## Evaluation signals

- Each assembler yields its own complete MAG set: bin metaSPAdes and MegaHIT contigs separately, never pooled
- Binning parameters recorded exactly as --maxP 99 --minS 98 -m 1500
- Downstream classification succeeds on the bins (e.g. kMetaShot run with -a 0.2 completes without reference-load failures; reference requires ~22 Gb storage / at least 32 Gb RAM)
- For the mock community benchmark (SRR606249), sensitivity, precision, FDR, balanced accuracy, and F1-score computed against the known composition (48 bacteria, 16 archaea) should reproduce the reported ranking where kMetaShot gave the most precise calls at species and genus levels on both meta-assembler MAG sets

## Limitations

- Binning quality depends on the meta-assembler: kMetaShot benchmark results were reported separately for metaSPAdes and MegaHIT MAG sets, so assembler choice can change downstream classifier rankings
- MetaBAT2 minimum contig size (-m 1500) excludes short contigs, so small plasmid/viral-like fragments are not recovered as MAGs
- kMetaShot, the intended downstream consumer of the bins, requires substantial resources (reference ~22 Gb on disk, at least 32 Gb RAM for reference loading)

## Evidence

- [methods] the provided contigs were binned separately with MetaBAT2 (v2:2.15) (--maxP 99 --minS 98 -m 1,500) to obtain MAGs: [evidence span withheld — anchor status: not_found]
- [methods] PE reads were meta-assembled with metaSpades (v3.15.2) and MegaHIT (v1.2.9) (for both -k 35, 57, 79, 99): [evidence span withheld — anchor status: not_found]
- [methods] After a quality check with FastQC and trimming with Sickle (-q 32 -l 50): "After a quality check with FastQC [58] and trimming with Sickle [59] (-q 32 -l 50)"
- [results] kMetaShot correctly assigned the highest number of MAGs with the lowest number of false positives at both species and genus levels: [evidence span withheld — anchor status: not_found]
- [readme] -b , --bins_dir (char) Path to a directory containing bins fasta files or path to a multi-fasta file: "Path to a directory containing bins fasta files or 
                        path to a multi-fasta file where each header corresponds
                        to a bin/MAG."
