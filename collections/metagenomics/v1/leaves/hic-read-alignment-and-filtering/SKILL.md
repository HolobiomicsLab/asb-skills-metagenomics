---
name: hic-read-alignment-and-filtering
description: Use when raw paired-end metagenomic Hi-C FASTQ files must be preprocessed
  and aligned to assembled contigs to produce a name-sorted BAM for Hi-C-based binning.
when_to_use_negative:
- Input is already a name-sorted, filtered BAM file from a prior ViralCC run — re-running
  cleaning and alignment would duplicate effort
- Data are shotgun-only reads with no Hi-C library, since the trimming steps (e.g.,
  ftl=10) and proximity-ligation alignment settings are Hi-C-specific
- The assembly contains contigs shorter than 1000 bp, which fall below the ViralCC
  --min-len default and should be filtered before binning
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3168
tools:
- name: BBTools
  role: bbduk.sh performs adaptor removal, quality trimming, and Hi-C first-10-nt
    trimming; clumpify.sh removes PCR optical and tile-edge duplicates
- name: BWA
  role: BWA MEM (v0.7.17) with -5SP flag aligns cleaned Hi-C paired-end reads to the
    assembled contigs
- name: samtools
  role: samtools view -F 0x904 filters unmapped, supplementary, and secondary alignments;
    samtools sort name-sorts the BAM
- name: MEGAHIT
  role: assembles the shotgun reads into contigs (--min-contig-len 1000) that serve
    as the alignment reference
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:fa0d3ca1c1cdce56d5c4c991b10bb57a3168abf7f95f5a946f2eb1d3906f187d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBTools
  - BWA
  - CoCoNet
  - HiCBin
  - MEGAHIT
  - MetaTOR
  - TAXAassign
  - ViralCC
  - bin3C
  - samtools
  - vRhyme
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# Hi-C read alignment and filtering

## Summary

Cleans metagenomic Hi-C paired-end reads (adaptor/quality/first-10-nt trimming, duplicate removal), aligns them to shotgun-assembled contigs with BWA MEM, filters the BAM with samtools, and name-sorts it to produce the BAM input required by ViralCC and other Hi-C binners.

## When to use

Use when raw paired-end metagenomic Hi-C FASTQ files must be preprocessed and aligned to assembled contigs to produce a name-sorted BAM for Hi-C-based binning.

## When NOT to use

- Input is already a name-sorted, filtered BAM file from a prior ViralCC run — re-running cleaning and alignment would duplicate effort
- Data are shotgun-only reads with no Hi-C library, since the trimming steps (e.g., ftl=10) and proximity-ligation alignment settings are Hi-C-specific
- The assembly contains contigs shorter than 1000 bp, which fall below the ViralCC --min-len default and should be filtered before binning

## Inputs

- Raw paired-end Hi-C FASTQ files (e.g., from SRA/ENA accessions ERX2333418, ERX2548555-56)
- Shotgun-assembled contigs in FASTA format (MEGAHIT assembly, min contig length 1000)

## Outputs

- Cleaned paired-end Hi-C FASTQ files with adaptors, low-quality bases, first 10 nucleotides, and PCR duplicates removed
- Name-sorted BAM file of filtered Hi-C alignments for input to ViralCC / binning pipelines

## How to apply

First clean reads with BBTools: remove adaptors with bbduk (ktrim=r k=23 mink=11 hdist=1 minlen=50 tpe tbo), quality-trim with bbduk (trimq=10 qtrim=r ftm=5 minlen=50), trim the first 10 nucleotides of Hi-C reads (ftl=10) to mitigate ligation artifacts, and remove identical PCR optical and tile-edge duplicates with clumpify.sh. Align the cleaned Hi-C pairs to contigs assembled by MEGAHIT (--min-contig-len 1000) using BWA MEM with the -5SP flag. Filter the alignment with samtools view -F 0x904 to drop unmapped reads, supplementary alignments, and secondary alignments, then sort the BAM by read name with samtools sort, since ViralCC requires a name-sorted BAM (the same cleaned shotgun reads are assembled first and ViralCC's contig-length minimum is enforced with --min-len default 1000 and --min-mapq default 30 downstream).

## Related tools

- **BBTools** (bbduk.sh performs adaptor removal, quality trimming, and Hi-C first-10-nt trimming; clumpify.sh removes PCR optical and tile-edge duplicates)
- **BWA** (BWA MEM (v0.7.17) with -5SP flag aligns cleaned Hi-C paired-end reads to the assembled contigs)
- **samtools** (samtools view -F 0x904 filters unmapped, supplementary, and secondary alignments; samtools sort name-sorts the BAM)
- **MEGAHIT** (assembles the shotgun reads into contigs (--min-contig-len 1000) that serve as the alignment reference)

## Examples

```
bwa mem -5SP contigs.fasta SGX_R1.clean.fastq.gz SGX_R2.clean.fastq.gz | samtools view -F 0x904 -b | samtools sort -n -o hicsorted.bam
```

## Evaluation signals

- BAM is sorted by read name (not coordinate) and accepted by ViralCC's pipeline without sort-order errors
- Filtered alignments contain no unmapped reads, secondary alignments, or supplementary alignments (consistent with samtools view -F 0x904)
- Hi-C reads show 10 fewer leading nucleotides than the raw reads, matching the ftl=10 trimming step
- Clumpify removes identical optical/tile-edge duplicates; retained read pairs are unique
- Alignment MAPQ and match-length distributions pass ViralCC defaults (--min-mapq 30, --min-match 30) downstream

## Limitations

- CheckV-based completeness evaluation can be insensitive — a random binning control is recommended when using completeness as a metric, so alignment quality alone does not validate final binning success
- NCBI download links for raw data may change, so preprocessing may require updated accession URLs
- Only alignments with sufficient quality survive the -F 0x904 filter and ViralCC's min-mapq/min-match thresholds; very sparse Hi-C contact maps are expected for viral contigs

## Evidence

- [methods] Adaptor sequences are removed by `bbduk` from the `BBTools` suite with parameter `ktrim=r k=23 mink=11 hdist=1 minlen=50 tpe tbo`: "Adaptor sequences are removed by `bbduk` from the `BBTools` suite with parameter `ktrim=r k=23 mink=11 hdist=1 minlen=50 tpe tbo`"
- [methods] Hi-C first 10 nt trimming: "the first 10 nucleotides of Hi-C reads are trimmed by `bbduk` with parameter `ftl=10`"
- [methods] samtools filtering: "samtools with parameters ‘view -F 0x904’ is applied to remove unmapped reads, supplementary alignments, and secondary alignments"
- [readme] Name-sort requirement: "BAM file needs to be sorted **by name** using 'samtools sort'"
- [methods] Duplicate removal: "Identical PCR optical and tile-edge duplicates for Hi-C reads were removed by the script `clumpify.sh` from `BBTools` suite."
