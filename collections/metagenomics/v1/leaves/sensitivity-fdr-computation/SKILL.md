---
name: sensitivity-fdr-computation
description: Use when evaluating breakpoint or structural-variant detection accuracy
  against known ground-truth positions from simulations, where sensitivity and FDR
  must be computed at multiple coverage levels (e.g. 2X, 5X, 70X).
when_to_use_negative:
- Input is real (non-simulated) metagenomic data with no ground-truth HGT breakpoint
  positions, so true/false classifications cannot be made
- The goal is strain reconstruction quality assessment rather than breakpoint detection
  accuracy benchmarking
- Evaluating tools whose output is gene-level HGT inference rather than nucleotide-resolution
  breakpoint coordinates
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: LEMON
  role: Graph-based HGT breakpoint detector under evaluation (get_raw_bkp.py, get_acc_bkp
    with DBSCAN)
  repo: https://github.com/lichen2018/hgt-detection
- name: LUMPY
  role: Structural-variant caller used as the comparison baseline at default parameters
- name: WGSIM
  role: Simulates paired-end reads from true local strains at 2X, 5X, and 70X coverage
- name: BWA
  role: Aligns simulated reads against the reference set before breakpoint calling
- name: samtools
  role: Extracts unique reads with view -q 20 as LEMON input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:a1179df7d8781eaa454f79f3217c629a08b33276eb46ae2972a0658981b283d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/hgt-detection
  tools:
  - BWA
  - LEMON
  - LUMPY
  - WGSIM
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# sensitivity-fdr-computation

## Summary

Benchmark protocol for scoring HGT breakpoint detection tools (LEMON vs LUMPY) by sensitivity and false discovery rate across sequencing coverages, using a 20 bp positional tolerance to classify true detections in simulated gut metagenomics data.

## When to use

Use when evaluating breakpoint or structural-variant detection accuracy against known ground-truth positions from simulations, where sensitivity and FDR must be computed at multiple coverage levels (e.g. 2X, 5X, 70X).

## When NOT to use

- Input is real (non-simulated) metagenomic data with no ground-truth HGT breakpoint positions, so true/false classifications cannot be made
- The goal is strain reconstruction quality assessment rather than breakpoint detection accuracy benchmarking
- Evaluating tools whose output is gene-level HGT inference rather than nucleotide-resolution breakpoint coordinates

## Inputs

- Reference metagenomic genome set (e.g. 100 bacterial genomes from NCBI; reference set of 16,093 species / 1,246,881 scaffolds)
- Simulated local strains with known 10k bp transferred segments and true HGT breakpoint coordinates
- Paired-end shotgun reads simulated with WGSIM at 2X, 5X, and 70X coverage
- BAM files of unique reads (samtools view -q 20) and split reads (lumpy-sv extractSplitReads_BwaMem)
- Breakpoint prediction files from LEMON (get_acc_bkp output) and LUMPY

## Outputs

- sensitivity_fdr_table.csv with sensitivity and FDR per tool per coverage level
- Comparison against reported values: LEMON 22.39%/51.19% sensitivity at 2X/5X, 0.002 FDR at 2X, 0.0034 FDR at 70X; LUMPY 6.86%/31.12% sensitivity, 0.016/0.010 FDR

## How to apply

Build simulated ground truth by inserting 10k bp transferred segments between randomly chosen donor and receptor reference genomes (e.g. 60 local strains with 4260 HGT breakpoints from 100 microbial genomes), then generate paired-end reads with WGSIM at 2X, 5X, and 70X coverage. Align reads with BWA, extract unique reads (samtools view -q 20) and split reads (lumpy-sv extractSplitReads_BwaMem), and run both LEMON (get_raw_bkp.py followed by get_acc_bkp with DBSCAN eps set to the average insert size and minsample 1) and LUMPY (default parameters) on each coverage level. Score each detection with a 20 bp tolerance: a detected breakpoint within 20 bp of a true position counts as a true positive. Compute sensitivity as true detected / all true breakpoints and FDR as false discovered / all discovered for each tool at each coverage, then compile into sensitivity_fdr_table.csv.

## Related tools

- **LEMON** (Graph-based HGT breakpoint detector under evaluation (get_raw_bkp.py, get_acc_bkp with DBSCAN)) — https://github.com/lichen2018/hgt-detection
- **LUMPY** (Structural-variant caller used as the comparison baseline at default parameters)
- **WGSIM** (Simulates paired-end reads from true local strains at 2X, 5X, and 70X coverage)
- **BWA** (Aligns simulated reads against the reference set before breakpoint calling)
- **samtools** (Extracts unique reads with view -q 20 as LEMON input)

## Examples

```
samtools view -q 20 aligned.bam > unique.bam; python LEMON/Scripts/get_raw_bkp.py -r ref.fa -u unique.bam -o raw_bkp.txt; LUMPY default run per coverage; score detections within 20 bp of true breakpoints and write sensitivity_fdr_table.csv
```

## Evaluation signals

- LEMON sensitivity at each coverage should fall near reported values (22.39% at 2X, 51.19% at 5X) and exceed LUMPY (6.86% and 31.12%)
- LEMON FDR should be lower than LUMPY at all coverages (LEMON ~0.002 at 2X and ~0.0034 at 70X vs LUMPY 0.016 and 0.010)
- A detection is only counted as true positive if it lies within 20 bp of a ground-truth breakpoint position
- Both tools must be scored on the same simulated data (60 local strains, 4260 true HGT breakpoints) at identical 2X/5X/70X coverage levels

## Limitations

- Sensitivity is low at very low coverage (2X: LEMON ~22%, LUMPY ~7%), so benchmark conclusions at low depth are noisy
- Results depend on simulation parameters (10k bp inserts, insert size used as DBSCAN eps); other simulation regimes may not reproduce reported figures
- Applies to breakpoint detection only — it does not evaluate the quality of reconstructed local strains

## Evidence

- [methods] All true local strains were used to generated 20X paired-end reads with WGSIM [28] as an input of LEMON.: "All true local strains were used to generated 20X paired-end reads with WGSIM [28] as an input of LEMON."
- [methods] we utilize BWA to align reads against the references: "we utilize BWA[20] to align reads against the references to identify the set of donors and receptors"
- [methods] DBSCAN clustering of breakpoint pair candidates: "We then apply the clustering algorithm DBSCAN [23] using Euclidean distance to cluster the breakpoint pair candidates."
- [methods] Most third-party tools used in this article are set default parameters, including BWA, LUMPY: "Most third-party tools used in this article are set default parameters, including BWA, LUMPY [25], iRep [26], and STAR-Fusion [27]."
- [abstract] The simulated results indicate that the method can identify transferred segments accurately on reference sequences of the microbiome.: "The simulated results indicate that the method can identify transferred segments accurately on reference sequences of the microbiome."
