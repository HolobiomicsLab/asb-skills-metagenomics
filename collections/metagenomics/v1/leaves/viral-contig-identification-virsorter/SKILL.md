---
name: viral-contig-identification-virsorter
description: Use when you have assembled metagenomic contigs (MEGAHIT, --min-contig-len
  1000) from shotgun data and need to flag viral contigs as input for Hi-C-based viral
  genome binning with ViralCC.
when_to_use_negative:
- Input is raw FASTQ reads rather than assembled contigs — VirSorter in this workflow
  is applied only after MEGAHIT assembly (min contig length 1000)
- Input contigs are already classified as viral or a VIRAL_file already exists for
  the assembly — re-identification is redundant
- The target task is bacterial/archaeal MAG binning with marker-gene tools such as
  CheckM rather than viral genome recovery
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: VirSorter
  role: Identifies viral contigs from assembled metagenomic contigs via wrapper_phage_contigs_sorter_iPlant.pl,
    producing the VIRAL_file input for ViralCC
- name: MEGAHIT
  role: Assembles cleaned shotgun reads into contigs (--min-contig-len 1000) that
    are the input to VirSorter
- name: ViralCC
  role: Consumes the VirSorter VIRAL_file together with the FASTA and name-sorted
    BAM to bin viral contigs via the Hi-C interaction graph and host proximity graph
  repo: https://github.com/dyxstat/ViralCC
- name: CheckV
  role: Estimates completeness of the recovered draft viral genomes against its curated
    GenBank/environmental database (v0.7.0)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:cc4e5bc268ceb091b04f9fef66f00fb82901aa3e33e19d761d9b6635e3f64135
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - BBTools
  - BWA
  - CheckV
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# viral-contig-identification-virsorter

## Summary

Identify putative viral contigs from metagenomic shotgun assemblies using VirSorter (wrapper_phage_contigs_sorter_iPlant.pl v1.0.6), producing the VIRAL_file that the ViralCC pipeline requires to restrict binning to viral contigs.

## When to use

Use when you have assembled metagenomic contigs (MEGAHIT, --min-contig-len 1000) from shotgun data and need to flag viral contigs as input for Hi-C-based viral genome binning with ViralCC.

## When NOT to use

- Input is raw FASTQ reads rather than assembled contigs — VirSorter in this workflow is applied only after MEGAHIT assembly (min contig length 1000)
- Input contigs are already classified as viral or a VIRAL_file already exists for the assembly — re-identification is redundant
- The target task is bacterial/archaeal MAG binning with marker-gene tools such as CheckM rather than viral genome recovery

## Inputs

- Assembled shotgun contigs in FASTA format (e.g. MEGAHIT final.contigs, contigs >= 1000 bp)
- Cleaned shotgun FASTQ reads (bbduk adaptor/quality-trimmed, clumpify.sh deduplicated) used to generate the assembly

## Outputs

- VIRAL_file listing viral contigs from each assembly, consumed by ViralCC as a required pipeline input
- Viral contigs eligible for downstream CheckV 0.7.0 completeness assessment (high/medium completion categories)

## How to apply

First assemble cleaned shotgun reads into contigs (MEGAHIT v1.2.9, --min-contig-len 1000 --k-min 21 --k-max 141 --k-step 12 --merge-level 20,0.95); VirSorter is designed for assembled contigs, not raw reads. Run VirSorter v1.0.6 via wrapper_phage_contigs_sorter_iPlant.pl on each assembly to classify contigs as viral, keeping reads cleaned per the ViralCC recipe (bbduk adaptor removal with ktrim=r k=23 mink=11 hdist=1 minlen=50 tpe tbo; quality trimming trimq=10 qtrim=r ftm=5 minlen=50; clumpify.sh duplicate removal). The output VIRAL_file is passed directly to ViralCC as the third argument: 'python ./viralcc.py pipeline FASTA_file BAM_file VIRAL_file OUTPUT_directory'. Downstream completeness of the resulting bins is judged with CheckV 0.7.0 against its curated GenBank/environmental database, so the viral-contig set directly conditions the high/medium-completion counts reported in Fig. 3 of the paper. Use the exact software versions listed in dyxstat/Reproduce_ViralCC to make counts comparable with the published benchmarks.

## Related tools

- **VirSorter** (Identifies viral contigs from assembled metagenomic contigs via wrapper_phage_contigs_sorter_iPlant.pl, producing the VIRAL_file input for ViralCC)
- **MEGAHIT** (Assembles cleaned shotgun reads into contigs (--min-contig-len 1000) that are the input to VirSorter)
- **ViralCC** (Consumes the VirSorter VIRAL_file together with the FASTA and name-sorted BAM to bin viral contigs via the Hi-C interaction graph and host proximity graph) — https://github.com/dyxstat/ViralCC
- **CheckV** (Estimates completeness of the recovered draft viral genomes against its curated GenBank/environmental database (v0.7.0))

## Examples

```
python ./viralcc.py pipeline FASTA_file BAM_file VIRAL_file OUTPUT_directory
```

## Evaluation signals

- VIRAL_file contains only contigs present in the assembly and flagged viral by VirSorter v1.0.6; no contigs shorter than the 1000 bp assembly threshold appear
- Running ViralCC with the VIRAL_file (python ./viralcc.py pipeline FASTA_file BAM_file VIRAL_file OUTPUT_directory) completes without missing-input errors
- High/medium-completion counts from CheckV 0.7.0 on downstream bins match the source data provided in dyxstat/Reproduce_ViralCC (Source Data folder for Fig. 3)
- Reproduced per-method percentage improvements (e.g. ViralCC exceeding VAMB, CoCoNet, vRhyme, bin3C, MetaTOR by 161%, 140%, 66.7%, 93.5%, 62.1% on cow fecal) are consistent with published values
- Software versions match the reproduction README (VirSorter v1.0.6, MEGAHIT v1.2.9, CheckV 0.7.0)

## Limitations

- CheckV assessment of completeness is not comprehensive and cannot estimate contamination of vMAGs because no universal viral marker-gene set exists
- A random binning control is necessary when using CheckV completeness as the evaluation metric, since random binning does not reduce CheckV completeness
- The benchmarking strategy underestimates true viral community diversity because only single-contig-recoverable viral genomes are chosen, and putative viral genomes tend to be small (size bias)

## Evidence

- [intro] Viral contigs are subsequently identified.: "Viral contigs are subsequently identified."
- [methods] Identify viral contigs from assembled contigs: "### Identify viral contigs from assembled contigs"
- [readme] VirSorter version used for reproduction: [evidence span withheld — anchor status: external_doc]
- [methods] ViralCC consumes the VirSorter VIRAL_file: "python ./viralcc.py pipeline [Parameters] FASTA_file BAM_file VIRAL_file OUTPUT_directory"
