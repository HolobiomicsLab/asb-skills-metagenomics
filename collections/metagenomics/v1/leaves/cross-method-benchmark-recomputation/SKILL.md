---
name: cross-method-benchmark-recomputation
description: Use when a binning/clustering tool must be benchmarked against published
  baselines on mock data with known per-contig genome labels, requiring recomputation
  of metrics (F-score, ARI, NMI, homogeneity) and vMAG quality counts.
when_to_use_negative:
- When no ground-truth genome labels exist for the contigs — the article notes true
  labels of viral contigs in real datasets are unknown, so metrics like the fraction
  of spurious host-proximity contacts cannot be computed
- When evaluating HiCBin for viral contigs — TAXAassign can hardly annotate viral
  contigs, so HiCBin cannot bin them and is not a valid comparator here
- When the goal is virus-host pair detection or CRISPR-based validation rather than
  binning performance benchmarking
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3174
tools:
- name: ViralCC
  role: Target method under benchmark; run with default parameters on contigs, BAM,
    and viral contig list
  repo: https://github.com/dyxstat/ViralCC
- name: VAMB
  role: Comparator binning method run on the same mock human gut inputs
- name: CoCoNet
  role: Comparator metagenomic viral binning tool using composition and co-occurrence
    features
- name: vRhyme
  role: Comparator metagenomic viral binning tool
- name: bin3C
  role: Comparator Hi-C-based binning method
- name: MetaTOR
  role: Comparator Hi-C-based binning method
- name: CheckV
  role: Assesses vMAG completeness categories for near-complete/high-quality vMAG
    counting
- name: Reproduce_ViralCC
  role: Provides source data of Fig. 2/3 and scripts for recomputing and plotting
    paper results
  repo: https://github.com/dyxstat/Reproduce_ViralCC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:b2114db7a0e5b83a277caf587a42a0b9e5fb6d842225eeac29c9b708a2e8832a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - CheckV
  - CoCoNet
  - MetaTOR
  - Reproduce_ViralCC
  - VAMB
  - ViralCC
  - bin3C
  - vRhyme
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# cross-method-benchmark-recomputation

## Summary

Recompute and compare clustering/binning performance metrics across multiple methods on a shared mock benchmark with ground-truth genome labels, as done for ViralCC vs. VAMB, CoCoNet, vRhyme, bin3C, and MetaTOR on the 1010 mock viral contigs of the mock human gut Hi-C dataset.

## When to use

Use when a binning/clustering tool must be benchmarked against published baselines on mock data with known per-contig genome labels, requiring recomputation of metrics (F-score, ARI, NMI, homogeneity) and vMAG quality counts.

## When NOT to use

- When no ground-truth genome labels exist for the contigs — the article notes true labels of viral contigs in real datasets are unknown, so metrics like the fraction of spurious host-proximity contacts cannot be computed
- When evaluating HiCBin for viral contigs — TAXAassign can hardly annotate viral contigs, so HiCBin cannot bin them and is not a valid comparator here
- When the goal is virus-host pair detection or CRISPR-based validation rather than binning performance benchmarking

## Inputs

- Assembled contigs FASTA (mock human gut assembly, min-contig-len 1000 via MEGAHIT)
- Read-pair-sorted Hi-C alignment BAM (name-sorted, filtered with samtools view -F 0x904)
- Viral contig list restricting analysis to the 1010 mock viral contigs (e.g., from VirSorter v1.0.6)
- Ground-truth genome labels per mock contig
- Bin/cluster assignments from each comparator tool (VAMB, CoCoNet, vRhyme, bin3C, MetaTOR)
- CheckV database (CheckV v0.7.0) for vMAG completeness assessment

## Outputs

- Per-tool table of F-score, ARI, NMI, and homogeneity on the 1010 mock viral contigs
- Per-tool counts of near-complete and high-quality vMAGs (CheckV categories)
- Random binning control results for completeness-based evaluation
- Comparison against published Fig. 2 source data values

## How to apply

Restrict all tools to the same input set — here the 1010 mock viral contigs of the mock human gut assembly — then run each method with its published default or paper-reported parameters (ViralCC: --min-len 1000, --min-mapq 30, --min-match 30, --min-k 4, --random-seed 42) on identical inputs (contigs FASTA, read-pair-sorted Hi-C BAM, viral contig list). Match each tool's cluster assignments against ground-truth genome labels to compute the four clustering metrics reported in Fig. 2 (F-score, ARI, NMI, homogeneity). Assess recovered vMAG completeness with CheckV (v0.7.0 in the paper) and count near-complete and high-quality vMAGs per tool; include a random binning control whenever CheckV completeness is the evaluation metric, since random binning does not reduce CheckV completeness. Assemble a per-tool table and validate that recomputed values match the published source data (available in the Reproduce_ViralCC 'Source Data' folder) within numerical tolerance while preserving the performance ordering (ViralCC F-score 0.795, ARI 0.787, 26 near-complete vMAGs vs. 1/5/0/5/22 for VAMB/CoCoNet/vRhyme/bin3C/MetaTOR). Pin all software versions, e.g. bbduk/clumpify v37.25, MEGAHIT v1.2.9, BWA v0.7.17, samtools v1.15.1, VirSorter v1.0.6, CheckV 0.7.0, as listed in the reproduction instructions.

## Related tools

- **ViralCC** (Target method under benchmark; run with default parameters on contigs, BAM, and viral contig list) — https://github.com/dyxstat/ViralCC
- **VAMB** (Comparator binning method run on the same mock human gut inputs)
- **CoCoNet** (Comparator metagenomic viral binning tool using composition and co-occurrence features)
- **vRhyme** (Comparator metagenomic viral binning tool)
- **bin3C** (Comparator Hi-C-based binning method)
- **MetaTOR** (Comparator Hi-C-based binning method)
- **CheckV** (Assesses vMAG completeness categories for near-complete/high-quality vMAG counting)
- **Reproduce_ViralCC** (Provides source data of Fig. 2/3 and scripts for recomputing and plotting paper results) — https://github.com/dyxstat/Reproduce_ViralCC

## Examples

```
python ./viralcc.py pipeline --min-len 1000 --min-mapq 30 --min-match 30 --random-seed 42 contigs.fasta hic.bam viral_contigs.txt OUT
```

## Evaluation signals

- Recomputed ViralCC F-score (0.795) and ARI (0.787) match published Fig. 2 mock human gut values within numerical tolerance
- Near-complete vMAG counts reproduce the reported ordering: ViralCC 26 vs. MetaTOR 22, bin3C 5, CoCoNet 5, VAMB 1, vRhyme 0
- Random binning control is included and reported whenever CheckV completeness is used as the evaluation metric
- All four metrics (F-score, ARI, NMI, homogeneity) preserve ViralCC outperforming all five comparator binners
- Software versions match the reproduction instructions (CheckV 0.7.0, MEGAHIT v1.2.9, BWA v0.7.17, samtools v1.15.1, VirSorter v1.0.6, BBTools v37.25)

## Limitations

- The benchmarking strategy only chooses viral genomes recoverable by a single contig, which inevitably underestimates the true diversity of the virus community
- CheckV assessment is not comprehensive and cannot estimate contamination of vMAGs since no viral marker gene set is available
- A random binning model is necessary as a control when CheckV completeness is the evaluation metric because random binning does not reduce CheckV completeness
- Putative viral genomes in the benchmarking tend to be small (size bias), and benchmarking is less convincing when there are few putative viral genomes

## Evidence

- [abstract] we demonstrate that ViralCC outperforms existing Hi-C-based binning methods as well as state-of-the-art tools specifically dedicated to metagenomic viral binning.: "we demonstrate that ViralCC outperforms existing Hi-C-based binning methods as well as state-of-the-art tools specifically dedicated to metagenomic viral binning."
- [methods] python ./viralcc.py pipeline [Parameters] FASTA_file BAM_file VIRAL_file OUTPUT_directory: "python ./viralcc.py pipeline [Parameters] FASTA_file BAM_file VIRAL_file OUTPUT_directory"
- [discussion] it is necessary to construct a random binning model as control experiments when the CheckV completeness is used as the evaluation metric: "it is necessary to construct a random binning model as control experiments when the CheckV completeness is used as the evaluation metric"
- [readme] Source data of Figure 2 and 3 in the main text and Figure S1 in the supplementary materials are provided in the folder: [evidence span withheld — anchor status: external_doc]
- [discussion] Since we only choose viral genomes that can be recovered by a single contig from the whole community, our benchmarking method inevitably underestimates the true diversity of the virus community.: "Since we only choose viral genomes that can be recovered by a single contig from the whole community, our benchmarking method inevitably underestimates the true diversity"
