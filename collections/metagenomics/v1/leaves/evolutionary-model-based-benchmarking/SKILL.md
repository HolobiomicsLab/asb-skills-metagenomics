---
name: evolutionary-model-based-benchmarking
description: Use when evaluating a strain-composition or haplotype-reconstruction
  method on simulated samples where ground-truth strain type distributions and mutation/recombination
  events must be generated under defined evolutionary models.
when_to_use_negative:
- Input is real sequencing reads rather than perfect allele calls — the read-to-allele
  pipeline (ART simulation, Bowtie v0.12.7 mapping, allele ILP) is a separate skill
- The organism/scheme has no curated set of known MLST strain types with edit-distance
  matrices, since candidate weighting relies on distances to 753 known strain types
- The goal is de novo strain discovery without a reference strain database — the SDP
  is explicitly reference-informed, unlike de novo methods such as DESMAN
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: CPLEX 12.6.3.0
  role: Solves the Strain Diversity Problem MILP via its Python 2.7 API (globalStrainDiversity.py)
    on the simulated perfect-allele inputs
  repo: https://github.com/WGS-TB/MLST
- name: editdistance
  role: Computes Levenshtein edit distances between candidate strain types and the
    753 known strain types for candidate weighting
  repo: https://github.com/aflc/editdistance
- name: PyEMD
  role: Computes Earth-Mover's distance between inferred and true strain distributions
    as an evaluation metric
  repo: https://github.com/garydoranjr/pyemd
- name: pandas
  role: Handles CSV input/output of allele proportions and strain distributions during
    benchmarking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:e6b66bdc4b78820ad2429d8d4cc5acf62c2e099ae6264c0f3096def3e6af9c25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGS-TB/MLST
  tools:
  - CPLEX 12.6.3.0
  - PyEMD
  - Python 2.7
  - editdistance
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# evolutionary-model-based-benchmarking

## Summary

Benchmark a strain-diversity MILP solver (SDP) against simulated ground truth generated under five evolutionary mechanisms (EvoMod1/2/2e/2n/3) on the B. burgdorferi MLST scheme, using precision, recall, TVD, soft metrics and Earth-Mover's distance to judge recovery of true within-host strain diversity.

## When to use

Use when evaluating a strain-composition or haplotype-reconstruction method on simulated samples where ground-truth strain type distributions and mutation/recombination events must be generated under defined evolutionary models.

## When NOT to use

- Input is real sequencing reads rather than perfect allele calls — the read-to-allele pipeline (ART simulation, Bowtie v0.12.7 mapping, allele ILP) is a separate skill
- The organism/scheme has no curated set of known MLST strain types with edit-distance matrices, since candidate weighting relies on distances to 753 known strain types
- The goal is de novo strain discovery without a reference strain database — the SDP is explicitly reference-informed, unlike de novo methods such as DESMAN

## Inputs

- B. burgdorferi MLST scheme: 8 housekeeping genes, 1726 known alleles, 753 known strain types
- Per-locus edit-distance matrices (provided 'editDist' folder) for candidate strain weighting
- Simulated ground-truth strain type distributions generated under EvoMod1/2/2e/2n/3 with random proportions summing to 1
- Perfect (ground-truth) allele calls per locus per sample

## Outputs

- Inferred strain types and proportions per sample (sampleX_strainsAndProportions.csv from globalStrainDiversity.py)
- Per-mechanism precision and recall on complete strain types
- Per-mechanism Total Variation Distance, soft-precision, soft-recall (TP within 5 SNPs) and Earth-Mover's distance
- Benchmark table comparable to Table 2's SDP simulation block

## How to apply

For each evolutionary mechanism, generate random ground-truth strain type distributions summing to 1: EvoMod1 (1 existing strain mutated m=2 times at edit distance ≤ d=15 to yield a novel strain), EvoMod2 (2 existing + 2 novel), EvoMod2e/EvoMod2n (EvoMod2 minus a random existing/novel strain), and EvoMod3 (EvoMod2 plus an allele-exchange recombination event among 4 available strains). Build each sample's candidate strain type set as the Cartesian product of the perfect alleles called per locus, and weight candidates by N times the minimum edit distance to the closest of the 753 known B. burgdorferi strain types (known strains get weight 0), using the supplied 'editDist' distance matrices and the aflc/editdistance package. Solve the SDP MILP with IBM CPLEX 12.6.3.0 via its Python 2.7 API (globalStrainDiversity.py) using error bound ε=0.1 and tolerance δ=0.001, minimizing summed weights of novel strains plus total proportion errors. Compare inferred vs. true distributions per mechanism using precision, recall, Total Variation Distance, soft-precision, soft-recall (TP within 5 SNPs), and Earth-Mover's distance (PyEMD), and check results against Table 2's SDP simulation block.

## Related tools

- **CPLEX 12.6.3.0** (Solves the Strain Diversity Problem MILP via its Python 2.7 API (globalStrainDiversity.py) on the simulated perfect-allele inputs) — https://github.com/WGS-TB/MLST
- **editdistance** (Computes Levenshtein edit distances between candidate strain types and the 753 known strain types for candidate weighting) — https://github.com/aflc/editdistance
- **PyEMD** (Computes Earth-Mover's distance between inferred and true strain distributions as an evaluation metric) — https://github.com/garydoranjr/pyemd
- **pandas** (Handles CSV input/output of allele proportions and strain distributions during benchmarking)

## Examples

```
python globalStrainDiversity.py -o strainsAndProp -oc all -g 5 -timelim 600 -pathToDistMat editDist
```

## Evaluation signals

- Reproduced statistics match Table 2's SDP simulation block, e.g. EvoMod1 precision 0.85±0.28, EvoMod3 precision 0.76±0.21, TVD 0.22±0.24
- Paper's qualitative criterion met: SDP maintains precision and recall of almost 75% even for the recombination model (EvoMod3) given perfect allele calls
- Simulated ground-truth strain proportions each sum to 1, and all known strains receive candidate weight 0
- Candidate strain set equals the Cartesian product of the alleles called per locus, weighted by N × minimum edit distance to the closest known strain type
- globalStrainDiversity.py terminates with relative gap and time-limit settings satisfied, emitting per-sample strain/proportion CSV files

## Limitations

- Requires an academic CPLEX license and Python 2.7; the PyEMD package used supports Python 2 only
- Benchmark relies on perfect allele calls, so it does not measure sensitivity to allele-calling or sequencing error introduced upstream
- Solver stopping is controlled jointly by relative gap (-g, default 5%) and time limit (-timelim, default 600 s), and the program will not stop until the gap is reached even past the time limit
- Handling of clusters of closely related strains is ad-hoc post-processing; more principled approaches remain unexplored, and Additional file 1 (supplementary methods for the NP-completeness proof) is referenced but not included

## Evidence

- [methods] SDP simulation: "For this simulation we selected random strain type distributions and tested the ability of our SDP method to recover the true diversity given perfect allele calls."
- [methods] EvoMod3 definition: "EvoMod3: we apply EvoMod2, then apply a recombination (allele exchange) event on two randomly chosen strains out of the 4 available strains."
- [readme] globalStrainDiversity.py options: "The `-oc` argument specifies different formulation of the objective function, 'all' includes all components of the objective function, 'noPropAndErr' omits the proportion and error component."
- [readme] edit distance package: "In computing edit distance, we used the package by https://github.com/aflc/editdistance and we hereby acknowledge Hiroyuki Tanaka, where the algorithm is proposed"
