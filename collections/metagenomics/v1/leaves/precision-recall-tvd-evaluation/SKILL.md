---
name: precision-recall-tvd-evaluation
description: Use when validating MLST allele- or strain-deconvolution results on simulated
  samples, comparing a pipeline against Kallisto or strainEST, or verifying recovery
  of true strain type distributions from ground-truth allele calls.
when_to_use_negative:
- Evaluating raw read-mapping output (SAM/BAM mapping statistics from Bowtie/samtools)
  before allele sets and proportions are inferred
- Comparing against de novo methods such as DESMAN, which cannot exploit the known
  reference alleles and strain types this metric setup assumes
- Datasets outside the MLST framework (no per-locus allele identities or known strain
  type database to define TP/FP and edit distances)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: editdistance
  role: Computes Levenshtein edit distances between allele sequences for soft-precision/soft-recall
    (TP within 5 SNPs) and candidate weighting
  repo: https://github.com/aflc/editdistance
- name: PyEMD
  role: Earth-Mover's distance calculation between true and inferred strain type distributions
    using the editDist matrices
  repo: https://github.com/garydoranjr/pyemd
- name: Kallisto
  role: Baseline isoform-abundance method whose per-locus allele calls are scored
    with precision/recall/TVD for comparison with B-MLST
- name: strainEST
  role: Baseline strain-composition method whose estimates are scored with soft-precision/soft-recall
    and EMD against the proposed pipeline
  repo: https://github.com/compmetagen/strainest
- name: pandas
  role: Assembles the averaged statistics (± standard deviations) into the comparison
    CSV
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:5e60cae1a6602a5006bf74544de2665adda93d57355d8fd6722d0e13532908c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aflc/editdistance
  tools:
  - ART
  - Bowtie v0.12.7
  - Kallisto
  - PyEMD
  - Python 2.7
  - editdistance
  - pandas
  - strainEST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# precision-recall-tvd-evaluation

## Summary

Evaluation skill for the MLST deconvolution pipeline (ADP/SDP) of the B. burgdorferi study: computes precision, recall, Total Variation Distance, soft-precision/soft-recall (TP within 5 SNPs) and Earth-Mover's distance to benchmark allele calls against Kallisto and strain compositions against strainEST, and to validate SDP recovery under perfect allele calls.

## When to use

Use when validating MLST allele- or strain-deconvolution results on simulated samples, comparing a pipeline against Kallisto or strainEST, or verifying recovery of true strain type distributions from ground-truth allele calls.

## When NOT to use

- Evaluating raw read-mapping output (SAM/BAM mapping statistics from Bowtie/samtools) before allele sets and proportions are inferred
- Comparing against de novo methods such as DESMAN, which cannot exploit the known reference alleles and strain types this metric setup assumes
- Datasets outside the MLST framework (no per-locus allele identities or known strain type database to define TP/FP and edit distances)

## Inputs

- Predicted per-locus allele sets and allele proportions from the ADP stage (geneX_proportions.csv)
- Per-sample strain type distributions and proportions from the SDP stage (sampleX_strainsAndProportions.csv)
- Ground-truth simulated allele compositions and strain type distributions for EvoMod1/2/2e/2n/3
- Kallisto per-locus allele calls and abundances (run_kallisto_sim.py, -i 40)
- strainEST strain composition and abundance estimates
- editDist per-gene edit distance matrices over the B. burgdorferi MLST allele database (8 genes, 1726 alleles, 753 strain types)

## Outputs

- Comparison CSV of precision, recall and TVD (ADP) and soft-precision, soft-recall, EMD (strain/SDP) averaged ± standard deviations across parameter combinations
- Per-statistic PNG boxplots/summary plots in the results folder
- Per-mechanism SDP recovery statistics matching the Table 2 SDP simulation block (e.g. EvoMod1 precision 0.85±0.28, EvoMod3 precision 0.76±0.21, TVD 0.22±0.24)

## How to apply

For per-gene allele comparison (B-MLST vs Kallisto across ART-simulated read sets at coverage c∈{30,100,300} and inter-allele edit distance d∈{5,10,15,20,25}), compute precision TP/(TP+FP), recall TP/(TP+FN) and Total Variation Distance per locus (clpA, clpX, nifS, pepX, pyrG, recG, rplB, uvrA), then average with standard deviations across parameter combinations. For strain composition comparison against strainEST and for the SDP simulation with perfect allele calls (EvoMod1, EvoMod2, EvoMod2e/2n, EvoMod3), additionally compute soft-precision/soft-recall where a true positive counts if it is within 5 SNPs (edit distance) of a predicted strain, and Earth-Mover's distance between distributions using PyEMD on editDist distance matrices. Decision point: B-MLST precision should exceed Kallisto at every locus (e.g. clpA 0.99±0.009 vs 0.97±0.014) with comparable TVD, and SDP should retain ~75% precision/recall even for the recombination model EvoMod3. Output averages ± standard deviations to a comparison CSV and render per-statistic PNG plots.

## Related tools

- **editdistance** (Computes Levenshtein edit distances between allele sequences for soft-precision/soft-recall (TP within 5 SNPs) and candidate weighting) — https://github.com/aflc/editdistance
- **PyEMD** (Earth-Mover's distance calculation between true and inferred strain type distributions using the editDist matrices) — https://github.com/garydoranjr/pyemd
- **Kallisto** (Baseline isoform-abundance method whose per-locus allele calls are scored with precision/recall/TVD for comparison with B-MLST)
- **strainEST** (Baseline strain-composition method whose estimates are scored with soft-precision/soft-recall and EMD against the proposed pipeline) — https://github.com/compmetagen/strainest
- **pandas** (Assembles the averaged statistics (± standard deviations) into the comparison CSV)

## Examples

```
pip install editdistance; pip install -e git+https://github.com/garydoranjr/pyemd.git#egg=pyemd; import editdistance; prec = TP/(TP+FP); rec = TP/(TP+FN); tvd = 0.5*sum(abs(p_true-p_pred))
```

## Evaluation signals

- B-MLST precision exceeds Kallisto at every locus, e.g. clpA 0.99±0.009 vs 0.97±0.014 and nifS 0.96 vs 0.89, while TVD is comparable
- SDP statistics with perfect allele calls match Table 2: EvoMod1 precision 0.85±0.28, EvoMod3 precision 0.76±0.21, TVD 0.22±0.24, with precision and recall near 75% even for the recombination model
- Soft-TP assignment consistent: a predicted strain counts as TP only if within 5 SNPs (edit distance) of a true strain, using the editDist matrices
- Comparison CSV contains averages and standard deviations across all parameter combinations (c∈{30,100,300}, d∈{5,10,15,20,25}) and all 8 loci (clpA, clpX, nifS, pepX, pyrG, recG, rplB, uvrA)
- Per-statistic PNG plots are rendered in the results folder without missing-locus entries

## Limitations

- SDP recovery is evaluated only with perfect (ground-truth) allele calls; propagation of ADP stage errors into SDP metrics is a separate concern
- Close strain clusters require ad-hoc post-processing rather than a principled method, so soft-metric clustering behavior may be unstable
- Simulated evaluation covers specific evolutionary mechanisms (EvoMod1/2/2e/2n/3) and may not reflect all real within-host diversity patterns

## Evidence

- [methods] SDP simulation design: "For this simulation we selected random strain type distributions and tested the ability of our SDP method to recover the true diversity given perfect allele calls."
- [methods] Kallisto baseline rationale: "we compared our results with the results obtained with Kallisto [20], a recent method for isoform abundance estimation"
- [methods] strainEST baseline rationale: "We compared our pipeline with strainEST [13], a recent method to estimate the strain composition and abundance in metagenomics datasets."
- [readme] Edit distance implementation: "In computing edit distance, we used the package by https://github.com/aflc/editdistance"
- [readme] EMD implementation: "We also use the package PyEMD for EMD calculations, where we hereby acknowledge Gary Doran for his PyEMD package: https://github.com/garydoranjr/pyemd."
- [abstract] Simulated-data accuracy claim: "Our method performs accurately on simulated data and generates results on a real data set of Borrelia burgdorferi genomes"
