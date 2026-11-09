---
name: mlst-allele-panel-simulation
description: Use when benchmarking whether a metagenomic allele/strain deconvolution
  pipeline recovers fractional allele composition per MLST locus from simulated WGS
  reads with known ground truth.
when_to_use_negative:
- Evaluating the Strain Diversity Problem stage — use the separate SDP simulation
  with random strain-type distributions under perfect allele calls instead
- Data are real WGS samples with unknown ground truth — precision/recall/TVD against
  truth cannot be computed
- Target organism/pathogen uses an MLST scheme other than the B. burgdorferi 8-locus
  scheme (the simulation draws from that specific allele database)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: run_sim.py
  role: 'Runs the allele-panel simulation: draws alleles/proportions and invokes ART
    to simulate reads'
  repo: https://github.com/WGS-TB/MLST
- name: ART
  role: Simulates Illumina reads from simulated allele panels following real tick-sample
    read characteristics
  repo: https://www.niehs.nih.gov/research/resources/software/biostatistics/art/
- name: Bowtie v0.12.7
  role: Aligns simulated reads to the allele database
  repo: https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- name: samtools v1.3.1
  role: Extracts mapping information from .sam files after Bowtie alignment
  repo: https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- name: CPLEX 12.6.3.0
  role: Solves the 0-1 ILP for the Allele Diversity Problem via its Python API
- name: GAML
  role: Selects among multiple optimal ILP solutions using a likelihood-based model
- name: pyemd
  role: Computes Earth Mover's / Total Variation Distance between predicted and true
    allele distributions
  repo: https://github.com/garydoranjr/pyemd
- name: editdistance
  role: Computes Levenshtein distance used to pick alleles within edit distance d
    of the seed allele
  repo: https://github.com/aflc/editdistance
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:9227a00b6da0a4723892fcc73c9335ce88440a6d3adba010d3b4119491f4ee42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGS-TB/MLST
  tools:
  - ART
  - Bowtie v0.12.7
  - CPLEX 12.6.3.0
  - GAML
  - Python 2.7
  - editdistance
  - pyemd
  - run_sim.py
  - samtools v1.3.1
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# mlst-allele-panel-simulation

## Summary

Simulates mixed B. burgdorferi MLST allele panels per locus to benchmark allele-diversity inference: draws random allele sets and proportions, simulates reads with ART, maps with Bowtie, and scores precision/recall/TVD against truth.

## When to use

Use when benchmarking whether a metagenomic allele/strain deconvolution pipeline recovers fractional allele composition per MLST locus from simulated WGS reads with known ground truth.

## When NOT to use

- Evaluating the Strain Diversity Problem stage — use the separate SDP simulation with random strain-type distributions under perfect allele calls instead
- Data are real WGS samples with unknown ground truth — precision/recall/TVD against truth cannot be computed
- Target organism/pathogen uses an MLST scheme other than the B. burgdorferi 8-locus scheme (the simulation draws from that specific allele database)

## Inputs

- B. burgdorferi MLST allele database (8 housekeeping genes, 1726 known alleles)
- Simulation parameters: allele count k in [2,7], edit distance d in {5,10,15,20,25}, coverage c in {30,100,300}, 40 iterations per combination
- Read simulation settings matching real tick-sample read characteristics (ART)

## Outputs

- Simulated FASTQ read sets per locus and parameter combination
- Predicted allele sets and fractional allele proportions per simulation
- adp_simulation_results.csv with per-locus Precision, Recall, and TVD averages and standard deviations across all (c,d) combinations

## How to apply

For each of the 8 B. burgdorferi MLST loci (clpA, clpX, nifS, pepX, pyrG, recG, rplB, uvrA), use run_sim.py to draw a random allele count k in [2,7], pick one database allele plus k-1 others within an edit distance d, and assign random proportions summing to 1; run 40 simulations per (c,d) combination with coverage c in {30,100,300} and d in {5,10,15,20,25}. Simulate reads with ART using characteristics of the real tick-sample reads, then map to the allele database with Bowtie v0.12.7 and extract mapping information with samtools. Solve the Allele Diversity Problem as a 0-1 ILP via the CPLEX Python API (objective minimizing number of selected alleles plus normalized Phred mismatch weights), selecting among multiple optima with GAML, and compute allele proportions by weighting selected-allele read mappings after filtering reads mapping equally well to all selected alleles. Score each simulation with Precision, Recall, and Total Variation Distance (via pyemd) against the simulated truth, then aggregate per-locus averages and standard deviations across parameter combinations.

## Related tools

- **run_sim.py** (Runs the allele-panel simulation: draws alleles/proportions and invokes ART to simulate reads) — https://github.com/WGS-TB/MLST
- **ART** (Simulates Illumina reads from simulated allele panels following real tick-sample read characteristics) — https://www.niehs.nih.gov/research/resources/software/biostatistics/art/
- **Bowtie v0.12.7** (Aligns simulated reads to the allele database) — https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- **samtools v1.3.1** (Extracts mapping information from .sam files after Bowtie alignment) — https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- **CPLEX 12.6.3.0** (Solves the 0-1 ILP for the Allele Diversity Problem via its Python API)
- **GAML** (Selects among multiple optimal ILP solutions using a likelihood-based model)
- **pyemd** (Computes Earth Mover's / Total Variation Distance between predicted and true allele distributions) — https://github.com/garydoranjr/pyemd
- **editdistance** (Computes Levenshtein distance used to pick alleles within edit distance d of the seed allele) — https://github.com/aflc/editdistance

## Examples

```
python run_sim.py -i 40 -f simulation_results -c 100 -e 10 -b ../bowtieFolder/ -s ../../samtools -a ../art_illumina
```

## Evaluation signals

- Reproduced per-locus B-MLST statistics match Table 1: average precision ~0.98 ± 0.011, average recall ~0.96 ± 0.026
- Per-gene Total Variation Distance falls in the ~0.07-0.12 range
- Predicted fractional allele proportions per simulation sum to 1 and match the randomly assigned truth proportions
- 40 simulations are completed for each (c,d) combination with c in {30,100,300} and d in {5,10,15,20,25}, producing results aggregated in adp_simulation_results.csv
- Simulated reads (ART) replicate the read characteristics of the real tick-sample data used as reference

## Limitations

- Results depend on ART read simulation following characteristics of one real dataset (SRR2034336-based tick samples); transfer to other sequencing characteristics is unverified
- The Python 2.7 stack and Bowtie v0.12.7 / samtools v1.3.1 versions are dated; pyemd requires Python 2 and does not support Python 3
- Simulation covers only panels of up to 7 alleles per locus (k in [2,7]); higher within-host allele diversity is not benchmarked
- When the ILP has multiple optimal solutions, solution selection relies on GAML, adding dependence on a likelihood-based assembly model

## Evidence

- [methods] ADP simulation design: "For each locus of the Borrelia MLST scheme, we drew a random number k∈[2,7], selected a random allele from the database"
- [methods] ART simulation rationale: "All reads are simulated using ART [18], following the characteristics of the reads from the real data set"
- [methods] Bowtie version: "To align the simulated reads to the alleles of the database, we used Bowtie v0.12.7 [19]."
- [readme] run_sim.py usage: "python run_sim.py [-h] [-i NUMOFITER] [-f SIMULATIONRESULTFOLDER] [-c COVERAGE] [-e EDITDIST] [-fp FULLPAPER]"
- [methods] GAML post-processing: "If the above 0-1 ILP has multiple optimal solutions, we resort to a likelihood based method to select one, namely GAML [17]"
- [abstract] Simulation accuracy finding: "Our method performs accurately on simulated data and generates results on a real data set of Borrelia burgdorferi genomes"
