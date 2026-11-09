---
name: viral-bin-clustering-metric-evaluation
description: Use when multiple Hi-C or composition-based binners have been run on
  a mock metagenomic dataset with known ground-truth genome labels and you must quantify
  binning accuracy via clustering metrics and vMAG completeness counts.
when_to_use_negative:
- Real datasets with no ground-truth genome labels — the clustering metrics require
  known labels, which the paper notes cannot be obtained from real samples
- Datasets with very few putative viral genomes, where the paper states benchmarking
  effectiveness is less convincing
- Evaluating shotgun-only binners lacking Hi-C co-occurrence profiles from multiple
  samples, which the paper notes impairs CoCoNet and vRhyme
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ViralCC
  role: Hi-C-based viral binner under evaluation; produces cluster_viral_contig.txt
    and VIRAL_BIN/*.fa draft viral bins
  repo: https://github.com/dyxstat/ViralCC
- name: VAMB
  role: Comparator binning method run on the same mock human gut inputs
- name: CoCoNet
  role: Comparator viral binning tool using composition and co-occurrence features
- name: vRhyme
  role: Comparator viral binning tool dedicated to metagenomic viral binning
- name: bin3C
  role: Comparator open-source Hi-C-based binning tool
- name: MetaTOR
  role: Comparator open-source Hi-C-based binning tool
- name: CheckV
  role: Estimates completeness of recovered vMAGs by comparison to a curated database
    (v0.7.0)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:6b47ee9fe54274d834a07e5a68411873f0e9e18ab088d2d5c98cfeb177904bb4
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# viral-bin-clustering-metric-evaluation

## Summary

Benchmarks metagenomic viral binning tools (ViralCC vs VAMB, CoCoNet, vRhyme, bin3C, MetaTOR) on the mock human gut Hi-C dataset by matching predicted clusters against ground-truth genome labels of the 1010 mock viral contigs and computing F-score, ARI, NMI, and homogeneity, plus CheckV-based near-complete/high-quality vMAG counts.

## When to use

Use when multiple Hi-C or composition-based binners have been run on a mock metagenomic dataset with known ground-truth genome labels and you must quantify binning accuracy via clustering metrics and vMAG completeness counts.

## When NOT to use

- Real datasets with no ground-truth genome labels — the clustering metrics require known labels, which the paper notes cannot be obtained from real samples
- Datasets with very few putative viral genomes, where the paper states benchmarking effectiveness is less convincing
- Evaluating shotgun-only binners lacking Hi-C co-occurrence profiles from multiple samples, which the paper notes impairs CoCoNet and vRhyme

## Inputs

- Assembled contigs FASTA from the mock human gut Hi-C dataset (min-contig-len 1000 via megahit)
- Read-pair-sorted Hi-C alignment BAM (samtools sort by name, filtered with samtools view -F 0x904)
- Viral contig list restricting analysis to the 1010 mock viral contigs
- Ground-truth genome labels for the 1010 mock viral contigs
- Bin assignment files from each binner (e.g., cluster_viral_contig.txt and VIRAL_BIN/*.fa from ViralCC)

## Outputs

- Per-tool table of F-score, ARI, NMI, and homogeneity against ground-truth labels
- Per-tool counts of near-complete and high-quality vMAGs (CheckV completeness categories, with random binning control)
- Reproduction of Fig. 2 mock human gut benchmarking values for comparison with the paper

## How to apply

Restrict the analysis to the 1010 mock viral contigs of the mock human gut dataset and run ViralCC with default parameters (--min-len 1000, --min-mapq 30, --min-match 30) alongside VAMB, CoCoNet, vRhyme, bin3C, and MetaTOR on identical inputs (assembled contigs FASTA, name-sorted read-pair Hi-C BAM, viral contig list). Match each tool's clusters (e.g., ViralCC's cluster_viral_contig.txt assignments) against ground-truth genome labels and compute the four clustering metrics reported in Fig. 2 (F-score, ARI, NMI, homogeneity). Assess recovered vMAGs with CheckV (v0.7.0) completeness categories and count near-complete and high-quality vMAGs per tool, including a random binning control since random binning does not reduce CheckV completeness. Assemble a per-tool table and compare against the published Fig. 2 values, which report ViralCC at F-score 0.795, ARI 0.787, and 26 near-complete vMAGs versus 1 (VAMB), 5 (CoCoNet), 0 (vRhyme), 5 (bin3C), and 22 (MetaTOR).

## Related tools

- **ViralCC** (Hi-C-based viral binner under evaluation; produces cluster_viral_contig.txt and VIRAL_BIN/*.fa draft viral bins) — https://github.com/dyxstat/ViralCC
- **VAMB** (Comparator binning method run on the same mock human gut inputs)
- **CoCoNet** (Comparator viral binning tool using composition and co-occurrence features)
- **vRhyme** (Comparator viral binning tool dedicated to metagenomic viral binning)
- **bin3C** (Comparator open-source Hi-C-based binning tool)
- **MetaTOR** (Comparator open-source Hi-C-based binning tool)
- **CheckV** (Estimates completeness of recovered vMAGs by comparison to a curated database (v0.7.0))

## Examples

```
python ./viralcc.py pipeline --min-len 1000 --min-mapq 30 --min-match 30 FASTA_file BAM_file VIRAL_file OUTPUT_directory
```

## Evaluation signals

- Recomputed ViralCC F-score on the mock human gut dataset matches the published value of 0.795 within numerical tolerance
- Recomputed ViralCC ARI matches the published value of 0.787 and preserves the performance ordering over the five comparators
- Near-complete vMAG counts reproduce the Fig. 2 ordering: ViralCC 26 > MetaTOR 22 > CoCoNet 5 = bin3C 5 > VAMB 1 > vRhyme 0
- A random binning control is included whenever CheckV completeness is used as the evaluation metric
- All tools are scored on the same restricted set of 1010 mock viral contigs with identical input files

## Limitations

- Random binning does not reduce CheckV completeness, so a random binning control model is necessary when CheckV completeness is the evaluation metric
- The benchmarking strategy underestimates true viral community diversity because only single-contig-recoverable viral genomes were chosen
- Putative viral genomes in the benchmarking tend to be small, and this size bias should be accounted for

## Evidence

- [discussion] CheckV completeness requires random binning control: "it is necessary to construct a random binning model as control experiments when the CheckV completeness is used as the evaluation metric"
- [intro] CheckV estimates vMAG completeness: "CheckV has been widely used to estimate the completeness of vMAGs by comparing them to a large database curated from NCBI GenBank and environmental samples43"
- [abstract] ViralCC is benchmarked against existing Hi-C and viral binning tools: "we demonstrate that ViralCC outperforms existing Hi-C-based binning methods as well as state-of-the-art tools specifically dedicated to metagenomic viral binning."
- [readme] Source data for Fig. 2 is publicly available: [evidence span withheld — anchor status: external_doc]
- [discussion] Benchmarking validity caveats: "The effectiveness of the benchmarking is also less convincing if there are few putative viral genomes."
