---
name: allele-diversity-integer-linear-programming
description: Use when WGS reads have been mapped to a curated MLST allele database
  and you must infer, per locus, which alleles are present and their fractional proportions
  in mixed (within-host heterogeneous) samples.
when_to_use_negative:
- No curated reference allele database exists for your species — the ADP is reference-based
  (unlike de novo methods such as DESMAN) and requires known MLST alleles
- The goal is direct strain-type calling from perfect allele inputs — that is the
  second-stage Strain Diversity Problem MILP, not this allele-assignment ILP
- Reads are unmapped or mapping information has not been extracted from SAM files
  — the ILP consumes the sampleX_geneY_paired_reads.txt files, not raw FASTQ
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CPLEX 12.6.3.0
  role: Formulates and solves the 0-1 ILP for the Allele Diversity Problem via its
    Python API
- name: GAML
  role: Likelihood-based post-processing to select among multiple optimal ILP solutions
- name: Bowtie v0.12.7
  role: Maps sample reads onto the known allele database to produce the SAM files
    feeding the ILP
  repo: https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- name: samtools v1.3.1
  role: Extracts per-read mapping information from SAM files into sampleX_geneY_paired_reads.txt
  repo: https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- name: Python 2.7
  role: All pipeline scripts (alleleDiversity.py, mapSamples.py) are written in Python
    2.7
  repo: https://github.com/WGS-TB/MLST
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:e4b6b88e10401a5110baed06f103661e9cdead0ec08930dc84d736f61fd6bbd1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ART
  - Bowtie v0.12.7
  - CPLEX 12.6.3.0
  - GAML
  - PyEMD
  - Python 2.7
  - editdistance
  - fastq-dump
  - samtools v1.3.1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# allele-diversity-integer-linear-programming

## Summary

Solves the Allele Diversity Problem (ADP) from the MLST strain-deconvolution pipeline: given per-locus read-to-allele mapping information, a 0-1 ILP selects the smallest set of known database alleles (with normalized Phred mismatch weights in the objective) that explains each sample's reads, then computes fractional allele proportions per locus.

## When to use

Use when WGS reads have been mapped to a curated MLST allele database and you must infer, per locus, which alleles are present and their fractional proportions in mixed (within-host heterogeneous) samples.

## When NOT to use

- No curated reference allele database exists for your species — the ADP is reference-based (unlike de novo methods such as DESMAN) and requires known MLST alleles
- The goal is direct strain-type calling from perfect allele inputs — that is the second-stage Strain Diversity Problem MILP, not this allele-assignment ILP
- Reads are unmapped or mapping information has not been extracted from SAM files — the ILP consumes the sampleX_geneY_paired_reads.txt files, not raw FASTQ

## Inputs

- Per-sample, per-gene read-to-allele mapping files (sampleX_geneY_paired_reads.txt in the variantsAndProp folder) extracted from Bowtie v0.12.7 SAM output via samtools v1.3.1
- B. burgdorferi MLST allele database (8 housekeeping genes: clpA, clpX, nifS, pepX, pyrG, recG, rplB, uvrA; 1726 known alleles)
- Paired-end FASTQ reads (2x76 bp) from the 24 tick samples downloaded via fastq-dump v2.8.2

## Outputs

- Per-locus allele sets with normalized allele proportions, written as 8 files geneX_proportions.csv per sample in variantsAndProp
- Downstream input for the Strain Diversity Problem MILP (per-sample allele distributions)

## How to apply

Build the weighted read-allele incidence matrix from the per-sample/per-gene mapping files (sampleX_geneY_paired_reads.txt), where each entry is the sum of Phred mismatch scores normalized over the 33-126 range (non-mapping reads set to infinity). Formulate the 0-1 ILP with binary allele-selection variables x_j and read-assignment variables y_ik, minimizing sum(q_ik * y_ik) + sum(x_j) subject to cover and uniqueness constraints, and solve with the CPLEX 12.6.3.0 Python API. When multiple optimal solutions exist, break ties with the GAML likelihood-based post-processing step. Compute allele proportions by discarding reads mapping equally well to all selected alleles, adding weight 1/h to each of the h alleles attaining the optimal score for each retained read, and normalizing weights to sum to 1. Expect roughly 20,112 ILP variables per gene and ~1.5 GB RAM for a large sample (e.g. SRR2034336, ~33 min for all 8 genes on 4 CPUs), so allocate resources accordingly.

## Related tools

- **CPLEX 12.6.3.0** (Formulates and solves the 0-1 ILP for the Allele Diversity Problem via its Python API)
- **GAML** (Likelihood-based post-processing to select among multiple optimal ILP solutions)
- **Bowtie v0.12.7** (Maps sample reads onto the known allele database to produce the SAM files feeding the ILP) — https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- **samtools v1.3.1** (Extracts per-read mapping information from SAM files into sampleX_geneY_paired_reads.txt) — https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- **Python 2.7** (All pipeline scripts (alleleDiversity.py, mapSamples.py) are written in Python 2.7) — https://github.com/WGS-TB/MLST

## Examples

```
python alleleDiversity.py -s SRR2034336
```

## Evaluation signals

- alleleDiversity.py runs to completion and writes exactly 8 geneX_proportions.csv files (one per locus) into each sample's variantsAndProp folder
- Per-locus allele proportions are normalized to sum to 1 after filtering reads that map equally well to all selected alleles
- ILP size and resources stay feasible: paper reports ~20,112 variables per gene on average and ~1.5 GB peak RAM for sample SRR2034336
- On simulated ADP benchmarks, per-locus B-MLST statistics match Table 1: average precision ~0.98 ± 0.011, average recall ~0.96 ± 0.026, TVD ~0.07-0.12
- Output allele distributions, when chained into the SDP stage, should yield strong ADP-stage precision ~0.92-0.96 and recall ~0.90-0.92 per EvoMod simulation

## Limitations

- Small ADP-stage errors propagate downstream: in full-pipeline EvoMod simulations, exact SDP precision/recall dropped from ~0.8 with perfect allele input to ~0.3-0.5 with ADP-inferred input, and TVD rose to ~0.6-0.8
- Multiple optimal ILP solutions require an external likelihood-based tie-breaker (GAML), an extra dependency and decision point
- Runtime scales with sample size: ~33 min for all 8 genes of one large sample on a 4-CPU machine, requiring CPLEX academic license and Python 2.7

## Evidence

- [methods] Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0.: "Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0."
- [methods] GAML post-processing for multiple optima: "If the above 0-1 ILP has multiple optimal solutions, we resort to a likelihood based method to select one, namely GAML [17]"
- [methods] computing allele proportions: "once the alleles have been identified for a given locus, we compute the proportion of each allele."
- [readme] README invocation of alleleDiversity.py: "This script will output some intermediate results to the screen and create 8 CSV files (for each locus) in each of the sample folder in `variantsAndProp`"
- [readme] README prerequisites for ILP solving: "CPLEX Python API: For solving ILP. You will need academic license to download CPLEX."
