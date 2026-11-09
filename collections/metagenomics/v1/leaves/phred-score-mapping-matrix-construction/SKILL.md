---
name: phred-score-mapping-matrix-construction
description: Use when short reads have been mapped to known MLST alleles and you need
  to encode read-allele mapping quality as a cost matrix (Phred-scaled mismatch scores)
  before solving the allele diversity ILP.
when_to_use_negative:
- No Phred quality scores are available for the reads (e.g. pseudo-aligned or quantified
  reads), since the matrix entries depend on per-base quality values
- Reads have not yet been mapped to the allele database with Bowtie — the matrix is
  built from .sam-derived mapping information, not raw reads
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Bowtie
  role: Maps sample reads to the MLST allele database; its .sam output is parsed to
    obtain read-allele pairs and mismatch information
  repo: https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- name: Samtools
  role: Extracts mapping information from .sam files that is written into the sampleX_geneY_paired_reads.txt
    input files
  repo: https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- name: Python 2.7
  role: Language in which the matrix construction and downstream ILP scripts (mapSamples.py,
    alleleDiversity.py) are implemented
  repo: https://github.com/WGS-TB/MLST
- name: CPLEX 12.6.3.0
  role: Solves the 0-1 ILP formulated on the constructed Phred-score matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:25691e07efe410e1e78ca1be4fec29be96a7a7f1283638609fb3ef14c231527e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bowtie
  - Bowtie v0.12.7
  - CPLEX 12.6.3.0
  - GAML
  - Python 2.7
  - Samtools
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# phred-score-mapping-matrix-construction

## Summary

Constructs the weighted bipartite read-allele incidence matrix M used by the Allele Diversity Problem ILP, where entry m_ij is the sum of Phred scores of mismatched bases (normalized over the ASCII 33-126 range) between read r_i and allele a_j, with non-mapping reads set to infinity.

## When to use

Use when short reads have been mapped to known MLST alleles and you need to encode read-allele mapping quality as a cost matrix (Phred-scaled mismatch scores) before solving the allele diversity ILP.

## When NOT to use

- No Phred quality scores are available for the reads (e.g. pseudo-aligned or quantified reads), since the matrix entries depend on per-base quality values
- Reads have not yet been mapped to the allele database with Bowtie — the matrix is built from .sam-derived mapping information, not raw reads

## Inputs

- sampleX_geneY_paired_reads.txt files in the sample's variantsAndProp folder (extracted from Bowtie .sam output via samtools)
- FASTQ read files with Phred quality scores (downloaded with fastq-dump v2.8.2)
- Database of known B. burgdorferi MLST alleles (8 housekeeping genes, 1726 alleles)

## Outputs

- Weighted bipartite read-allele incidence matrix M with entries m_ij = sum of normalized Phred scores of mismatched bases (infinity for non-mapping reads)
- Matrix consumed by the alleleDiversity.py 0-1 ILP to select alleles and assign per-locus proportions

## How to apply

For each sample-gene pair, load the per-sample, per-gene read mapping file (sampleX_geneY_paired_reads.txt in the variantsAndProp folder) that mapSamples.py extracted from the .sam files produced by Bowtie v0.12.7. For each read i and allele j, compute m_ij as the sum of normalized Phred scores of the mismatched bases, normalizing scores over the 33-126 ASCII range so entries reflect sequencing-error-scaled mismatch cost rather than raw quality values. Set m_ij to infinity for reads that do not map to an allele, so the ILP can never select that pairing. Feed the resulting weighted incidence matrix into the 0-1 ILP with variables x_j (allele selected) and y_ik (read explained via mapping score q_ik), minimizing sum(q_ik*y_ik) + sum(x_j). The Phred-based weighting is the rationale: it lets the solver prefer allele assignments whose mismatches are more likely explained by sequencing error.

## Related tools

- **Bowtie** (Maps sample reads to the MLST allele database; its .sam output is parsed to obtain read-allele pairs and mismatch information) — https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- **Samtools** (Extracts mapping information from .sam files that is written into the sampleX_geneY_paired_reads.txt input files) — https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- **Python 2.7** (Language in which the matrix construction and downstream ILP scripts (mapSamples.py, alleleDiversity.py) are implemented) — https://github.com/WGS-TB/MLST
- **CPLEX 12.6.3.0** (Solves the 0-1 ILP formulated on the constructed Phred-score matrix)

## Examples

```
python mapSamples.py -c 4 -b ../bowtieFolder/ -s ../../samtools; python alleleDiversity.py -s SRR2034336
```

## Evaluation signals

- For every read i, the uniqueness constraint sum_k y_ik = 1 must be satisfiable: each read is explained via exactly one allele mapping score
- Non-mapping reads must carry infinite (or prohibitively large) cost so no optimal solution assigns them
- For sample SRR2034336, the resulting ILP should average ~20,112 variables per gene and solve within ~1.5GB RAM / ~33 min for all 8 genes on a 4-CPU machine
- Each sample folder in variantsAndProp must yield 8 geneX_proportions.csv files whose allele weights sum to 1 after normalization
- Simulated-data accuracy should match reported levels (e.g. average B-MLST precision 0.98 +/- 0.011)

## Limitations

- Matrix size grows with the number of reads and alleles; the ILP built on it is large (tens of thousands of variables per gene) and requires a commercial solver (CPLEX academic license)
- Phred normalization assumes scores in the ASCII 33-126 range; other encodings would distort the mismatch costs
- If multiple optimal ILP solutions exist, matrix construction alone cannot disambiguate them — a likelihood-based post-processing step (GAML) is needed

## Evidence

- [methods] Weighted bipartite read-allele incidence matrix with Phred-score costs: [evidence span withheld — anchor status: not_found]
- [methods] ILP formulated on the matrix, solved with CPLEX: "Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0."
- [readme] Mapping input files produced per sample-gene: "The script will create a `variantsAndProp` folder containing a folder for each sample, where each of these folder contains a list of `sampleX_geneY_paired_reads.txt` text files"
- [methods] Reads are aligned to the allele database with Bowtie before matrix construction: "To align the simulated reads to the alleles of the database, we used Bowtie v0.12.7 [19]."
- [readme] Downstream output of the ILP that consumes the matrix: "This script will output some intermediate results to the screen and create 8 CSV files (for each locus) in each of the sample folder in `variantsAndProp`, named as `geneX_proportions.csv`"
