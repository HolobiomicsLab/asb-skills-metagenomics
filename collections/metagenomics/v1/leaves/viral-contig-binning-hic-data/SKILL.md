---
name: viral-contig-binning-hic-data
description: Use when metagenomic Hi-C data (assembled contig FASTA, read-pair-sorted
  BAM, viral contig list) must be binned into viral genomes and compared against other
  binning tools using clustering metrics and completeness-based vMAG counts.
when_to_use_negative:
- Input contigs have not been filtered to putative viral contigs — shotgun/Hi-C-based
  general binners like HiCBin cannot bin viral contigs because TAXAassign can hardly
  annotate viral contigs
- Only a handful of putative viral genomes are present — the paper notes benchmarking
  effectiveness is less convincing when there are few putative viral genomes
- The goal is contamination estimation of vMAGs via marker genes — CheckV cannot estimate
  contamination of vMAGs since no marker gene set is available for viruses
edam_operation: http://edamontology.org/operation_3459
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0622
tools:
- name: ViralCC
  role: Primary Hi-C-based viral binning pipeline using Hi-C interaction graph plus
    host proximity graph and Leiden clustering
  repo: https://github.com/dyxstat/ViralCC
- name: VAMB
  role: Comparator binner run on identical mock inputs for benchmarking
- name: CoCoNet
  role: Comparator viral binning tool using composition and co-occurrence features
- name: vRhyme
  role: Comparator viral binning tool
- name: bin3C
  role: Comparator Hi-C-based binning tool
- name: MetaTOR
  role: Comparator Hi-C-based binning tool
- name: CheckV
  role: Estimates completeness of recovered vMAGs against a curated reference database
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:305653af493869a5505d0534860a159a05578d0c50d97bab526643df25b0a29c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - CheckV
  - CoCoNet
  - MetaTOR
  - VAMB
  - ViralCC
  - bin3C
  - vRhyme
  edam_operation: http://edamontology.org/operation_3459
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# viral-contig-binning-hic-data

## Summary

Benchmark viral contig binning from metagenomic Hi-C data by running ViralCC against comparator binners (VAMB, CoCoNet, vRhyme, bin3C, MetaTOR) on the same contigs and read-pair-sorted Hi-C BAM, then scoring clusters against ground-truth genome labels with F-score, ARI, NMI, homogeneity and CheckV-based vMAG counts.

## When to use

Use when metagenomic Hi-C data (assembled contig FASTA, read-pair-sorted BAM, viral contig list) must be binned into viral genomes and compared against other binning tools using clustering metrics and completeness-based vMAG counts.

## When NOT to use

- Input contigs have not been filtered to putative viral contigs — shotgun/Hi-C-based general binners like HiCBin cannot bin viral contigs because TAXAassign can hardly annotate viral contigs
- Only a handful of putative viral genomes are present — the paper notes benchmarking effectiveness is less convincing when there are few putative viral genomes
- The goal is contamination estimation of vMAGs via marker genes — CheckV cannot estimate contamination of vMAGs since no marker gene set is available for viruses

## Inputs

- Assembled metagenomic contigs (FASTA), restricted to a list of putative viral contigs
- Hi-C paired-end alignment BAM sorted by read-pair name (after samtools view -F 0x904 filtering)
- Viral contig list file (e.g., from VirSorter)
- Ground-truth genome labels for mock/benchmark viral contigs
- CheckV database for completeness estimation

## Outputs

- cluster_viral_contig.txt: contig-to-cluster assignments
- VIRAL_BIN/*.fa: draft viral genome bins (vMAGs)
- Per-tool table of F-score, ARI, NMI, homogeneity and near-complete/high-quality vMAG counts
- CheckV completeness categories per recovered vMAG

## How to apply

Restrict the analysis to the putative viral contigs (e.g., the 1010 mock viral contigs of the mock human gut dataset, min-len 1000) and run ViralCC via 'python viralcc.py pipeline FASTA BAM VIRAL OUTPUT' with default parameters (--min-len 1000, --min-mapq 30, --min-match 30, --min-k 4, --random-seed 42) to produce cluster_viral_contig.txt and VIRAL_BIN/*.fa. Run each comparator (VAMB, CoCoNet, vRhyme, bin3C, MetaTOR) on identical inputs so that differences reflect binning algorithms rather than preprocessing. Match each tool's clusters to ground-truth genome labels of the mock contigs and compute F-score, ARI, NMI and homogeneity as in Fig. 2. Assess recovered vMAGs with CheckV (v0.7.0 in the paper) for completeness categories and count near-complete and high-quality vMAGs per tool, including a random binning control since random binning does not reduce CheckV completeness. Validate by requiring recomputed ViralCC values to match published Fig. 2 results (e.g., 26 near-complete vMAGs, F-score 0.795, ARI 0.787 vs 22/5/5/1/0 for MetaTOR/bin3C/vRhyme/CoCoNet/VAMB) within tolerance and preserving the performance ordering.

## Related tools

- **ViralCC** (Primary Hi-C-based viral binning pipeline using Hi-C interaction graph plus host proximity graph and Leiden clustering) — https://github.com/dyxstat/ViralCC
- **VAMB** (Comparator binner run on identical mock inputs for benchmarking)
- **CoCoNet** (Comparator viral binning tool using composition and co-occurrence features)
- **vRhyme** (Comparator viral binning tool)
- **bin3C** (Comparator Hi-C-based binning tool)
- **MetaTOR** (Comparator Hi-C-based binning tool)
- **CheckV** (Estimates completeness of recovered vMAGs against a curated reference database)

## Examples

```
python ./viralcc.py pipeline --min-len 1000 --min-mapq 30 --min-match 30 --min-k 4 --random-seed 42 contigs.fa reads_pair_sorted.bam viral_contigs.txt ViralCC_OUT
```

## Evaluation signals

- Recomputed ViralCC metrics on the mock human gut dataset match published Fig. 2 values (F-score 0.795, ARI 0.787, 26 near-complete vMAGs) within numerical tolerance
- Performance ordering is preserved: ViralCC > MetaTOR (22 vMAGs) > bin3C (5) = vRhyme (5) > CoCoNet (1) > VAMB (0)
- ViralCC produces cluster_viral_contig.txt and VIRAL_BIN/*.fa without errors when run with default parameters on the 1010 mock viral contigs
- A random binning control is included whenever CheckV completeness is the evaluation metric
- Every tool is benchmarked on the same preprocessed contigs and BAM so metric differences are attributable to binning

## Limitations

- CheckV completeness as a metric requires a random binning control because random binning does not reduce CheckV completeness
- The benchmarking strategy only uses viral genomes recoverable by a single contig, underestimating true viral community diversity
- Putative viral genomes in the benchmarking tend to be small, introducing a size bias
- Benchmarking effectiveness is less convincing when there are few putative viral genomes

## Evidence

- [abstract] ViralCC outperforms existing Hi-C-based binning methods: "we demonstrate that ViralCC outperforms existing Hi-C-based binning methods as well as state-of-the-art tools specifically dedicated to metagenomic viral binning."
- [methods] ViralCC pipeline invocation: "python ./viralcc.py pipeline [Parameters] FASTA_file BAM_file VIRAL_file OUTPUT_directory"
- [methods] ViralCC parameter defaults: "--min-len: Minimum acceptable contig length (default 1000)"
- [intro] CheckV for vMAG completeness: "CheckV has been widely used to estimate the completeness of vMAGs by comparing them to a large database curated from NCBI GenBank and environmental samples"
- [discussion] Random binning control needed: "it is necessary to construct a random binning model as control experiments when the CheckV completeness is used as the evaluation metric"
- [readme] Reproduction repo: "If you want to reproduce results in our ViralCC paper, please read our instructions [here](https://github.com/dyxstat/Reproduce_ViralCC)."
