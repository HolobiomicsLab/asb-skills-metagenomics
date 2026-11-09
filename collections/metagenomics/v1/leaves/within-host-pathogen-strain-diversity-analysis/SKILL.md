---
name: within-host-pathogen-strain-diversity-analysis
description: Use when WGS read pairs from mixed-infected samples must be typed at
  MLST loci (e.g. the 8-gene B. burgdorferi scheme with 1726 alleles and 753 strain
  types) to infer per-sample allele proportions and strain-type mixtures, including
  novel strains.
when_to_use_negative:
- De novo strain reconstruction is required where reference alleles or known MLST
  strain types are unavailable — the paper explicitly notes de novo methods like DESMAN
  'cannot take advantage of known reference strains', and this pipeline depends on
  the MLST database
- Samples with a single, clonal strain and no within-host heterogeneity, where a simple
  allele caller suffices
- Species or pathogens other than B. burgdorferi without an equivalent MLST allele/strain-type
  database and per-gene edit-distance matrices
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0199
tools:
- name: fastq-dump (SRA toolkit)
  role: downloads the sample read pairs from SRA (v2.8.2 used)
  repo: https://github.com/WGS-TB/MLST
- name: Bowtie v0.12.7
  role: maps sample reads onto the 1726 known MLST alleles
- name: samtools v1.3.1
  role: extracts mapping information from SAM files after Bowtie mapping
- name: CPLEX 12.6.3.0
  role: solves the 0-1 ILP (Allele Diversity Problem) and the MILP (Strain Diversity
    Problem) via its Python API
- name: GAML
  role: likelihood-based tie-breaking among multiple optimal ILP solutions
- name: editdistance (aflc/roy-ht)
  role: computes edit distances from novel strains to the 753 known strain types for
    SDP objective weights
  repo: https://github.com/aflc/editdistance
- name: PyEMD
  role: Earth Mover's Distance calculations used in the pipeline
  repo: https://github.com/garydoranjr/pyemd
- name: strainEST
  role: comparison baseline for strain composition and abundance estimation
  repo: https://github.com/compmetagen/strainest
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:05ab43bd8b167d905aa9eb6dcafb0e75b55b66985ea8d91dd926de457626eee1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGS-TB/MLST
  tools:
  - Bowtie v0.12.7
  - CPLEX 12.6.3.0
  - GAML
  - PyEMD
  - Python 2.7
  - editdistance (aflc/roy-ht)
  - fastq-dump
  - fastq-dump (SRA toolkit)
  - samtools v1.3.1
  - strainEST
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# within-host-pathogen-strain-diversity-analysis

## Summary

Deconvolves within-host pathogen strain composition by applying the paper's two-stage mixed integer linear programming pipeline (Allele Diversity Problem, then Strain Diversity Problem) to MLST-sequenced Borrelia burgdorferi tick samples, detecting both known and novel strains despite within-host heterogeneity.

## When to use

Use when WGS read pairs from mixed-infected samples must be typed at MLST loci (e.g. the 8-gene B. burgdorferi scheme with 1726 alleles and 753 strain types) to infer per-sample allele proportions and strain-type mixtures, including novel strains.

## When NOT to use

- De novo strain reconstruction is required where reference alleles or known MLST strain types are unavailable — the paper explicitly notes de novo methods like DESMAN 'cannot take advantage of known reference strains', and this pipeline depends on the MLST database
- Samples with a single, clonal strain and no within-host heterogeneity, where a simple allele caller suffices
- Species or pathogens other than B. burgdorferi without an equivalent MLST allele/strain-type database and per-gene edit-distance matrices

## Inputs

- Paired-end WGS reads (2x76bp FASTQ) from B. burgdorferi-infected tick samples, e.g. 24 samples including SRR2034336 downloaded from SRA
- B. burgdorferi MLST allele database: 8 housekeeping genes with 1726 known alleles and 753 known strain types
- Edit-distance matrices for each gene (editDist folder) used to weight novel strains in the SDP objective

## Outputs

- Per-locus allele sets and normalized allele proportions (geneX_proportions.csv per sample)
- Per-sample strain types and proportions (sampleX_strainsAndProportions.csv)
- Tally of inferred novel versus existing database strains (e.g. 60 novel and 10 existing strains across the 24 real tick samples)

## How to apply

Stage 1 processes each sample individually: download the read pairs (2x76bp) with fastq-dump v2.8.2 via download_samples.py, map reads onto the 1726 known alleles of the 8-gene MLST scheme with Bowtie v0.12.7, extract SAM-file mapping information with samtools v1.3.1 via mapSamples.py, then solve the Allele Diversity Problem as a 0-1 ILP with CPLEX 12.6.3.0 (alleleDiversity.py) whose objective minimizes summed normalized Phred mismatch weights plus the number of selected alleles; when multiple optimal solutions exist, break ties with the likelihood-based GAML method, then compute normalized allele proportions after filtering reads mapping equally well to all selected alleles. Stage 2 aggregates allele distributions across all samples (e.g. the 24 Vermont/New York/Massachusetts/Connecticut tick samples from 2007) and solves the Strain Diversity Problem as an MILP with CPLEX (globalStrainDiversity.py), penalizing novel strains by edit distance to the 753 known strain types and respecting per-allele proportion error bounds (epsilon=0.1, smallest-proportion tolerance delta=0.001); stop when the relative gap is within the set percentage or the time limit (the paper used 10% gap and 24 hours) is reached. Output per-sample strain types and proportions to sampleX_strainsAndProportions.csv and tally novel versus existing database strains.

## Related tools

- **fastq-dump (SRA toolkit)** (downloads the sample read pairs from SRA (v2.8.2 used)) — https://github.com/WGS-TB/MLST
- **Bowtie v0.12.7** (maps sample reads onto the 1726 known MLST alleles)
- **samtools v1.3.1** (extracts mapping information from SAM files after Bowtie mapping)
- **CPLEX 12.6.3.0** (solves the 0-1 ILP (Allele Diversity Problem) and the MILP (Strain Diversity Problem) via its Python API)
- **GAML** (likelihood-based tie-breaking among multiple optimal ILP solutions)
- **editdistance (aflc/roy-ht)** (computes edit distances from novel strains to the 753 known strain types for SDP objective weights) — https://github.com/aflc/editdistance
- **PyEMD** (Earth Mover's Distance calculations used in the pipeline) — https://github.com/garydoranjr/pyemd
- **strainEST** (comparison baseline for strain composition and abundance estimation) — https://github.com/compmetagen/strainest

## Examples

```
python globalStrainDiversity.py -o strainsAndProp -g 10 -timelim 86400 -pathToDistMat editDist
```

## Evaluation signals

- Acceptance criterion for the real data set: the inferred candidate set across the 24 tick samples contains exactly 60 novel and 10 existing strains, matching the paper's report
- Each sample folder in variantsAndProp contains 8 geneX_proportions.csv files (one per MLST locus) with alleles summing to normalized proportions 0-1
- The output folder from globalStrainDiversity.py contains a sampleX_strainsAndProportions.csv for each of the 24 samples
- Simulated-data validation: ADP accuracy on fractional allele composition and complete strain types under the EvoMod1/EvoMod2/EvoMod3 simulation schemes, with Kallisto and strainEST used as comparison baselines
- Solver run must stop only when both the relative gap threshold and the time limit condition are satisfied as defined by the -g and -timelim arguments

## Limitations

- Both the Allele Diversity Problem and the Strain Diversity Problem are NP-hard (reductions from 3-dimensional matching and 3-partition), so large instances require stopping criteria (relative gap, time limit) and may return approximate solutions
- Handling of closely related strain clusters relies on ad-hoc post-processing rather than a principled method; more principled approaches remain unexplored
- High sensitivity to the chosen error bound (epsilon=0.1 per allele proportion) and smallest-proportion tolerance (delta=0.001); results depend on the mapping quality of Bowtie onto the allele database

## Evidence

- [abstract] First we process each sample individually by assigning it, for each locus in the MLST scheme, a set of alleles and a proportion for each allele.: "First we process each sample individually by assigning it, for each locus in the MLST scheme, a set of alleles and a proportion for each allele."
- [methods] 60 novel and 10 existing strains were inferred from 24 tick samples: "60 novel and 10 existing strains were inferred to be potential candidates for explaining the strain diversity in this large sample of ticks."
- [methods] The traditional B. burgdorferi MLST scheme with 1726 alleles: "the traditional B. burgdorferi MLST scheme [11] composed of 8 housekeeping genes having a combined total of 1726 known alleles"
- [readme] README instruction to solve allele diversity: "python alleleDiversity.py [-h] [-s SAMPLE]"
- [readme] README stopping criterion for global strain diversity solver: "The program will stop if and only if `-g` and `-timelim` are both satisfied."
- [methods] GAML tie-breaking rationale: "If the above 0-1 ILP has multiple optimal solutions, we resort to a likelihood based method to select one, namely GAML [17]"
