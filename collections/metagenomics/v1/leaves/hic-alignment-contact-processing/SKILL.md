---
name: hic-alignment-contact-processing
description: Use when raw Hi-C paired-end and shotgun FASTQ reads must be converted
  into a name-sorted, filtered BAM aligned to assembled contigs before running a Hi-C-based
  binning pipeline such as ViralCC.
when_to_use_negative:
- The Hi-C BAM already exists, is read-pair (name) sorted, and filtered of unmapped/secondary/supplementary
  alignments
- Input is single-cell or non-proximity-ligation data without paired-end Hi-C contacts,
  where a read-pair contact map is meaningless
- The goal is taxonomic annotation of host contigs (e.g. TAXAassign) rather than producing
  ViralCC's alignment input
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: bbduk (BBTools suite)
  role: Adaptor removal, quality trimming, first-10-nucleotide trimming of Hi-C reads
  repo: https://github.com/dyxstat/Reproduce_ViralCC
- name: clumpify.sh (BBTools suite)
  role: Removal of PCR optical and tile-edge duplicate Hi-C reads
  repo: https://github.com/dyxstat/Reproduce_ViralCC
- name: MEGAHIT
  role: Assembles cleaned shotgun reads into contigs with min contig length 1000
  repo: https://github.com/dyxstat/Reproduce_ViralCC
- name: BWA MEM
  role: Aligns cleaned Hi-C paired-end reads to assembled contigs
  repo: https://github.com/dyxstat/Reproduce_ViralCC
- name: samtools
  role: Filters alignments (view -F 0x904) and sorts the BAM by name for ViralCC
  repo: https://github.com/dyxstat/Reproduce_ViralCC
- name: ViralCC
  role: Downstream consumer of the FASTA + BAM + viral contig list using --min-len
    1000, --min-mapq 30, --min-match 30
  repo: https://github.com/dyxstat/ViralCC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:719926a1bfe1a1e5623d19327a6c1faffbb99d06205547e5b3012e803d7541b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/Reproduce_ViralCC
  tools:
  - BWA MEM
  - CoCoNet
  - MEGAHIT
  - MetaTOR
  - VAMB
  - ViralCC
  - bbduk (BBTools suite)
  - bin3C
  - clumpify.sh (BBTools suite)
  - samtools
  - vRhyme
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# hic-alignment-contact-processing

## Summary

Preprocess raw shotgun and Hi-C reads, assemble contigs, align Hi-C paired-end reads to the assembly, and filter/sort the BAM so it is suitable for ViralCC's Hi-C interaction and host proximity graph construction.

## When to use

Use when raw Hi-C paired-end and shotgun FASTQ reads must be converted into a name-sorted, filtered BAM aligned to assembled contigs before running a Hi-C-based binning pipeline such as ViralCC.

## When NOT to use

- The Hi-C BAM already exists, is read-pair (name) sorted, and filtered of unmapped/secondary/supplementary alignments
- Input is single-cell or non-proximity-ligation data without paired-end Hi-C contacts, where a read-pair contact map is meaningless
- The goal is taxonomic annotation of host contigs (e.g. TAXAassign) rather than producing ViralCC's alignment input

## Inputs

- Raw Hi-C paired-end FASTQ reads (e.g. FASTQ.gz from SRA)
- Raw shotgun FASTQ reads
- Assembled contigs FASTA (e.g. MEGAHIT output)
- Viral contig list for downstream ViralCC binning

## Outputs

- Cleaned, adapter- and duplicate-trimmed FASTQ files
- Assembly contigs FASTA (min contig length 1000)
- Filtered, name-sorted BAM of Hi-C pairs aligned to contigs

## How to apply

First clean the raw reads with the BBTools suite (v37.25 in the paper): remove adaptors with bbduk (ktrim=r k=23 mink=11 hdist=1 minlen=50 tpe tbo), quality-trim with bbduk (trimq=10 qtrim=r ftm=5 minlen=50), trim the first 10 nucleotides of Hi-C reads (ftl=10), and remove PCR optical and tile-edge duplicates with clumpify.sh. Assemble cleaned shotgun reads with MEGAHIT (v1.2.9, --min-contig-len 1000), then align cleaned Hi-C paired-end reads to the contigs with BWA MEM (v0.7.17). Filter the alignment with 'samtools view -F 0x904' to drop unmapped reads, supplementary alignments, and secondary alignments, and ensure the BAM is sorted by name using samtools sort, since ViralCC requires read-pair-sorted input with thresholds --min-len 1000, --min-mapq 30, and --min-match 30.

## Related tools

- **bbduk (BBTools suite)** (Adaptor removal, quality trimming, first-10-nucleotide trimming of Hi-C reads) — https://github.com/dyxstat/Reproduce_ViralCC
- **clumpify.sh (BBTools suite)** (Removal of PCR optical and tile-edge duplicate Hi-C reads) — https://github.com/dyxstat/Reproduce_ViralCC
- **MEGAHIT** (Assembles cleaned shotgun reads into contigs with min contig length 1000) — https://github.com/dyxstat/Reproduce_ViralCC
- **BWA MEM** (Aligns cleaned Hi-C paired-end reads to assembled contigs) — https://github.com/dyxstat/Reproduce_ViralCC
- **samtools** (Filters alignments (view -F 0x904) and sorts the BAM by name for ViralCC) — https://github.com/dyxstat/Reproduce_ViralCC
- **ViralCC** (Downstream consumer of the FASTA + BAM + viral contig list using --min-len 1000, --min-mapq 30, --min-match 30) — https://github.com/dyxstat/ViralCC

## Examples

```
megahit -1 SG1.fastq.gz -2 SG2.fastq.gz -o ASSEMBLY --min-contig-len 1000
```

## Evaluation signals

- BAM is sorted by name (read-pair sorted), as explicitly required by ViralCC
- BAM contains no unmapped reads, secondary alignments, or supplementary alignments (view -F 0x904 filter already applied)
- All assembly contigs are >= 1000 bp (megahit --min-contig-len 1000)
- Software versions match the paper's stack (BBTools v37.25, MEGAHIT v1.2.9, BWA v0.7.17, samtools v1.15.1) when reproducing results
- Downstream ViralCC run with defaults (--min-mapq 30, --min-match 30, --random-seed 42) produces cluster_viral_contig.txt and VIRAL_BIN/*.fa outputs

## Limitations

- NCBI download links for raw data may change and require checking the latest NCBI links when reproducing
- CheckV-based completeness benchmarking needs a random binning control because random binning does not reduce CheckV completeness
- Alignment quality thresholds (min-mapq 30, min-match 30) may drop contacts for short or repetitive contigs

## Evidence

- [methods] Adaptor sequences are removed by `bbduk` from the `BBTools` suite with parameter `ktrim=r k=23 mink=11 hdist=1 minlen=50 tpe tbo`: "Adaptor sequences are removed by `bbduk` from the `BBTools` suite with parameter `ktrim=r k=23 mink=11 hdist=1 minlen=50 tpe tbo`"
- [methods] quality trimming parameters: "reads are quality-trimmed using `bbduk` with parameters `trimq=10 qtrim=r ftm=5 minlen=50`"
- [methods] samtools filter flag: "samtools with parameters ‘view -F 0x904’ is applied to remove unmapped reads, supplementary alignments, and secondary alignments"
- [methods] BAM name sorting: "BAM file needs to be sorted **by name** using 'samtools sort'"
- [methods] assembly minimum length: "megahit -1 SG1.fastq.gz -2 SG2.fastq.gz -o ASSEMBLY --min-contig-len 1000"
- [readme] software versions used for reproduction: [evidence span withheld — anchor status: external_doc]
- [methods] ViralCC alignment quality threshold: "--min-mapq: Minimum acceptable alignment quality (default 30)"
