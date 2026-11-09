---
name: metagenomic-hic-binning-benchmarking
description: Use when comparing viral binning tools on metagenomic Hi-C datasets (shotgun
  + Hi-C FASTQ pairs from SRA/ENA) and evaluating recovered vMAGs by completeness
  categories.
when_to_use_negative:
- The goal is completeness assessment for prokaryotic MAGs — CheckM with universal
  single-copy marker genes is the appropriate tool, since viruses lack universal single-copy
  marker genes
- Estimating contamination of vMAGs — CheckV cannot estimate contamination because
  no marker gene set is available for viruses
- Few putative viral genomes are available in the sample — the article notes benchmarking
  effectiveness is less convincing in this regime, and the strategy underestimates
  true viral community diversity because only single-contig-recoverable genomes were
  chosen
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: ViralCC
  role: Hi-C-based viral binning method under evaluation; integrates Hi-C interaction
    graph and host proximity graph with Leiden clustering to reconstruct draft viral
    genomes
  repo: https://github.com/dyxstat/ViralCC
- name: MetaTOR
  role: Hi-C-based binning baseline for comparison
- name: bin3C
  role: Hi-C-based binning baseline for comparison
- name: HiCBin
  role: Hi-C-based binning baseline; requires TAXAassign taxonomic annotation and
    cannot bin viral contigs
- name: CoCoNet
  role: shotgun-based viral binning baseline using composition and co-occurrence features
- name: vRhyme
  role: shotgun-based viral binning baseline for comparison
- name: TAXAassign
  role: taxonomic annotation of contigs required by HiCBin to generate intra-species
    contacts
  repo: https://github.com/umerijaz/TAXAassign
- name: CheckV
  role: estimates completeness of recovered vMAGs against a curated GenBank/environmental
    database; versions v0.7.0 used
- name: MEGAHIT
  role: assembles cleaned shotgun reads (v1.2.9, --min-contig-len 1000)
- name: BWA
  role: BWA MEM (-5SP) aligns Hi-C reads to assembled contigs (v0.7.17)
- name: samtools
  role: filters alignments (view -F 0x904) and name-sorts the BAM (v1.15.1)
- name: BBTools
  role: bbduk for adaptor/quality trimming and Hi-C ftl=10 trimming; clumpify.sh for
    duplicate removal (v37.25)
- name: VirSorter
  role: identifies viral contigs from assemblies to produce the VIRAL_file (v1.0.6)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:3c0e660d1dd95029958336d8ea8d9a18319d617cb007b7f32229bc5239abd0c9
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# metagenomic-hic-binning-benchmarking

## Summary

A benchmarking workflow for comparing metagenomic Hi-C-based viral binning methods (ViralCC, MetaTOR, bin3C, HiCBin, CoCoNet, vRhyme) on real human gut, cow fecal, and wastewater datasets, using CheckV-estimated high/medium-completion draft viral genome counts as the evaluation metric.

## When to use

Use when comparing viral binning tools on metagenomic Hi-C datasets (shotgun + Hi-C FASTQ pairs from SRA/ENA) and evaluating recovered vMAGs by completeness categories.

## When NOT to use

- The goal is completeness assessment for prokaryotic MAGs — CheckM with universal single-copy marker genes is the appropriate tool, since viruses lack universal single-copy marker genes
- Estimating contamination of vMAGs — CheckV cannot estimate contamination because no marker gene set is available for viruses
- Few putative viral genomes are available in the sample — the article notes benchmarking effectiveness is less convincing in this regime, and the strategy underestimates true viral community diversity because only single-contig-recoverable genomes were chosen

## Inputs

- Raw shotgun FASTQ files from NCBI SRA/ENA (accessions SRR6131122-24, ERX2333418/ERX2548555-56, SRR8239392-93)
- Raw metagenomic Hi-C paired-end FASTQ files
- Assembled contigs FASTA (MEGAHIT, --min-contig-len 1000)
- Name-sorted BAM of Hi-C reads aligned with BWA MEM and filtered with samtools view -F 0x904
- VIRAL_file: viral contigs identified by VirSorter
- CheckV curated GenBank/environmental reference database

## Outputs

- Draft viral genome bins (vMAGs) per method per dataset
- CheckV completeness estimates with high/medium completion categories per bin
- Per-method, per-dataset high/medium-completion counts and percentage improvements of ViralCC over baselines
- Comparison against Figure 3 values from the ViralCC paper

## How to apply

Download raw shotgun and Hi-C FASTQ files (e.g., ERR2282092, ERR2530126/27) and clean them with BBTools: bbduk adaptor removal (ktrim=r k=23 mink=11 hdist=1 minlen=50 tpe tbo), quality trimming (trimq=10 qtrim=r ftm=5 minlen=50), trim the first 10 nucleotides of Hi-C reads (ftl=10), and remove PCR optical/tile-edge duplicates with clumpify.sh. Assemble shotgun reads with MEGAHIT (--min-contig-len 1000), align Hi-C reads with BWA MEM (-5SP), filter with samtools view -F 0x904, and name-sort the BAM. Identify viral contigs with VirSorter (wrapper_phage_contigs_sorter_iPlant.pl) to produce the VIRAL_file, then run ViralCC (python ./viralcc.py pipeline FASTA_file BAM_file VIRAL_file OUTPUT_directory) and baselines (MetaTOR, bin3C, HiCBin, CoCoNet, vRhyme). Estimate completeness of all recovered bins with CheckV v0.7.0 against its curated GenBank/environmental database, assign high/medium completion categories, and compute percentage improvements of ViralCC over each baseline. Success is judged by whether counts and percentage improvements match Fig. 3 (e.g., on cow fecal, ViralCC exceeded VAMB, CoCoNet, vRhyme, bin3C, MetaTOR by 161%, 140%, 66.7%, 93.5%, 62.1%).

## Related tools

- **ViralCC** (Hi-C-based viral binning method under evaluation; integrates Hi-C interaction graph and host proximity graph with Leiden clustering to reconstruct draft viral genomes) — https://github.com/dyxstat/ViralCC
- **MetaTOR** (Hi-C-based binning baseline for comparison)
- **bin3C** (Hi-C-based binning baseline for comparison)
- **HiCBin** (Hi-C-based binning baseline; requires TAXAassign taxonomic annotation and cannot bin viral contigs)
- **CoCoNet** (shotgun-based viral binning baseline using composition and co-occurrence features)
- **vRhyme** (shotgun-based viral binning baseline for comparison)
- **TAXAassign** (taxonomic annotation of contigs required by HiCBin to generate intra-species contacts) — https://github.com/umerijaz/TAXAassign
- **CheckV** (estimates completeness of recovered vMAGs against a curated GenBank/environmental database; versions v0.7.0 used)
- **MEGAHIT** (assembles cleaned shotgun reads (v1.2.9, --min-contig-len 1000))
- **BWA** (BWA MEM (-5SP) aligns Hi-C reads to assembled contigs (v0.7.17))
- **samtools** (filters alignments (view -F 0x904) and name-sorts the BAM (v1.15.1))
- **BBTools** (bbduk for adaptor/quality trimming and Hi-C ftl=10 trimming; clumpify.sh for duplicate removal (v37.25))
- **VirSorter** (identifies viral contigs from assemblies to produce the VIRAL_file (v1.0.6))

## Examples

```
python ./viralcc.py pipeline FASTA_file BAM_file VIRAL_file OUTPUT_directory
```

## Evaluation signals

- Reproduced per-method, per-dataset high/medium-completion counts match the values reported in Fig. 3 of the ViralCC paper (source data available in dyxstat/Reproduce_ViralCC Source Data folder)
- On cow fecal, ViralCC exceeds VAMB, CoCoNet, vRhyme, bin3C, and MetaTOR by 161%, 140%, 66.7%, 93.5%, and 62.1% respectively
- All alignments passing filter are mapped, primary, non-supplementary (samtools view -F 0x904 removes 0x4, 0x100, 0x800 flags)
- HiCC BAM is sorted by name before running ViralCC, which requires name-sorted input
- Software versions match those reported: BBTools v37.25, MEGAHIT v1.2.9, BWA v0.7.17, samtools v1.15.1, VirSorter v1.0.6, CheckV 0.7.0

## Limitations

- CheckV completeness as an evaluation metric requires a random binning control model, since random binning does not reduce CheckV completeness
- CheckV cannot estimate contamination of vMAGs (no viral marker gene set exists), so assessment by CheckV is not comprehensive
- The benchmarking strategy underestimates true viral community diversity because only viral genomes recoverable by a single contig were chosen, and putative viral genomes in the benchmarking tend to be small (size bias)

## Evidence

- [methods] samtools filtering: "samtools with parameters 'view -F 0x904' is applied to remove unmapped reads, supplementary alignments, and secondary alignments"
- [intro] CheckV usage: "CheckV has been widely used to estimate the completeness of vMAGs by comparing them to a large database curated from NCBI GenBank and environmental samples"
- [discussion] random binning control: "it is necessary to construct a random binning model as control experiments when the CheckV completeness is used as the evaluation metric"
- [results] cow fecal finding: [evidence span withheld — anchor status: not_found]
- [readme] software versions: "wrapper_phage_contigs_sorter_iPlant.pl command from VirSorter: v1.0.6

checkv command from CheckV: 0.7.0"
