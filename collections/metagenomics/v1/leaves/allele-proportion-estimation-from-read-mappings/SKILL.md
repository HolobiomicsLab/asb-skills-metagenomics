---
name: allele-proportion-estimation-from-read-mappings
description: Use when processing WGS tick samples (2x76bp read pairs) against the
  1726-allele B. burgdorferi MLST database and you need, per locus, the set of alleles
  and their fractional composition under within-host strain heterogeneity.
when_to_use_negative:
- You need to jointly infer strain types and novel strains across multiple samples
  — that is the separate Stage-2 Strain Diversity Problem (globalStrainDiversity.py),
  not per-locus allele proportioning
- The organism has no curated reference allele database (e.g. no MLST scheme) — de
  novo methods such as DESMAN are the alternative the paper positions against
- Samples lack within-host heterogeneity and a single reference strain suffices for
  typing
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: Bowtie
  role: Maps sample reads onto the database of known MLST alleles prior to proportion
    estimation
  repo: https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- name: samtools
  role: Extracts per-read mapping information from SAM files for the ILP input files
  repo: https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- name: CPLEX
  role: Solves the 0-1 ILP formulation of the Allele Diversity Problem that determines
    which alleles are present before proportions are computed
- name: GAML
  role: Likelihood-based selection among multiple optimal ILP solutions to break ties
- name: ART
  role: Simulates reads (matching real tick-sample characteristics) for validating
    allele-composition recovery on simulated samples
  repo: https://www.niehs.nih.gov/research/resources/software/biostatistics/art/
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:93283715b5df5e140552c64def7b073c76f932dd753f03d253745e8e173902a3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ART
  - Bowtie
  - Bowtie v0.12.7
  - CPLEX
  - GAML
  - Python 2.7
  - fastq-dump
  - samtools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# allele-proportion-estimation-from-read-mappings

## Summary

A stage-1 MLST deconvolution step that, for each of the 8 B. burgdorferi housekeeping loci (clpA, clpX, nifS, pepX, pyrG, recG, rplB, uvrA), selects the allele set present in a sample via a 0-1 ILP (Allele Diversity Problem) and computes normalized per-allele proportions from Bowtie/samtools read-mapping information.

## When to use

Use when processing WGS tick samples (2x76bp read pairs) against the 1726-allele B. burgdorferi MLST database and you need, per locus, the set of alleles and their fractional composition under within-host strain heterogeneity.

## When NOT to use

- You need to jointly infer strain types and novel strains across multiple samples — that is the separate Stage-2 Strain Diversity Problem (globalStrainDiversity.py), not per-locus allele proportioning
- The organism has no curated reference allele database (e.g. no MLST scheme) — de novo methods such as DESMAN are the alternative the paper positions against
- Samples lack within-host heterogeneity and a single reference strain suffices for typing

## Inputs

- FASTQ read pairs (2x76bp) for the 24 tick samples downloaded with fastq-dump v2.8.2
- B. burgdorferi MLST allele database (8 housekeeping genes, 1726 known alleles)
- Per-read mapping information extracted from SAM files (sampleX_geneY_paired_reads.txt in variantsAndProp/)

## Outputs

- geneX_proportions.csv — one per locus (8 per sample), listing selected alleles and their normalized proportions
- Per-locus allele composition usable as input to the downstream Strain Diversity Problem (globalStrainDiversity.py)

## How to apply

Map sample reads to the allele database with Bowtie v0.12.7 and extract per-read mapping weights with samtools v1.3.1 (mapSamples.py produces sampleX_geneY_paired_reads.txt). For each sample and locus, solve the Allele Diversity Problem as a 0-1 ILP (objective: number of selected alleles plus summed normalized Phred mismatch weights) using the CPLEX 12.6.3.0 Python API via alleleDiversity.py; when multiple optima exist, select among them with GAML. Once alleles are identified, compute each allele's proportion by weighting the ILP-selected read mappings and normalizing to sum to 1, after filtering reads that map equally well to all selected alleles (these carry no discriminative information). Write one geneX_proportions.csv per locus per sample into the sample's folder under variantsAndProp.

## Related tools

- **Bowtie** (Maps sample reads onto the database of known MLST alleles prior to proportion estimation) — https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- **samtools** (Extracts per-read mapping information from SAM files for the ILP input files) — https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- **CPLEX** (Solves the 0-1 ILP formulation of the Allele Diversity Problem that determines which alleles are present before proportions are computed)
- **GAML** (Likelihood-based selection among multiple optimal ILP solutions to break ties)
- **ART** (Simulates reads (matching real tick-sample characteristics) for validating allele-composition recovery on simulated samples) — https://www.niehs.nih.gov/research/resources/software/biostatistics/art/

## Examples

```
python alleleDiversity.py -s SRR2034333
```

## Evaluation signals

- On simulated samples (run_sim.py, 40 iterations per c in {30,100,300} x d in {5,10,15,20,25}), per-locus precision should average ~0.98 ± 0.011 and recall ~0.96 ± 0.026
- Total Variation Distance of predicted vs. true per-gene allele distributions should fall in the ~0.07–0.12 range
- Each run of alleleDiversity.py produces exactly 8 geneX_proportions.csv files (one per locus) in the sample's variantsAndProp folder
- Allele proportions must be normalized and must come only from reads mapping differentially to the ILP-selected allele set (reads mapping equally well to all selected alleles are filtered out)
- When the 0-1 ILP has multiple optima, a GAML-selected solution must be reported rather than an arbitrary solver optimum

## Limitations

- The paper's approach to closely related strain clusters relies on ad-hoc post-processing; more principled methods remain unexplored
- Only the Python 2.7-era pipeline is documented (Bowtie 0.12.7, CPLEX 12.6.3.0, samtools 1.3.1, fastq-dump 2.8.2); compatibility with modern toolchains is not covered
- Performance degrades at low coverage — c=30 coverage is the lowest simulated condition, and accuracy is characterized across edit-distance d up to 25

## Evidence

- [methods] Computing allele proportions: "Computing allele proportions.Finally, once the alleles have been identified for a given locus, we compute the proportion of each allele."
- [methods] ILP formulation solved with CPLEX: "Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0."
- [methods] GAML tie-breaking: "If the above 0-1 ILP has multiple optimal solutions, we resort to a likelihood based method to select one, namely GAML [17]"
- [readme] Proportion output files: "named as `geneX_proportions.csv` which contains the alleles and their proportions identified at `geneX`."
- [readme] Mapping inputs: "The script will create a `variantsAndProp` folder containing a folder for each sample"
