---
name: metagenomic-read-quality-trimming
description: Use when raw paired-end Illumina shotgun metagenomic reads (e.g. SRA
  accession SRR606249, 2 x 100 bp) must be cleaned prior to meta-assembly with metaSPAdes
  or MegaHIT.
when_to_use_negative:
- Input is already assembled contigs or binned MAGs (FASTA) — kMetaShot classification
  works directly on binning output and requires no read trimming
- Reads are long single-end (2nd/3rd generation) reads rather than paired-end Illumina,
  where Sickle paired-end mode and the -l 50 length cutoff from this benchmark would
  not map to the data
- The analysis goal is MAG classification only and a pre-trimmed or QC'd dataset is
  already available
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: FastQC
  role: performs the initial quality check on raw paired-end reads before trimming
- name: Sickle
  role: trims reads by quality (-q 32) and minimum length (-l 50) to produce the assembly-ready
    read set
- name: kMetaShot
  role: 'downstream consumer: trimmed reads feed the assembly-binning pipeline whose
    MAGs are classified by kMetaShot'
  repo: https://github.com/gdefazio/kMetaShot
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:0ac19d336297885725eed6ac0c93acab281ee13c35cbaed0421a6ee504393e37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - CAMITAX
  - FastQC
  - GTDBtk
  - Mash
  - MegaHIT
  - Sickle
  - kMetaShot
  - metaSPAdes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# metagenomic-read-quality-trimming

## Summary

Quality-control and trim paired-end shotgun metagenomic reads with FastQC followed by Sickle (quality cutoff -q 32, minimum length -l 50) before meta-assembly, as used in the kMetaShot benchmark on the real mock community dataset (SRR606249).

## When to use

Use when raw paired-end Illumina shotgun metagenomic reads (e.g. SRA accession SRR606249, 2 x 100 bp) must be cleaned prior to meta-assembly with metaSPAdes or MegaHIT.

## When NOT to use

- Input is already assembled contigs or binned MAGs (FASTA) — kMetaShot classification works directly on binning output and requires no read trimming
- Reads are long single-end (2nd/3rd generation) reads rather than paired-end Illumina, where Sickle paired-end mode and the -l 50 length cutoff from this benchmark would not map to the data
- The analysis goal is MAG classification only and a pre-trimmed or QC'd dataset is already available

## Inputs

- raw paired-end FASTQ shotgun metagenomic reads (e.g. SRR606249, Illumina HiSeq 2000, 2 x 100 bp)

## Outputs

- quality-checked reports (FastQC output)
- trimmed paired-end FASTQ reads passing -q 32 and -l 50 filters, ready for meta-assembly with metaSPAdes or MegaHIT

## How to apply

Run a quality check on the FASTQ files with FastQC, then trim the paired-end reads with Sickle using the paper's parameters -q 32 (minimum per-base quality) and -l 50 (minimum read length after trimming). The trimmed reads then serve as input to meta-assembly with metaSPAdes (v3.15.2) or MegaHIT (v1.2.9), both with k-mers 35, 57, 79, 99, and downstream binning with MetaBAT2 (--maxP 99 --minS 98 -m 1500). This step was applied to the real mock community reads from SRA accession SRR606249 (Illumina HiSeq 2000, 2 x 100 bp) in the kMetaShot benchmark against GTDBtk and CAMITAX, and trimming should be run identically for each assembler arm so that the two MAG sets remain comparable.

## Related tools

- **FastQC** (performs the initial quality check on raw paired-end reads before trimming)
- **Sickle** (trims reads by quality (-q 32) and minimum length (-l 50) to produce the assembly-ready read set)
- **kMetaShot** (downstream consumer: trimmed reads feed the assembly-binning pipeline whose MAGs are classified by kMetaShot) — https://github.com/gdefazio/kMetaShot

## Examples

```
sickle pe -f reads_R1.fastq -r reads_R2.fastq -o trimmed_R1.fastq -p trimmed_R2.fastq -s trimmed_SE.fastq -q 32 -l 50
```

## Evaluation signals

- FastQC reports confirm the raw paired-end data were assessed before trimming (2 x 100 bp reads)
- Trimmed output reads all satisfy the cutoffs: per-base quality >= 32 and length >= 50 after trimming
- Downstream metaSPAdes and MegaHIT are each run on the same trimmed read set with identical k-mers (-k 35, 57, 79, 99), yielding two comparable MAG sets after MetaBAT2 binning (--maxP 99 --minS 98 -m 1500)
- Both assembler arms proceed to classification (kMetaShot, GTDBtk, CAMITAX) and reproduce the paper's finding that kMetaShot performance is meta-assembler independent
- Final metrics (sensitivity, precision, FDR, balanced accuracy, F1-score) against the known mock community composition (48 bacteria, 16 archaea) are computable end-to-end

## Limitations

- Quality cutoffs (-q 32 -l 50) are the benchmark's parameters; applying them unchanged to other datasets should be justified by a per-dataset FastQC check
- This step only prepares reads for assembly; correctness must ultimately be judged by downstream classification metrics, not by trimming alone

## Evidence

- [methods] After a quality check with FastQC and trimming with Sickle (-q 32 -l 50): "After a quality check with FastQC [58] and trimming with Sickle [59] (-q 32 -l 50)"
- [methods] PE reads were meta-assembled with metaSpades and MegaHIT using k-mer list 35, 57, 79, 99: "PE reads were meta-assembled with metaSpades (v3.15.2) [44] and MegaHIT (v1.2.9) [45] (for both -k 35, 57, 79, 99)"
