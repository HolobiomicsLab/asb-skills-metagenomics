---
name: mixed-integer-linear-programming-strain-deconvolution
description: Use when WGS reads from heterogeneous samples must be decomposed into
  known MLST strain types plus novel strains, given a reference allele database (e.g.
  8 housekeeping genes, 1726 alleles, 753 strain types) and mixed samples where strain
  proportions vary.
when_to_use_negative:
- When reference genomes or strain types are unknown — de novo methods such as DESMAN
  are the appropriate choice since this pipeline relies on a known MLST allele/strain
  database
- When samples are single-strain (no within-host heterogeneity), where the two-stage
  deconvolution adds unnecessary complexity
- 'When exact strain-level precision is the sole requirement even from imperfect allele
  calls: the paper''s Table 2 shows full-pipeline SDP exact precision/recall drops
  to ~0.3-0.5 versus ~0.8 with perfect allele input'
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: CPLEX 12.6.3.0
  role: Formulates and solves both the ADP 0-1 ILP and the SDP MILP via its Python
    API
- name: Bowtie v0.12.7
  role: Maps sample/simulated reads to the allele database; produces .sam files consumed
    by the ADP stage
  repo: https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- name: samtools v1.3.1
  role: Extracts mapping information from .sam files into text files feeding the first-stage
    ILP
  repo: https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- name: GAML
  role: Likelihood-based selection among multiple optimal ADP ILP solutions
- name: ART
  role: Simulates reads following real dataset characteristics for pipeline validation
  repo: https://www.niehs.nih.gov/research/resources/software/biostatistics/art/
- name: editdistance
  role: Computes Levenshtein edit distances between alleles/strain types for novel-strain
    weights and SNP-based soft evaluation
  repo: https://github.com/aflc/editdistance
- name: PyEMD
  role: Computes Earth Mover's Distance between true and inferred allele/strain distributions
    during evaluation
  repo: https://github.com/garydoranjr/pyemd
- name: fastq-dump (SRA toolkit)
  role: Downloads sample reads (v2.8.2) as pipeline input
  repo: https://www.ncbi.nlm.nih.gov/sra/docs/toolkitsoft/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:2a26d8867553510849ad12ed1db126296606a0d30b50f1e663c1f0ad48dd536e
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
  - fastq-dump (SRA toolkit)
  - samtools v1.3.1
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# mixed-integer-linear-programming-strain-deconvolution

## Summary

A two-stage MILP pipeline that deconvolves within-host pathogen strain mixtures from WGS reads in the B. burgdorferi MLST framework: an Allele Diversity Problem (ADP) ILP infers per-locus allele sets and proportions, and a Strain Diversity Problem (SDP) MILP jointly infers strain types across samples, including novel strains.

## When to use

Use when WGS reads from heterogeneous samples must be decomposed into known MLST strain types plus novel strains, given a reference allele database (e.g. 8 housekeeping genes, 1726 alleles, 753 strain types) and mixed samples where strain proportions vary.

## When NOT to use

- When reference genomes or strain types are unknown — de novo methods such as DESMAN are the appropriate choice since this pipeline relies on a known MLST allele/strain database
- When samples are single-strain (no within-host heterogeneity), where the two-stage deconvolution adds unnecessary complexity
- When exact strain-level precision is the sole requirement even from imperfect allele calls: the paper's Table 2 shows full-pipeline SDP exact precision/recall drops to ~0.3-0.5 versus ~0.8 with perfect allele input

## Inputs

- FASTQ reads for each sample (downloaded via fastq-dump v2.8.2, e.g. 24 tick samples infected with B. burgdorferi)
- B. burgdorferi MLST allele database (8 housekeeping genes, 1726 alleles, 753 strain types)
- Per-gene edit distance matrices for strain/allele similarity (editDist folder)
- Simulated reads from ART following real dataset read characteristics (for validation runs)

## Outputs

- geneX_proportions.csv per sample per locus: alleles and their normalized proportions (ADP stage)
- sampleX_strainsAndProportions.csv per sample: inferred strain types and proportions (SDP stage)
- Evaluation statistics: precision, recall, TVD, EMD, and soft precision/soft recall (5-SNP tolerance)

## How to apply

Stage 1: map reads per sample to the allele database with Bowtie v0.12.7 and extract mapping information from .sam files using samtools v1.3.1; then solve the ADP set-cover 0-1 ILP per sample and locus with the CPLEX 12.6.3.0 Python API, minimizing the number of alleles plus normalized Phred-score mismatch weights, selecting among multiple optima with GAML, and computing normalized allele proportions per locus (8 CSV files named geneX_proportions.csv per sample). Stage 2: feed the inferred allele distributions into the SDP MILP in CPLEX, which minimizes the sum of weights of novel strain types (edit-distance-based, epsilon=0.1 error bound, delta=0.001 tolerance) plus total proportion deviation across all samples, stopping when the relative gap (-g, default 5%) and time limit (-timelim, default 600 s) are both satisfied; outputs sampleX_strainsAndProportions.csv. Evaluate with precision, recall, TVD and EMD (using the aflc/editdistance and PyEMD packages), plus soft precision/recall where a called strain counts as TP if it differs from the true strain by at most 5 SNPs. On simulated end-to-end runs (EvoMod1/2/3), expect strong ADP performance (precision ~0.92-0.96, recall ~0.90-0.92) but degraded SDP exact precision/recall (~0.3-0.5 versus ~0.8 with perfect input, TVD ~0.6-0.8).

## Related tools

- **CPLEX 12.6.3.0** (Formulates and solves both the ADP 0-1 ILP and the SDP MILP via its Python API)
- **Bowtie v0.12.7** (Maps sample/simulated reads to the allele database; produces .sam files consumed by the ADP stage) — https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- **samtools v1.3.1** (Extracts mapping information from .sam files into text files feeding the first-stage ILP) — https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- **GAML** (Likelihood-based selection among multiple optimal ADP ILP solutions)
- **ART** (Simulates reads following real dataset characteristics for pipeline validation) — https://www.niehs.nih.gov/research/resources/software/biostatistics/art/
- **editdistance** (Computes Levenshtein edit distances between alleles/strain types for novel-strain weights and SNP-based soft evaluation) — https://github.com/aflc/editdistance
- **PyEMD** (Computes Earth Mover's Distance between true and inferred allele/strain distributions during evaluation) — https://github.com/garydoranjr/pyemd
- **fastq-dump (SRA toolkit)** (Downloads sample reads (v2.8.2) as pipeline input) — https://www.ncbi.nlm.nih.gov/sra/docs/toolkitsoft/

## Examples

```
python mapSamples.py -c 4 && python alleleDiversity.py && python globalStrainDiversity.py -o strainsAndProp -oc all -g 5 -timelim 600 -pathToDistMat editDist
```

## Evaluation signals

- ADP stage on simulated data should reach precision ~0.92-0.96 and recall ~0.90-0.92 per locus across EvoMod1/2/3 mechanisms
- Full-pipeline SDP with chained ADP output should reproduce the Table 2 pattern: exact precision/recall ~0.3-0.5 (vs ~0.8 with perfect input) and TVD rising to ~0.6-0.8
- Per-locus ADP outputs must be 8 geneX_proportions.csv files per sample with normalized allele proportions summing to 1
- SDP solver must respect its stop rule: it halts only when both the relative gap (-g, default 5%) and time limit (-timelim, default 600 s) are satisfied
- Soft recall/precision under the 5-SNP tolerance should substantially exceed exact-strain metrics; strain types differing from truth by ≤5 SNPs count as true positives

## Limitations

- Small ADP errors propagate: chaining inferred allele distributions into the SDP dramatically degrades exact strain typing relative to perfect allele input (exact precision/recall ~0.3-0.5, TVD ~0.6-0.8)
- Closely related strain clusters are handled by ad-hoc post-processing rather than a principled method; more principled approaches remain unexplored
- The ADP and SDP are both NP-hard; the SDP may require stopping at a 5% relative gap or a 600-second time limit rather than proven optimality
- The pipeline depends on Python 2.7 and requires an academic CPLEX license; PyEMD requires Python 2 (Python 3 is not supported)
- Some reported parameters (e.g. epsilon=0.1, delta=0.001, d=15, 5-SNP soft tolerance) and Table 2 content appear in supplementary material (Additional file 1) not fully included in the article text

## Evidence

- [methods] Both ILPs are formulated and solved using the Python API of IBM's CPLEX 12.6.3.0.: [evidence span withheld — anchor status: not_found]
- [methods] Multiple optima handled by GAML: "If the above 0-1 ILP has multiple optimal solutions, we resort to a likelihood based method to select one, namely GAML [17]"
- [methods] GAML post-processing rationale: "we resort to a likelihood based method to select one, namely GAML [17], a probabilistic model for genome assembly."
- [methods] Read mapping with Bowtie: "To align the simulated reads to the alleles of the database, we used Bowtie v0.12.7 [19]."
- [methods] Evaluation metrics include edit distance and EMD packages: "In computing edit distance, we used the package by https://github.com/aflc/editdistance"
- [readme] SDP solver stopping rule from README: "The program will stop if and only if `-g` and `-timelim` are both satisfied."
- [readme] SDP output files from README: "The folder contains a list of `sampleX_strainsAndProportions.csv` with sampleX corresponds a particular sample"
- [readme] ADP output files from README: "create 8 CSV files (for each locus) in each of the sample folder in `variantsAndProp`, named as `geneX_proportions.csv`"
- [abstract] Real-data diversity finding: "Our method performs accurately on simulated data and generates results on a real data set of Borrelia burgdorferi genomes"
- [intro] De novo methods comparison: "De novo methods, such as DESMAN [12], cannot take advantage of known reference strains or alleles"
