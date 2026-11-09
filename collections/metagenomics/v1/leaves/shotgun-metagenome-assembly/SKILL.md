---
name: shotgun-metagenome-assembly
description: Use when starting from cleaned paired-end shotgun FASTQ files from a
  metagenomic Hi-C experiment and you need assembled contigs as the substrate for
  aligning Hi-C reads and binning viral genomes.
when_to_use_negative:
- Input is already an assembly or contig FASTA — skip assembly and proceed directly
  to Hi-C alignment (BWA MEM -5SP) and viral contig identification
- Data are single-cell or long-read assemblies where MEGAHIT's short-read de Bruijn
  graph (k up to 141) is not the appropriate assembler
- Only Hi-C reads are available without matching shotgun reads — co-assembly from
  Hi-C alone is not the designed workflow
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MEGAHIT
  role: Assembles cleaned shotgun paired-end reads into contigs (>=1000 bp) used as
    the ViralCC FASTA_file reference
- name: BBTools
  role: bbduk.sh removes adaptors and quality-trims shotgun reads; clumpify.sh removes
    PCR optical/tile-edge duplicates prior to assembly
- name: BWA
  role: Aligns Hi-C paired-end reads to the assembled contigs with BWA MEM -5SP
- name: samtools
  role: Filters alignments (view -F 0x904) and name-sorts the BAM (samtools sort -n)
    for downstream binning
- name: VirSorter
  role: Identifies viral contigs from the assembly (wrapper_phage_contigs_sorter_iPlant.pl)
    to produce the VIRAL_file input
- name: ViralCC
  role: Consumes the assembly FASTA, sorted BAM, and VIRAL_file to reconstruct draft
    viral genomes
  repo: https://github.com/dyxstat/ViralCC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:30fb96a9f1719859a8ae22cedc6698cf285d001e8151e3f0b15f12f17acbce0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - BBTools
  - BWA
  - CoCoNet
  - HiCBin
  - MEGAHIT
  - MetaTOR
  - TAXAassign
  - VirSorter
  - ViralCC
  - bin3C
  - samtools
  - vRhyme
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# shotgun-metagenome-assembly

## Summary

Assemble cleaned shotgun metagenomic reads into long contigs with MEGAHIT (min-contig-len 1000, k=21-141 step 12) to serve as the reference for Hi-C read alignment and downstream viral binning with ViralCC.

## When to use

Use when starting from cleaned paired-end shotgun FASTQ files from a metagenomic Hi-C experiment and you need assembled contigs as the substrate for aligning Hi-C reads and binning viral genomes.

## When NOT to use

- Input is already an assembly or contig FASTA — skip assembly and proceed directly to Hi-C alignment (BWA MEM -5SP) and viral contig identification
- Data are single-cell or long-read assemblies where MEGAHIT's short-read de Bruijn graph (k up to 141) is not the appropriate assembler
- Only Hi-C reads are available without matching shotgun reads — co-assembly from Hi-C alone is not the designed workflow

## Inputs

- Cleaned paired-end shotgun FASTQ files (e.g. ERR2282092_1/2.fastq.gz from cow fecal, SRR6131122-24 human gut, SRR8239392-93 wastewater)
- BBTools v37.25 (bbduk.sh, clumpify.sh) for read cleaning
- MEGAHIT v1.2.9 assembler

## Outputs

- Assembly FASTA file of contigs >= 1000 bp (FASTA_file input to ViralCC)
- VIRAL_file of viral contigs identified by VirSorter from the assembly
- Name-sorted BAM of filtered Hi-C alignments against the contigs

## How to apply

After cleaning raw shotgun and Hi-C reads with BBTools (bbduk adaptor removal with ktrim=r k=23 mink=11 hdist=1, quality-trimming with trimq=10 qtrim=r ftm=5 minlen=50), assemble the shotgun pairs using MEGAHIT with --min-contig-len 1000 --k-min 21 --k-max 141 --k-step 12 --merge-level 20,0.95. The resulting FASTA of contigs (each >=1000 bp) becomes the reference to which Hi-C paired-end reads are aligned with BWA MEM (-5SP). Filter alignments with samtools view -F 0x904 to drop unmapped, supplementary, and secondary alignments, then name-sort the BAM with samtools sort -n. These contigs are also screened for viral origin with VirSorter (wrapper_phage_contigs_sorter_iPlant.pl) to produce the VIRAL_file required by ViralCC. The assembly version (MEGAHIT v1.2.9) and parameters matter for reproducibility of the paper's completeness comparison against VAMB, CoCoNet, vRhyme, bin3C, and MetaTOR.

## Related tools

- **MEGAHIT** (Assembles cleaned shotgun paired-end reads into contigs (>=1000 bp) used as the ViralCC FASTA_file reference)
- **BBTools** (bbduk.sh removes adaptors and quality-trims shotgun reads; clumpify.sh removes PCR optical/tile-edge duplicates prior to assembly)
- **BWA** (Aligns Hi-C paired-end reads to the assembled contigs with BWA MEM -5SP)
- **samtools** (Filters alignments (view -F 0x904) and name-sorts the BAM (samtools sort -n) for downstream binning)
- **VirSorter** (Identifies viral contigs from the assembly (wrapper_phage_contigs_sorter_iPlant.pl) to produce the VIRAL_file input)
- **ViralCC** (Consumes the assembly FASTA, sorted BAM, and VIRAL_file to reconstruct draft viral genomes) — https://github.com/dyxstat/ViralCC

## Examples

```
megahit -1 SG1.fastq.gz -2 SG2.fastq.gz -o ASSEMBLY --min-contig-len 1000 --k-min 21 --k-max 141 --k-step 12 --merge-level 20,0.95
```

## Evaluation signals

- Assembly FASTA contains only contigs of at least 1000 bp, consistent with megahit --min-contig-len 1000
- Downstream BAM shows no unmapped (0x4), secondary (0x100), or supplementary (0x800) records after samtools view -F 0x904
- BAM is sorted by read name (samtools sort -n), a required input property for ViralCC
- Running the full pipeline on the paper's datasets reproduces the CheckV high/medium-completion vMAG counts and percentage improvements over VAMB, CoCoNet, vRhyme, bin3C, and MetaTOR reported in Fig. 3
- Software versions match the reproducibility record: MEGAHIT v1.2.9, BWA v0.7.17, samtools v1.15.1, BBTools v37.25, VirSorter v1.0.6, CheckV 0.7.0

## Limitations

- The benchmarking strategy only chooses viral genomes recoverable by a single contig, so it underestimates true viral community diversity
- CheckV cannot estimate contamination of vMAGs since no universal marker gene set exists for viruses, so completeness assessment is not comprehensive
- A random binning control model is necessary when using CheckV completeness as an evaluation metric, because random binning does not reduce CheckV completeness
- Putative viral genomes in the benchmark tend to be small, introducing a size bias that should be accounted for

## Evidence

- [methods] MEGAHIT assembly parameters: "megahit -1 SG1.fastq.gz -2 SG2.fastq.gz -o ASSEMBLY --min-contig-len 1000"
- [methods] alignment filtering: "samtools with parameters 'view -F 0x904' is applied to remove unmapped reads, supplementary alignments, and secondary alignments"
- [methods] adaptor removal: "Adaptor sequences are removed by `bbduk` from the `BBTools` suite with parameter `ktrim=r k=23 mink=11 hdist=1 minlen=50 tpe tbo`"
- [intro] CheckV contamination limitation: "CheckV is unable to estimate the contamination of vMAGs since there is no such marker gene set available for viruses"
- [readme] software versions: [evidence span withheld — anchor status: external_doc]
