---
name: metagenome-short-read-assembly
description: Use when metagenomics short reads (not assembled genomes) are the genomic
  counterpart to GNPS mass spectrometry data and RiPP discovery requires contigs as
  input to 6-frame translation and motif-ORF/all-ORF precursor mining.
when_to_use_negative:
- Input is already assembled genomic contigs or complete RefSeq genomes — assembly
  is unnecessary; proceed directly to 6-frame translation.
- The spectra to be searched lack paired genomics/metagenomics data — the article
  notes 99% of GNPS spectra cannot be searched this way.
- Only a previously described biosynthetic gene cluster is available and no genome
  or metagenome sequence exists (as with the lacticin producer).
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaMiner
  role: Command line tool that first assembles short reads into contigs, then translates,
    detects modification enzymes, and builds RiPP precursor databases
  repo: https://github.com/mohimanilab/MetaMiner
- name: HMMer
  role: Scans assembled and translated contig sequences to detect RiPP modification
    enzymes that anchor 10-kbp biosynthetic gene cluster windows
- name: GNPS
  role: Source of the paired mass spectrometry datasets (e.g. MSV000078839, MSV000078604)
    whose metagenomic short reads are assembled
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:bd670e0024f92382110a97c93ad4484d226f358bbb73589059d6b71c5400ee45
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - Dereplicator
  - GNPS
  - HMMer
  - MS-DPR
  - MetaMiner
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metagenome-short-read-assembly

## Summary

Assembles microbiome/metagenome short reads (e.g. simulated metagenome FASTQ paired with ACTI spectral data) into contigs as the first step of the MetaMiner peptidogenomics pipeline, so the contigs can be 6-frame translated and mined for RiPP precursor peptides.

## When to use

Use when metagenomics short reads (not assembled genomes) are the genomic counterpart to GNPS mass spectrometry data and RiPP discovery requires contigs as input to 6-frame translation and motif-ORF/all-ORF precursor mining.

## When NOT to use

- Input is already assembled genomic contigs or complete RefSeq genomes — assembly is unnecessary; proceed directly to 6-frame translation.
- The spectra to be searched lack paired genomics/metagenomics data — the article notes 99% of GNPS spectra cannot be searched this way.
- Only a previously described biosynthetic gene cluster is available and no genome or metagenome sequence exists (as with the lacticin producer).

## Inputs

- Metagenome short-read sequencing data (e.g. simulated metagenome short reads for the ACTI dataset, FASTQ)
- Paired GNPS mass spectrometry datasets (MSV000078839, MSV000078604)

## Outputs

- Assembled contigs ready for 6-frame translation and modification-enzyme detection
- Downstream: precursor peptide target database (motif-ORF and all-ORF modes) and shuffled decoy database

## How to apply

Retrieve the paired GNPS spectral datasets (e.g. MSV000078839 and MSV000078604 for the ACTI Actinomycetes data with 36 RefSeq genomes and simulated metagenome short reads) and the metagenome FASTQ input. Feed the short reads to the MetaMiner command line tool (github.com/mohimanilab/MetaMiner), which first assembles the reads into contigs before downstream processing. The contigs are then 6-frame translated into protein sequence, HMMer scans the translated sequences to detect modification enzymes, and 10-kbp putative biosynthetic gene cluster windows centered at each detected enzyme are constructed for motif-ORF/all-ORF precursor database building. Assembly is only performed when the input is short reads; if complete genomes (e.g. the paired RefSeq genomes) are available, skip assembly and translate directly.

## Related tools

- **MetaMiner** (Command line tool that first assembles short reads into contigs, then translates, detects modification enzymes, and builds RiPP precursor databases) — https://github.com/mohimanilab/MetaMiner
- **HMMer** (Scans assembled and translated contig sequences to detect RiPP modification enzymes that anchor 10-kbp biosynthetic gene cluster windows)
- **GNPS** (Source of the paired mass spectrometry datasets (e.g. MSV000078839, MSV000078604) whose metagenomic short reads are assembled)

## Examples

```
git clone https://github.com/mohimanilab/MetaMiner && python MetaMiner.py --input acti_metagenome_reads.fastq --assemble
```

## Evaluation signals

- Contigs are produced from short reads before 6-frame translation; no ORF enumeration should occur on unassembled reads.
- Downstream HMMer scan must detect modification enzymes on assembled contigs and yield 10-kbp putative biosynthetic gene cluster windows centered at each enzyme.
- On the ACTI dataset, the assembled contigs must support the reference result: at 0% FDR motif-ORF mode identifies 3 unknown + 5 known RiPPs while all-ORF identifies only 2 known RiPPs; at 1% FDR, 8 RiPPs total are identified.
- Validation table written to acti_ripp_mode_fdr_comparison.csv must match the reported mode-by-FDR counts.
- If complete genomes are used instead of short reads, the assembly step should be skipped without changing the precursor-database construction logic.

## Limitations

- The motif-ORF strategy on assembled contigs is not sensitive — peptides without sequence similarity to known RiPPs are missed; all-ORF mode (ORFs shorter than the default 200 amino acid threshold) recovers some but identified fewer RiPPs on ACTI.
- No blind modification searches are currently performed for cyclic peptides due to computational complexity.
- Short-read input requires assembly as a prerequisite; spectra without paired genomic/metagenomic data (99% of GNPS) cannot be searched at all.

## Evidence
