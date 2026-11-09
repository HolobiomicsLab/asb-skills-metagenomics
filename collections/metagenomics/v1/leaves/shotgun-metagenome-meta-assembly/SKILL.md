---
name: shotgun-metagenome-meta-assembly
description: Use when paired-end shotgun metagenomic reads (e.g. an SRA run such as
  SRR606249) must be turned into MAGs to feed taxonomic classifiers like kMetaShot,
  GTDBtk, or CAMITAX, starting from raw FASTQ.
when_to_use_negative:
- Input is already binned MAG FASTA files — go directly to the classifier (kMetaShot_classifier_NV.py
  -b bins/) instead of re-assembling
- The task is reference-based strain typing of isolated genomes rather than de novo
  recovery of community MAGs from mixed reads
- Only short contigs (< 1500 bp) are expected, since MetaBAT2 here bins with -m 1500
  and would discard them
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: FastQC
  role: Quality check of the paired-end reads before trimming
- name: Sickle
  role: Read trimming with -q 32 -l 50
- name: metaSPAdes
  role: First meta-assembler (v3.15.2, -k 35,57,79,99) producing one of two contig
    sets
- name: MegaHIT
  role: Second meta-assembler (v1.2.9, -k 35,57,79,99) producing the parallel contig
    set
- name: MetaBAT2
  role: Contig binning (v2:2.15, --maxP 99 --minS 98 -m 1500) to obtain MAGs per assembler
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:41f6d36e8e43af04c037b83ecdf85470dba27b205dbb2864beac5ff2ae26eba2
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
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# shotgun-metagenome-meta-assembly

## Summary

A meta-assembly workflow that converts paired-end shotgun metagenomic reads (e.g. SRA run SRR606249, Illumina HiSeq 2000, 2 x 100 bp) into MAGs by quality-trimming reads, meta-assembling them with two independent assemblers (metaSPAdes and MegaHIT), and binning contigs with MetaBAT2. It supplies the MAG inputs that downstream taxonomic classifiers such as kMetaShot, GTDBtk, and CAMITAX require.

## When to use

Use when paired-end shotgun metagenomic reads (e.g. an SRA run such as SRR606249) must be turned into MAGs to feed taxonomic classifiers like kMetaShot, GTDBtk, or CAMITAX, starting from raw FASTQ.

## When NOT to use

- Input is already binned MAG FASTA files — go directly to the classifier (kMetaShot_classifier_NV.py -b bins/) instead of re-assembling
- The task is reference-based strain typing of isolated genomes rather than de novo recovery of community MAGs from mixed reads
- Only short contigs (< 1500 bp) are expected, since MetaBAT2 here bins with -m 1500 and would discard them

## Inputs

- Paired-end shotgun metagenomic FASTQ reads (e.g. SRR606249, Illumina HiSeq 2000, 2 x 100 bp)
- Known mock community composition for validation (e.g. 48 bacteria and 16 archaea)

## Outputs

- Trimmed paired-end FASTQ reads
- metaSPAdes contigs (k 35,57,79,99)
- MegaHIT contigs (k 35,57,79,99)
- MetaBAT2 MAG bin sets, one per assembler (contigs >= 1500 bp)

## How to apply

Run a quality check on the paired-end reads with FastQC, then trim with Sickle using '-q 32 -l 50' (minimum quality 32, minimum length 50). Meta-assemble the trimmed reads separately with metaSPAdes (v3.15.2) and MegaHIT (v1.2.9), both with k-mers 35, 57, 79, 99, so that assembler choice can be evaluated independently. Bin the contigs from each assembler separately with MetaBAT2 (v2:2.15) using '--maxP 99 --minS 98 -m 1500' to obtain the MAGs. Keep the two meta-assembler MAG sets distinct, since the kMetaShot benchmark showed the tool was most precise at all taxonomic levels with both meta-assembler MAG sets. The rationale for dual assembly is reproducibility of the benchmark: classification accuracy (sensitivity, precision, FDR, balanced accuracy, F1-score) is then computed per assembler against the known community composition (48 bacteria and 16 archaea in the mock community).

## Related tools

- **FastQC** (Quality check of the paired-end reads before trimming)
- **Sickle** (Read trimming with -q 32 -l 50)
- **metaSPAdes** (First meta-assembler (v3.15.2, -k 35,57,79,99) producing one of two contig sets)
- **MegaHIT** (Second meta-assembler (v1.2.9, -k 35,57,79,99) producing the parallel contig set)
- **MetaBAT2** (Contig binning (v2:2.15, --maxP 99 --minS 98 -m 1500) to obtain MAGs per assembler)

## Examples

```
metaspades.py -1 trimmed_R1.fastq.gz -2 trimmed_R2.fastq.gz -k 35,57,79,99 -o metaspades_out && metabat2 -i metaspades_out/contigs.fasta -o mag_set/mb2 -t 10 -m 1500
```

## Evaluation signals

- Both metaSPAdes and MegaHIT were run with the exact k-mer list 35, 57, 79, 99, and contigs from each assembler were binned separately (never pooled).
- MetaBAT2 was invoked with --maxP 99 --minS 98 -m 1500; bins below the 1500 bp minimum contig size should not appear.
- Trimmed reads retain minimum length 50 and were filtered at quality 32, verifiable by re-running FastQC/Sickle logs.
- Downstream, the resulting MAGs classified with kMetaShot (ass2ref 0.2) should reproduce the reported ranking: highest correct assignments with lowest false positives at species and genus levels versus GTDBtk and CAMITAX.
- Sensitivity, precision, FDR, balanced accuracy, and F1-score computed against the known composition (48 bacteria, 16 archaea) should match the paper's reported tool comparison.

## Limitations

- Assembler and binning parameter choices (k-mers 35/57/79/99, MetaBAT2 -m 1500) are dataset-specific for the mock community benchmark and may need retuning for other read depths or community complexities.
- MetaBAT2's -m 1500 excludes shorter contigs, so MAG completeness depends on assembler contiguity.
- Only one real mock community dataset (SRR606249) plus in silico CAMI II multi-sample data were used for validation; results may not generalize to all environments.

## Evidence

- [methods] PE reads were meta-assembled with metaSpades (v3.15.2) [44] and MegaHIT (v1.2.9) [45] (for both -k 35, 57, 79, 99): "PE reads were meta-assembled with metaSpades (v3.15.2) [44] and MegaHIT (v1.2.9) [45] (for both -k 35, 57, 79, 99)"
- [methods] After a quality check with FastQC and trimming with Sickle: "After a quality check with FastQC [58] and trimming with Sickle [59] (-q 32 -l 50)"
- [methods] the provided contigs were binned separately with MetaBAT2: "the provided contigs were binned separately with MetaBAT2 (v2:2.15) [35] (--maxP 99 --minS 98 -m 1,500) to obtain MAGs"
- [readme] bins fasta files or multi-fasta bin headers: "Path to a directory containing bins fasta files or path to a multi-fasta file where each header corresponds to a bin/MAG."
