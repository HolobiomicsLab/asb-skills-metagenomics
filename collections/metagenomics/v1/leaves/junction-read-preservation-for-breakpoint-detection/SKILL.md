---
name: junction-read-preservation-for-breakpoint-detection
description: Use when paired-end shotgun metagenomic reads must be mapped against
  many reference genomes while preserving discordant/junction read pairs spanning
  two references as input for HGT breakpoint detection.
when_to_use_negative:
- Downstream steps needing only concordant alignments — the -M flag's split alignments
  would be discarded or treated as noise
- Single-genome resequencing where discordant cross-reference pairs are irrelevant;
  a plain bwa mem without the multi-reference junction-preservation rationale suffices
- Long-read or transcriptome data, since LEMON's workflow targets paired-end shotgun
  NGS datasets
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: BWA
  role: Aligns paired-end shotgun reads against the reference genomes with 'bwa mem
    -M' to retain split/junction alignments
  repo: https://github.com/lichen2018/LEMON
- name: Samtools
  role: Converts SAM to BAM and coordinate-sorts the mapped reads for downstream breakpoint
    detection
  repo: https://github.com/samtools/htslib
- name: LEMON
  role: Consumes the sorted unique-reads BAM via get_raw_bkp.py to detect raw HGT
    breakpoints from junction pairs
  repo: https://github.com/lichen2018/LEMON
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:d0e992eaa6b11fa9f85effbc8c4622d05e1e4c3e5c0f5b2cb688cf701fc501b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - LEMON
  - Samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Junction-read preservation for breakpoint detection

## Summary

Aligns paired-end shotgun metagenomic reads against a multi-genome reference with BWA mem using the -M split-alignment flag so that junction read pairs whose two sides map to different references are retained, producing the mapped-read BAM that LEMON's raw breakpoint detection (get_raw_bkp.py) requires.

## When to use

Use when paired-end shotgun metagenomic reads must be mapped against many reference genomes while preserving discordant/junction read pairs spanning two references as input for HGT breakpoint detection.

## When NOT to use

- Downstream steps needing only concordant alignments — the -M flag's split alignments would be discarded or treated as noise
- Single-genome resequencing where discordant cross-reference pairs are irrelevant; a plain bwa mem without the multi-reference junction-preservation rationale suffices
- Long-read or transcriptome data, since LEMON's workflow targets paired-end shotgun NGS datasets

## Inputs

- Metagenomic reference FASTA containing all donor/receptor genomes (Metagenomic_reference.fasta)
- Paired-end shotgun metagenomic reads as two FASTQ files (sample.1.fq, sample.2.fq)

## Outputs

- Coordinate-sorted BAM file (sample.bam) of mapped reads, including junction read pairs whose two sides map to two different references

## How to apply

Index the combined metagenomic reference FASTA (Metagenomic_reference.fasta) with BWA to build a single Burrows-Wheeler Transform index over all donor and receptor genomes. Align the paired-end FASTQ files (sample.1.fq, sample.2.fq) with 'bwa mem -M -t 8' plus a read-group header (@RG ID/SM/LB); the -M flag keeps split alignments so pairs whose two sides map to two different references are not discarded. Convert the SAM output to BAM with 'samtools view -bhS' and sort it with 'samtools sort -o sample.bam'. The sorted BAM is consumed by LEMON's get_raw_bkp.py (-u unique reads BAM) to identify the set of donor and receptor genomes and candidate HGT breakpoints, so the critical decision point is retaining junction pairs rather than filtering them as discordant.

## Related tools

- **BWA** (Aligns paired-end shotgun reads against the reference genomes with 'bwa mem -M' to retain split/junction alignments) — https://github.com/lichen2018/LEMON
- **Samtools** (Converts SAM to BAM and coordinate-sorts the mapped reads for downstream breakpoint detection) — https://github.com/samtools/htslib
- **LEMON** (Consumes the sorted unique-reads BAM via get_raw_bkp.py to detect raw HGT breakpoints from junction pairs) — https://github.com/lichen2018/LEMON

## Examples

```
bwa mem -M -t 8 -R '@RG\tID:id\tSM:sample\tLB:lib' Metagenomic_reference.fasta sample.1.fq sample.2.fq | samtools view -bhS - | samtools sort -o sample.bam
```

## Evaluation signals

- sample.bam exists and is a valid, coordinate-sorted BAM (samtools quickcheck passes)
- The BAM contains mapped read pairs, including junction pairs whose two sides map to two different references
- Read-group header (@RG ID/SM/LB) is present in the BAM header
- get_raw_bkp.py can consume the BAM (-u argument) and emit a non-empty raw breakpoints file
- Split alignments flagged by -M are present rather than being marked as supplementary/discarded

## Limitations

- If junction pairs are lost (e.g., omitting the -M flag or aggressive discordant filtering), the subsequent raw-breakpoint detection step (get_raw_bkp.py) loses its evidence for donor/receptor identification
- All reference genomes must be indexed together in one BWA index so cross-reference junction pairs can be detected in a single alignment pass
- BWA 0.7.12+, Samtools 1.3.1+ and HTSlib 1.9 are required by the LEMON workflow

## Evidence

- [methods] Firstly, we map paired-end shotgun reads against reference genomes using Burrows Wheeler Aligner (BWA).: "Firstly, we map paired-end shotgun reads against reference genomes using Burrows Wheeler Aligner (BWA)."
- [methods] Then, we select junction reads, whose two sides are mapped to two different references, from the set of mapped reads.: "Then, we select junction reads, whose two sides are mapped to two different references, from the set of mapped reads."
- [methods] we utilize BWA[20] to align reads against the references to identify the set of donors and receptors: "we utilize BWA[20] to align reads against the references to identify the set of donors and receptors"
- [readme] bwa mem -M -t 8 -R "@RG\tID:id\tSM:sample\tLB:lib": "# Align the data
bwa mem -M -t 8 -R "@RG\tID:id\tSM:sample\tLB:lib""
