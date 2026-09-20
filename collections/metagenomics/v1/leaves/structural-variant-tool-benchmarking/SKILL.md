---
name: structural-variant-tool-benchmarking
description: Use when evaluating or reproducing comparisons of HGT breakpoint detection
  or local strain reconstruction methods on simulated shotgun metagenomic reads, where
  ground-truth breakpoints and transferred segments are known.
when_to_use_negative:
- Input data are real metagenomic samples without known ground-truth breakpoints —
  sensitivity/FDR against true positions cannot be computed
- The goal is de novo HGT discovery rather than head-to-head tool comparison against
  reported benchmark figures
- Data are long-read assemblies or non-shotgun data; the LEMON/LUMPY pipeline assumes
  short paired-end shotgun reads aligned with BWA
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: LEMON
  role: 'Graph-based method under test: detects raw and accurate HGT breakpoints (DBSCAN
    clustering, split-read refinement) and reconstructs the local strain'
  repo: https://github.com/lichen2018/LEMON
- name: LUMPY
  role: Baseline structural-variant caller run with default parameters on the same
    BAM inputs for comparison
- name: WGSIM
  role: Simulates paired-end reads from true local strains at defined coverage levels
    (2X, 5X, 30X, 70X)
- name: BWA
  role: Aligns simulated reads against the metagenomic reference to produce the BAM
    used by both tools
- name: Smith-Waterman algorithm
  role: Scores similarity between reconstructed and true strain segment sequences
    (match_score=1, mismatch_score=-1) to compute reconstruction accuracy
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:fa8462cc8f26bcc223a55fbd9cfcbb04401a3ec6d5c73b5faa7dfcfa0139b77a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - LEMON
  - LUMPY
  - Smith-Waterman algorithm
  - WGSIM
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# structural-variant-tool-benchmarking

## Summary

A benchmarking skill for comparing structural-variant / HGT-breakpoint detection tools (LEMON vs LUMPY) on simulated gut metagenomics data, measuring breakpoint sensitivity/FDR with a 20 bp tolerance and local-strain reconstruction accuracy (RA) via Smith-Waterman alignment across coverage levels.

## When to use

Use when evaluating or reproducing comparisons of HGT breakpoint detection or local strain reconstruction methods on simulated shotgun metagenomic reads, where ground-truth breakpoints and transferred segments are known.

## When NOT to use

- Input data are real metagenomic samples without known ground-truth breakpoints — sensitivity/FDR against true positions cannot be computed
- The goal is de novo HGT discovery rather than head-to-head tool comparison against reported benchmark figures
- Data are long-read assemblies or non-shotgun data; the LEMON/LUMPY pipeline assumes short paired-end shotgun reads aligned with BWA

## Inputs

- Paired-end simulated shotgun reads (FASTQ) from known local strains at defined coverages (2X, 5X, 30X, 70X) generated with WGSIM
- Metagenomic reference set (e.g., 16,093 species, 1,246,881 scaffolds from NCBI bacterial genomes)
- Unique-read BAM (samtools view -q 20), split-read BAM (extractSplitReads_BwaMem), and coverage file
- Ground-truth HGT breakpoint positions and true transferred segments per simulated strain

## Outputs

- sensitivity_fdr_table.csv: sensitivity and FDR per tool per coverage level
- reconstruction_accuracy_comparison.csv: RA values per tool per coverage level (e.g., LEMON 0.6087/5X, 1.0/30X; LUMPY 0.5238/5X, 0.9565/30X)
- Raw and accurate breakpoint call files and reconstructed local strain sequences per tool

## How to apply

Simulate ground-truth local strains by inserting 10k bp transferred segments into reference genomes (e.g., 60 local strains, 4260 HGT breakpoints from 100 randomly selected microbials), then generate paired-end reads with WGSIM at multiple coverages (e.g., 2X, 5X, 30X, 70X). Align reads with BWA and extract unique reads (samtools view -q 20) and split reads (lumpy-sv extractSplitReads_BwaMem) as required by the LEMON preprocessing workflow. Run each tool on identical BAM inputs — LEMON via get_raw_bkp.py, get_acc_bkp (DBSCAN with eps equal to average insert size, minsample=1), and reconstruct_HGT_strain.py; LUMPY with default parameters. Score breakpoint detections with a 20 bp tolerance, computing sensitivity (true detected / all true breakpoints) and FDR (false discovered / all discovered); score reconstructed strains with Smith-Waterman (match_score=1, mismatch_score=-1) counting transferred segments as matched when both breakpoint positions fall within 20 bp, RA = SW(hi,Hi)/(mi*match_score). Compile results into comparison tables (e.g., sensitivity_fdr_table.csv, reconstruction_accuracy_comparison.csv) and check them against reported figures.

## Related tools

- **LEMON** (Graph-based method under test: detects raw and accurate HGT breakpoints (DBSCAN clustering, split-read refinement) and reconstructs the local strain) — https://github.com/lichen2018/LEMON
- **LUMPY** (Baseline structural-variant caller run with default parameters on the same BAM inputs for comparison)
- **WGSIM** (Simulates paired-end reads from true local strains at defined coverage levels (2X, 5X, 30X, 70X))
- **BWA** (Aligns simulated reads against the metagenomic reference to produce the BAM used by both tools)
- **Smith-Waterman algorithm** (Scores similarity between reconstructed and true strain segment sequences (match_score=1, mismatch_score=-1) to compute reconstruction accuracy)

## Examples

```
python LEMON/Scripts/get_raw_bkp.py -r metagenomic_reference.fa -u unique_reads.bam -o raw_bkp.txt -t 4
```

## Evaluation signals

- LEMON sensitivity/FDR should exceed LUMPY at every coverage level; reproduce reported values (LEMON 22.39%/51.19% sensitivity at 2X/5X, 0.002 FDR at 2X, 0.0034 FDR at 70X; LUMPY 6.86%/31.12% sensitivity, 0.016/0.010 FDR)
- Reconstruction accuracy should match reported figures: LEMON RA of 0.6087 at 5X and 1.0 at 30X; LUMPY RA of 0.5238 at 5X and 0.9565 at 30X
- A detected breakpoint counts as true only when within 20 bp of the true position; verify the 20 bp tolerance is applied consistently for both tools
- RA must equal SW(hi,Hi)/(mi*match_score); check RA stays in [0,1] and equals 1.0 only when all transferred segments match within 20 bp on both breakpoints
- Both tools must be run on identical unique-read BAM, split-read BAM, and coverage file inputs to make the comparison fair

## Limitations

- Benchmark relies on simulated strains with simple inserted segments (e.g., two HGT events, five transferred segments each); real strains may harbor more complicated structure variation
- Third-party tools are run with default parameters (LUMPY, BWA), so results may not reflect tuned per-tool optima
- Performance degrades at very low coverage (LEMON only 22.39% sensitivity at 2X); coverage level strongly affects sensitivity

## Evidence

- [abstract] In simulations, the graph-based method implemented in LEMON identifies transferred segments accurately on reference sequences of the microbiome.: "The simulated results indicate that the method can identify transferred segments accurately on reference sequences of the microbiome."
- [methods] Simulation setup of 60 local strains with 4260 HGT breakpoints and scoring with 20 bp tolerance via Smith-Waterman.: "We apply Smith-Waterman algorithm to measure the similarity between segment sequences of Hi and hi."
- [methods] LUMPY and other third-party tools run with default parameters for the benchmark.: "Most third-party tools used in this article are set default parameters, including BWA, LUMPY [25], iRep [26], and STAR-Fusion [27]."
- [methods] DBSCAN clustering parameters for accurate breakpoint detection used in the benchmark pipeline.: "We then apply the clustering algorithm DBSCAN [23] using Euclidean distance to cluster the breakpoint pair candidates."
- [readme] LEMON pipeline commands (get_raw_bkp.py, get_acc_bkp, reconstruct_HGT_strain.py) and required BAM inputs from the README.: "usage: python LEMON/Scripts/reconstruct_HGT_strain.py [options]"
