---
name: roc-curve-analysis
description: Use when selecting or validating an overlap cutoff for read recruitment
  in TAR-VIR, or when you need an ROC/AUC justification that a sequence-similarity
  threshold separates within-quasispecies overlap from cross-species conservation.
when_to_use_negative:
- The chosen cutoff has no relation to read recruitment string matching (e.g. classifying
  classifier-level alignment scores or mapping quality) — the ROC here is specifically
  over LCS sizes between genome pairs
- You need a general binary-classifier evaluation rather than a sequence-similarity
  threshold justification between reference genomes
- The target virus shares no conservation with any reference genome, since LCS-based
  cutoffs cannot be calibrated for it
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0781
tools:
- name: TAR-VIR
  role: Pipeline whose overlap-extension read recruitment step consumes the cutoff
    justified by this ROC analysis; cutoffs are passed to the overlap module via the
    -c parameter
  repo: https://github.com/chjiao/TAR-VIR
- name: Python
  role: Used to compute pairwise LCS sizes (generalized suffix array or dynamic programming)
    and to build the ROC curve and AUC from the resulting distributions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:9e2572e0373f428115f71ab3ca521c51726f84927655efd877bea9859b62bda3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chjiao/TAR-VIR
  tools:
  - Python
  - TAR-VIR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# roc-curve-analysis

## Summary

Reproduce the longest-common-substring (LCS) ROC analysis that justifies TAR-VIR's read-recruitment overlap cutoff (e.g. 100 bp or more, with 150 bp used in simulations), by computing LCS sizes between viral genomes and between viruses and other microbes, then evaluating true/false positive rates for candidate cutoffs.

## When to use

Use when selecting or validating an overlap cutoff for read recruitment in TAR-VIR, or when you need an ROC/AUC justification that a sequence-similarity threshold separates within-quasispecies overlap from cross-species conservation.

## When NOT to use

- The chosen cutoff has no relation to read recruitment string matching (e.g. classifying classifier-level alignment scores or mapping quality) — the ROC here is specifically over LCS sizes between genome pairs
- You need a general binary-classifier evaluation rather than a sequence-similarity threshold justification between reference genomes
- The target virus shares no conservation with any reference genome, since LCS-based cutoffs cannot be calibrated for it

## Inputs

- Viral reference genomes in FASTA from NCBI (e.g. NC_003977.2 HBV, NC_004102.1 HCV, NC_001710.1 HPgV, Vaccinia and Variola RefSeq records)
- Microbial (non-viral) RefSeq genomes for cross-species LCS background estimation
- Candidate overlap cutoff values (e.g. 100 bp, 150 bp, 180 bp)

## Outputs

- lcs_roc_analysis.csv containing LCS size distributions, key pair values (469 bp Vaccinia-Variola, 154 bp HCV g7-g5), and ROC/AUC results
- ROC curve with AUC (expected close to 1) and false positive rate near zero for cutoffs above 100 bp
- A justified overlap cutoff value for TAR-VIR read recruitment (e.g. 150 bp)

## How to apply

Download the reference genomes named in the article (HBV NC_003977.2, seven HCV genotypes including NC_009827.1 and NC_004102.1, HPgV NC_001710.1, plus viral/microbial RefSeq records) from NCBI. Compute the longest common substring (LCS) size for each pair of viral genomes and each virus-vs-other-microbe pair using an efficient string algorithm such as a generalized suffix array or dynamic programming (Additional file 1, Section 1). Build an ROC curve where, for each candidate LCS cutoff l, the true positive rate is the probability that two strains within a quasispecies share an LCS of at least l, and the false positive rate is the probability that two different microbial species share an LCS of at least l; report the AUC. Verify the known extreme values to check correctness: the largest between-virus LCS is 469 bp (Vaccinia vs Variola) and 154 bp (HCV genotype 7 vs genotype 5), and only a small number (e.g. 86) of the ~142 million virus-vs-other pairs exceed 100 bp. Write the LCS size distributions, key pair values, and ROC/AUC results to a table such as lcs_roc_analysis.csv, and use the near-zero false positive rate above ~100 bp to justify overlap cutoffs (the overlap threshold must also exceed half the read size, e.g. 150 bp for 250 bp MiSeq reads).

## Related tools

- **TAR-VIR** (Pipeline whose overlap-extension read recruitment step consumes the cutoff justified by this ROC analysis; cutoffs are passed to the overlap module via the -c parameter) — https://github.com/chjiao/TAR-VIR
- **Python** (Used to compute pairwise LCS sizes (generalized suffix array or dynamic programming) and to build the ROC curve and AUC from the resulting distributions)

## Examples

```
./overlap -S test_data/HIV.sam -x virus -f test_data/virus.fa -c 180 -o virus_recruit.fa
```

## Evaluation signals

- Recomputed Vaccinia-vs-Variola LCS equals the reported 469 bp
- Recomputed HCV genotype 7 vs genotype 5 LCS equals the reported 154 bp
- Only a small number of the ~142 million virus-vs-other-microbe pairs (e.g. 86 pairs) exceed 100 bp
- ROC curve achieves an AUC close to 1 with a near-zero false positive rate at overlap cutoffs above 100 bp
- Chosen cutoff exceeds half the read size (e.g. >125 bp for 250 bp reads), as required to prevent recruiting chimeric reads

## Limitations

- A completely new virus that shares no conservation with any reference genome cannot be detected or have its cutoff calibrated by this method
- No single universally recommended overlap threshold value is stated; guidance must be derived from the common string size analysis between the user's own target species
- If local-mode alignment is used instead, no quantitative criteria are given for accepting or rejecting reads by alignment length and score

## Evidence

- [methods] Overlap cutoff rationale: "we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads"
- [discussion] Cutoff guidance from LCS analysis: "We have provided guidance on choosing appropriate overlap cutoffs based on the analysis of the common string sizes between different species."
- [readme] Overlap module invocation with cutoff: "./overlap -S test_data/HIV.sam -x virus -f test_data/virus.fa -c 180 -o virus_recruit.fa"
- [results] Specificity of recruited reads at chosen cutoff: "Worth noting is all the recruited reads are from SARS-CoV (no contamination from Influenza and HBV)."
