---
name: local-strain-reconstruction
description: Use when paired-end shotgun metagenomic reads must be mapped to a microbiome
  reference to detect HGT breakpoints, identify transferred genome segments, and reconstruct
  the local strain harboring the HGT events.
when_to_use_negative:
- Input is long-read sequencing data or transcriptomic data rather than paired-end
  shotgun NGS reads, since LEMON's breakpoint detection relies on unique and split
  shotgun read alignment via BWA
- The goal is ancient/lineage HGT inference from genomic sequences rather than reconstructing
  recent HGT events from metagenomic samples
- No metagenomic reference sequences are available against which reads can be aligned
  for donor/receptor identification
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: LEMON
  role: Core package implementing breakpoint detection (get_raw_bkp.py, get_acc_bkp)
    and graph-based local strain reconstruction (reconstruct_HGT_strain.py)
  repo: https://github.com/lichen2018/LEMON
- name: BWA
  role: Aligns paired-end shotgun reads against the metagenomic reference to identify
    donors, receptors, and junction/split reads
- name: LUMPY
  role: Third-party structural variant caller run with default parameters on the same
    BAM inputs as a comparison baseline for reconstruction accuracy
- name: WGSIM
  role: Simulates paired-end reads from true local strains at defined coverage (e.g.
    5X, 30X) for benchmarking
- name: Smith-Waterman algorithm
  role: Measures similarity between reconstructed and true strain segment sequences
    to compute reconstruction accuracy (RA)
- name: DBSCAN
  role: Clusters breakpoint pair candidates using Euclidean distance during raw breakpoint
    detection
- name: HTSlib
  role: Dependency (version 1.9) used by LEMON's getAccBkp C++ module for BAM file
    access
  repo: https://github.com/samtools/htslib
- name: Gurobipy
  role: Optimization library used in LEMON's graph balancing / Eulerian circuit workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:aaa3d1b30150c07b6bb89873952f0bf6ccf27ec0e5d7fefaf251325d19f1bb6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - DBSCAN
  - Gurobipy
  - HTSlib
  - LEMON
  - LUMPY
  - Smith-Waterman algorithm
  - WGSIM
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# local-strain-reconstruction

## Summary

Reconstructs the inserted local strain at horizontal gene transfer (HGT) sites from gut metagenomics shotgun data using the graph-based LEMON method, which detects raw and accurate HGT breakpoints and assembles linked transferred segments into an Eulerian circuit.

## When to use

Use when paired-end shotgun metagenomic reads must be mapped to a microbiome reference to detect HGT breakpoints, identify transferred genome segments, and reconstruct the local strain harboring the HGT events.

## When NOT to use

- Input is long-read sequencing data or transcriptomic data rather than paired-end shotgun NGS reads, since LEMON's breakpoint detection relies on unique and split shotgun read alignment via BWA
- The goal is ancient/lineage HGT inference from genomic sequences rather than reconstructing recent HGT events from metagenomic samples
- No metagenomic reference sequences are available against which reads can be aligned for donor/receptor identification

## Inputs

- Paired-end shotgun metagenomic reads (FASTQ, e.g. simulated with WGSIM at 5X–30X coverage)
- Metagenomic reference sequences (e.g. NCBI bacterial genome reference set)
- Unique-read BAM file
- Split-read BAM file
- Coverage file
- Raw breakpoints file (intermediate)

## Outputs

- Raw HGT breakpoints file (get_raw_bkp.py)
- Accurate HGT breakpoints file (get_acc_bkp, 20 bp tolerance)
- Reconstructed local strain sequence at HGT sites
- Optional replication-time-annotated strain (reconstruct_HGT_strain_for_replication_time.py)
- Reconstruction accuracy values (RA) vs. LUMPY benchmark

## How to apply

Map paired-end shotgun reads against the metagenomic reference with BWA, then produce the unique-read BAM, split-read BAM, and coverage file required by the LEMON preprocessing workflow. Run get_raw_bkp.py on the unique-read BAM to detect raw HGT breakpoints, cluster candidates with DBSCAN (Euclidean distance), then run get_acc_bkp using split reads to refine exact breakpoint positions with a 20 bp bearing tolerance. Run reconstruct_HGT_strain.py, which builds a connected graph of linked segments, inserts dummy edges to balance it, and finds an Eulerian circuit to assemble the local strain sequence. Benchmark against LUMPY on the same BAM inputs by scoring each reconstructed strain against the true strain with the Smith-Waterman algorithm (match_score = 1, mismatch_score = -1) and computing RA = SW(hi,Hi)/(mi*match_score), counting segments as matched when both breakpoints fall within 20 bp of truth.

## Related tools

- **LEMON** (Core package implementing breakpoint detection (get_raw_bkp.py, get_acc_bkp) and graph-based local strain reconstruction (reconstruct_HGT_strain.py)) — https://github.com/lichen2018/LEMON
- **BWA** (Aligns paired-end shotgun reads against the metagenomic reference to identify donors, receptors, and junction/split reads)
- **LUMPY** (Third-party structural variant caller run with default parameters on the same BAM inputs as a comparison baseline for reconstruction accuracy)
- **WGSIM** (Simulates paired-end reads from true local strains at defined coverage (e.g. 5X, 30X) for benchmarking)
- **Smith-Waterman algorithm** (Measures similarity between reconstructed and true strain segment sequences to compute reconstruction accuracy (RA))
- **DBSCAN** (Clusters breakpoint pair candidates using Euclidean distance during raw breakpoint detection)
- **HTSlib** (Dependency (version 1.9) used by LEMON's getAccBkp C++ module for BAM file access) — https://github.com/samtools/htslib
- **Gurobipy** (Optimization library used in LEMON's graph balancing / Eulerian circuit workflow)

## Examples

```
python LEMON/Scripts/get_raw_bkp.py -r reference.fa -u unique.bam -o raw_breakpoints.txt
```

## Evaluation signals

- Reproduced reconstruction accuracy should match reported figures: LEMON RA = 0.6087 at 5X and 1.0 at 30X; LUMPY RA = 0.5238 at 5X and 0.9565 at 30X
- Each transferred segment counts as matched only when both breakpoint positions fall within 20 bp of the true breakpoints
- RA is computed as RA = SW(hi,Hi)/(mi*match_score) with Smith-Waterman match_score = 1 and mismatch_score = -1
- Raw breakpoint output from get_raw_bkp.py should be produced before get_acc_bkp refinement, and the accurate-breakpoint file must feed reconstruct_HGT_strain.py
- In real-data validation, detected gene fusion points should lie near HGT breakpoints and reconstructed strains should show replication time profiles with lower standard error

## Limitations

- Reconstruction accuracy degrades at low sequencing coverage: LEMON achieved RA 0.6087 at 5X versus 1.0 at 30X in simulations
- HGT inference from genomic sequences is affected by complex external factors such as gene loss, mutation, duplication, and structural variations at insertion sites
- Requires a curated reference set (16,093 species, 1,246,881 scaffolds) and a Python 3.5+ environment with Gurobipy, plus pinned HTSlib 1.9 for compilation

## Evidence

- [intro] LEMON takes use of existing shotgun NGS datasets to detect HGT breakpoints, identify the transferred segments, and reconstruct the local strain: "LEMON(https://github.com/lichen2018/LEMON) takes use of existing shotgun NGS datasets to detect HGT breakpoints, identify the transferred segments, and reconstruct the local strain"
- [methods] Map reads with BWA: "Firstly, we map paired-end shotgun reads against reference genomes using Burrows Wheeler Aligner (BWA)."
- [methods] DBSCAN clustering of breakpoint candidates: "We then apply the clustering algorithm DBSCAN [23] using Euclidean distance to cluster the breakpoint pair candidates."
- [methods] Split reads for exact breakpoints: "Fourthly, we utilize split reads to get the exact positions of HGT breakpoints."
- [methods] Smith-Waterman accuracy scoring: "We apply Smith-Waterman algorithm to measure the similarity between segment sequences of Hi and hi."
- [readme] get_raw_bkp.py usage: "usage: python LEMON/Scripts/get_raw_bkp.py [options]"
- [readme] reconstruct_HGT_strain.py usage: "usage: python LEMON/Scripts/reconstruct_HGT_strain.py [options]"
