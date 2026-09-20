---
name: metagenomic-read-alignment
description: Use when paired-end shotgun metagenomic FASTQ reads must be mapped to
  a combined microbiome reference FASTA while preserving junction pairs mapping across
  different references, as the first stage of LEMON's HGT detection pipeline.
when_to_use_negative:
- Input is long-read data or single-end reads incompatible with the paired-end 'bwa
  mem -M' workflow the article specifies
- The analysis goal does not require cross-reference junction reads (e.g. simple per-genome
  variant calling), where retaining supplementary/split alignments via -M adds no
  value
- A valid sorted BAM with mapped read pairs already exists from a prior LEMON preprocessing
  run
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BWA
  role: Aligns paired-end shotgun metagenomic reads against the combined reference
    genomes to identify the set of donor and receptor genomes; -M flag keeps split
    alignments (junction pairs).
  repo: https://github.com/lichen2018/LEMON
- name: Samtools
  role: Converts SAM to BAM ('samtools view -bhS') and coordinate-sorts it ('samtools
    sort -o sample.bam').
  repo: https://github.com/lichen2018/LEMON
- name: LEMON
  role: 'Downstream consumer: get_raw_bkp.py uses the unique-read BAM (with junction
    pairs) to cluster raw HGT breakpoint candidates.'
  repo: https://github.com/lichen2018/LEMON
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:b29d53e40156a080ac634a68ee257849dc62acb12350c75dae00613c89da1bd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - Gurobipy
  - LEMON
  - LUMPY
  - Samtools
  - WGSIM
  - iRep
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# metagenomic-read-alignment

## Summary

Align paired-end shotgun metagenomic reads against a multi-species reference FASTA using BWA mem with the -M flag, then convert/sort to BAM with samtools, so that junction read pairs whose sides map to two different references are retained for LEMON's downstream HGT breakpoint detection.

## When to use

Use when paired-end shotgun metagenomic FASTQ reads must be mapped to a combined microbiome reference FASTA while preserving junction pairs mapping across different references, as the first stage of LEMON's HGT detection pipeline.

## When NOT to use

- Input is long-read data or single-end reads incompatible with the paired-end 'bwa mem -M' workflow the article specifies
- The analysis goal does not require cross-reference junction reads (e.g. simple per-genome variant calling), where retaining supplementary/split alignments via -M adds no value
- A valid sorted BAM with mapped read pairs already exists from a prior LEMON preprocessing run

## Inputs

- Paired-end shotgun metagenomic reads (sample.1.fq, sample.2.fq FASTQ)
- Metagenomic reference FASTA (Metagenomic_reference.fasta; e.g. 16,093 species, 1,246,881 scaffolds)

## Outputs

- Sorted, indexed BAM file of mapped reads (sample.bam) including junction read pairs whose sides map to different references

## How to apply

First index the metagenomic reference FASTA (e.g. Metagenomic_reference.fasta, the 16,093-species / 1,246,881-scaffold set) with BWA to build the Burrows-Wheeler Transform index for all references together. Align sample.1.fq and sample.2.fq with 'bwa mem -M -t 8' plus a read-group header (@RG ID/SM/LB); the -M flag is essential because it marks split alignments so junction pairs whose two sides map to two different references (candidate donor/receptor pairs) are retained. Convert the SAM output to BAM with 'samtools view -bhS' and coordinate-sort with 'samtools sort -o sample.bam'. The resulting mapped-read set, including junction reads, is consumed by LEMON's get_raw_bkp.py, which clusters breakpoint-pair candidates with DBSCAN, so validation of this step hinges on the BAM containing such cross-reference junction pairs.

## Related tools

- **BWA** (Aligns paired-end shotgun metagenomic reads against the combined reference genomes to identify the set of donor and receptor genomes; -M flag keeps split alignments (junction pairs).) — https://github.com/lichen2018/LEMON
- **Samtools** (Converts SAM to BAM ('samtools view -bhS') and coordinate-sorts it ('samtools sort -o sample.bam').) — https://github.com/lichen2018/LEMON
- **LEMON** (Downstream consumer: get_raw_bkp.py uses the unique-read BAM (with junction pairs) to cluster raw HGT breakpoint candidates.) — https://github.com/lichen2018/LEMON

## Examples

```
bwa mem -M -t 8 -R "@RG\tID:id\tSM:sample\tLB:lib" Metagenomic_reference.fasta sample.1.fq sample.2.fq | samtools view -bhS - | samtools sort -o sample.bam
```

## Evaluation signals

- sample.bam exists and is a valid, coordinate-sorted BAM (verify with samtools quickcheck / samtools view -H showing the @RG header)
- BAM contains mapped read pairs, including junction pairs whose two sides map to two different references (required by get_raw_bkp.py)
- Alignment used 'bwa mem -M -t 8' with an @RG read-group header (ID/SM/LB) so split alignments were kept
- Downstream LEMON step (get_raw_bkp.py) produces a non-empty raw breakpoints file from the unique-read BAM
- In the simulation benchmark, the alignment feeds through the full LEMON pipeline to reproduce reported reconstruction accuracies (LEMON RA 0.6087 at 5X and 1.0 at 30X)

## Limitations

- Requires BWA 0.7.12+ and samtools 1.3.1+ per the LEMON README requirements
- Omitting the -M flag drops split alignments, losing the junction reads whose two sides map to different references, which breaks raw-breakpoint detection
- Designed for existing short-read shotgun NGS datasets from human microbiome studies; reference must contain all candidate donor and receptor genomes for junction reads to be informative

## Evidence

- [methods] BWA-based alignment identifies donor/receptor set: "we utilize BWA[20] to align reads against the references to identify the set of donors and receptors"
- [methods] First stage maps paired-end reads: "Firstly, we map paired-end shotgun reads against reference genomes using Burrows Wheeler Aligner (BWA)."
- [methods] Junction reads feed breakpoint detection: "Then, we select junction reads, whose two sides are mapped to two different references, from the set of mapped reads."
- [readme] README alignment command: "bwa mem -M -t 8 -R "@RG\tID:id\tSM:sample\tLB:lib""
