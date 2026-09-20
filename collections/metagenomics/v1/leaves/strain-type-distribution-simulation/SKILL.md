---
name: strain-type-distribution-simulation
description: 'Use when benchmarking a strain-deconvolution or MLST strain-typing solver
  against known ground truth: you need simulated multi-strain samples with controlled
  novel-strain and recombination events, plus precision/recall/TVD/EMD evaluation
  of inferred strain type proportions.'
when_to_use_negative:
- When the goal is typing real samples without ground truth — this skill is a simulation/benchmark
  harness, not a production typing pipeline
- When evaluating only the ADP (per-locus allele) stage — allele-level simulation
  without strain type distributions is a separate, simpler task
- When reference strain types are unavailable for the organism, since novel-strain
  weighting depends on edit distance to the 753 known MLST strain types
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: CPLEX 12.6.3.0
  role: Solves the SDP MILP via its Python 2.7 API (globalStrainDiversity.py) on the
    simulated distributions
  repo: https://github.com/WGS-TB/MLST
- name: editdistance
  role: Computes Levenshtein distances between candidate strain types and the 753
    known strains for novel-strain weighting
  repo: https://github.com/aflc/editdistance
- name: PyEMD
  role: Computes Earth-Mover's distance between true and inferred strain type distributions
  repo: https://github.com/garydoranjr/pyemd
- name: ART
  role: Simulates reads from ground-truth strains for the full-pipeline (ADP→SDP)
    degradation experiment
- name: Bowtie v0.12.7
  role: Maps ART-simulated reads to the B. burgdorferi allele database before ADP
    inference
- name: GAML
  role: Selects among multiple optimal ADP ILP solutions before allele distributions
    feed the SDP
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:52f524a731b4afd342a7f4c71390ab057a2dbeba6c5e9b5660f22037de3bbe64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGS-TB/MLST
  tools:
  - ART
  - Bowtie v0.12.7
  - CPLEX 12.6.3.0
  - GAML
  - PyEMD
  - Python 2.7
  - editdistance
  - pandas
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# strain-type-distribution-simulation

## Summary

Simulates ground-truth strain type distributions under five evolutionary mechanisms (EvoMod1/2/2e/2n/3) for the B. burgdorferi MLST scheme, then scores SDP strain-typing recovery with precision, recall, TVD, soft metrics (5-SNP threshold), and Earth-Mover's distance. Used both with perfect allele input and chained behind ADP-inferred alleles to quantify error propagation.

## When to use

Use when benchmarking a strain-deconvolution or MLST strain-typing solver against known ground truth: you need simulated multi-strain samples with controlled novel-strain and recombination events, plus precision/recall/TVD/EMD evaluation of inferred strain type proportions.

## When NOT to use

- When the goal is typing real samples without ground truth — this skill is a simulation/benchmark harness, not a production typing pipeline
- When evaluating only the ADP (per-locus allele) stage — allele-level simulation without strain type distributions is a separate, simpler task
- When reference strain types are unavailable for the organism, since novel-strain weighting depends on edit distance to the 753 known MLST strain types

## Inputs

- B. burgdorferi MLST allele database (8 housekeeping genes, 1726 alleles, 753 strain types)
- edit-distance matrices per gene ('editDist' folder) computed with aflc/editdistance
- Random seed/iteration count and evolutionary mechanism choice (EvoMod1/2/2e/2n/3)
- For full-pipeline mode: ADP-inferred allele sets and proportions per locus (geneX_proportions.csv)
- ART-simulated reads following real-dataset read characteristics (coverage c=30 default, 40 simulations per gene)

## Outputs

- Inferred strain types and proportions per sample (sampleX_strainsAndProportions.csv)
- Per-mechanism evaluation table: precision, recall, Total Variation Distance, soft-precision, soft-recall (within 5 SNPs), and Earth-Mover's distance
- Comparison against Table 2 reference blocks (SDP simulation and full-pipeline simulation)

## How to apply

Draw random ground-truth strain type distributions per mechanism: EvoMod1 mutates one existing strain m=2 times at edit distance ≤ d=15; EvoMod2 does this in parallel from two existing strains; EvoMod2e/EvoMod2n remove a random existing/novel strain from EvoMod2; EvoMod3 adds an allele-exchange recombination event on two of the four strains, with proportions summing to 1. Build the candidate strain type set as the Cartesian product of alleles called per locus over the 8 housekeeping genes (1726 alleles, 753 known strain types), weighting each candidate by N times its minimum edit distance to the closest known strain (known strains get weight 0) via the 'editDist' matrices and the aflc/editdistance package. Solve the SDP MILP with CPLEX 12.6.3.0 (Python 2.7, globalStrainDiversity.py) using error bound ε=0.1 and tolerance δ=0.001; either feed perfect allele calls or chain ADP-inferred allele distributions (from set-cover ILP + GAML post-processing on ART-simulated reads mapped with Bowtie v0.12.7) to measure degradation. Score against truth with precision, recall, TVD, soft-precision/soft-recall (a called strain is TP if within 5 SNPs of the true type), and Earth-Mover's distance using PyEMD. For the perfect-input run, validation should match Table 2's SDP simulation block (e.g., EvoMod1 precision 0.85±0.28, EvoMod3 precision 0.76±0.21, TVD 0.22±0.24, with precision/recall near 75% even for the recombination model); the full-pipeline run should show SDP exact precision/recall dropping from ~0.8 to ~0.3–0.5 with TVD rising to ~0.6–0.8.

## Related tools

- **CPLEX 12.6.3.0** (Solves the SDP MILP via its Python 2.7 API (globalStrainDiversity.py) on the simulated distributions) — https://github.com/WGS-TB/MLST
- **editdistance** (Computes Levenshtein distances between candidate strain types and the 753 known strains for novel-strain weighting) — https://github.com/aflc/editdistance
- **PyEMD** (Computes Earth-Mover's distance between true and inferred strain type distributions) — https://github.com/garydoranjr/pyemd
- **ART** (Simulates reads from ground-truth strains for the full-pipeline (ADP→SDP) degradation experiment)
- **Bowtie v0.12.7** (Maps ART-simulated reads to the B. burgdorferi allele database before ADP inference)
- **GAML** (Selects among multiple optimal ADP ILP solutions before allele distributions feed the SDP)

## Examples

```
python run_sim.py -i 40 -f simulation_results -c 30 -e 15 -fp 1
```

## Evaluation signals

- Perfect-input SDP results match Table 2's simulation block: EvoMod1 precision 0.85±0.28, EvoMod3 precision 0.76±0.21, TVD 0.22±0.24
- Precision and recall stay near 75% even for the recombination model (EvoMod3) when allele calls are perfect
- Full-pipeline run reproduces the degradation pattern: exact precision/recall drops from ~0.8 to ~0.3–0.5, TVD rises to ~0.6–0.8, while ADP-stage precision stays ~0.92–0.96
- Simulated strain proportions per sample sum to 1 and novel strains satisfy the m=2 mutation, edit distance ≤ 15 constraint
- Soft-precision/soft-recall (5-SNP tolerance) exceed exact precision/recall, confirming errors are concentrated in near-identical strain types

## Limitations

- SDP accuracy degrades sharply when ADP-stage allele errors propagate, despite ADP itself performing well — small allele mistakes cost strain typing
- Handling closely related strain clusters relies on ad-hoc post-processing rather than a principled method
- Pipeline requires Python 2.7 and CPLEX 12.6.3.0 (academic license); PyEMD requires Python 2 and numpy/scipy
- The SDP MILP solver may stop on a relative gap (default 5%) or time limit (default 600 s) rather than proving optimality
- Simulation parameters are B. burgdorferi-specific (8-gene MLST scheme, d=15, m=2); transfer to other organisms requires re-tuning

## Evidence

- [methods] EvoMod1: We select a random existing strain S, which is then mutated m=2 times to obtain a new strain S′: "EvoMod1: We select a random existing strain S, which is then mutated m=2 times to obtain a new strain S′"
- [methods] EvoMod3 recombination definition: "EvoMod3: we apply EvoMod2, then apply a recombination (allele exchange) event on two randomly chosen strains out of the 4 available strains."
- [methods] SDP simulation description: "For this simulation we selected random strain type distributions and tested the ability of our SDP method to recover the true diversity given perfect allele calls."
- [intro] Evaluation metrics on simulated samples: "We evaluate our approach on simulated samples and find that it is accurate in identifying both the fractional allele composition at each housekeeping gene"
- [methods] CPLEX version and API: "Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0."
- [readme] Edit distance package used: "In computing edit distance, we used the package by https://github.com/aflc/editdistance and we hereby acknowledge Hiroyuki Tanaka"
- [readme] globalStrainDiversity.py CLI with gap and time limit parameters: "`-g` specifies the relative gap in % to stop the solver, default is 5%. `-timelim` specifies the maximum time limit in seconds before stopping the solver, default is 600 seconds."
- [readme] Run simulation script invocation: "python run_sim.py [-h] [-i NUMOFITER] [-f SIMULATIONRESULTFOLDER] [-c COVERAGE] [-e EDITDIST] [-fp FULLPAPER] [-b BOWTIE] [-s SAMTOOLS] [-a ART]"
